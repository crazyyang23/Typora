## **Nginx Upstream 权重分配手册**

### **1. 概述**

本手册详细说明了 Nginx 中 `upstream` 模块的权重分配配置。通过权重分配，可以控制请求在不同服务器之间的分发比例，从而实现负载均衡。

------

### **2. Upstream 配置**

以下是 `upstream` 模块的配置示例：

nginx

复制

```
upstream backend {
    least_conn;
    server 10.2.255.60:30038 weight=1  max_fails=3 fail_timeout=10s;  # 服务器1的容器A，权重为1
    server 10.2.255.60:30091 weight=1  max_fails=3 fail_timeout=10s;  # 服务器1的容器B，权重为1
    server 10.2.255.15:30038 weight=99 max_fails=3 fail_timeout=10s;  # 服务器2的容器A，权重为99
    server 10.2.255.15:30091 weight=99 max_fails=3 fail_timeout=10s;  # 服务器2的容器B，权重为99
    server 10.2.255.15:30092 weight=99 max_fails=3 fail_timeout=10s;  # 服务器2的容器C，权重为99
    server 10.2.255.15:30093 weight=99 max_fails=3 fail_timeout=10s;  # 服务器2的容器D，权重为99
    server 10.2.255.15:30094 weight=99 max_fails=3 fail_timeout=10s;  # 服务器2的容器E，权重为99
    server 10.2.255.15:30095 weight=99 max_fails=3 fail_timeout=10s;  # 服务器2的容器F，权重为99
    keepalive 100;
}
```

------

### **3. 权重分配表**

以下是权重分配的详细表格：

| 服务器地址    | 端口  | 权重 | 最大失败次数 (`max_fails`) | 失败超时时间 (`fail_timeout`) | 说明                     |
| :------------ | :---- | :--- | :------------------------- | :---------------------------- | :----------------------- |
| `10.2.255.60` | 30038 | 1    | 3                          | 10s                           | 服务器1的容器A，权重为1  |
| `10.2.255.60` | 30091 | 1    | 3                          | 10s                           | 服务器1的容器B，权重为1  |
| `10.2.255.15` | 30038 | 99   | 3                          | 10s                           | 服务器2的容器A，权重为99 |
| `10.2.255.15` | 30091 | 99   | 3                          | 10s                           | 服务器2的容器B，权重为99 |
| `10.2.255.15` | 30092 | 99   | 3                          | 10s                           | 服务器2的容器C，权重为99 |
| `10.2.255.15` | 30093 | 99   | 3                          | 10s                           | 服务器2的容器D，权重为99 |
| `10.2.255.15` | 30094 | 99   | 3                          | 10s                           | 服务器2的容器E，权重为99 |
| `10.2.255.15` | 30095 | 99   | 3                          | 10s                           | 服务器2的容器F，权重为99 |

------

### **4. 权重分配说明**

1. **权重为1的服务器**：
   - `10.2.255.60:30038` 和 `10.2.255.60:30091` 的权重为1。
   - 这两个服务器分配的请求比例较低，适合处理少量请求或作为备用服务器。
2. **权重为99的服务器**：
   - `10.2.255.15` 的所有容器（30038、30091、30092、30093、30094、30095）的权重为99。
   - 这些服务器分配的请求比例较高，适合处理大量请求。
3. **请求分配比例**：
   - 总权重 = (1 + 1) + (99 × 6) = 2 + 594 = 596。
   - 权重为1的服务器分配的请求比例 = 1 / 596 ≈ 0.17%。
   - 权重为99的服务器分配的请求比例 = 99 / 596 ≈ 16.61%。
4. **负载均衡策略**：
   - 使用 `least_conn` 策略，优先将请求分配给当前连接数最少的服务器。
   - 权重仅在 `least_conn` 策略无法区分时生效。

------

### **5. 配置优化建议**

1. **调整权重**：
   - 如果 `10.2.255.60` 的服务器性能较弱，可以保持低权重。
   - 如果 `10.2.255.15` 的服务器性能较强，可以保持高权重。
2. **监控服务器性能**：
   - 使用监控工具（如 Prometheus、Zabbix）监控服务器的负载情况，动态调整权重。
3. **增加健康检查**：
   - 使用 Nginx Plus 或第三方工具（如 Consul）实现更灵活的健康检查机制。

------

### **6. 示例日志**

以下是一个示例日志，展示了请求分配的情况：

plaintext

复制

```
10.0.104.51 - - [16/Mar/2025:16:38:50 +0800] "GET /report/DataQuery/GetLastOneYearTransaction HTTP/1.1" 200 244 "http://cibu5.gwkf.cn/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/128.0.0.0 Safari/537.36" "-" "-" "- " nginx_server:"10.0.104.50" upstream:"10.2.255.15:30038" rt=0.885 uct="0.000" uht="0.881" urt="0.886"
```

------

### **7. 总结**

通过合理的权重分配，可以实现高效的负载均衡，确保请求能够均匀分配到各个服务器。建议根据实际业务需求和服务器性能动态调整权重。





## **Nginx Upstream 参数详解**

### **1. `least_conn`**

- **作用**：负载均衡策略，优先将请求分配给当前连接数最少的服务器。

- **适用场景**：适用于长连接或请求处理时间不均匀的场景。

- **示例**：

  nginx

  复制

  ```
  upstream backend {
      least_conn;
      server 10.2.255.60:30038;
      server 10.2.255.15:30038;
  }
  ```

------

### **2. `server`**

- **作用**：定义后端服务器的地址和端口。

- **参数**：

  - `address:port`：服务器的 IP 地址和端口。
  - `weight`：权重，默认为 1。权重越高，分配的请求越多。
  - `max_fails`：最大失败次数。在 `fail_timeout` 时间内，如果失败次数达到 `max_fails`，则认为服务器不可用。
  - `fail_timeout`：失败超时时间。在 `fail_timeout` 时间内，如果失败次数达到 `max_fails`，则服务器被标记为不可用，并在 `fail_timeout` 时间后重新尝试。
  - `backup`：将服务器标记为备用服务器，只有在其他服务器不可用时才会使用。
  - `down`：将服务器标记为永久不可用。

- **示例**：

  nginx

  复制

  ```
  server 10.2.255.60:30038 weight=1 max_fails=3 fail_timeout=10s;
  ```

------

### **3. `weight`**

- **作用**：定义服务器的权重，权重越高，分配的请求越多。

- **默认值**：1。

- **示例**：

  nginx

  复制

  ```
  server 10.2.255.60:30038 weight=1;  # 权重为1
  server 10.2.255.15:30038 weight=99; # 权重为99
  ```

------

### **4. `max_fails`**

- **作用**：在 `fail_timeout` 时间内，允许的最大失败次数。如果失败次数达到 `max_fails`，则认为服务器不可用。

- **默认值**：1。

- **示例**：

  nginx

  复制

  ```
  server 10.2.255.60:30038 max_fails=3;
  ```

------

### **5. `fail_timeout`**

- **作用**：定义失败超时时间。在 `fail_timeout` 时间内，如果失败次数达到 `max_fails`，则服务器被标记为不可用，并在 `fail_timeout` 时间后重新尝试。

- **默认值**：10 秒。

- **示例**：

  nginx

  复制

  ```
  server 10.2.255.60:30038 fail_timeout=10s;
  ```

------

### **6. `keepalive`**

- **作用**：定义与后端服务器保持的长连接数。减少频繁建立和关闭连接的开销。

- **默认值**：无（需要显式配置）。

- **示例**：

  nginx

  复制

  ```
  upstream backend {
      server 10.2.255.60:30038;
      keepalive 100;  # 保持100个长连接
  }
  ```

------

### **7. `backup`**

- **作用**：将服务器标记为备用服务器。只有在其他服务器不可用时才会使用。

- **示例**：

  nginx

  复制

  ```
  server 10.2.255.60:30038 backup;
  ```

------

### **8. `down`**

- **作用**：将服务器标记为永久不可用。

- **示例**：

  nginx

  复制

  ```
  server 10.2.255.60:30038 down;
  ```

------

## **Nginx 健康检查配置**

Nginx 开源版本本身不支持主动健康检查，但可以通过以下方式实现：

------

### **1. 被动健康检查**

Nginx 默认支持被动健康检查，基于 `max_fails` 和 `fail_timeout` 参数。

- **配置示例**：

  nginx

  复制

  ```
  upstream backend {
      server 10.2.255.60:30038 max_fails=3 fail_timeout=10s;
      server 10.2.255.15:30038 max_fails=3 fail_timeout=10s;
  }
  ```

- **工作原理**：

  - 在 `fail_timeout` 时间内，如果请求失败次数达到 `max_fails`，则服务器被标记为不可用。
  - 在 `fail_timeout` 时间后，Nginx 会重新尝试将请求分配到该服务器。

------

### **2. 主动健康检查（Nginx Plus）**

Nginx Plus 支持主动健康检查，可以定期向后端服务器发送健康检查请求。

- **配置示例**：

  nginx

  复制

  ```
  upstream backend {
      zone backend 64k;
      server 10.2.255.60:30038;
      server 10.2.255.15:30038;
  
      health_check interval=5s fails=3 passes=2 uri=/health;
  }
  ```

- **参数说明**：

  - `interval`：健康检查的间隔时间。
  - `fails`：连续失败次数，达到后标记服务器为不可用。
  - `passes`：连续成功次数，达到后标记服务器为可用。
  - `uri`：健康检查的 URI 路径。

------

### **3. 使用第三方工具实现健康检查**

如果使用 Nginx 开源版本，可以通过第三方工具（如 Consul、Prometheus）实现健康检查。

#### **3.1 使用 Consul**

- **步骤**：

  1. 在 Consul 中注册后端服务器。
  2. 使用 `nginx-upsync-module` 动态更新 Nginx 的 `upstream` 配置。

- **配置示例**：

  nginx

  复制

  ```
  upstream backend {
      upsync 127.0.0.1:8500/v1/kv/upstreams/backend upsync_timeout=6m upsync_interval=500ms upsync_type=consul strong_dependency=off;
      upsync_dump_path /etc/nginx/conf.d/backend.conf;
      include /etc/nginx/conf.d/backend.conf;
  }
  ```

#### **3.2 使用 Prometheus + Nginx Exporter**

- **步骤**：
  1. 使用 Prometheus 监控后端服务器的健康状态。
  2. 使用 Nginx Exporter 将 Nginx 的状态暴露给 Prometheus。
  3. 根据 Prometheus 的监控数据动态调整 Nginx 的 `upstream` 配置。

------

### **4. 健康检查的最佳实践**

1. **合理设置 `max_fails` 和 `fail_timeout`**：
   - 根据后端服务器的性能调整 `max_fails` 和 `fail_timeout`，避免误判。
2. **使用主动健康检查**：
   - 如果使用 Nginx Plus，建议启用主动健康检查，确保后端服务器的可用性。
3. **结合第三方工具**：
   - 如果使用 Nginx 开源版本，可以结合 Consul、Prometheus 等工具实现更灵活的健康检查。

------

## **总结**

```
for i in {1..100}; do curl -s -o /dev/null -w "%{time_total} %{url_effective}\n" http://center.gwkf.cn/mes/; sleep 0.1; done
```

通过合理配置 `upstream` 参数和健康检查机制，可以确保 Nginx 的高可用性和高性能。如果有其他问题，欢迎继续提问！