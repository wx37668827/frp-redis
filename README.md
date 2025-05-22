## 功能描述
仅允许来自特定 IP 地址请求访问代理服务，IP白名单存放在REDIS中。

## 生成EXE命令
```sh
go build -o frps.exe ./cmd/frps
 ```
## 配置文件frps.ini
```text
[common]
bind_port = 7000
enable_redis_ip_whitelist = true
redis_addr = 127.0.0.1:6379 
redis_password = redispass
redis_db = 2
redis_whitelist_prefix = ip_whitelist_  #redis key 前缀，存在ip_whitelist_192.168.100.200这样的key则IP:192.168.100.200允许访问
```
## 启动命令
```sh
frps.exe -c frps.ini

 ```
