在CentOS 7.9上离线安装Nginx并包含`ngx_http_upstream_least_time_module`模块的步骤如下：

### 一、准备工作（在联网环境中进行）

1. **下载Nginx及依赖源码包**：

   - **Nginx**：从[官网](https://nginx.org/download/)下载最新稳定版，如`nginx-1.25.3.tar.gz`。
   - **PCRE**：下载[pcre-8.45](https://sourceforge.net/projects/pcre/files/pcre/8.45/pcre-8.45.tar.gz)。
   - **zlib**：下载[zlib-1.3](https://zlib.net/zlib-1.3.tar.gz)。
   - **OpenSSL**：下载[openssl-3.0.12](https://www.openssl.org/source/openssl-3.0.12.tar.gz)。
   - **第三方模块**：下载[ngx_http_upstream_least_time_module](https://github.com/yzprofile/ngx_http_upstream_least_time_module/archive/refs/heads/master.zip)。

2. **下载编译工具和依赖的RPM包**：

   bash

   ```
   # 停止 Nginx 服务
   sudo systemctl stop nginx
   
   # 禁用开机启动
   sudo systemctl disable nginx
   
   # 卸载 Nginx
   sudo yum remove nginx
   ```

   ```
   mkdir rpms
   yum install yum-utils -y
   yumdownloader --resolve --destdir=rpms gcc make automake autoconf libtool pcre-devel zlib-devel openssl-devel
   
   yum -y install gcc pcre-devel  zlib-devel openssl-devel libxml2-devel libxslt-devel gd-devel GeoIP-devel jemalloc-devel libatomic_ops-devel perl-devel  perl-ExtUtils-Embed
    
   
   
   yum -y install gcc pcre-devel  zlib-devel openssl-devel libxml2-devel libxslt-devel gd-devel GeoIP-devel jemalloc-devel libatomic_ops-devel perl-devel  perl-ExtUtils-Embed
    
   
   yum -y install yum-utils  gcc make automake autoconf libtool pcre-devel zlib-devel openssl-devel
   ```

3. **打包所有文件**：
   将Nginx、依赖库源码、第三方模块和`rpms`目录打包，传输到离线服务器。

------

### 二、离线环境安装

1. **安装编译工具和依赖**：

   bash

   复制

   ```
   cd rpms
   rpm -ivh *.rpm --nodeps --force  # 强制安装，忽略依赖（确保所有包已下载）
   ```

2. **解压源码包**：

   bash

   复制

   ```
   tar -xzf nginx-1.25.3.tar.gz
   tar -xzf pcre-8.45.tar.gz
   tar -xzf zlib-1.3.tar.gz
   tar -xzf openssl-3.0.12.tar.gz
   unzip ngx_http_upstream_least_time_module-master.zip -d ngx_http_upstream_least_time_module
   ```

------

### 三、编译安装Nginx

1. **进入Nginx源码目录并配置**：

   bash

   复制

   ```
   cd nginx-1.25.3
   ./configure \
   --prefix=/usr/local/nginx \
   --sbin-path=/usr/sbin/nginx \
   --conf-path=/etc/nginx/nginx.conf \
   --error-log-path=/var/log/nginx/error.log \
   --http-log-path=/var/log/nginx/access.log \
   --pid-path=/var/run/nginx.pid \
   --lock-path=/var/run/nginx.lock \
   --user=nginx \
   --group=nginx \
   --with-pcre=../pcre-8.45 \
   --with-zlib=../zlib-1.3 \
   --with-openssl=../openssl-3.0.12 \
   --with-http_ssl_module \
   --with-http_realip_module \
   --with-http_stub_status_module \
   --with-http_v2_module \
   --with-threads \
   --with-stream \
   --add-module=../ngx_http_upstream_least_time_module-master
   ```

   ```
   cd nginx-1.28.0/
   ```

   

   ```
   ./configure \
   --prefix=/usr/local/nginx \
   --sbin-path=/usr/sbin/nginx \
   --conf-path=/etc/nginx/nginx.conf \
   --error-log-path=/var/log/nginx/error.log \
   --http-log-path=/var/log/nginx/access.log \
   --pid-path=/var/run/nginx.pid \
   --lock-path=/var/run/nginx.lock \
   --user=nginx \
   --group=nginx \
   --with-cc-opt="-Wno-error"  \
   --with-http_ssl_module \
   --with-http_sub_module \
   --with-http_random_index_module \
   --with-http_auth_request_module \
   --with-http_realip_module \
   --with-http_stub_status_module \
   --with-http_gunzip_module \
   --with-http_gzip_static_module \
   --with-http_v2_module \
   --with-http_addition_module \
   --with-threads \
   --with-stream \
   --add-module=../ngx_http_upstream_least_time-main \
   --add-module=../nginx_upstream_check_module-master \
   --add-module=../nginx-upstream-fair-master
   ```

   ```bash
   make distclean  # 清理之前的配置
   ```

   **说明**：根据需求添加其他模块，或使用`--with-..._module`启用更多官方模块。

2. **编译并安装**：

   bash

   复制

   ```
   make && make install
   ```

------

### 四、配置系统服务

1. **创建Nginx用户和日志目录**：

   bash

   复制

   ```
   useradd -M -s /sbin/nologin nginx
   mkdir -p /var/log/nginx
   ```

2. **创建Systemd服务文件**：

   bash

   复制

   ```
   cat > /etc/systemd/system/nginx.service <<EOF
   [Unit]
   Description=The nginx HTTP and reverse proxy server
   After=network.target
   
   [Service]
   Type=forking
   PIDFile=/var/run/nginx.pid
   ExecStartPre=/usr/sbin/nginx -t
   ExecStart=/usr/sbin/nginx
   ExecReload=/usr/sbin/nginx -s reload
   KillSignal=SIGQUIT
   TimeoutStopSec=5
   KillMode=process
   PrivateTmp=true
   
   [Install]
   WantedBy=multi-user.target
   EOF
   ```

3. **启动Nginx并设置开机自启**：

   bash

   复制

   ```
   systemctl daemon-reload
   systemctl start nginx
   systemctl enable nginx
   ```

------

### 五、验证安装

1. **检查模块是否加载**：

   bash

   复制

   ```
   nginx -V 2>&1 | grep least_time_module
   ```

   输出应包含`--add-module=../ngx_http_upstream_least_time_module`。

2. **访问测试**：

   bash

   复制

   ```
   curl http://localhost
   ```

------

五、验证安装

1. **修改 Nginx 配置**：

   nginx

   复制

   ```
   http {
       map $time_iso8601 $logdate {
           '~^(?<ymd>\d{4}-\d{2}-\d{2})' $ymd;
           default 'date-not-found';
       }
       
       access_log /var/log/nginx/access-$logdate.log main;
   }
   ```

```
sudo chown -R nginx:root /var/log/nginx/
sudo chmod -R 755 /var/log/nginx/
```

**注意事项**：

- 确保所有依赖库路径正确，避免编译错误。
- 若第三方模块与Nginx版本不兼容，需调整版本或修改模块代码。
- 离线安装需提前测试所有步骤，避免依赖缺失。