#### 背景
运维突然报告，线上服务CPU 400%，服务无法响应了，dump出stack后，重启服务实例，暂时度过了危机。

#### 分析
  1. 查看stack，发现下面的调用数量比较多，其他的线程看起来是正常的。

``` java

  at org.springframework.security.crypto.bcrypt.BCrypt.key(BCrypt.java:447)
	at org.springframework.security.crypto.bcrypt.BCrypt.crypt_raw(BCrypt.java:510)
	at org.springframework.security.crypto.bcrypt.BCrypt.hashpw(BCrypt.java:590)
	at org.springframework.security.crypto.bcrypt.BCrypt.checkpw(BCrypt.java:659)
	at org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder.matches(BCryptPasswordEncoder.java:94)
	at org.springframework.security.crypto.password.DelegatingPasswordEncoder.matches(DelegatingPasswordEncoder.java:201)
	at org.springframework.security.oauth2.config.annotation.web.configurers.AuthorizationServerSecurityConfigurer$1.matches(AuthorizationServerSecurityConfigurer.java:155)
	at org.springframework.security.authentication.dao.DaoAuthenticationProvider.mitigateAgainstTimingAttack(DaoAuthenticationProvider.java:149)
	at org.springframework.security.authentication.dao.DaoAuthenticationProvider.retrieveUser(DaoAuthenticationProvider.java:116)
	at org.springframework.security.authentication.dao.AbstractUserDetailsAuthenticationProvider.authenticate(AbstractUserDetailsAuthenticationProvider.java:144)
	at org.springframework.security.authentication.ProviderManager.authenticate(ProviderManager.java:175)
	at org.springframework.security.web.authentication.www.BasicAuthenticationFilter.doFilterInternal(BasicAuthenticationFilter.java:180)
	at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:118)
	at org.springframework.security.web.FilterChainProxy$VirtualFilterChain.doFilter(FilterChainProxy.java:334)
	at org.springframework.security.web.authentication.logout.LogoutFilter.doFilter(LogoutFilter.java:116)

```
  1. 初步怀疑是这个地方把CPU打满了。CPU被占满的原因一般会是出现了dead loop。
  2. 查看源码，BCrpyt的调用是使用brpyt算法验证密码是否正确，卡带的代码不像是会出现死循环；
```
  		for (i = 0; i < slen; i += 2) {
			encipher(lr, 0);
			S[i] = lr[0];
			S[i + 1] = lr[1];
		}
```
  4. 通过google发现，BCrpyt确实是一个安全，但是耗能的算法，而线程里显示这个调用有8条，所以怀疑是当时有很多这样的请求涌入，造成服务过载；nginx的日志也证明半小时内有1000多获取token的请求；
    ![performance](../images/bcrypt-performance.png?raw=true "Performance")
  5. 服务里面用到BCrypt的地方比较容易找到，我们使用了spring-security-oauth功能，并且配置了PasswordEncoder, 默认算法使用了Bcrypt算法

```
        String encodingId = "bcrypt";
        Map<String, PasswordEncoder> encoders = new HashMap<>();
        encoders.put(encodingId, new BCryptPasswordEncoder());
        encoders.put("ldap", new org.springframework.security.crypto.password.LdapShaPasswordEncoder());
        encoders.put("MD4", new org.springframework.security.crypto.password.Md4PasswordEncoder());
        encoders.put("MD5", new org.springframework.security.crypto.password.MessageDigestPasswordEncoder("MD5"));
        encoders.put("noop", org.springframework.security.crypto.password.NoOpPasswordEncoder.getInstance());
        encoders.put("pbkdf2", new Pbkdf2PasswordEncoder());
        encoders.put("scrypt", new SCryptPasswordEncoder());
        encoders.put("SHA-1", new org.springframework.security.crypto.password.MessageDigestPasswordEncoder("SHA-1"));
        encoders
            .put("SHA-256", new org.springframework.security.crypto.password.MessageDigestPasswordEncoder("SHA-256"));
        encoders.put("sha256", new org.springframework.security.crypto.password.StandardPasswordEncoder());

        DelegatingPasswordEncoder delegatingPasswordEncoder = new DelegatingPasswordEncoder(encodingId, encoders);
        delegatingPasswordEncoder.setDefaultPasswordEncoderForMatches(new BCryptPasswordEncoder(10));
        return delegatingPasswordEncoder;
```

  6. 既然Bcrypt是一个耗能的算法，那替换成一个高效算法是不是就可以解决问题了呢？比如SHA-256. 似乎是一个可行的办法。
  7. 但是等等，Password-Encoder应该只在登录服务，获取token的时候才会被使用，为啥会有这么多请求呢，而且从日志发现，这1000多请求的id都是同一个。从日志看起来这些请求都被拒绝了。既然如此，为啥还是会出现CPU过载呢？如果有很多请求，不应该是线程被耗尽吗？
  8. id被拒绝的原因很快就查到了，它事实上还没有被创建出来，但是client服务启动后，就开始使用它了，并且失败后，就会一直重试（差劲的错误处理）。所以才会在短时间出现大量的请求；
  9. 但既然id不存在，为啥还会去验证密码呢？这时我注意到了调用栈中的方法

```
  at org.springframework.security.authentication.dao.DaoAuthenticationProvider.mitigateAgainstTimingAttack(DaoAuthenticationProvider.java:149)
```
  10. 然后找到调用的地方，发现它确实是在未找到id的时候，去执行的。

```
  protected final UserDetails retrieveUser(String username,
			UsernamePasswordAuthenticationToken authentication)
			throws AuthenticationException {
		prepareTimingAttackProtection();
		try {
			UserDetails loadedUser = this.getUserDetailsService().loadUserByUsername(username);
			if (loadedUser == null) {
				throw new InternalAuthenticationServiceException(
						"UserDetailsService returned null, which is an interface contract violation");
			}
			return loadedUser;
		}
		catch (UsernameNotFoundException ex) {
			mitigateAgainstTimingAttack(authentication);
			throw ex;
		}
		catch (InternalAuthenticationServiceException ex) {
			throw ex;
		}
		catch (Exception ex) {
			throw new InternalAuthenticationServiceException(ex.getMessage(), ex);
		}
	}
```

  11. 继续跟踪，发现它的作用就是用userNotFoundPassword这个字符串和密码进行比对；但比对的结果不会被使用；

```


	/**
	 * The plaintext password used to perform
	 * PasswordEncoder#matches(CharSequence, String)}  on when the user is
	 * not found to avoid SEC-2056.
	 */
	private static final String USER_NOT_FOUND_PASSWORD = "userNotFoundPassword";

```

  1.  spring-security 这么做的目的是为了防止(https://jira.spring.io/browse/SEC-2056?redirect=false)。如果账号存在，就会去验证密码，不存在跳过验证，那么攻击者就可以通过这个时间差，来探查账号是否存在。但是加入了这段代码后，账号存在与否的处理时间将是一致的。

#### 结论
  应用程序错误的获取token方式（在失败的时候一直重试），服务使用了耗时的密码验证算法（单次需要300ms），形成了类似DDOS的攻击效果；
  在现有的情况下解决方案是，
  1. 先配置id；
  2. 应用程序修改错误处理逻辑。
  3. 服务限制获取token的频次，似乎是一个更好的办法。
