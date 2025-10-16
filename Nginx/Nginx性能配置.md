Ngxin性能优化文件

优化Nginx配置文件可以从多个方面入手，包括性能调优、安全性增强、以及代码结构的优化。以下是一些优化建议，帮助你提高Nginx的性能：

### 1. **全局配置优化**

- **worker_processes**: 你已经设置为 `auto`，这是推荐的设置，Nginx 会自动根据 CPU 核心数设置 worker 进程数。
- **worker_rlimit_nofile**: 你已经设置为 `100000`，这是一个合理的值，确保系统能够处理大量并发连接。

### 2. **事件模块优化**

- **worker_connections**: 你已经设置为 `4096`，这是一个合理的值。如果你有更高的并发需求，可以适当增加这个值，但要确保系统的文件描述符限制足够高。
- **multi_accept**: 你已经设置为 `on`，这允许 Nginx 一次性接受多个连接，有助于提高并发性能。
- **use epoll**: 你已经设置为 `epoll`，这是 Linux 系统下的最佳选择。

### 3. **HTTP模块优化**

- **keepalive_timeout**: 你已经设置为 `30`，这是一个合理的值。如果你的应用有大量的长连接需求，可以适当增加这个值。
- **keepalive_requests**: 你已经设置为 `2000`，这是一个合理的值。如果你有更高的长连接需求，可以适当增加这个值。
- **client_max_body_size**: 你已经设置为 `500m`，确保它符合你的应用需求。如果上传文件较大，可以适当增加这个值。

### 4. **缓冲区优化**

- **client_body_buffer_size**: 你已经设置为 `16K`，这是一个合理的值。如果你的应用有较大的请求体，可以适当增加这个值。
- **client_header_buffer_size**: 你已经设置为 `2k`，这是一个合理的值。如果你的请求头较大，可以适当增加这个值。
- **large_client_header_buffers**: 你已经设置为 `4 8k`，这是一个合理的值。如果你的请求头非常大，可以适当增加这个值。

### 5. **超时设置**

- **client_body_timeout** 和 **client_header_timeout**: 你已经设置为 `12` 秒，这是一个合理的值。如果你的应用有较慢的客户端，可以适当增加这个值。
- **send_timeout**: 你已经设置为 `10` 秒，这是一个合理的值。如果你的应用有较慢的客户端，可以适当增加这个值。

### 6. **压缩配置**

- **gzip**: 你已经启用 Gzip 压缩，并且设置了合理的压缩级别和类型。确保你压缩的内容类型符合你的应用需求。
- **gzip_min_length**: 你已经设置为 `512`，这是一个合理的值。如果你的应用有大量小文件，可以适当降低这个值。

### 7. **缓存配置**

- **open_file_cache**: 你已经启用了文件缓存，并且设置了合理的缓存大小和验证时间。确保缓存大小符合你的应用需求。

### 8. **反向代理优化**

- **proxy_buffering**: 你已经启用代理缓冲，并且设置了合理的缓冲区大小和数量。确保这些值符合你的应用需求。
- **proxy_read_timeout**, **proxy_connect_timeout**, **proxy_send_timeout**: 你已经设置为 `60s`，这是一个合理的值。如果你的后端服务器响应较慢，可以适当增加这些值。

### 9. **SSL/TLS 优化**

- **ssl_protocols**: 你已经设置为 `TLSv1.2 TLSv1.3`，这是推荐的设置，确保使用最新的安全协议。
- **ssl_ciphers**: 你已经设置为 `HIGH:!aNULL:!MD5`，这是推荐的设置，确保使用强加密套件。
- **ssl_session_cache**: 你已经设置为 `shared:SSL:10m`，这是一个合理的值。如果你的应用有大量的 SSL/TLS 连接，可以适当增加这个值。

### 10. **日志优化**

- **access_log**: 你已经启用了访问日志，并且设置了合理的日志格式。如果你的应用有大量的访问日志，可以考虑关闭不必要的日志记录，或者将日志写入到高性能的存储设备中。

### 11. **负载均衡优化**

- 如果你有多个后端服务器，可以考虑启用负载均衡，并且使用 `least_conn` 或 `ip_hash` 等负载均衡算法。

### 12. **静态文件缓存**

- 你已经注释掉了静态文件缓存配置。如果你的应用有大量的静态文件，建议启用静态文件缓存，并且设置合理的缓存时间。

### 13. **错误页面优化**

- 你已经配置了错误页面，确保错误页面的路径和内容符合你的应用需求。

### 14. **HTTPS 重定向**

- 你已经配置了 HTTPS 重定向到 HTTP，确保这个配置符合你的应用需求。如果你需要强制使用 HTTPS，可以调整这个配置。

### 15. **其他优化**

- **TCP_NOPUSH** 和 **TCP_NODELAY**: 你已经启用了这些选项，确保它们符合你的应用需求。
- **sendfile**: 你已经启用了 `sendfile`，确保它符合你的应用需求。

### 优化后的配置示例

nginx

复制

```
# 全局配置
user nginx;
worker_processes auto;
worker_rlimit_nofile 100000;

# 错误日志配置
error_log /var/log/nginx/error.log warn;
pid /run/nginx.pid;
include /usr/share/nginx/modules/*.conf;

# 事件模块配置
events {
    worker_connections 4096;
    multi_accept on;
    use epoll;
}

# HTTP模块配置
http {
    include /etc/nginx/mime.types;
    default_type application/octet-stream;
    charset UTF-8;
    include /etc/nginx/conf.d/*.conf;

    # 日志格式
    log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                    '$status $body_bytes_sent "$http_referer" '
                    '"$http_user_agent" "$http_x_forwarded_for" "$http_x_forwarded_port" "$http_port "    '
                      'rt=$request_time uct="$upstream_connect_time" uht="$upstream_header_time" urt="$upstream_response_time"'  
    log_format log404 '$remote_addr - $proxy_add_x_forwarded_for - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for" "$http_x_forwarded_port" "$http_port "';

    access_log /var/log/nginx/access.log main;
    access_log /var/log/nginx/host.access.404.log log404;

    # 性能优化配置
    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    keepalive_timeout 30;
    keepalive_requests 2000;
    types_hash_max_size 2048;

    # 缓冲区优化
    client_body_buffer_size 16K;
    client_header_buffer_size 2k;
    client_max_body_size 500m;
    large_client_header_buffers 4 8k;

    # 超时设置
    client_body_timeout 12;
    client_header_timeout 12;
    send_timeout 10;
    proxy_connect_timeout 300;
    proxy_send_timeout 300;
    proxy_read_timeout 300;
    proxy_buffer_size 8k;
    proxy_buffers 16 32k;
    proxy_busy_buffers_size 112k;
    proxy_temp_file_write_size 128k;
    fastcgi_connect_timeout 300;
    fastcgi_send_timeout 300;
    fastcgi_read_timeout 300;

    # 压缩配置
    gzip on;
    gzip_min_length 512;
    gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;
    gzip_comp_level 5;
    gzip_vary on;
    gzip_disable "msie6";

    # 缓存配置
    open_file_cache max=20000 inactive=30s;
    open_file_cache_valid 60s;
    open_file_cache_min_uses 2;
    open_file_cache_errors on;

    # 定义一个映射，用于提取 X-Forwarded-For 头部中的第一个 IP 地址
    map $http_x_forwarded_for $client_ip {
        ~^[0-9\.]+,? $1;
        default $remote_addr;
    }

    # 定义一个映射，用于检查提取的 IP 是否是 10.0.104.51
    map $client_ip $is_blocked {
        "10.0.104.51" 1;
        default 0;
    }

    # 虚拟主机配置
    server {
        listen 80;
        server_name center.gwkf.cn;

        location / {
            proxy_pass http://127.0.0.1:8001;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            proxy_http_version 1.1;
            proxy_set_header Connection "";
            proxy_buffering on;
            proxy_buffer_size 4k;
            proxy_buffers 8 16k;
            proxy_read_timeout 60s;
            proxy_connect_timeout 60s;
            proxy_send_timeout 60s;
            proxy_next_upstream error timeout invalid_header http_500 http_502 http_503 http_504;
            proxy_next_upstream_tries 3;
            proxy_next_upstream_timeout 60s;
        }

        location /nginx_status {
            stub_status on;
            access_log off;
            allow 127.0.0.1;
            allow 10.2.255.50;
            allow 10.0.104.51;
            deny all;
        }

        error_page 500 502 503 504 /50x.html;
        location = /50x.html {
            root /usr/share/nginx/html;
        }
    }

    server {
        listen 80;
        server_name szwms.gwkf.cn;

        location / {
            proxy_pass http://127.0.0.1:30441;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            proxy_http_version 1.1;
            proxy_set_header Connection "";
            proxy_buffering on;
            proxy_buffer_size 4k;
            proxy_buffers 8 16k;
            proxy_read_timeout 60s;
            proxy_connect_timeout 60s;
            proxy_send_timeout 60s;
        }

        error_page 500 502 503 504 /50x.html;
        location = /50x.html {
            root /usr/share/nginx/html;
        }
    }

    server {
        listen 80;
        server_name aps-uat.gwkf.cn;

        location / {
            proxy_pass http://127.0.0.1:8008;
            proxy_set_header Host $http_host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            proxy_http_version 1.1;
            proxy_set_header Connection "";
            proxy_buffering on;
            proxy_buffer_size 4k;
            proxy_buffers 8 16k;
            proxy_read_timeout 60s;
            proxy_connect_timeout 60s;
            proxy_send_timeout 60s;
        }

        error_page 500 502 503 504 /50x.html;
        location = /50x.html {
            root /usr/share/nginx/html;
        }
    }

    server {
        listen 80;
        server_name sz_1Panel.gwkf.cn;

        location / {
            proxy_pass http://10.2.255.15:14617;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "upgrade";
            proxy_set_header Connection "";
            proxy_buffering on;
            proxy_buffer_size 4k;
            proxy_buffers 8 16k;
            proxy_read_timeout 60s;
            proxy_connect_timeout 60s;
            proxy_send_timeout 60s;
        }

        error_page 500 502 503 504 /50x.html;
        location = /50x.html {
            root /usr/share/nginx/html;
        }
    }

    ## HTTPS 重定向至 HTTP
    server {
        listen 443 ssl;
        server_name center.gwkf.cn;
        ssl_certificate /etc/nginx/ssl/gwkf.crt;
        ssl_certificate_key /etc/nginx/ssl/gwkf.key;
        ssl_protocols TLSv1.2 TLSv1.3;
        ssl_ciphers HIGH:!aNULL:!MD5;
        ssl_prefer_server_ciphers on;
        ssl_session_cache shared:SSL:10m;
        ssl_session_timeout 10m;
        ssl_buffer_size 4k;

        if ($scheme = https) {
            return 301 http://$server_name$request_uri;
        }
    }
}
```

### 总结

通过以上优化，Nginx 的性能应该会有所提升。你可以根据实际的负载情况和应用需求进一步调整这些参数。建议在每次修改配置后，使用 `nginx -t` 测试配置文件的正确性，并逐步上线观察效果。