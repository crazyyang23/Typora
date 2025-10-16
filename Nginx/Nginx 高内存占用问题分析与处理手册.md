# **Nginx 高内存占用问题分析与处理手册**

**版本：1.0**
**最后更新：2025-07-04**

------

## **1. 问题概述**

### **1.1 现象描述**

- Nginx 内存占用异常高（6.5GB+）
- 部分 Worker 进程无法正常退出（`is shutting down` 状态）
- 重启后内存仍然居高不下

### **1.2 影响范围**

- 可能导致 **服务响应变慢**
- 极端情况下触发 **OOM Killer**，导致 Nginx 被强制终止
- 客户端可能遇到 **499（Client Closed Request）** 错误

------

## **2. 问题诊断**

### **2.1 快速检查命令**

bash



复制



下载

```
# 查看 Nginx 状态
systemctl status nginx

# 检查内存占用
ps aux | grep nginx | grep -v grep | awk '{print $4,$11}' | sort -nr

# 查看当前连接数
ss -ant | grep ':80\|:443' | wc -l

# 检查配置语法
nginx -T
```

### **2.2 常见原因**

| 可能原因              | 检查方法                             | 相关日志/指标                |
| :-------------------- | :----------------------------------- | :--------------------------- |
| **Worker 进程过多**   | `nginx -T | grep worker_processes`   | `Tasks` 数量异常             |
| **Worker 连接数过高** | `nginx -T | grep worker_connections` | `Memory` 持续增长            |
| **缓冲区配置过大**    | `nginx -T | grep buffer`             | `client_body_buffer_size` 等 |
| **第三方模块泄漏**    | `nginx -V | grep modules`            | `error.log` 内存错误         |
| **后端应用占用高**    | `top -p $(pgrep nginx)`              | 后端服务日志                 |

------

## **3. 解决方案**

### **3.1 紧急处理**

#### **(1) 强制清理僵尸进程**

bash



复制



下载

```
# 查找并杀死旧 Worker
ps aux | grep 'nginx.*shutting down' | awk '{print $2}' | xargs sudo kill -9

# 完全重启 Nginx
sudo systemctl restart nginx
```

#### **(2) 优化关键配置**

nginx



复制



下载

```
# /etc/nginx/nginx.conf
worker_processes auto;                  # 自动匹配 CPU 核心数
events {
    worker_connections 2048;           # 建议值：CPU核心数 * 1024
    multi_accept on;
}
http {
    keepalive_timeout 30s;             # 减少长连接时间
    client_body_buffer_size 16k;       # 调低缓冲区
    client_header_buffer_size 4k;
}
```

### **3.2 长期优化**

#### **(1) 内存监控**

bash



复制



下载

```
# 每 5 秒记录内存使用
watch -n 5 "smem -P nginx -c 'pid user pss rss command'"
```

#### **(2) 限制系统资源**

bash

```
# 增加文件描述符限制
echo "fs.file-max = 65535" >> /etc/sysctl.conf
echo "nginx soft nofile 65535" >> /etc/security/limits.conf
sysctl -p
```

#### **(3) 定时维护**

bash

```
# 每天凌晨 3 点优雅重启
0 3 * * * /usr/sbin/nginx -s reload
```

------

## **4. 高级调试**

### **4.1 内存泄漏检测**

bash

```
# 使用 Valgrind（需调试版本）
valgrind --leak-check=full /usr/sbin/nginx -c /etc/nginx/nginx.conf
```

### **4.2 核心转储分析**

bash

```
# 生成并分析 core dump
gdb /usr/sbin/nginx core.<pid>
bt full
```

------

## **5. 附录**

### **5.1 相关日志位置**

- **Nginx 错误日志**：`/var/log/nginx/error.log`
- **系统日志**：`journalctl -u nginx --since "1 hour ago"`

### **5.2 配置文件示例**

nginx

```
user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log warn;
pid /var/run/nginx.pid;

events {
    worker_connections 2048;
    use epoll;
}

http {
    keepalive_timeout 30s;
    client_max_body_size 10m;
    client_body_buffer_size 16k;
    include /etc/nginx/mime.types;
    default_type application/octet-stream;
    # 其他配置...
}
```

### **5.3 应急预案**

1. **内存突然暴涨**
   - 立即重启 Nginx：`sudo systemctl restart nginx`
   - 临时限制连接数：`limit_conn_zone`
2. **OOM Killer 触发**
   - 检查 `/var/log/messages` 中的 `killed process`
   - 调整 OOM 优先级：`echo -17 > /proc/$(pgrep nginx)/oom_adj`

------

**注**：本手册需根据实际环境调整参数，建议先在测试环境验证。
**维护团队**：运维部
**联系方式**：[admin@example.com](https://mailto:admin@example.com/)