# Nginx日志按天生成并压缩操作手册

## 1. 手册概述

本手册详细介绍了如何配置Nginx服务器使其日志按天生成并自动压缩的三种方法，包括使用Nginx内置功能+logrotate、cron脚本实现以及Linux系统时间变量方法。

## 2. 准备工作

### 2.1 系统要求

- Linux服务器
- 已安装Nginx
- root或sudo权限

### 2.2 检查当前Nginx配置

bash

复制

```
nginx -V
ls -l /var/log/nginx/
```

### 2.3 备份现有配置

bash

复制

```
cp /etc/nginx/nginx.conf /etc/nginx/nginx.conf.bak
```

## 3. 方法一：使用Nginx内置功能+logrotate

### 3.1 配置Nginx日志

1. 编辑Nginx主配置文件：

bash

复制

```
sudo vi /etc/nginx/nginx.conf
```

1. 在http块中添加或修改以下内容：

nginx

复制

```
http {
    log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                    '$status $body_bytes_sent "$http_referer" '
                    '"$http_user_agent" "$http_x_forwarded_for"';

    access_log /var/log/nginx/access-%Y-%m-%d.log main;
    error_log /var/log/nginx/error-%Y-%m-%d.log;
    
    # 其他现有配置保持不变...
}
```

1. 检查配置语法：

bash

复制

```
sudo nginx -t
```

1. 重新加载Nginx：

bash

复制

```
sudo systemctl reload nginx
```

### 3.2 配置logrotate

1. 创建或编辑logrotate配置文件：

bash

复制

```
sudo vi /etc/logrotate.d/nginx
```

1. 添加以下内容：

复制

```
/var/log/nginx/*.log {
    daily
    missingok
    rotate 30
    compress
    delaycompress
    notifempty
    create 0640 www-data adm
    sharedscripts
    postrotate
        if [ -f /var/run/nginx.pid ]; then
            kill -USR1 `cat /var/run/nginx.pid`
        fi
    endscript
}
```

1. 测试logrotate配置：

bash

复制

```
sudo logrotate -d /etc/logrotate.d/nginx
```

1. 手动运行logrotate（可选）：

bash

复制

```
sudo logrotate -f /etc/logrotate.d/nginx
```

## 4. 方法二：使用cron脚本实现

### 4.1 创建日志切割脚本

1. 创建脚本文件：

bash

复制

```
sudo vi /usr/local/bin/nginx_logrotate.sh
```

1. 添加以下内容：

bash

复制

```
#!/bin/bash

# 定义日志目录
LOG_DIR="/var/log/nginx"
# 获取昨天的日期
YESTERDAY=$(date -d "yesterday" +%Y-%m-%d)

# 压缩昨天的日志
find $LOG_DIR -type f -name "access-${YESTERDAY}.log" -exec gzip {} \;
find $LOG_DIR -type f -name "error-${YESTERDAY}.log" -exec gzip {} \;

# 重新加载Nginx以创建新的日志文件
kill -USR1 $(cat /var/run/nginx.pid)
```

1. 设置执行权限：

bash

复制

```
sudo chmod +x /usr/local/bin/nginx_logrotate.sh
```

### 4.2 设置cron任务

1. 编辑crontab：

bash

复制

```
sudo crontab -e
```

1. 添加以下行（每天午夜执行）：

复制

```
0 0 * * * /usr/local/bin/nginx_logrotate.sh
```

1. 保存并退出。

## 5. 方法三：使用Linux系统时间变量（推荐）

### 5.1 修改Nginx配置

1. 编辑Nginx配置文件：

bash

复制

```
sudo vi /etc/nginx/nginx.conf
```

1. 在http块中添加：

nginx

复制

```
http {
    map $time_iso8601 $logdate {
        '~^(?<ymd>\d{4}-\d{2}-\d{2})' $ymd;
        default 'date-not-found';
    }

    access_log /var/log/nginx/access-$logdate.log main;
    error_log /var/log/nginx/error-$logdate.log;
    
    # 其他现有配置保持不变...
}
```

1. 检查并重新加载配置：

bash

复制

```
sudo nginx -t && sudo systemctl reload nginx
```

### 5.2 配置logrotate（同方法一）

## 6. 验证配置

### 6.1 检查日志文件

bash

复制

```
ls -lh /var/log/nginx/
```

### 6.2 模拟日志生成

bash

复制

```
sudo tail -f /var/log/nginx/access-$(date +%Y-%m-%d).log
```

### 6.3 检查压缩文件

等待第二天检查：

bash

复制

```
ls -lh /var/log/nginx/*.gz
```

## 7. 常见问题解决

### 7.1 权限问题

bash

复制

```
sudo chown -R www-data:adm /var/log/nginx
sudo chmod -R 750 /var/log/nginx
```

### 7.2 日志不滚动

检查Nginx进程是否收到USR1信号：

bash

复制

```
ps aux | grep nginx
```

### 7.3 磁盘空间不足

调整保留日志天数（修改rotate值）或增加磁盘空间。

## 8. 维护建议

1. 定期检查日志文件大小和数量
2. 根据业务需求调整日志保留天数
3. 考虑将日志归档到其他存储系统
4. 设置日志监控告警

## 9. 附录

### 9.1 相关命令参考

- `nginx -t` - 测试Nginx配置
- `systemctl reload nginx` - 重新加载Nginx配置
- `logrotate -f /etc/logrotate.d/nginx` - 强制运行logrotate

### 9.2 配置文件位置

- Nginx主配置：/etc/nginx/nginx.conf
- logrotate配置：/etc/logrotate.d/nginx
- cron脚本：/usr/local/bin/nginx_logrotate.sh

### 9.3 推荐阅读

- Nginx官方文档：http://nginx.org/en/docs/
- logrotate手册页：`man logrotate`
- cron使用指南：`man crontab`

------

本手册提供了三种实现Nginx日志按天生成并压缩的方法，推荐使用方法一或方法三，它们在大多数生产环境中表现更稳定可靠。根据实际需求和服务器环境选择最适合的方案。