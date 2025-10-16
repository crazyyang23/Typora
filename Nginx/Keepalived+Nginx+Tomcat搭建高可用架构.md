![Keepalived+Nginx+Tomcat搭建高可用架构](https://pica.zhimg.com/70/v2-e9dd1f9340d7b0c0e8e3e06395bbd094_1440w.image?source=172ae18b&biz_tag=Post)

# Keepalived+Nginx+Tomcat搭建高可用架构

[![星火技术](https://pica.zhimg.com/v2-7058ba672b65516068e5bc366b13dd28_l.jpg?source=172ae18b)](https://www.zhihu.com/people/hopeit)

[星火技术](https://www.zhihu.com/people/hopeit)[](https://www.zhihu.com/question/48510028)

信息技术行业 从业人员

关注她

4 人赞同了该文章





要详细地在 [CentOS 7](https://zhida.zhihu.com/search?content_id=250602788&content_type=Article&match_order=1&q=CentOS+7&zhida_source=entity) 环境下使用 [Nginx](https://zhida.zhihu.com/search?content_id=250602788&content_type=Article&match_order=1&q=Nginx&zhida_source=entity) 和 [Keepalived](https://zhida.zhihu.com/search?content_id=250602788&content_type=Article&match_order=1&q=Keepalived&zhida_source=entity) 实现高可用和负载均衡，我们需要深入理解每个组件的作用和配置方法。以下是详细的步骤和原理说明：

### 一、环境准备

1. **安装 CentOS 7**:
2. 安装两台或更多服务器作为后端节点（例如 [Tomcat](https://zhida.zhihu.com/search?content_id=250602788&content_type=Article&match_order=1&q=Tomcat&zhida_source=entity) 服务器）。
3. 安装一台或多台前端节点作为 Nginx 和 Keepalived 服务器。
4. **关闭防火墙** (可选):
5. 如果你的系统中有防火墙，你可能需要关闭它或者配置允许相关的端口。 `bash systemctl stop firewalld systemctl disable firewalld`
6. **禁用 [SELinux](https://zhida.zhihu.com/search?content_id=250602788&content_type=Article&match_order=1&q=SELinux&zhida_source=entity)** (可选):
7. 如果 SELinux 阻碍了服务运行，可以暂时禁用它。 `bash sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config setenforce 0`

### 二、配置 Keepalived

### 2.1 安装 Keepalived

```bash
yum install keepalived -y
```

### 2.2 配置 Keepalived

Keepalived 是一个基于 [VRRP](https://zhida.zhihu.com/search?content_id=250602788&content_type=Article&match_order=1&q=VRRP&zhida_source=entity)（Virtual Router Redundancy Protocol）的高可用解决方案，用于监控主服务器的状态并在主服务器出现故障时接管其虚拟 IP 地址。

- **编辑 `/etc/keepalived/keepalived.conf` 文件**:

\```conf global_defs { router_id LVS_DEVEL }

vrrp_script chk_nginx { script "/usr/local/bin/check_nginx.sh" interval 2 weight -2 }

vrrp_instance VI_1 { state MASTER interface eth0 virtual_router_id 51 priority 100 advert_int 1 authentication { auth_type PASS auth_pass 1111 } virtual_ipaddress { 192.168.1.100 } track_script { chk_nginx } } ```

- **参数解释**:
- `global_defs`: 全局定义。
- `vrrp_script`: 定义一个外部脚本来检查 Nginx 的状态。
- `interval`: 检查间隔。
- `weight`: 当脚本失败时减少的优先级。
- `state`: MASTER 或 BACKUP，表示当前实例的状态。
- `interface`: 监听的网络接口。
- `virtual_router_id`: 虚拟路由器 ID，必须所有实例相同。
- `priority`: 优先级，MASTER 应该比 BACKUP 的优先级更高。
- `advert_int`: 广播间隔。
- `authentication`: 认证方式和密码。
- `virtual_ipaddress`: 虚拟 IP 地址。
- `track_script`: 跟踪脚本，当脚本返回非零状态码时，优先级将被降低。

### 2.3 编写检查脚本

创建 `/usr/local/bin/check_nginx.sh` 并设置权限。

```bash
#!/bin/bash
if [ "$(curl -Is http://127.0.0.1 | head -n 1)" = "HTTP/1.1 200 OK" ]; then
    exit 0
else
    exit 2
fi
chmod +x /usr/local/bin/check_nginx.sh
```

- **脚本作用**: 检查 Nginx 是否正常运行。

### 2.4 启动 Keepalived

```bash
systemctl start keepalived
systemctl enable keepalived
```

### 三、配置 Nginx

### 3.1 安装 Nginx

```bash
yum install epel-release -y
yum install nginx -y
```

### 3.2 配置 Nginx

Nginx 是一款高性能的 HTTP 和反向代理 Web 服务器，用于实现负载均衡。

- **编辑 `/etc/nginx/nginx.conf` 或 `/etc/nginx/conf.d/default.conf`**:

\```nginx upstream tomcat_cluster { server 192.168.1.200:8080; server 192.168.1.201:8080; }

server { listen 80; server_name example.com;

```text
location / {
      proxy_pass http://tomcat_cluster;
      proxy_set_header Host $host;
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
  }
```

} ```

- **参数解释**:
- `upstream`: 定义一组后端服务器。
- `server`: 定义单个后端服务器地址和端口。
- `listen`: 监听端口。
- `server_name`: 定义域名。
- `location`: URL 匹配模式。
- `proxy_pass`: 指定后端服务器组。
- `proxy_set_header`: 设置代理请求头。

### 3.3 启动 Nginx

```bash
systemctl start nginx
systemctl enable nginx
```

### 四、配置 Tomcat

### 4.1 安装 Tomcat

- **下载并解压 Tomcat**: `bash wget https://mirrors.tuna.tsinghua.edu.cn/apache/tomcat/tomcat-8/v8.5.80/bin/apache-tomcat-8.5.80.tar.gz tar -zxvf apache-tomcat-8.5.80.tar.gz cd apache-tomcat-8.5.80/`

### 4.2 配置 Tomcat

Tomcat 是一个开源的 Servlet 容器，用于部署 Java 应用程序。

- **修改 `server.xml`**:
- 开启集群支持。
- 配置 `Catalina` 引擎的 `cluster` 属性。
- 可以使用 `DeltManager` 来同步会话状态。

### 五、测试

- **使用浏览器访问 Nginx 的虚拟 IP 地址进行测试**。
- **检查 Keepalived 的状态是否正确切换**。

以上步骤提供了从环境搭建到配置的详细过程。在实际环境中还需要考虑网络配置、安全策略等因素。