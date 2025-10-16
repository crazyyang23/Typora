# 实现Nginx文件显示与下载功能

## 完整配置方案

以下是实现文件列表显示并可点击下载的完整Nginx配置方案：

### 1. Nginx配置文件 (`/etc/nginx/conf.d/filebrowser.conf`)

nginx

复制

```
server {
    listen 80;
    server_name center.gwkf.cn;
    
    # 文件浏览和下载配置
    location /logs/ {
        alias /var/www/logs/;  # 实际存储目录
        
        # 启用目录列表
        autoindex on;
        autoindex_exact_size off;  # 显示易读的文件大小
        autoindex_localtime on;    # 显示本地时间
        
        # 确保文件可下载
        types {
            application/octet-stream bin exe dll;
            text/plain log txt;
            text/css css;
            text/javascript js;
            image/gif gif;
            image/jpeg jpg jpeg;
            image/png png;
            image/svg+xml svg;
            application/pdf pdf;
            application/json json;
        }
        default_type application/octet-stream;
        
        # 禁用某些文件类型的执行
        location ~* \.(php|sh|py|pl)$ {
            deny all;
            return 403;
        }
        
        # 安全设置
        disable_symlinks if_not_owner;
    }
    
    # 错误页面
    error_page 403 /403.html;
    error_page 404 /404.html;
    location = /403.html, /404.html {
        root /usr/share/nginx/html;
        internal;
    }
}
```

### 2. 实现步骤

#### 2.1 准备文件目录

bash

复制

```
# 创建日志目录
sudo mkdir -p /var/www/logs
sudo chown -R nginx:nginx /var/www/logs
sudo chmod -R 755 /var/www/logs

# 创建测试文件
echo "This is a test log file" | sudo tee /var/www/logs/test.log
dd if=/dev/zero of=/var/www/logs/largefile.bin bs=1M count=10
```

#### 2.2 配置Nginx

bash

复制

```
# 检查配置语法
sudo nginx -t

# 重启Nginx
sudo systemctl restart nginx
```

### 3. 增强功能

#### 3.1 添加密码保护

bash

复制

```
# 安装htpasswd工具
sudo yum install httpd-tools -y

# 创建密码文件
sudo htpasswd -c /etc/nginx/.htpasswd username
```

在Nginx配置中添加：

nginx

复制

```
location /logs/ {
    auth_basic "Restricted Access";
    auth_basic_user_file /etc/nginx/.htpasswd;
    # ...其他配置...
}
```

#### 3.2 启用HTTPS

bash

复制

```
# 使用Certbot获取证书
sudo yum install certbot python3-certbot-nginx -y
sudo certbot --nginx -d center.gwkf.cn
```

### 4. 高级功能

#### 4.1 文件上传功能

nginx

复制

```
location /upload/ {
    # 允许上传
    client_max_body_size 100M;
    dav_methods PUT DELETE MKCOL COPY MOVE;
    create_full_put_path on;
    dav_access user:rw group:rw all:r;
    
    # 需要认证
    auth_basic "Restricted Upload";
    auth_basic_user_file /etc/nginx/.htpasswd;
}
```

#### 4.2 文件搜索功能

nginx

复制

```
location /logs/ {
    # 启用SSI执行
    ssi on;
    
    # 添加搜索表单
    autoindex_format html;
    autoindex_localtime on;
    add_before_body /.autoindex/search.html;
}
```

创建搜索页面 `/var/www/logs/.autoindex/search.html`:

html

复制

```
<form action="/logs/" method="get">
  <input type="text" name="q" placeholder="Search files...">
  <button type="submit">Search</button>
</form>
```

运行 HTML

### 5. 安全加固

#### 5.1 限制IP访问

nginx

复制

```
location /logs/ {
    allow 192.168.1.0/24;
    allow 10.0.0.1;
    deny all;
    # ...其他配置...
}
```

#### 5.2 防止目录遍历

nginx

复制

```
location /logs/ {
    # 禁止访问隐藏文件
    location ~ /\. {
        deny all;
    }
    
    # 禁止访问上级目录
    if ($request_filename ~* "\.\./") {
        return 403;
    }
}
```

## 最终效果

1. 访问 `http://center.gwkf.cn/logs/` 显示文件列表
2. 点击文件名直接下载文件
3. 管理员可通过认证上传文件
4. 所有传输可通过HTTPS加密

## 维护建议

1. 定期检查日志：

   bash

   复制

   ```
   sudo tail -f /var/log/nginx/access.log
   sudo tail -f /var/log/nginx/error.log
   ```

2. 设置日志轮转：

   bash

   复制

   ```
   sudo logrotate -f /etc/logrotate.d/nginx
   ```

3. 监控磁盘空间：

   bash

   复制

   ```
   df -h /var/www/logs
   ```

此配置提供了完整的文件浏览和下载功能，同时考虑了安全性和可维护性。