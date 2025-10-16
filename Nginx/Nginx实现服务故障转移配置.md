# **Nginx 多级故障转移操作手册**

**版本：1.0**
**适用场景**：当主服务（A）不可用时自动切换到备用服务（B），若 A 和 B 均不可用，则切换到最终备用服务（C）。

------

## **1. 环境准备**

### **1.1 服务器信息**

| 服务器 | IP 地址       | 端口 | 角色     |
| :----- | :------------ | :--- | :------- |
| 服务A  | 192.168.1.100 | 80   | 主服务器 |
| 服务B  | 192.168.1.101 | 80   | 第一备份 |
| 服务C  | 192.168.1.102 | 80   | 第二备份 |

### **1.2 依赖检查**

- 确保 Nginx 已安装（`nginx -v` 检查版本）
- 确保各后端服务（A/B/C）可正常访问

------

## **2. Nginx 配置**

### **2.1 修改 Nginx 配置文件**

编辑 Nginx 主配置文件（通常位于 `/etc/nginx/nginx.conf` 或 `/etc/nginx/conf.d/default.conf`），在 `http` 块内添加 `upstream` 和 `server` 配置：

```
http {
    upstream backend {
        # 主服务器（A），失败 3 次后 10 秒内不再尝试
        server 192.168.1.100:80 max_fails=3 fail_timeout=10s;

        # 第一备份（B），仅当主服务不可用时启用
        server 192.168.1.101:80 backup max_fails=3 fail_timeout=10s;

        # 第二备份（C），仅当主服务和第一备份均不可用时启用
        server 192.168.1.102:80 backup;
    }

    server {
        listen 80;
        server_name example.com;  # 替换为你的域名或IP

        location / {
            proxy_pass http://backend;
            proxy_next_upstream error timeout http_500 http_502 http_503 http_504;
            proxy_connect_timeout 2s;  # 连接超时时间
            proxy_read_timeout 5s;     # 读取超时时间
        }
    }
}
```

### **2.2 检查并重载 Nginx

```
# 检查配置语法是否正确
sudo nginx -t

# 重新加载 Nginx 使配置生效
sudo systemctl reload nginx
```

------

## **3. 测试故障转移**

### **3.1 测试主服务（A）不可用**

1. **停止服务A**（模拟宕机）：

   ```
   # 如果是 Web 服务（如 Nginx/Apache）
   sudo systemctl stop nginx
   ```

2. **访问测试**：

   ```
   curl http://your-nginx-server/
   ```

   - **预期结果**：请求自动切换到服务 B。

### **3.2 测试主服务（A）和第一备份（B）均不可用**

1. **停止服务B**：

   ```
   sudo systemctl stop nginx  # 假设 B 也是 Nginx
   ```

2. **访问测试**：

   ```
   curl http://your-nginx-server/
   ```

   - **预期结果**：请求自动切换到服务 C。

### **3.3 恢复服务A，观察是否自动切换回主服务**

1. **启动服务A**：

   ```
   sudo systemctl start nginx
   ```

2. **访问测试**：

   ```
   curl http://your-nginx-server/
   ```

   

   - **预期结果**：流量自动切回服务 A。

------

## **4. 监控与日志**

### **4.1 查看 Nginx 访问日志

```
tail -f /var/log/nginx/access.log
```

- 检查请求是否按预期路由到 A/B/C。

### **4.2 查看 Nginx 错误日志

```
tail -f /var/log/nginx/error.log
```

- 检查是否有后端服务连接失败记录。

### **4.3 使用 `nginx-status`（Nginx Plus 适用）

```
curl http://localhost/nginx-status
```

- 查看当前活跃的后端服务器。

------

## **5. 高级配置（可选）**

### **5.1 使用 Nginx Plus 增强健康检查

```
upstream backend {
    zone backend 64k;
    server 192.168.1.100:80 resolve;
    server 192.168.1.101:80 resolve backup;
    server 192.168.1.102:80 resolve backup;
    health_check interval=5s fails=3 passes=2 uri=/health;
}
```

### **5.2 会话保持（Sticky Session）**

如果需要会话保持，可添加 `ip_hash`：

```
upstream backend {
    ip_hash;  # 基于客户端 IP 分配固定后端
    server 192.168.1.100:80;
    server 192.168.1.101:80 backup;
    server 192.168.1.102:80 backup;
}
```

------

## **6. 故障排查**

| 问题                 | 可能原因                               | 解决方案                                         |
| :------------------- | :------------------------------------- | :----------------------------------------------- |
| 请求未切换至备份服务 | `backup` 配置错误                      | 检查 `upstream` 是否正确定义 `backup`            |
| 切换延迟             | `max_fails` 或 `fail_timeout` 设置过长 | 调整超时时间（如 `max_fails=1 fail_timeout=5s`） |
| 502 Bad Gateway      | 所有后端均不可用                       | 检查服务 C 是否正常运行                          |

------

## **7. 总结**

本方案实现：
✅ **主服务（A）优先**
✅ **A 不可用时自动切换至 B**
✅ **A 和 B 均不可用时切换至 C**
✅ **A 恢复后自动切回主服务**

适用于高可用 Web 服务、API 网关等场景。