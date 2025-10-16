# 为什么你的 Nginx 撑不住高并发？

2025-08-01 13:05·[不秃头程序员](https://www.toutiao.com/c/user/token/MS4wLjABAAAAcdzXEbn9TB9MSzXQtHdwga_hLZNs68gVde3uII3Zdjw/?source=tuwen_detail)

![img](https://p3-sign.toutiaoimg.com/tos-cn-i-axegupay5k/bf23fffebb7d4e08982483580dff1426~tplv-tt-origin-web:gif.jpeg?_iz=58558&from=article.pc_detail&lk3s=953192f4&x-expires=1756534760&x-signature=mM9esQmG40%2B%2BQyFVBKbzXawJwLM%3D)



在处理高并发场景时，很多运维同学会遇到这些痛点：

- 并发量一上来就502/504
- CPU使用率飙升，但吞吐量上不去
- 连接数达到瓶颈，新请求被拒绝
- 响应时间越来越慢，用户体验极差

这些问题的根源往往不是硬件性能，而是配置和调优不到位。让我们从基础配置开始，一步步打造能扛住高并发的Nginx。

# 第一部分：Nginx核心配置优化

**1. 工作进程与连接数配置**

```
# 工作进程数 = CPU核心数
worker_processes auto;

# 每个进程的最大连接数
worker_connections 65535;

# 工作进程优先级（-20到20，数值越小优先级越高）
worker_priority -10;

# CPU亲和性绑定，避免进程在CPU间切换
worker_cpu_affinity auto;

# 工作进程打开文件数限制
worker_rlimit_nofile 100000;
```

**实战经验**：worker_connections 不是越大越好，需要考虑内存消耗。每个连接大约消耗232-248字节内存。

**2. 事件驱动模型优化**

```
events {
    # 使用epoll事件驱动（Linux最优选择）
    useepoll;
    
    # 允许同时接受多个连接
    multi_accepton;
    
    # 连接数配置
    worker_connections65535;
    
    # 接受连接后立即禁用accpet锁
    accept_mutexoff;
}
```

**3. HTTP核心配置优化**

```
http {
    # 高效文件传输
    sendfileon;
    tcp_nopushon;
    tcp_nodelayon;
    
    # 连接超时配置
    keepalive_timeout60;
    keepalive_requests10000;
    client_header_timeout10;
    client_body_timeout10;
    send_timeout10;
    
    # 缓冲区大小优化
    client_header_buffer_size4k;
    large_client_header_buffers88k;
    client_body_buffer_size128k;
    client_max_body_size50m;
    
    # 开启gzip压缩
    gzipon;
    gzip_varyon;
    gzip_min_length1000;
    gzip_comp_level6;
    gzip_types
        text/plain
        text/css
        text/javascript
        application/javascript
        application/json
        application/xml;
    
    # 隐藏版本信息
    server_tokensoff;
}
```

# 第二部分：高级性能优化配置

**1. 连接池和缓存优化**

```
http {
    # 连接池配置
    upstream backend {
        keepalive300;
        keepalive_requests10000;
        keepalive_timeout60s;
        
        server192.168.1.100:8080 weight=3 max_fails=3 fail_timeout=30s;
        server192.168.1.101:8080 weight=2 max_fails=3 fail_timeout=30s;
    }
    
    # 打开文件缓存
    open_file_cache max=100000 inactive=20s;
    open_file_cache_valid30s;
    open_file_cache_min_uses2;
    open_file_cache_errorson;
    
    # FastCGI缓存配置（适用于PHP）
    fastcgi_cache_path /var/cache/nginx levels=1:2 keys_zone=fcgi:100m inactive=60m;
    fastcgi_cache_key"$scheme$request_method$host$request_uri";
    fastcgi_cache_use_staleerror timeout invalid_header http_500;
}
```

**2. 限流和防护配置**

```
http {
    # 限制请求速率
    limit_req_zone$binary_remote_addr zone=api:10m rate=100r/s;
    limit_req_zone$binary_remote_addr zone=login:10m rate=5r/s;
    
    # 限制连接数
    limit_conn_zone$binary_remote_addr zone=perip:10m;
    limit_conn_zone$server_name zone=perserver:10m;
    
    server {
        # 应用限流规则
        limit_req zone=api burst=200 nodelay;
        limit_conn perip 20;
        limit_conn perserver 2000;
        
        # 静态资源长缓存
        location~* \.(jpg|jpeg|png|gif|ico|css|js)$ {
            expires1y;
            add_header Cache-Control "public, immutable";
        }
        
        # API接口优化
        location /api/ {
            limit_req zone=api burst=50 nodelay;
            proxy_pass http://backend;
            proxy_http_version1.1;
            proxy_set_header Connection "";
            proxy_connect_timeout5s;
            proxy_send_timeout10s;
            proxy_read_timeout10s;
        }
    }
}
```

# 第三部分：操作系统内核参数优化

高并发场景下，操作系统层面的优化同样重要。以下是经过实战验证的内核参数配置：

**1. 网络参数优化**

```
# 编辑 /etc/sysctl.conf
cat >> /etc/sysctl.conf << EOF

# 网络核心参数
net.core.somaxconn = 65535
net.core.netdev_max_backlog = 30000
net.core.rmem_default = 262144
net.core.rmem_max = 16777216
net.core.wmem_default = 262144
net.core.wmem_max = 16777216

# TCP参数优化
net.ipv4.tcp_max_syn_backlog = 65535
net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_fin_timeout = 15
net.ipv4.tcp_keepalive_time = 600
net.ipv4.tcp_keepalive_probes = 3
net.ipv4.tcp_keepalive_intvl = 15

# 连接跟踪表大小
net.netfilter.nf_conntrack_max = 1048576
net.nf_conntrack_max = 1048576

# 文件描述符限制
fs.file-max = 1048576

EOF

# 应用配置
sysctl -p
```

**2. 进程和内存参数**

```
# 编辑 /etc/security/limits.conf
cat >> /etc/security/limits.conf << EOF

# 用户进程限制
* soft nofile 1048576
* hard nofile 1048576
* soft nproc 1048576
* hard nproc 1048576

# nginx用户特殊配置
nginx soft nofile 1048576
nginx hard nofile 1048576

EOF
```

# 第四部分：实战监控与调优

**1. 性能监控脚本**

```
#!/bin/bash
# nginx_monitor.sh - Nginx性能监控脚本

echo"=== Nginx连接状态 ==="
ss -s

echo"=== 活跃连接数 ==="
netstat -n | awk '/^tcp/ {++S[$NF]} END {for(a in S) print a, S[a]}'

echo"=== Nginx进程状态 ==="
ps aux | grep nginx | grep -v grep

echo"=== 系统负载 ==="
uptime

echo"=== 内存使用情况 ==="
free -h

echo"=== Nginx访问统计 ==="
curl -s http://localhost/nginx_status
```

**2. 性能测试命令**

```
# wrk压测工具使用
wrk -t20 -c1000 -d60s --latency http://your-domain.com/

# ab压测命令
ab -n 100000 -c 1000 http://your-domain.com/

# 系统资源监控
top -p $(pgrep nginx | tr '\n' ',' | sed 's/,$//')
```

# 第五部分：高并发架构最佳实践

**1. 多层缓存策略**

```
# 三级缓存配置示例
location / {
    # L1: Nginx本地缓存
    try_files$uri@proxy;
}

location@proxy {
    # L2: 代理缓存
    proxy_cache my_cache;
    proxy_cache_valid2003021h;
    proxy_cache_valid4041m;
    proxy_pass http://backend;
    
    # 缓存锁，防止缓存击穿
    proxy_cache_lockon;
    proxy_cache_lock_timeout5s;
}
```

**2. 负载均衡策略**

```
upstream backend {
    # 一致性哈希，保证会话粘性
    hash $remote_addr consistent;
    
    # 健康检查（需要nginx-plus或第三方模块）
    server 192.168.1.100:8080 weight=3 max_fails=3 fail_timeout=30s;
    server 192.168.1.101:8080 weight=3 max_fails=3 fail_timeout=30s;
    server 192.168.1.102:8080 weight=2 max_fails=3 fail_timeout=30s backup;
}
```

# 第六部分：故障排查与性能调优

**常见问题及解决方案**

1. **502 Bad Gateway**

- 检查upstream服务状态
- 调整proxy_connect_timeout
- 增加worker_processes

1. **连接数不够**

- 调整worker_connections
- 检查系统文件描述符限制
- 优化keepalive配置

1. **内存占用过高**

- 调整缓存配置
- 检查日志文件大小
- 优化gzip配置

**性能调优checklist**

- CPU核心数与worker_processes匹配
- 文件描述符限制已调整
- 内核网络参数已优化
- 启用了适当的缓存策略
- 配置了合理的超时时间
- 实施了请求限流措施
- 监控系统已部署

# 总结

通过本文的全面优化，你的Nginx服务器应该能够：

- 支持10万+并发连接
- 响应时间控制在100ms以内
- CPU使用率保持在合理范围
- 内存使用更加高效

记住，性能优化是一个持续的过程，需要根据实际业务场景不断调整。建议在生产环境应用前，先在测试环境进行充分验证。

**实战提醒**：配置修改后记得重载配置（nginx -s reload），并监控系统状态，确保优化效果符合预期。