# Redis Exporter 详细操作手册

## 目录

1. 概述
2. 环境准备
3. 安装部署
   - 3.1 二进制方式安装
   - 3.2 Docker方式安装
   - 3.3 系统服务配置
4. 配置详解
   - 4.1 基础配置
   - 4.2 多实例监控
   - 4.3 安全配置
5. 集成Prometheus
6. Grafana可视化
7. 高级功能
8. 维护与排错
9. 附录

## 1. 概述

Redis Exporter是将Redis监控指标导出为Prometheus格式的开源工具，主要用于Redis数据库的性能监控和告警。

## 2. 环境准备

### 2.1 硬件要求

- 最低配置：1核CPU，512MB内存
- 推荐配置：2核CPU，1GB内存

### 2.2 软件依赖

- Redis服务器（2.8+版本）
- Prometheus服务（可选）
- Grafana服务（可选）

### 2.3 网络要求

- Exporter主机与Redis服务器网络连通
- 开放Exporter监听端口（默认9121）

## 3. 安装部署

### 3.1 二进制方式安装

#### 3.1.1 下载安装包

bash

 [redis_exporter-v1.50.0.linux-amd64.tar.gz](..\..\..\GPT浏览器下载\redis_exporter-v1.50.0.linux-amd64.tar.gz) 

```
# 创建安装目录
sudo mkdir -p /opt/redis_exporter
cd /opt/redis_exporter

# 下载最新版本（请替换为最新版本号）
VERSION="v1.50.0"
wget https://github.com/oliver006/redis_exporter/releases/download/${VERSION}/redis_exporter-${VERSION}.linux-amd64.tar.gz

# 解压安装包
tar xvfz redis_exporter-${VERSION}.linux-amd64.tar.gz

# 创建软链接
sudo ln -s /opt/redis_exporter/redis_exporter-${VERSION}.linux-amd64/redis_exporter /usr/local/bin/redis_exporter
```

#### 3.1.2 验证安装

bash

复制

```
redis_exporter --version
```

### 3.2 Docker方式安装

#### 3.2.1 基础运行

bash

复制

```
docker run -d \
  --name redis_exporter \
  -p 9121:9121 \
  oliver006/redis_exporter
```

#### 3.2.2 带认证的运行方式

bash

复制

```
docker run -d \
  --name redis_exporter \
  -p 9121:9121 \
  -e REDIS_ADDR="redis://redis-host:6379" \
  -e REDIS_PASSWORD="yourpassword" \
  oliver006/redis_exporter
```

### 3.3 系统服务配置

#### 3.3.1 创建系统用户

bash

复制

```
sudo useradd --no-create-home --shell /bin/false redis_exporter
```

#### 3.3.2 创建Systemd服务文件

创建`/etc/systemd/system/redis_exporter.service`文件：

ini

复制

```
[Unit]
Description=Redis Exporter
Documentation=https://github.com/oliver006/redis_exporter
After=network.target

[Service]
User=redis_exporter
Group=redis_exporter
Type=simple
ExecStart=/usr/local/bin/redis_exporter \
  -redis.addr redis://localhost:6379 \
  -redis.password "your_redis_password" \
  -web.listen-address ":9121" \
  -log-format json
Restart=always
RestartSec=5
StartLimitInterval=0

[Install]
WantedBy=multi-user.target
```

#### 3.3.3 启动服务

bash

复制

```
sudo systemctl daemon-reload
sudo systemctl enable redis_exporter
sudo systemctl start redis_exporter
```

#### 3.3.4 验证服务状态

bash

复制

```
sudo systemctl status redis_exporter
journalctl -u redis_exporter -f
```

## 4. 配置详解

### 4.1 基础配置

#### 4.1.1 常用参数说明

| 参数                  | 默认值                 | 说明                      |
| :-------------------- | :--------------------- | :------------------------ |
| `-redis.addr`         | redis://localhost:6379 | Redis服务器地址           |
| `-redis.password`     | 无                     | Redis认证密码             |
| `-web.listen-address` | :9121                  | Exporter监听地址          |
| `-namespace`          | 无                     | 指标命名空间前缀          |
| `-check-keys`         | 无                     | 监控特定键（支持通配符）  |
| `-script`             | 无                     | 加载Lua脚本收集自定义指标 |
| `-log-format`         | txt                    | 日志格式（json/txt）      |
| `-debug`              | false                  | 启用调试模式              |

#### 4.1.2 配置文件示例

创建配置文件`/etc/redis_exporter.conf`：

ini

复制

```
redis_addr = "redis://localhost:6379"
redis_password = "yourpassword"
web_listen_address = ":9121"
check_keys = "user*,session*"
log_format = "json"
```

使用配置文件启动：

bash

复制

```
redis_exporter -config.file /etc/redis_exporter.conf
```

### 4.2 多实例监控

#### 4.2.1 文件服务发现方式

1. 创建目标文件`/etc/redis_targets.json`：

json

复制

```
[
  {
    "targets": ["redis1:6379"],
    "labels": {
      "env": "production",
      "role": "master"
    }
  },
  {
    "targets": ["redis2:6379", "redis3:6379"],
    "labels": {
      "env": "production",
      "role": "replica"
    }
  }
]
```

1. 启动Exporter：

bash

复制

```
redis_exporter -redis.targets-file /etc/redis_targets.json
```

#### 4.2.2 多Exporter实例方式

bash

复制

```
# 实例1
redis_exporter -redis.addr redis1:6379 -web.listen-address :9121

# 实例2
redis_exporter -redis.addr redis2:6379 -web.listen-address :9122
```

### 4.3 安全配置

#### 4.3.1 TLS加密配置

1. 生成证书（测试用）：

bash

复制

```
openssl req -new -newkey rsa:2048 -days 365 -nodes -x509 \
  -keyout redis_exporter.key -out redis_exporter.crt \
  -subj "/CN=redis-exporter.example.com"
```

1. 创建web配置文件`/etc/redis_exporter_web.yml`：

yaml

复制

```
tls_server_config:
  cert_file: /etc/redis_exporter.crt
  key_file: /etc/redis_exporter.key
```

1. 启动Exporter：

bash

复制

```
redis_exporter -web.config.file=/etc/redis_exporter_web.yml
```

#### 4.3.2 基本认证配置

1. 生成密码哈希：

bash

复制

```
echo "mypassword" | htpasswd -nBC 10 "" | tr -d ':\n'
```

1. 更新web配置文件：

yaml

复制

```
basic_auth_users:
  admin: "$2y$10$hashedpassword"
```

## 5. 集成Prometheus

### 5.1 Prometheus配置示例

编辑`prometheus.yml`：

yaml

复制

```
scrape_configs:
  - job_name: 'redis'
    scrape_interval: 15s
    static_configs:
      - targets: ['redis-exporter:9121']
    relabel_configs:
      - source_labels: [__address__]
        target_label: instance
        regex: (.*):.*
        replacement: '$1'
```

### 5.2 文件服务发现配置

yaml

复制

```
scrape_configs:
  - job_name: 'redis'
    file_sd_configs:
      - files:
        - '/etc/prometheus/redis_targets.json'
        refresh_interval: 5m
```

## 6. Grafana可视化

### 6.1 导入官方仪表板

1. 登录Grafana
2. 点击"+" > "Import"
3. 输入仪表板ID `11835`
4. 选择对应的Prometheus数据源

### 6.2 自定义仪表板关键指标

| 指标名称                         | 说明          |
| :------------------------------- | :------------ |
| `redis_up`                       | Redis实例状态 |
| `redis_connected_clients`        | 客户端连接数  |
| `redis_memory_used_bytes`        | 内存使用量    |
| `redis_commands_processed_total` | 命令处理总数  |
| `redis_keys`                     | 数据库键数量  |

## 7. 高级功能

### 7.1 自定义Lua脚本监控

1. 创建脚本`custom_metrics.lua`：

lua

复制

```
local slow_log = redis.call('SLOWLOG', 'GET', 5)
local long_running = 0

for i, log in ipairs(slow_log) do
  if log[3] > 1000 then  -- 执行时间超过1ms
    long_running = long_running + 1
  end
end

return {long_running}
```

1. 启动Exporter：

bash

复制

```
redis_exporter -script=custom_metrics.lua -script-values=slow_queries
```

### 7.2 键空间监控

bash

复制

```
# 监控所有以"cache:"开头的键
redis_exporter -check-keys="cache:*"

# 监控多个键模式
redis_exporter -check-keys="user:*,session:*,cache:*"
```

## 8. 维护与排错

### 8.1 日常维护

1. 日志检查：

   bash

   复制

   ```
   journalctl -u redis_exporter --since "1 hour ago"
   ```

2. 指标端点检查：

   bash

   复制

   ```
   curl http://localhost:9121/metrics
   ```

### 8.2 常见问题排查

#### 问题1：无法连接Redis

**现象**：

复制

```
error connecting to redis instance: dial tcp 127.0.0.1:6379: connect: connection refused
```

**解决方案**：

1. 确认Redis服务是否运行
2. 检查网络连接
3. 验证认证信息

#### 问题2：指标缺失

**现象**：部分指标未显示

**解决方案**：

1. 检查Redis版本是否支持该指标
2. 增加`-debug`参数查看详细日志
3. 确认是否有足够的权限

## 9. 附录

### 9.1 常用指标说明

| 指标                           | 类型    | 说明               |
| :----------------------------- | :------ | :----------------- |
| redis_up                       | Gauge   | 服务可用性         |
| redis_connected_clients        | Gauge   | 客户端连接数       |
| redis_memory_used_bytes        | Gauge   | 内存使用量         |
| redis_commands_processed_total | Counter | 处理的命令总数     |
| redis_db_keys                  | Gauge   | 每个数据库的键数量 |

### 9.2 参考资源

- 官方GitHub仓库：https://github.com/oliver006/redis_exporter
- Prometheus文档：https://prometheus.io/docs
- Redis监控最佳实践：https://redis.io/topics/monitoring

### 9.3 版本更新记录

| 版本 | 日期       | 更新说明           |
| :--- | :--------- | :----------------- |
| v1.0 | 2023-10-01 | 初始版本           |
| v1.1 | 2023-10-15 | 增加多实例监控说明 |