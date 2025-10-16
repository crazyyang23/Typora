# Nginx高可用集群部署手册（主备架构+后端代理）

## 一、架构说明



```
客户端 → VIP(192.168.1.100) → [主Nginx(192.168.1.101) / 备Nginx(192.168.1.102)] → 后端应用服务器(192.168.1.105-107)
```

## 二、环境准备

### 1. 服务器规划

| 角色        | IP地址        | 主机名       |
| :---------- | :------------ | :----------- |
| Nginx主节点 | 192.168.1.101 | nginx-master |
| Nginx备节点 | 192.168.1.102 | nginx-backup |
| 虚拟IP(VIP) | 192.168.1.100 | -            |
| 应用服务器1 | 192.168.1.105 | app-server1  |
| 应用服务器2 | 192.168.1.106 | app-server2  |
| 应用服务器3 | 192.168.1.107 | app-server3  |

### 2. 系统要求

- 所有节点时间同步（执行 `ntpdate ntp.aliyun.com`）
- 关闭防火墙或开放端口（80、443、VRRP协议）
- 所有节点SSH互信配置（可选）

## 三、Nginx安装与配置（主备节点执行）

### 1. 安装Nginx

bash



复制



下载

```
# CentOS
yum install -y epel-release
yum install -y nginx

# Ubuntu
apt-get update
apt-get install -y nginx
```

### 2. 配置Nginx反向代理

编辑 `/etc/nginx/nginx.conf`：

nginx



复制



下载

```
http {
    upstream app_servers {
        server 192.168.1.105:80 weight=3;
        server 192.168.1.106:80 weight=2;
        server 192.168.1.107:80 weight=1;
        
        # 健康检查参数
        max_fails=3 fail_timeout=30s;
    }

    server {
        listen 80;
        server_name _;

        location / {
            proxy_pass http://app_servers;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            
            # 重要超时参数
            proxy_connect_timeout 5s;
            proxy_read_timeout 60s;
            proxy_send_timeout 30s;
        }

        # 状态监控页面（可选）
        location /nginx_status {
            stub_status on;
            access_log off;
            allow 192.168.1.0/24;
            deny all;
        }
    }
}
```

### 3. 验证配置并启动

bash



复制



下载

```
nginx -t  # 测试配置
systemctl start nginx
systemctl enable nginx
```

## 四、Keepalived安装配置

### 1. 安装Keepalived（主备节点执行）

bash



复制



下载

```
# CentOS
yum install -y keepalived

# Ubuntu
apt-get install -y keepalived
```

### 2. 主节点配置（192.168.1.101）

创建 `/etc/keepalived/keepalived.conf`：

conf



复制



下载

```
! Configuration File for keepalived

global_defs {
    router_id nginx_master
    script_user root
    enable_script_security
}

vrrp_script chk_nginx {
    script "/etc/keepalived/check_nginx.sh"
    interval 2
    weight -50
    fall 2
    rise 1
}

vrrp_instance VI_1 {
    state MASTER
    interface ens33  # 修改为实际网卡名
    virtual_router_id 51
    priority 100
    advert_int 1
    
    authentication {
        auth_type PASS
        auth_pass d0cker@123  # 建议修改为复杂密码
    }
    
    virtual_ipaddress {
        192.168.1.100/24 dev ens33 label ens33:vip
    }
    
    track_script {
        chk_nginx
    }
    
    # 当keepalived停止时释放VIP
    notify_stop "/etc/keepalived/release_vip.sh"
}
```

### 3. 备节点配置（192.168.1.102）

conf



复制



下载

```
! Configuration File for keepalived

global_defs {
    router_id nginx_backup
    script_user root
    enable_script_security
}

vrrp_script chk_nginx {
    script "/etc/keepalived/check_nginx.sh"
    interval 2
    weight -50
}

vrrp_instance VI_1 {
    state BACKUP
    interface ens33
    virtual_router_id 51
    priority 90
    advert_int 1
    
    authentication {
        auth_type PASS
        auth_pass d0cker@123  # 与主节点保持一致
    }
    
    virtual_ipaddress {
        192.168.1.100/24 dev ens33 label ens33:vip
    }
    
    track_script {
        chk_nginx
    }
}
```

### 4. 创建健康检查脚本（主备节点）

创建 `/etc/keepalived/check_nginx.sh`：

bash



复制



下载

```
#!/bin/bash
count=$(ps -ef | grep nginx | grep -v "grep" | wc -l)
if [ $count -lt 2 ]; then  # 根据实际进程数调整
    systemctl stop keepalived
    exit 1
else
    exit 0
fi
```

创建VIP释放脚本 `/etc/keepalived/release_vip.sh`：

bash



复制



下载

```
#!/bin/bash
ip addr del 192.168.1.100/24 dev ens33
```

设置权限：

bash



复制



下载

```
chmod +x /etc/keepalived/*.sh
chown -R root:root /etc/keepalived
```

### 5. 启动服务

bash



复制



下载

```
systemctl start keepalived
systemctl enable keepalived
```

## 五、后端应用服务器配置（示例）

在192.168.1.105-107上部署应用后，确保：

1. 应用服务监听80端口

2. 配置不同的HTTP响应头用于测试：

   bash

   

   复制

   

   下载

   ```
   # 在105上执行
   echo "Server: AppServer1" > /var/www/html/index.html
   
   # 在106上执行
   echo "Server: AppServer2" > /var/www/html/index.html
   
   # 在107上执行
   echo "Server: AppServer3" > /var/www/html/index.html
   ```

## 六、验证与测试

### 1. 基础验证

bash



复制



下载

```
# 查看VIP绑定
ip addr show ens33 | grep 192.168.1.100

# 查看Nginx状态
systemctl status nginx

# 查看Keepalived状态
systemctl status keepalived
```

### 2. 故障转移测试

bash



复制



下载

```
# 在主节点执行
systemctl stop nginx
# 观察约5秒后VIP应自动切换到备节点

# 恢复测试
systemctl start nginx
# VIP应自动切回主节点
```

### 3. 负载均衡测试

bash



复制



下载

```
# 连续访问测试
for i in {1..10}; do curl http://192.168.1.100; done
# 应看到不同后端服务器的响应
```

## 七、监控与维护

### 1. 日志查看

bash



复制



下载

```
# Nginx访问日志
tail -f /var/log/nginx/access.log

# Keepalived日志
journalctl -u keepalived -f
```

### 2. 日常维护命令

bash



复制



下载

```
# 手动切换VIP（在主节点执行）
systemctl restart keepalived

# 检查Nginx与后端连接
netstat -antp | grep nginx
```

## 八、注意事项

1. 确保所有节点时间同步（chrony或ntp）
2. 生产环境建议：
   - 使用HTTPS加密传输
   - 配置日志轮转
   - 设置防火墙白名单
3. Keepalived的virtual_router_id在同一个网络内必须唯一
4. 建议配置Zabbix/Prometheus监控服务状态

------

以上配置实现了：

- Nginx主备高可用（通过Keepalived VIP）
- 后端应用服务器的负载均衡
- 自动故障检测与转移
- 基本的健康检查机制

实际部署时请根据网络环境调整网卡名称、IP地址等参数。