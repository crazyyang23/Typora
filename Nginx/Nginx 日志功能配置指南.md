# Nginx 日志功能配置指南

Nginx 的日志功能主要包括访问日志(access_log)和错误日志(error_log)，以下是启用和关闭这些日志的方法：

## 一、访问日志配置

### 1. 启用访问日志

nginx

复制

```
http {
    # 定义日志格式
    log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                   '$status $body_bytes_sent "$http_referer" '
                   '"$http_user_agent" "$http_x_forwarded_for"';

    # 启用访问日志并指定路径和格式
    access_log /var/log/nginx/access.log main;
}
```

### 2. 关闭访问日志

方法一：设置为 off

nginx

复制

```
access_log off;
```

方法二：注释掉 access_log 指令

nginx

复制

```
# access_log /var/log/nginx/access.log;
```

### 3. 为特定 location 关闭访问日志

nginx

复制

```
location /static/ {
    access_log off;
    ...
}
```

## 二、错误日志配置

### 1. 启用错误日志

nginx

复制

```
# 在全局、http、server 或 location 上下文中配置
error_log /var/log/nginx/error.log warn;  # warn级别及以上记录
```

日志级别（从低到高）：

- debug
- info
- notice
- warn
- error
- crit
- alert
- emerg

### 2. 关闭错误日志

nginx

复制

```
error_log /dev/null;  # 将错误日志重定向到空设备
```

或者（不推荐）：

nginx

复制

```
error_log off;  # 某些版本可能不支持
```

## 三、动态日志配置

### 1. 条件日志记录

nginx

复制

```
map $status $loggable {
    ~^[23]  0;  # 2xx和3xx状态码不记录
    default 1;  # 其他状态码记录
}

access_log /var/log/nginx/access.log main if=$loggable;
```

### 2. 按日期分割日志（已在您的配置中使用）

nginx

复制

```
map $time_iso8601 $logdate {
    '~^(?<ymd>\d{4}-\d{2}-\d{2})' $ymd;
    default 'date-not-found';
}

access_log /var/log/nginx/access-$logdate.log main;
```

## 四、配置示例

### 完整示例（启用日志）

nginx

复制

```
http {
    log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                   '$status $body_bytes_sent "$http_referer" '
                   '"$http_user_agent" "$http_x_forwarded_for"';

    access_log /var/log/nginx/access.log main;
    error_log /var/log/nginx/error.log warn;

    server {
        listen 80;
        server_name example.com;
        
        location / {
            root /var/www/html;
        }
        
        location /private/ {
            access_log off;  # 关闭此位置的访问日志
            ...
        }
    }
}
```

### 完整示例（关闭日志）

nginx

复制

```
http {
    # 不定义log_format
    access_log off;
    error_log /dev/null;

    server {
        listen 80;
        server_name example.com;
        ...
    }
}
```

## 五、注意事项

1. 修改配置后需要重新加载Nginx使更改生效：

   bash

   复制

   ```
   nginx -s reload
   ```

2. 生产环境不建议完全关闭错误日志，至少保留warn级别

3. 高流量网站应考虑：

   - 日志轮转（使用logrotate）
   - 日志压缩
   - 只记录必要的请求

4. 调试时可临时启用debug级别：

   nginx

   

   ```
   error_log /var/log/nginx/debug.log debug;
   ```

   调试完成后记得改回正常级别