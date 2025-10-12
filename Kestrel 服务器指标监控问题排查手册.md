# Kestrel 服务器指标监控问题排查手册

------

## **一、快速排查步骤**

### 1. 基础检查流程

mermaid

复制

```
graph TD
A[指标未显示] --> B{应用是否暴露指标?}
B -->|否| C[检查代码/Prometheus中间件]
B -->|是| D{Prometheus配置正确?}
D -->|否| E[修正prometheus.yml]
D -->|是| F{网络是否连通?}
F -->|否| G[检查容器网络/端口]
F -->|是| H[验证指标名称版本]
```

### 2. 5分钟快速检查清单

1. 访问 `/metrics` 端点是否返回数据
2. 检查 Prometheus Targets 状态是否为 **UP**
3. 验证容器间网络连通性
4. 确认使用正确的指标名称（新旧版本差异）
5. 查看应用日志是否有错误输出

------

## **二、详细问题排查指南**

### 1. 指标未暴露验证

#### 1.1 容器内直接检查

bash

复制

```
# 进入应用容器执行
docker exec -it <容器ID> curl -s http://localhost:80/metrics | grep -E 'kestrel_active_connections|microsoft_aspnetcore_server_kestrel_connections_active'

# 预期输出示例
# HELP microsoft_aspnetcore_server_kestrel_connections_active Current number of active connections
# TYPE microsoft_aspnetcore_server_kestrel_connections_active gauge
microsoft_aspnetcore_server_kestrel_connections_active 23
```

#### 1.2 代码配置验证

csharp

复制

```
// Program.cs 必要配置
var builder = WebApplication.CreateBuilder(args);

// 必须的中间件配置
var app = builder.Build();
app.UseMetricServer("/metrics");  // 暴露端点
app.UseHttpMetrics();             // 收集HTTP指标
```

#### 1.3 NuGet包验证

bash

复制

```
# 检查项目依赖
dotnet list package | grep -E 'prometheus-net|opentelemetry'

# 必要依赖
prometheus-net.AspNetCore >= 7.0.0
prometheus-net.SystemMetrics >= 7.0.0
```

### 2. Prometheus 配置验证

#### 2.1 基础配置模板

yaml

复制

```
# prometheus.yml 示例
scrape_configs:
  - job_name: 'aspnetcore'
    scrape_interval: 15s
    metrics_path: /metrics
    static_configs:
      - targets: ['宿主机真实IP:5000']  # 使用实际映射端口
        labels:
          env: 'production'
```

#### 2.2 容器网络配置要点

dockerfile

复制

```
# Dockerfile 关键配置
EXPOSE 80
ENV ASPNETCORE_URLS=http://+:80
```

#### 2.3 动态发现配置

yaml

复制

```
# 适用于Docker Swarm/K8s环境
scrape_configs:
  - job_name: 'docker-aspnetcore'
    docker_sd_configs:
      - host: unix:///var/run/docker.sock
    relabel_configs:
      - source_labels: [__meta_docker_container_label_prometheus_enable]
        regex: "true"
        action: keep
```

### 3. 网络连通性验证

#### 3.1 跨容器连通性测试

bash

复制

```
# 从Prometheus容器测试
docker exec -it prometheus-container curl http://aspnet-container:80/metrics

# 成功响应特征
HTTP/1.1 200 OK
Content-Type: text/plain; version=0.0.4
```

#### 3.2 端口映射检查

bash

复制

```
# 查看应用容器端口映射
docker port <容器ID>

# 正确输出示例
80/tcp -> 0.0.0.0:5000
```

#### 3.3 防火墙规则验证

bash

复制

```
# 检查宿主机防火墙
sudo ufw status
sudo iptables -L -n | grep ':5000'

# 临时开放端口（测试用）
sudo ufw allow 5000/tcp
```

### 4. 指标版本兼容处理

#### 4.1 新旧指标对照表

| .NET 版本 | 指标前缀                             | 示例                                                   |
| :-------- | :----------------------------------- | :----------------------------------------------------- |
| <6.0      | kestrel_                             | kestrel_active_connections                             |
| ≥6.0      | microsoft_aspnetcore_server_kestrel_ | microsoft_aspnetcore_server_kestrel_connections_active |

#### 4.2 兼容查询方法

promql

复制

```
# 同时查询新旧指标
{__name__=~"kestrel_active_connections|microsoft_aspnetcore_server_kestrel_connections_active"}
```

------

## **三、常见问题解决方案**

### 1. 指标端点返回404

**现象**：
`curl: (52) Empty reply from server` 或 `HTTP 404`

**解决方案**：

csharp

复制

```
// 检查中间件顺序
var app = builder.Build();
app.UseRouting();
app.UseMetricServer("/metrics");  // 必须在UseEndpoints之前
app.UseEndpoints(...);
```

### 2. Prometheus显示"Connection refused"

**现象**：
Target状态显示 **DOWN**

**解决方案**：

yaml

复制

```
# prometheus.yml 修正示例
static_configs:
  - targets: ['host.docker.internal:5000']  # Docker特殊域名
```

### 3. 指标数值始终为0

**验证方法**：

bash

复制

```
# 生成测试流量
docker run --rm alpine/bombardier -c 100 -d 60s http://aspnet-app:80
```

### 4. 数据抓取间隔过长

**优化配置**：

yaml

复制

```
scrape_configs:
  - job_name: 'aspnetcore'
    scrape_interval: 5s      # 缩短抓取间隔
    scrape_timeout: 3s
```

------

## **四、附录：配置参考**

### 1. 完整docker-compose.yml示例

yaml

复制

```
version: '3.8'

services:
  aspnet-app:
    image: your-app-image
    ports:
      - "5000:80"
    environment:
      - ASPNETCORE_ENVIRONMENT=Production
    networks:
      - monitor-net

  prometheus:
    image: prom/prometheus:v2.40.0
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
    networks:
      - monitor-net

networks:
  monitor-net:
    driver: bridge
```

### 2. Prometheus告警规则示例

yaml

复制

```
groups:
- name: kestrel-alerts
  rules:
  - alert: HighConnectionUsage
    expr: |
      microsoft_aspnetcore_server_kestrel_connections_active 
        / on(instance) 
        kestrel_limits_max_concurrent_connections > 0.8
    for: 5m
    labels:
      severity: critical
    annotations:
      summary: "High connection usage ({{ $value }}%)"
```

------

## **五、验证与维护**

### 1. 定期检查清单

| 检查项             | 频率 | 方法                            |
| :----------------- | :--- | :------------------------------ |
| 指标端点可用性     | 每日 | curl -I http://app:port/metrics |
| Prometheus存储增长 | 每周 | 检查Prometheus TSDB状态         |
| 告警规则有效性     | 每月 | 模拟触发条件验证                |
| 容器资源使用情况   | 实时 | Grafana仪表板监控               |

### 2. 维护建议

1. **版本升级**：保持ASP.NET Core和Prometheus为最新稳定版
2. **文档同步**：当架构变更时及时更新prometheus.yml
3. **压力测试**：每季度执行全链路负载测试
4. **备份策略**：定期备份Prometheus数据目录

------

**手册版本控制**

| 版本 | 更新日期   | 修改内容             |
| :--- | :--------- | :------------------- |
| 1.0  | 2023-08-15 | 初始版本             |
| 1.1  | 2023-09-01 | 新增动态发现配置示例 |