# DTM 分布式事务管理器部署与使用手册

## 1. 产品概述

DTM 是一款开源的分布式事务管理器，提供跨服务、跨数据库的分布式事务能力。主要特性包括：

- 支持 Saga、TCC、XA、二阶段消息等事务模式
- 与主流微服务框架无缝集成
- 提供友好的管理界面
- 高性能、高可用的架构设计

## 2. 系统架构

### 2.1 网络访问架构

text

复制下载

```
公网用户/内网用户
        ↓
[ admin.dtm.pub ] ← 统一认证网关
        ↓ (认证后重定向)
[ 10.2.255.67:36789 ] ← DTM管理后台
        ↓
[ 业务服务集群 ]
```



### 2.2 核心组件

- **DTM Server**: 事务协调器
- **管理界面**: 提供Web操作界面
- **存储后端**: Redis/MySQL/PostgreSQL
- **认证网关**: 统一访问入口

## 3. 安装部署

### 3.1 下载预编译二进制包

#### 下载地址

**GitHub Releases** (国际):

bash

复制下载

```
# Linux AMD64
wget https://github.com/dtm-labs/dtm/releases/latest/download/dtm-linux-amd64.tar.gz

# Linux ARM64
wget https://github.com/dtm-labs/dtm/releases/latest/download/dtm-linux-arm64.tar.gz

# macOS
wget https://github.com/dtm-labs/dtm/releases/latest/download/dtm-darwin-amd64.tar.gz
```



**Gitee 镜像** (国内推荐):

bash

复制下载

```
# Linux AMD64
wget https://gitee.com/dtm-labs/dtm/releases/latest/download/dtm-linux-amd64.tar.gz
```



#### 一键安装脚本

bash

复制下载

```
# 官方脚本
curl -sSL https://github.com/dtm-labs/dtm/releases/latest/download/install.sh | bash

# 国内镜像
curl -sSL https://gitee.com/dtm-labs/dtm/releases/latest/download/install.sh | bash
```



### 3.2 安装步骤

bash

复制下载

```
# 解压安装包
tar -xzf dtm-linux-amd64.tar.gz

# 安装到系统路径
sudo mv dtm /usr/local/bin/
sudo chmod +x /usr/local/bin/dtm

# 验证安装
dtm version
```



### 3.3 Docker 部署

bash

复制下载

```
# 快速启动
docker run -d -p 36789:36789 \
  --name dtm \
  yedf/dtm

# 使用自定义配置
docker run -d -p 36789:36789 \
  -v /path/to/config.yml:/app/dtm/configs/config.yml \
  yedf/dtm
```



## 4. 配置说明

### 4.1 基础配置文件

创建 `config.yml`:

yaml

复制下载

```
# 微服务配置
MicroService:
  Driver: 'dtm-driver-gozero'  # 支持: dtm-driver-gozero, nacos, etcd, consul
  Target: 'etcd://localhost:2379/dtmservice'
  EndPoint: 'localhost:36789'

# 存储配置
Store:
  Driver: 'redis'  # 支持: redis, mysql, postgres, boltdb
  Host: 'localhost'
  Port: 6379
  User: ''
  Password: ''
  
# 数据库配置 (如使用MySQL/PostgreSQL)
# Store:
#   Driver: 'mysql'
#   Host: 'localhost'
#   Port: 3306
#   User: 'dtm'
#   Password: 'your_password'
#   Database: 'dtm'

# HTTP服务配置
HTTPServer:
  Port: 36789
  Address: '0.0.0.0'

# 日志配置
Log:
  Level: 'info'
  Outputs: ['stderr', 'dtm.log']
```



### 4.2 生产环境配置建议

yaml

复制下载

```
Store:
  Driver: 'mysql'
  Host: 'mysql-cluster.internal.com'
  Port: 3306
  User: 'dtm_prod'
  Password: '${DB_PASSWORD}'  # 建议使用环境变量
  Database: 'dtm_production'
  MaxOpenConns: 50
  MaxIdleConns: 10

HTTPServer:
  Port: 36789
  Address: '0.0.0.0'
  ReadTimeout: '30s'
  WriteTimeout: '30s'

Log:
  Level: 'warn'
  Outputs: ['/var/log/dtm/dtm.log']
```



## 5. 启动服务

### 5.1 直接启动

bash

复制下载

```
# 开发环境
dtm -c config.yml

# 指定端口启动
dtm -p 36789

# 后台运行
nohup dtm -c config.yml > dtm.log 2>&1 &
```



### 5.2 Systemd 服务配置

创建 `/etc/systemd/system/dtm.service`:

ini

复制下载

```
[Unit]
Description=DTM Distributed Transaction Manager
After=network.target redis.service

[Service]
Type=simple
User=dtm
Group=dtm
WorkingDirectory=/opt/dtm
ExecStart=/usr/local/bin/dtm -c /etc/dtm/config.yml
Restart=always
RestartSec=5
Environment=DB_PASSWORD=your_secure_password

[Install]
WantedBy=multi-user.target
```



启用服务:

bash

复制下载

```
sudo systemctl daemon-reload
sudo systemctl enable dtm
sudo systemctl start dtm
sudo systemctl status dtm
```



## 6. 访问管理界面

### 6.1 访问方式

- **内网直接访问**: `http://10.2.255.67:36789`
- **公网统一入口**: `https://admin.dtm.pub` (重定向至内网地址)

### 6.2 访问流程说明

1. 用户访问统一入口 `admin.dtm.pub`
2. 认证网关进行身份验证
3. 验证通过后重定向至内网地址 `10.2.255.67:36789`
4. 用户直接与DTM管理后台交互

### 6.3 安全说明

- 内网地址不直接暴露给公网
- 统一入口提供集中认证和访问控制
- 建议配置防火墙规则，限制内网地址的访问来源

## 7. 运维管理

### 7.1 服务监控

#### 健康检查

bash

复制下载

```
# 检查服务状态
curl http://localhost:36789/api/ping

# 检查版本信息
curl http://localhost:36789/api/version
```



#### 日志查看

bash

复制下载

```
# 查看实时日志
tail -f /var/log/dtm/dtm.log

# 查看systemd服务日志
journalctl -u dtm -f
```



### 7.2 数据备份

#### Redis 数据备份

bash

复制下载

```
# 如果使用Redis作为存储
redis-cli SAVE  # 立即创建RDB快照
# 备份RDB文件
cp /var/lib/redis/dump.rdb /backup/dtm-redis-$(date +%Y%m%d).rdb
```



#### MySQL 数据备份

bash

复制下载

```
# 如果使用MySQL作为存储
mysqldump -u dtm -p dtm > /backup/dtm-mysql-$(date +%Y%m%d).sql
```



### 7.3 性能监控

关键监控指标:

- 事务处理吞吐量
- 事务成功率/失败率
- 存储后端连接数
- 系统资源使用率

## 8. 故障排查

### 8.1 常见问题

#### 服务无法启动

bash

复制下载

```
# 检查端口占用
netstat -tlnp | grep 36789

# 检查配置文件语法
dtm -c config.yml check

# 查看详细错误日志
dtm -c config.yml --log-level debug
```



#### 无法访问管理界面

bash

复制下载

```
# 检查服务状态
systemctl status dtm

# 检查防火墙规则
iptables -L -n | grep 36789

# 检查网络连通性
telnet 10.2.255.67 36789
```



#### 存储连接问题

bash

复制下载

```
# 测试Redis连接
redis-cli -h localhost -p 6379 ping

# 测试MySQL连接
mysql -h localhost -P 3306 -u dtm -p -e "SELECT 1;"
```



### 8.2 日志分析

常见错误日志及解决方案:

log

复制下载

```
# 存储连接失败
ERROR: dial tcp 127.0.0.1:6379: connect: connection refused
# 解决方案: 检查Redis服务状态

# 端口被占用
ERROR: listen tcp :36789: bind: address already in use  
# 解决方案: 更换端口或停止占用进程

# 配置错误
ERROR: parse config file error: yaml: unmarshal errors
# 解决方案: 检查YAML文件格式和语法
```



## 9. 版本升级

### 9.1 平滑升级步骤

1. 备份当前配置和数据
2. 停止当前服务
3. 下载新版本二进制文件
4. 替换二进制文件
5. 启动新版本服务
6. 验证服务功能

### 9.2 回滚方案

1. 停止新版本服务
2. 恢复旧版本二进制文件
3. 恢复原有配置
4. 启动旧版本服务

## 10. 安全建议

1. **网络隔离**: DTM服务应部署在内网，通过网关对外提供访问
2. **认证授权**: 配置统一的认证机制
3. **数据加密**: 敏感配置信息使用环境变量或密钥管理服务
4. **访问审计**: 开启访问日志，定期审计
5. **定期更新**: 及时更新到最新安全版本

## 附录

### A. 常用命令速查

bash

复制下载

```
# 启动服务
dtm -c config.yml

# 查看版本
dtm version

# 检查配置
dtm -c config.yml check

# 查看帮助
dtm --help
```



### B. 配置文件模板

参考 `config.example.yml` 文件创建自定义配置。

### C. 技术支持

- 官方文档: [https://dtm.pub](https://dtm.pub/)
- GitHub: https://github.com/dtm-labs/dtm
- 问题反馈: GitHub Issues

------

*本手册对应 DTM 版本: v2.0+*
*最后更新: 2024年*