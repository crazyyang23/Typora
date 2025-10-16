### **基于 `nginx-vts-exporter` 和 `vhost_traffic_status` 模块的进阶监控配置**

------

#### **一、模块概述**

1. **`vhost_traffic_status` 模块**
   - **描述**：Nginx 第三方模块，提供细粒度的流量统计，支持按 **虚拟主机（server）、位置块（location）、上游服务器（upstream）** 分类监控。
   - **核心功能**：
     - 统计请求数、响应状态码（2xx/3xx/4xx/5xx）。
     - 记录请求流量（进/出字节数）。
     - 跟踪上游服务器响应时间和请求成功率。
   - **适用场景**：多域名、多应用服务器环境，需分服务监控流量及性能。
2. **`nginx-vts-exporter`**
   - **描述**：Prometheus Exporter，将 `vhost_traffic_status` 模块的数据转换为 Prometheus 格式。
   - **特点**：
     - 支持多维度指标（如按虚拟主机、状态码过滤）。
     - 提供历史数据聚合功能（需配置时间窗口）。

------

#### **二、模块安装与配置**

------

##### **1. 编译安装 `vhost_traffic_status` 模块**

###### **1.1 下载模块源码**

bash

复制

```
# 进入Nginx源码目录（假设已安装Nginx）
cd /usr/src/
git clone https://github.com/vozlt/nginx-module-vts.git
```

###### **1.2 重新编译Nginx**

查看当前Nginx版本及编译参数：

bash

复制

```
nginx -V
```

输出示例：

复制

```
nginx version: nginx/1.18.0
built with OpenSSL 1.1.1k  25 Mar 2021
configure arguments: --prefix=/etc/nginx --sbin-path=/usr/sbin/nginx ...
```

重新配置并添加 `--add-module` 参数：

bash

复制

```
# 进入Nginx源码目录（需与当前运行版本一致）
cd /usr/src/nginx-1.18.0/
./configure \
    <原有参数> \
    --add-module=/usr/src/nginx-module-vts
make
```

###### **1.3 替换Nginx二进制文件**

bash

复制

```
# 备份旧版本
cp /usr/sbin/nginx /usr/sbin/nginx.bak
# 替换新编译的二进制文件
cp objs/nginx /usr/sbin/nginx
# 测试配置并重启
nginx -t && systemctl restart nginx
```

###### **1.4 配置Nginx启用模块**

在 `nginx.conf` 的 `http` 块中添加：

nginx

复制

```
http {
    vhost_traffic_status_zone shared:vhost_traffic_status:10m;  # 分配10MB共享内存

    server {
        listen 80;
        server_name status.example.com;

        location /vhost_status {
            vhost_traffic_status_display;          # 启用状态页
            vhost_traffic_status_display_format html;  # 输出为HTML格式
            access_log off;
            allow 192.168.1.0/24;   # 限制访问IP
            deny all;
        }
    }
}
```

###### **1.5 验证状态页**

访问 `http://status.example.com/vhost_status`，应看到包含虚拟主机、上游服务器等统计的HTML页面。

------

##### **2. 部署 `nginx-vts-exporter`**

###### **2.1 下载并安装Exporter**

bash

复制

```
wget https://github.com/hnlq715/nginx-vts-exporter/releases/download/v0.10.3/nginx-vts-exporter-0.10.3.linux-amd64.tar.gz
tar -xzf nginx-vts-exporter-*.tar.gz
sudo mv nginx-vts-exporter /usr/local/bin/
```

###### **2.2 配置Systemd服务**

创建文件 `/etc/systemd/system/nginx-vts-exporter.service`：

ini

复制

```
[Unit]
Description=Nginx VTS Exporter
After=network.target

[Service]
User=nginx
ExecStart=/usr/local/bin/nginx-vts-exporter \
    -nginx.scrape_uri http://status.example.com/vhost_status/format/json  # 指向JSON格式状态页
Restart=always

[Install]
WantedBy=multi-user.target
```

###### **2.3 启动Exporter**

bash

复制

```
sudo systemctl daemon-reload
sudo systemctl start nginx-vts-exporter
sudo systemctl enable nginx-vts-exporter
```

###### **2.4 验证Exporter指标**

访问 `http://<exporter-ip>:9913/metrics`，应看到包含 `nginx_vts_*` 的指标。

------

#### **三、Prometheus 配置**

修改 `prometheus.yml`，添加Job：

yaml

复制

```
scrape_configs:
  - job_name: 'nginx-vts'
    static_configs:
      - targets: ['<exporter-ip>:9913']
    metrics_path: /metrics
```

重启Prometheus：

bash

复制

```
systemctl restart prometheus
```

------

#### **四、Grafana 仪表盘导入**

1. **数据源**：确保已添加Prometheus数据源。
2. **仪表盘ID**：使用官方模板 [2949](https://grafana.com/grafana/dashboards/2949) 或 [13457](https://grafana.com/grafana/dashboards/13457)。
3. **关键面板**：
   - 按虚拟主机的请求速率、流量分布。
   - 上游服务器响应时间百分位数（P95/P99）。
   - 状态码（4xx/5xx）错误率。

------

#### **五、关键参数详解**

##### **1. `vhost_traffic_status` 模块参数**

| 参数                                  | 描述                                                         |
| :------------------------------------ | :----------------------------------------------------------- |
| `vhost_traffic_status_zone`           | 定义共享内存区域名称及大小，格式：`name:size`（如 `shared:name:10m`）。 |
| `vhost_traffic_status_display`        | 启用状态页显示。                                             |
| `vhost_traffic_status_display_format` | 输出格式：`html`（默认）、`json`、`jsonp`、`prometheus`。    |
| `vhost_traffic_status_filter`         | 按域名或上游组过滤统计，如 `vhost_traffic_status_filter_by_host on;`。 |

##### **2. `nginx-vts-exporter` 启动参数**

| 参数                | 描述                                                         |
| :------------------ | :----------------------------------------------------------- |
| `-nginx.scrape_uri` | 指定 `vhost_traffic_status` 的JSON状态页URL（必须为`/format/json`）。 |
| `-telemetry.addr`   | Exporter监听地址（默认 `:9913`）。                           |
| `-interval`         | 数据抓取间隔（默认 `5s`）。                                  |

------

#### **六、指标详解（示例）**

prometheus

复制

```
# HELP nginx_vts_server_requests_total Total requests processed by server zone
# TYPE nginx_vts_server_requests_total counter
nginx_vts_server_requests_total{zone="example.com"} 1500

# HELP nginx_vts_server_request_seconds_total Request processing time in seconds
# TYPE nginx_vts_server_request_seconds_total counter
nginx_vts_server_request_seconds_total{zone="example.com"} 123.45

# HELP nginx_vts_upstream_response_seconds Upstream response time histogram
# TYPE nginx_vts_upstream_response_seconds histogram
nginx_vts_upstream_response_seconds_bucket{backend="app-server",le="1"} 100
```

------

#### **七、故障排查**

1. **模块未加载**
   - **现象**：Nginx启动报错 `unknown directive "vhost_traffic_status_zone"`。
   - **解决**：确认编译时添加了 `--add-module` 参数，并重启Nginx。
2. **Exporter无数据**
   - **检查点**：
     - 访问 `http://status.example.com/vhost_status/format/json` 是否返回JSON数据。
     - Exporter日志：`journalctl -u nginx-vts-exporter -f`。
3. **共享内存不足**
   - **现象**：Nginx日志报错 `could not build the variables_hash, you should increase vhost_traffic_status_zone_size`。
   - **解决**：增大 `vhost_traffic_status_zone` 的size值（如 `20m`）。

------

#### **八、安全建议**

1. **限制状态页访问**：

   nginx

   复制

   ```
   location /vhost_status {
       allow 监控服务器IP;
       deny all;
   }
   ```

2. **启用HTTPS**：为状态页配置SSL加密。

3. **定期清理数据**：调整 `vhost_traffic_status_zone` 的size，避免内存占用过高。

------

通过以上配置，您将获得比 `stub_status` 更精细的监控能力，尤其适用于多服务、多域名的复杂Nginx环境。