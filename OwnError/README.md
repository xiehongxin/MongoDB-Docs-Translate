# 博猪遇到的错误

1.F CONTROL [main] Failed global initialization: BadValue: Invalid or no user locale set. Please ensure LANG and/or LC_* environment variables are set correctly.

解决：https://www.jianshu.com/p/6ac188bfcac3


2.Automatically disabling TLS 1.0
```python
[hongxin@localhost conf]$ mongod -f ~/mongodb/conf/config.conf
2018-12-02T12:07:53.835+0800 I CONTROL  [main] Automatically disabling TLS 1.0, to force-enable TLS 1.0 specify --sslDisabledProtocols 'none'
about to fork child process, waiting until server is ready for connections.
forked process: 24717
ERROR: child process failed, exited with error number 1
To see additional information in this output, start without the "--fork" option.

# 配置文件
# 配置文件内容 
pidfilepath = ~/mongodb/config/log/configsrv.pid 
dbpath = ~/mongodb/config/data 
logpath = ~/mongodb/config/log/congigsrv.log 
logappend = true 
  
bind_ip = 127.0.0.1 
port = 21000 
fork = true 
  
#declare this is a config db of a cluster; 
configsvr = true 
 
#副本集名称 
replSet=configs 
  
#设置最大连接数 
maxConns=2000 

```
TLS： 传输层安全性协议（英语：Transport Layer Security，缩写作TLS），及其前身安全套接层（Secure Sockets Layer，缩写作SSL）是一种安全协议。

https://docs.mongodb.com/manual/release-notes/4.0/#disable-tls


从MongoDB4.0开始，MongoDB开始禁用TLS1.0, 并转向TLS1.1的怀抱

如何使用TLS1.0
https://docs.mongodb.com/manual/release-notes/4.0/#disable-tls

TLS1.0 和 TLS1.1 在mongodb中的区别是什么，为什么我上述的配置是1.0的？
