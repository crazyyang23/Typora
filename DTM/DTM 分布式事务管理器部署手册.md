# **DTM 分布式事务管理器部署手册**

**版本**：v1.0
**适用环境**：Linux (x86_64/arm64)
**最后更新**：2025-07-29

------

## **目录**

1. 系统要求
2. 二进制安装
3. 配置文件
4. 服务管理
5. 验证部署
6. 故障排查
7. 附录

------

## **1. 系统要求**

- **操作系统**：Linux (CentOS 7+/Ubuntu 18.04+)
- **依赖工具**：`systemd`、`wget`、`curl`
- **权限**：`root` 或 `sudo` 权限
- **资源**：
  - CPU：≥ 1 Core
  - 内存：≥ 512MB
  - 磁盘：≥ 100MB

------

## **2. 二进制安装**

### **2.1 下载二进制包**

bash



复制



下载

```
# 下载最新版本（替换版本号）
VERSION="1.18.0"
wget https://github.com/dtm-labs/dtm/releases/download/v${VERSION}/dtm-linux-amd64 -O /usr/local/bin/dtm

# 设置可执行权限
chmod +x /usr/local/bin/dtm
```

### **2.2 创建专用用户（可选）**

bash



复制



下载

```
useradd -r -s /bin/false dtm
chown dtm:dtm /usr/local/bin/dtm
```

------

## **3. 配置文件**

### **3.1 最小配置示例**

创建 `/etc/dtm/config.yaml`：

yaml

```
"Store": {
  "Driver": "redis",
  "Host": "10.2.255.15",  # 填入你的 Redis 服务器地址
  "Port": 6380,
  "PoolSize": 20,
  "MinIdleConns": 5,
  "MaxConnAge": "30m",
  "DialTimeout": "5s",
  "ReadTimeout": "3s",
  "WriteTimeout": "3s",
  "RedisPrefix": "{dtm}",
  "DataExpire": 86400,
  "FinishedDataExpire": 7200,
  # ... 其他配置保持不变 ...
}
```

下载

```
# 基础配置
MicroService:
  Driver: 'dtm-drivers-kratos'  # 微服务框架类型（如无则填 ''）
  Target: 'etcd://localhost:2379'  # 服务发现地址

# 存储配置（使用内置SQLite示例）
Store:
  Driver: 'sqlite'
  DBFile: '/var/lib/dtm/dtm.db'
```

### **3.2 数据库配置（MySQL）**

yaml



复制



下载

```
Store:
  Driver: 'mysql'
  Host: '127.0.0.1'
  Port: 3306
  User: 'dtm_user'
  Password: 'your_password'
  Database: 'dtm'
```

------

## **4. 服务管理**

### **4.1 创建 Systemd 服务**

编辑 `/etc/systemd/system/dtm.service`：

ini

```
[Unit]
Description=DTM Distributed Transaction Manager
After=network.target

[Service]
ExecStart=/usr/local/bin/dtm -c /root/dtm/conf.sample.yml

Restart=always
User=root
Group=root

[Install]
WantedBy=multi-user.target
```

复制



下载

```
[Unit]
Description=DTM Distributed Transaction Manager
After=network.target

[Service]
User=dtm  # 或 root
Group=dtm
ExecStart=/usr/local/bin/dtm -c /etc/dtm/config.yaml
Restart=on-failure
RestartSec=5s

[Install]
WantedBy=multi-user.target
```

### **4.2 启动与自启**

bash



复制



下载

```
# 重载配置
systemctl daemon-reload

# 启动服务
systemctl start dtm

# 设置开机自启
systemctl enable dtm
```

------

## **5. 验证部署**

### **5.1 检查服务状态**

bash



复制



下载

```
systemctl status dtm
```

**预期输出**：

text



复制



下载

```
Active: active (running)
```

### **5.2 测试 API 接口**

bash



复制



下载

```
curl http://localhost:36789/api/ping
```

**预期响应**：

json



复制



下载

```
"pong"
```

------

## **6. 故障排查**

### **6.1 常见错误**

| **错误现象**      | **解决方法**                                                |
| :---------------- | :---------------------------------------------------------- |
| `status=217/USER` | 检查 `/etc/systemd/system/dtm.service` 中的 `User` 是否存在 |
| `端口冲突`        | 修改 `config.yaml` 中的 `HTTPPort` 或 `GRPCPort`            |
| `数据库连接失败`  | 验证数据库账号权限及网络连通性                              |

### **6.2 日志查看**

bash



复制



下载

```
journalctl -u dtm --no-pager -n 50  # 查看最近50行日志
```

------

## **7. 附录**

### **7.1 卸载 DTM**

bash



复制



下载

```
systemctl stop dtm
rm /usr/local/bin/dtm
rm -rf /etc/dtm
```

### **7.2 资源链接**

- [官方文档](https://en.dtm.pub/)
- [GitHub Releases](https://github.com/dtm-labs/dtm/releases)

------

**修订记录**

| **日期**   | **版本** | **修改说明** |
| :--------- | :------- | :----------- |
| 2025-07-29 | v1.0     | 初始版本     |

------

**备注**：生产环境建议使用 MySQL/PostgreSQL 作为存储驱动，并配置高可用方案。