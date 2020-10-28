* 1\.  参考
  * a\. http://travistidwell.com/jsencrypt/
  * b\. https://www.devglan.com/spring-mvc/rsa-encryption-in-javascript-and-decryption-in-java
  * c\. https://www.baeldung.com/java-read-pem-file-keys

* 2\. 具体步骤
  * a\. 生成rsa公钥和私钥
    
    ```
    openssl genrsa -out keypair.pem 2048
    ```

    ```
      openssl rsa -pubout -outform pem -in keypair.pem -out public.pem
    ```

    ```
    openssl pkcs8 -topk8 -in keypair.pem -inform pem -out pkcs8.key -outform pem -nocrypt

    ```

  * b\. javascript 代码
     * 1\. install jsencrypt
     ```npm
      npm install jsencrypt
     ```

     * 2\. util methods
    
     ```javascript

        export function encrypt(plain) {
        let crypt = new JSEncrypt();

        crypt.setPublicKey(
          "-----BEGIN PUBLIC KEY-----\n88888888\n-----END PUBLIC KEY-----"
        );

        let res = crypt.encrypt(plain);

        return "encrypted:" + res;
      }

     ```

   * c\. java 代码
   ```java

    public class RSACryptoUtil {
      private static final Logger logger = LoggerFactory.getLogger(RSACryptoUtil.class);
      private static final String PRIVATE_PEM = "rsa/pkcs8.key";
      private static final String PREFIX = "encrypted:";
      private static final String PRIVATE_KEY_BEGIN = "-----BEGIN PRIVATE KEY-----";
      private static final String PRIVATE_KEY_END = "-----END PRIVATE KEY-----";

      private static volatile PrivateKey privateKey;

      private static PrivateKey getPrivateKey() throws Exception {
        if (privateKey != null) {
            return privateKey;
        }

        synchronized (RSACryptoUtil.class) {
            try (InputStream in = RSACryptoUtil.class.getClassLoader().getResourceAsStream(PRIVATE_PEM);) {
                byte[] keyBytes = in.readAllBytes();
                String pem = new String(keyBytes, "UTF8");
                pem = pem.replace(PRIVATE_KEY_BEGIN, "").replace(PRIVATE_KEY_END, "").replaceAll("\\n", "");

                byte[] key = Base64.getDecoder().decode(pem.getBytes("UTF8"));

                KeyFactory kf = KeyFactory.getInstance("RSA");

                privateKey = kf.generatePrivate(new PKCS8EncodedKeySpec(key));
            }
        }

        return privateKey;
    }

    public static String decrypt(String encrypted) {
        if (!StringUtils.startsWith(encrypted, PREFIX)) {
            return encrypted;
        }

        try {
            encrypted = encrypted.substring(PREFIX.length());

            byte[] bytes = Base64.getDecoder().decode(encrypted.getBytes("UTF8"));

            Cipher rsa = Cipher.getInstance("RSA");

            rsa.init(Cipher.DECRYPT_MODE, getPrivateKey());
            byte[] res = rsa.doFinal(bytes);

            return new String(res, "UTF8");
        } catch (Exception ex) {
            logger.error("failed to decrypt {}", ex);
            throw new RuntimeException(ex);
        }
    }
}



   ```