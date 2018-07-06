* 1 delete keys
```
 ETCDCTL_API=3 etcdctl del /registry --prefix
```

* 2 get all keys
```
 ETCDCTL_API=3 etcdctl get / --prefix --keys-only
```