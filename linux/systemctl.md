* 1. useful commands
```
  systemctl daemon-reload
  systemctl status xxx
  systemctl start xxx
  systemctl stop xxx
  systemctl restart xxx
```

* 2. when service is failed, how to check
```
  systemctl --failed
  journalctl _PID=xxxx
```