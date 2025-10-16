## Nginx 离线安装操作手册

### 1. 环境准备

#### 1.1 系统要求

- 操作系统：CentOS 7.9
- 用户权限：root 或具有 sudo 权限的用户

#### 1.2 依赖包准备

在一台可以访问互联网的机器上下载以下依赖包和源码包：

- **Nginx 源码包**：

  bash

  复制

  ```
  wget https://nginx.org/download/nginx-1.26.0.tar.gz
  ```

- **依赖库源码包**：

  bash

  复制

  ```
  wget https://ftp.pcre.org/pub/pcre/pcre-8.45.tar.gz
  wget https://zlib.net/zlib-1.2.11.tar.gz
  wget https://www.openssl.org/source/openssl-1.1.1w.tar.gz
  ```

- **编译工具**（如果离线机器上没有）：

  - `gcc`
  - `make`
  - `pcre`
  - `zlib`
  - `openssl`

将这些文件传输到离线的 CentOS 7.9 机器上。

------

### 2. 安装编译工具

在离线机器上安装编译工具（如果尚未安装）：

bash

复制

```
sudo yum install -y gcc make
```

如果无法使用 `yum`，可以手动下载 RPM 包并安装（参考前面的步骤）。

------

### 3. 解压源码包

解压所有下载的源码包：

bash

复制

```
tar -zxvf nginx-1.26.0.tar.gz
tar -zxvf pcre-8.45.tar.gz
tar -zxvf zlib-1.2.11.tar.gz
tar -zxvf openssl-1.1.1w.tar.gz
```

------

### 4. 编译依赖库

进入每个依赖库的目录并编译：

#### 4.1 编译 PCRE

bash

复制

```
cd pcre-8.45
./configure
make
sudo make install
cd ..
```

#### 4.2 编译 zlib

bash

复制

```
cd zlib-1.2.11
./configure
make
sudo make install
cd ..
```

#### 4.3 编译 OpenSSL

bash

复制

```
cd openssl-1.1.1w
./config --prefix=/usr/local/openssl --openssldir=/usr/local/openssl
make
sudo make install
cd ..
```

------

### 5. 编译 Nginx

进入 Nginx 源码目录并编译：

bash

复制

```
cd nginx-1.26.0
./configure \
--prefix=/usr/local/nginx \
--with-pcre=../pcre-8.45 \
--with-zlib=../zlib-1.2.11 \
--with-openssl=../openssl-1.1.1w \
--with-http_ssl_module \
--with-http_stub_status_module
make
sudo make install
```

------

### 6. 配置 Nginx

#### 6.1 修改配置文件

编辑 Nginx 配置文件 `/usr/local/nginx/conf/nginx.conf`，确保配置正确。例如：

nginx

复制

```
user nginx;
worker_processes auto;

events {
    worker_connections 1024;
}

http {
    include       mime.types;
    default_type  application/octet-stream;

    server {
        listen       80;
        server_name  localhost;

        location / {
            root   html;
            index  index.html index.htm;
        }

        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
}
```

#### 6.2 测试配置文件

检查配置文件是否正确：

bash

复制

```
/usr/local/nginx/sbin/nginx -t
```

如果显示 `syntax is ok` 和 `test is successful`，说明配置文件正确。

------

### 7. 配置 Systemd 服务

#### 7.1 创建服务文件

创建 Systemd 服务文件 `/etc/systemd/system/nginx.service`：

bash

复制

```
sudo vi /etc/systemd/system/nginx.service
```

添加以下内容：

ini

复制

```
[Unit]
Description=The NGINX HTTP and reverse proxy server
After=network.target

[Service]
Type=forking
PIDFile=/usr/local/nginx/logs/nginx.pid
ExecStartPre=/usr/local/nginx/sbin/nginx -t
ExecStart=/usr/local/nginx/sbin/nginx
ExecReload=/usr/local/nginx/sbin/nginx -s reload
ExecStop=/bin/kill -s QUIT $MAINPID
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```

#### 7.2 设置权限

确保服务文件的权限正确：

bash

复制

```
sudo chmod 644 /etc/systemd/system/nginx.service
```

#### 7.3 重新加载 Systemd 配置

重新加载 Systemd 配置：

bash

复制

```
sudo systemctl daemon-reload
```

#### 7.4 启用并启动服务

启用并启动 Nginx 服务：

bash

复制

```
sudo systemctl enable nginx
sudo systemctl start nginx
```

#### 7.5 检查服务状态

检查 Nginx 服务状态：

bash

复制

```
sudo systemctl status nginx
```

------

### 8. 解决 `nginx` 命令不可用问题

#### 8.1 临时添加 PATH

在当前会话中临时添加 PATH：

bash

复制

```
export PATH=/usr/local/nginx/sbin:$PATH
```

验证是否生效：

bash

复制

```
nginx -v
```

#### 8.2 永久添加 PATH

编辑 `~/.bashrc` 文件：

bash

复制

```
vi ~/.bashrc
```

在文件末尾添加以下内容：

bash

复制

```
export PATH=/usr/local/nginx/sbin:$PATH
```

保存并退出，然后运行以下命令使配置生效：

bash

复制

```
source ~/.bashrc
```

#### 8.3 创建符号链接（可选）

创建符号链接到 `/usr/local/bin` 或 `/usr/bin`：

bash

复制

```
sudo ln -s /usr/local/nginx/sbin/nginx /usr/local/bin/nginx
```

验证符号链接：

bash

复制

```
nginx -v
```

------

### 9. 验证安装

访问服务器的 IP 地址或域名，确认 Nginx 正常运行：

bash

复制

```
curl http://localhost
```

如果看到 Nginx 的欢迎页面，说明安装成功。

------

### 10. 日志管理

#### 10.1 修改日志路径

将日志文件路径修改为 `/var/log/nginx`：

1. 创建日志目录：

   bash

   复制

   ```
   sudo mkdir -p /var/log/nginx
   sudo chown -R nginx:nginx /var/log/nginx
   ```

2. 修改 Nginx 配置文件：

   nginx

   复制

   ```
   error_log  /var/log/nginx/error.log;
   access_log /var/log/nginx/access.log;
   ```

3. 重启 Nginx：

   bash

   复制

   ```
   sudo systemctl restart nginx
   ```

#### 10.2 配置日志轮转

创建 Logrotate 配置文件 `/etc/logrotate.d/nginx`：

bash

复制

```
sudo vi /etc/logrotate.d/nginx
```

添加以下内容：

logrotate

复制

```
/var/log/nginx/*.log {
    daily
    missingok
    rotate 30
    compress
    delaycompress
    notifempty
    create 0640 nginx nginx
    sharedscripts
    postrotate
        /usr/local/nginx/sbin/nginx -s reload > /dev/null 2>&1 || true
    endscript
}
```

测试 Logrotate：

bash

复制

```
sudo logrotate -vf /etc/logrotate.d/nginx
```

------

### 总结

通过以上步骤，您已经完成了 Nginx 的离线安装、Systemd 服务配置、`nginx` 命令不可用问题的解决以及日志管理。Nginx 已成功安装并运行。