# **Nginx日志目录访问配置手册**

## **通过Nginx实现`/root/center/logs`文件浏览与下载**

------

## **1. 目标**

配置Nginx，使`http://center.gwkf.cn/logs/`能够安全地浏览和下载`/root/center/logs`下的日志文件。

------

## **2. 前提条件**

- CentOS 7/8 系统
- Nginx 已安装并运行
- `root` 权限或 `sudo` 权限

------

## **3. 方案选择**

### **方案一：调整`/root`目录权限（测试用，不推荐生产环境）**

**适用场景**：快速测试，临时使用
**风险**：降低`/root`目录安全性

### **方案二：迁移日志目录至`/var/log/center_logs`（推荐生产环境）**

**适用场景**：长期稳定运行
**优势**：符合Linux目录规范，安全性更高

------

## **4. 方案一：调整`/root`权限（临时方案）**

### **4.1 修改`/root`目录权限**

bash

复制

```
sudo chmod 751 /root  # 允许组用户进入
```

### **4.2 验证Nginx能否访问**

bash

复制

```
sudo -u nginx ls -l /root/center/logs
```

### **4.3 修改Nginx配置**

bash

复制

```
sudo vi /etc/nginx/conf.d/logs.conf
```

写入：

nginx

复制

```
server {
    listen 80;
    server_name center.gwkf.cn;

    location /logs/ {
        alias /root/center/logs/;
        autoindex on;
        autoindex_exact_size off;
        autoindex_localtime on;
        
        # 可选：添加HTTP认证
        auth_basic "Restricted Access";
        auth_basic_user_file /etc/nginx/.htpasswd;
    }
}
```

### **4.4 测试并重启Nginx**

bash

复制

```
sudo nginx -t
sudo systemctl restart nginx
```

### **4.5 访问测试**

bash

复制

```
curl -I http://center.gwkf.cn/logs/
```

**⚠️ 警告**：此方案会降低系统安全性，仅建议临时测试使用。

------

## **5. 方案二：迁移日志目录（推荐生产环境）**

### **5.1 创建新的日志目录**

bash

复制

```
sudo mkdir -p /var/log/center_logs
sudo chown -R nginx:nginx /var/log/center_logs
sudo chmod -R 755 /var/log/center_logs
```

### **5.2 迁移现有日志**

bash

复制

```
sudo mv /root/center/logs/* /var/log/center_logs/
```

### **5.3 配置Nginx**

bash

复制

```
sudo vi /etc/nginx/conf.d/logs.conf
```

写入：

nginx

复制

```
server {
    listen 80;
    server_name center.gwkf.cn;

    location /logs/ {
        alias /var/log/center_logs/;
        autoindex on;
        autoindex_exact_size off;
        autoindex_localtime on;
        
        # 可选：HTTP认证
        auth_basic "Restricted Access";
        auth_basic_user_file /etc/nginx/.htpasswd;
    }
}
```

### **5.4 设置HTTP认证（可选）**

bash

复制

```
# 安装htpasswd工具（若未安装）
sudo yum install httpd-tools -y

# 创建认证文件
sudo sh -c "echo -n 'admin:' >> /etc/nginx/.htpasswd"
sudo sh -c "openssl passwd -apr1 >> /etc/nginx/.htpasswd"
```

### **5.5 测试并重启Nginx**

bash

复制

```
sudo nginx -t
sudo systemctl restart nginx
```

### **5.6 访问测试**

bash

复制

```
curl -I http://center.gwkf.cn/logs/
```

或浏览器访问：

复制

```
http://center.gwkf.cn/logs/
```

------

## **6. SELinux配置（如启用）**

### **6.1 检查SELinux状态**

bash

复制

```
getenforce  # 返回Enforcing表示启用
```

### **6.2 设置SELinux策略**

bash

复制

```
sudo semanage fcontext -a -t httpd_sys_content_t "/var/log/center_logs(/.*)?"
sudo restorecon -Rv /var/log/center_logs
sudo setsebool -P httpd_read_user_content 1
```

------

## **7. 日志轮转（Logrotate）**

### **7.1 创建日志轮转配置**

bash

复制

```
sudo vi /etc/logrotate.d/center_logs
```

写入：

复制

```
/var/log/center_logs/*.log {
    daily
    missingok
    rotate 30
    compress
    delaycompress
    notifempty
    create 640 nginx nginx
}
```

### **7.2 手动测试日志轮转**

bash

复制

```
sudo logrotate -vf /etc/logrotate.d/center_logs
```

------

## **8. 验证与排错**

### **8.1 检查Nginx错误日志**

bash

复制

```
sudo tail -f /var/log/nginx/error.log
```

### **8.2 验证目录权限**

bash

复制

```
namei -l /var/log/center_logs
```

### **8.3 模拟Nginx访问**

bash

复制

```
sudo -u nginx ls -la /var/log/center_logs
```

------

## **9. 安全加固建议**

1. **启用HTTPS**：使用Let's Encrypt免费证书
2. **IP访问限制**：仅允许特定IP访问`/logs/`
3. **定期审计**：检查日志目录权限是否被篡改

------

## **10. 总结**

| 方案            | 适用场景 | 安全性 | 推荐度 |
| :-------------- | :------- | :----- | :----- |
| 调整`/root`权限 | 临时测试 | 低     | ⭐☆☆☆☆  |
| 迁移日志目录    | 生产环境 | 高     | ⭐⭐⭐⭐⭐  |

**最终建议**：采用**方案二**，并启用HTTP认证和HTTPS加密访问。

------

**✅ 完成！** 现在可通过 `http://center.gwkf.cn/logs/` 安全访问日志文件。