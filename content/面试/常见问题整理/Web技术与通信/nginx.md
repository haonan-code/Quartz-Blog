##### 什么是负载均衡？
> **负载均衡**（Load Balancing）是一种分发技术，它将客户端请求分摊到多台后端服务器上，以提升系统性能、稳定性与可用性。
##### Nginx 负载均衡的原理
Nginx 作为高性能反向代理服务器，可以把请求按特定规则分发到多个后端服务器（称为 upstream server）上，从而实现负载均衡。
##### Nginx 配置负载均衡
**配置示例**
```nginx
http {
    upstream backend {
        server 192.168.1.101;
        server 192.168.1.102;
        server 192.168.1.103;
    }

    server {
        listen 80;
        server_name example.com;

        location / {
            proxy_pass http://backend;
        }
    }
}
```

**含义说明**
- `upstream backend`：定义一个上游服务组名 `backend`。
- `server`：指定上游服务器地址。
- `proxy_pass`：请求将转发给 `backend` 定义的服务器池。

##### Nginx 支持的负载均衡策略
| 策略                              | 描述                             |
| ------------------------------- | ------------------------------ |
| **轮询（默认）**                      | 请求依次分配到每台服务器。                  |
| **weight**                      | 根据权重分配请求。权重高的服务器分配更多请求。        |
| **ip_hash**                     | 同一个 IP 的请求会被分配到同一台服务器，常用于会话保持。 |
| **least_conn**（需要 nginx 1.3.1+） | 将请求分配给连接数最少的服务器。               |
| **hash**（需要第三方模块）               | 自定义 hash 规则进行分配，如按请求参数、cookie。 |