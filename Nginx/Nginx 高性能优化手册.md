# **Nginx 高性能优化手册**

## **1. 简介**

本手册提供针对 Nginx 的性能优化配置，适用于高并发、高负载场景，包括全局优化、HTTP 优化、代理缓存优化、SSL 优化等。

------

## **2. 全局优化**

### **2.1 工作进程与连接数**

nginx



复制



下载

```
worker_processes auto;  # 自动匹配 CPU 核心数
worker_rlimit_nofile 200000;  # 提高文件描述符限制

events {
    worker_connections 8192;  # 每个 worker 进程的最大连接数
    multi_accept on;  # 允许同时接受多个连接
    use epoll;  # Linux 高性能事件驱动模型
}
```

**优化说明**：

- `worker_processes auto` 自动匹配 CPU 核心数，提高并发能力。
- `worker_rlimit_nofile` 提高文件描述符限制，防止 `Too many open files` 错误。
- `worker_connections` 提高单进程连接数，适用于高并发场景。

------

### **2.2 文件缓存优化**

nginx



复制



下载

```
open_file_cache max=50000 inactive=60s;  # 缓存 50,000 个文件
open_file_cache_valid 120s;  # 缓存验证时间
open_file_cache_min_uses 3;  # 最少访问 3 次才缓存
open_file_cache_errors on;  # 缓存错误文件信息
```

**优化说明**：

- 减少磁盘 I/O，提高静态文件访问速度。
- `open_file_cache_valid` 控制缓存刷新频率，避免频繁检查文件变更。

------

## **3. HTTP 优化**

### **3.1 缓冲区优化**

nginx

```
client_body_buffer_size 32K;  # 请求体缓冲区
client_header_buffer_size 4k;  # 请求头缓冲区
large_client_header_buffers 4 64k;  # 大请求头缓冲区
client_max_body_size 500m;  # 最大上传文件大小
```

**优化说明**：

- 减少内存碎片，提高大请求处理能力。

------

### **3.2 超时优化**

nginx

```
keepalive_timeout 30;  # 长连接超时时间
keepalive_requests 5000;  # 单个连接最大请求数
client_body_timeout 30s;  # 请求体超时
client_header_timeout 30s;  # 请求头超时
send_timeout 30s;  # 发送超时
```

**优化说明**：

- 减少空闲连接占用资源，提高连接复用率。

------

### **3.3 TCP 优化**

nginx

```
sendfile on;  # 零拷贝传输
tcp_nopush on;  # 减少小包传输
tcp_nodelay on;  # 禁用 Nagle 算法，提高实时性
```

**优化说明**：

- `sendfile` 减少 CPU 拷贝开销，提高文件传输效率。
- `tcp_nopush` 和 `tcp_nodelay` 优化 TCP 数据包传输。

------

### **3.4 Gzip 压缩优化**

nginx

```
gzip on;
gzip_min_length 1024;  # 最小压缩文件大小
gzip_comp_level 6;  # 压缩级别（1-9）
gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;
gzip_vary on;
gzip_proxied any;  # 代理请求也压缩
```

**优化说明**：

- 减少传输数据量，提高网页加载速度。

------

## **4. 代理缓存优化**

### **4.1 缓存路径与策略**

nginx

```
proxy_cache_path /var/cache/nginx levels=1:2 keys_zone=my_cache:100m inactive=24h use_temp_path=off max_size=2g;
proxy_cache_key "$scheme$request_method$host$request_uri";
proxy_cache_lock on;  # 防止缓存击穿
proxy_cache_lock_timeout 5s;
proxy_cache_use_stale error timeout updating http_500 http_502 http_503 http_504;
proxy_cache_background_update on;
```

**优化说明**：

- `proxy_cache_path` 定义缓存存储位置、大小和失效时间。
- `proxy_cache_lock` 防止多个请求同时回源，减轻后端压力。

------

### **4.2 代理优化

```
proxy_buffers 32 16k;  # 缓冲区数量与大小
proxy_buffer_size 32k;  # 代理缓冲区大小
proxy_busy_buffers_size 256k;  # 忙时缓冲区大小
proxy_temp_file_write_size 256k;  # 临时文件写入大小
proxy_connect_timeout 60s;  # 连接超时
proxy_send_timeout 60s;  # 发送超时
proxy_read_timeout 60s;  # 读取超时
```

**优化说明**：

- 提高代理吞吐量，减少后端服务器压力。

------

## **5. SSL 优化（HTTPS 场景）

```
ssl_session_cache shared:SSL:50m;  # SSL 会话缓存
ssl_session_timeout 1d;  # 会话超时时间
ssl_session_tickets off;  # 禁用 Tickets
ssl_prefer_server_ciphers on;  # 优先使用服务器加密套件
ssl_stapling on;  # OCSP 装订
ssl_stapling_verify on;  # 验证 OCSP 响应
```

**优化说明**：

- 减少 SSL 握手时间，提高 HTTPS 性能。

------

## **6. 日志优化**

```
access_log /var/log/nginx/access.log main buffer=64k flush=5m;  # 缓冲写入日志
error_log /var/log/nginx/error.log warn;  # 只记录警告及以上级别
```

**优化说明**：

- `buffer=64k flush=5m` 减少磁盘 I/O，提高日志写入效率。

------

## **7. 静态资源优化**

nginx

```
location ~* \.(jpg|jpeg|png|gif|ico|css|js|pdf|woff|woff2|ttf|svg)$ {
    expires 365d;  # 长期缓存
    access_log off;  # 关闭日志
    add_header Cache-Control "public, immutable";  # 强制浏览器缓存
    add_header X-Static "true";  # 自定义头标识
    try_files $uri =404;  # 防止目录遍历
}
```

**优化说明**：

- 减少静态资源请求，提高页面加载速度。

------

## **8. 监控与调优建议**

1. **使用 `nginx -t` 测试配置**，确保无语法错误。
2. **监控工具推荐**：
   - `htop`（CPU/内存监控）
   - `iftop`（网络流量监控）
   - `nginx_status`（Nginx 状态监控）
3. **压测工具**：
   - `ab`（Apache Benchmark）
   - `wrk`（高性能 HTTP 压测）
   - `jmeter`（复杂场景测试）

------

## **9. 总结**

通过本手册的优化，Nginx 可以：
✅ 提高并发处理能力
✅ 减少磁盘 I/O 和 CPU 开销
✅ 优化 HTTPS 性能
✅ 增强缓存机制，降低后端负载
✅ 提高静态资源访问速度

**建议**：逐步应用优化，每次调整后测试性能，确保稳定性。

📌 **附：完整优化配置参考**
可在 `/etc/nginx/nginx.conf` 中应用上述优化，并根据业务需求调整参数。