### **Nginx监控操作手册（基于Prometheus + Grafana）**

------

### **目录**

1. **环境准备**
2. **启用Nginx状态监控**
3. **部署Nginx Prometheus Exporter**
4. **配置Prometheus抓取数据**
5. **配置Grafana仪表盘**
6. **关键指标说明**
7. **故障排查**
8. **附录：安全建议**

------

### **1. 环境准备**

#### **1.1 组件要求**

- **Nginx**：已安装并运行，支持`stub_status`模块。
- **Prometheus**：已部署，默认端口`9090`。
- **Grafana**：已部署，默认端口`3000`。
- **服务器权限**：需具备`sudo`权限操作Nginx及服务管理。

#### **1.2 网络要求**

- Prometheus可访问Nginx Exporter的`9113`端口。
- Grafana可访问Prometheus的`9090`端口。
- Exporter可访问Nginx的`80/443`端口（用于抓取状态页）。

------

### **2. 启用Nginx状态监控**

#### **2.1 确认`stub_status`模块已启用**

bash

复制

```
nginx -V 2>&1 | grep -o with-http_stub_status_module
```

- **输出结果**：若显示`with-http_stub_status_module`，则已启用；否则需重新编译Nginx。

#### **2.2 配置Nginx状态页**

编辑Nginx配置文件（如`/etc/nginx/conf.d/status.conf`）：

nginx

复制

```
server {
    listen 80;
    server_name localhost;

    location /nginx_status {
        stub_status on;          # 启用状态页
        access_log off;         # 关闭日志
        allow 127.0.0.1;         # 仅允许本地访问
        deny all;                # 拒绝其他IP
    }
}
```

#### **2.3 重启Nginx并验证**

bash

复制

```
sudo systemctl restart nginx
curl http://localhost/nginx_status
```

- **预期输出**：

  复制

  ```
  Active connections: 3 
  server accepts handled requests
   100 100 200 
  Reading: 0 Writing: 1 Waiting: 2 
  ```

------

### **3. 部署Nginx Prometheus Exporter**

#### **3.1 下载并安装Exporter**

bash

复制

```
# 下载二进制文件（示例版本：0.11.0）
wget https://github.com/nginxinc/nginx-prometheus-exporter/releases/download/v0.11.0/nginx-prometheus-exporter_0.11.0_linux_amd64.tar.gz
tar -xzf nginx-prometheus-exporter_*.tar.gz
sudo mv nginx-prometheus-exporter /usr/local/bin/
```

#### **3.2 配置Systemd服务**

创建文件`/etc/systemd/system/nginx-exporter.service`：

ini

复制

```
[Unit]
Description=Nginx Prometheus Exporter
After=network.target

[Service]
User=nginx
ExecStart=/usr/local/bin/nginx-prometheus-exporter \
    -nginx.scrape-uri http://localhost/nginx_status  # Nginx状态页地址
Restart=always

[Install]
WantedBy=multi-user.target
```

#### **3.3 启动Exporter**

bash

复制

```
sudo systemctl daemon-reload
sudo systemctl start nginx-exporter
sudo systemctl enable nginx-exporter
```

#### **3.4 验证Exporter**

bash

复制

```
curl http://localhost:9113/metrics
```

- **预期输出**：Prometheus格式的指标（如`nginx_connections_active`）。

------

### **4. 配置Prometheus抓取数据**

#### **4.1 修改Prometheus配置**

编辑`prometheus.yml`，添加Job：

yaml

复制

```
scrape_configs:
  - job_name: 'nginx'
    static_configs:
      - targets: ['<exporter-ip>:9113']  # 替换为Exporter实际IP
    metrics_path: /metrics
```

#### **4.2 重启Prometheus**

bash

复制

```
sudo systemctl restart prometheus
```

#### **4.3 验证Target状态**

1. 访问Prometheus Web界面：`http://<prometheus-ip>:9090/targets`
2. 确认`nginx` Job的状态为**UP**。

------

### **5. 配置Grafana仪表盘**

#### **5.1 添加Prometheus数据源**

1. 登录Grafana，进入 **Configuration > Data Sources**。
2. 选择 **Prometheus**，填写以下信息：
   - **Name**: `Prometheus`
   - **URL**: `http://<prometheus-ip>:9090`
3. 点击 **Save & Test**，确认返回“Data source is working”。

#### **5.2 导入Nginx仪表盘**

1. 进入 **Create > Import**。
2. 输入仪表盘ID：`12708`（官方推荐）或 `2949`，点击 **Load**。
3. 选择数据源为`Prometheus`，点击 **Import**。

#### **5.3 查看监控仪表盘**

- **关键面板**：
  - 请求速率（Requests per Second）
  - 活跃连接数（Active Connections）
  - 连接状态（Reading/Writing/Waiting）
  - 历史趋势（Historical Metrics）

------

### **6. 关键指标说明**

| 指标名称                     | 描述                   |
| :--------------------------- | :--------------------- |
| `nginx_connections_accepted` | 总接受的客户端连接数   |
| `nginx_connections_handled`  | 总处理的客户端连接数   |
| `nginx_requests_total`       | 总处理的请求数         |
| `nginx_connections_active`   | 当前活跃连接数         |
| `nginx_connections_reading`  | 正在读取请求头的连接数 |
| `nginx_connections_writing`  | 正在发送响应的连接数   |
| `nginx_connections_waiting`  | 空闲客户端连接数       |

------

### **7. 故障排查**

#### **7.1 Exporter无法访问Nginx状态页**

- **现象**：`curl http://localhost:9113/metrics` 返回空或无数据。
- **解决方案**：
  1. 检查Nginx配置中的`allow`规则，确保Exporter服务器IP被允许。
  2. 确认Nginx状态页路径是否为`/nginx_status`。

#### **7.2 Prometheus Target状态为DOWN**

- **现象**：Prometheus的Target列表中`nginx` Job状态为**DOWN**。

- **解决方案**：

  1. 检查防火墙是否放行`9113`端口：

     bash

     复制

     ```
     sudo ufw allow 9113/tcp
     ```

  2. 确认Exporter服务是否正常运行：

     bash

     复制

     ```
     sudo systemctl status nginx-exporter
     ```

#### **7.3 Grafana仪表盘无数据**

- **现象**：仪表盘显示“No data”。
- **解决方案**：
  1. 确认Grafana数据源配置的Prometheus地址正确。
  2. 检查Prometheus是否成功抓取Exporter指标（通过`http://prometheus-ip:9090/graph`查询`nginx_connections_active`）。

------

### **8. 附录：安全建议**

1. **限制Nginx状态页访问**：
   - 使用防火墙或Nginx的`allow/deny`规则，仅允许Exporter和本地访问。
2. **启用HTTPS**：
   - 为Exporter配置HTTPS（需在启动参数中添加`-web.listen-address`和证书）。
3. **监控Exporter性能**：
   - 通过Prometheus监控Exporter本身的资源使用情况（如CPU、内存）。

------

**文档版本**：1.0
**最后更新**：2023-10-05
**维护团队**：运维中心