# Nginx 负载均衡配置手册

## 目录

1. 简介
2. 基本配置
3. 详细参数说明
   - upstream 模块参数
   - proxy_pass 相关参数
4. 完整配置示例
5. 总结

------

## 简介

Nginx 是一个高性能的 HTTP 和反向代理服务器，广泛用于负载均衡。通过 `upstream` 模块，Nginx 可以将客户端的请求分发到多台后端服务器，从而实现负载均衡。本手册详细介绍如何配置 Nginx 实现负载均衡，并通过域名访问。

------

## 基本配置

以下是一个通过域名访问并将请求分发到多台后端服务器的基本配置示例：

nginx

复制

```
http {
    # 定义 upstream 组，名称为 backend
    upstream backend {
        # 定义后端服务器列表
        server 192.168.1.1;
        server 192.168.1.2;
        server 192.168.1.3;
    }

    # 配置虚拟主机
    server {
        # 监听 80 端口
        listen 80;

        # 域名配置
        server_name example.com;

        # 路由配置
        location / {
            # 将请求转发到 upstream 组
            proxy_pass http://backend;

            # 以下是一些常用的 proxy 配置
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }
    }
}
```

------

## 详细参数说明

### upstream 模块参数

- **`server`**:

  - 语法: `server address [parameters];`

  - 作用: 定义后端服务器的地址。

  - 参数:

    - `weight`: 权重，默认为 1。权重越高，分配的请求越多。

      nginx

      复制

      ```
      server 192.168.1.1 weight=5;
      ```

    - `max_fails`: 最大失败次数，默认为 1。超过此次数后，服务器将被标记为不可用。

      nginx

      复制

      ```
      server 192.168.1.2 max_fails=3;
      ```

    - `fail_timeout`: 失败超时时间，默认为 10 秒。服务器被标记为不可用后，在此时间内不再尝试。

      nginx

      复制

      ```
      server 192.168.1.3 fail_timeout=30s;
      ```

    - `backup`: 标记为备用服务器，只有在其他服务器不可用时才使用。

      nginx

      复制

      ```
      server 192.168.1.4 backup;
      ```

    - `down`: 标记服务器为永久不可用。

      nginx

      复制

      ```
      server 192.168.1.5 down;
      ```

- **`least_conn`**:

  - 语法: `least_conn;`

  - 作用: 使用最少连接负载均衡算法，将请求分配给当前连接数最少的服务器。

    nginx

    复制

    ```
    upstream backend {
        least_conn;
        server 192.168.1.1;
        server 192.168.1.2;
    }
    ```

- **`ip_hash`**:

  - 语法: `ip_hash;`

  - 作用: 基于客户端 IP 的哈希值进行负载均衡，确保同一客户端的请求总是分配到同一服务器。

    nginx

    复制

    ```
    upstream backend {
        ip_hash;
        server 192.168.1.1;
        server 192.168.1.2;
    }
    ```

- **`hash`**:

  - 语法: `hash key [consistent];`

  - 作用: 基于指定键的哈希值进行负载均衡。`key` 可以是变量或字符串，`consistent` 表示使用一致性哈希算法。

    nginx

    复制

    ```
    upstream backend {
        hash $request_uri consistent;
        server 192.168.1.1;
        server 192.168.1.2;
    }
    ```

- **`keepalive`**:

  - 语法: `keepalive connections;`

  - 作用: 设置与后端服务器的长连接数，减少连接建立的开销，提升性能。

    nginx

    复制

    ```
    upstream backend {
        server 192.168.1.1;
        server 192.168.1.2;
        keepalive 32;
    }
    ```

- **`zone`**:

  - 语法: `zone name [size];`

  - 作用: 定义共享内存区域，用于存储负载均衡状态。`name` 是区域名称，`size` 是内存大小。

    nginx

    复制

    ```
    upstream backend {
        zone backend 64k;
        server 192.168.1.1;
        server 192.168.1.2;
    }
    ```

### proxy_pass 相关参数

- **`proxy_set_header`**:

  - 作用: 设置转发到后端服务器的请求头。

  - 常用配置:

    nginx

    复制

    ```
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
    ```

- **`proxy_http_version`**:

  - 作用: 设置与后端服务器通信的 HTTP 协议版本。

    nginx

    复制

    ```
    proxy_http_version 1.1;
    ```

- **`proxy_connect_timeout`**:

  - 作用: 设置与后端服务器建立连接的超时时间。

    nginx

    复制

    ```
    proxy_connect_timeout 60s;
    ```

- **`proxy_read_timeout`**:

  - 作用: 设置从后端服务器读取数据的超时时间。

    nginx

    复制

    ```
    proxy_read_timeout 60s;
    ```

- **`proxy_send_timeout`**:

  - 作用: 设置向后端服务器发送数据的超时时间。

    nginx

    复制

    ```
    proxy_send_timeout 60s;
    ```

------

## 完整配置示例

以下是一个完整的负载均衡配置示例，支持域名访问，并包含健康检查、长连接和负载均衡算法优化：

nginx

复制

```
http {
    # 定义 upstream 组
    upstream backend {
        # 使用最少连接算法
        least_conn;

        # 定义后端服务器
        server 192.168.1.1 weight=5 max_fails=3 fail_timeout=30s;
        server 192.168.1.2 weight=3 max_fails=3 fail_timeout=30s;
        server 192.168.1.3 backup;

        # 启用长连接
        keepalive 32;

        # 定义共享内存区域
        zone backend 64k;
    }

    # 配置虚拟主机
    server {
        listen 80;
        server_name example.com;

        location / {
            # 转发请求到 upstream 组
            proxy_pass http://backend;

            # 设置请求头
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;

            # 设置超时时间
            proxy_connect_timeout 60s;
            proxy_read_timeout 60s;
            proxy_send_timeout 60s;

            # 启用 HTTP/1.1
            proxy_http_version 1.1;
            proxy_set_header Connection "";
        }
    }
}
```

------

## 总结

- **负载均衡算法**:
  - 默认轮询：适合服务器性能相近的场景。
  - 加权轮询：适合服务器性能差异较大的场景。
  - 最少连接：适合长连接或处理时间差异较大的场景。
  - IP 哈希：适合需要会话保持的场景。
- **性能优化**:
  - 使用 `keepalive` 减少连接建立的开销。
  - 使用 `zone` 共享内存区域提升多 worker 进程的协作效率。
  - 合理配置 `max_fails` 和 `fail_timeout` 实现健康检查。

通过以上配置，Nginx 可以实现高效的负载均衡功能，支持域名访问，并将请求分发到多台后端服务器。

### 1. `upstream`块中的优化参数

#### 1.1 `zone`

- **作用**：定义一个共享内存区域，用于存储上游服务器的状态信息。

- **语法**：`zone <name> <size>;`

- **示例**：

  nginx

  复制

  ```
  upstream backend {
      zone backend_zone 64k;
      server 10.24.241.102:30255;
      server 10.24.241.103:30255;
  }
  ```

- **说明**：

  - 共享内存区域允许多个工作进程共享上游服务器的状态信息（如健康检查状态）。
  - `64k`是共享内存的大小，可以根据需要调整。

#### 1.2 `queue`

- **作用**：当所有后端服务器都不可用时，将请求放入队列中等待。

- **语法**：`queue <number> [timeout=<time>];`

- **示例**：

  nginx

  复制

  ```
  upstream backend {
      server 10.24.241.102:30255;
      server 10.24.241.103:30255;
      queue 100 timeout=30s;
  }
  ```

- **说明**：

  - `100`：队列中最多可以存放100个请求。
  - `timeout=30s`：请求在队列中的最大等待时间为30秒，超时后返回错误。

#### 1.3 `keepalive_requests`

- **作用**：限制单个长连接可以处理的请求数量。

- **语法**：`keepalive_requests <number>;`

- **示例**：

  nginx

  复制

  ```
  upstream backend {
      server 10.24.241.102:30255;
      server 10.24.241.103:30255;
      keepalive 32;
      keepalive_requests 1000;
  }
  ```

- **说明**：

  - `1000`：单个长连接最多可以处理1000个请求，超过后连接会被关闭。
  - 适用于高并发场景，避免长连接占用过多资源。

#### 1.4 `keepalive_timeout`

- **作用**：设置长连接的超时时间。

- **语法**：`keepalive_timeout <time>;`

- **示例**：

  nginx

  复制

  ```
  upstream backend {
      server 10.24.241.102:30255;
      server 10.24.241.103:30255;
      keepalive 32;
      keepalive_timeout 60s;
  }
  ```

- **说明**：

  - `60s`：长连接在空闲60秒后会被关闭。
  - 避免长时间占用连接资源。

------

### 2. `server`块中的优化参数

#### 2.1 `proxy_buffer_size`

- **作用**：设置代理缓冲区的大小，用于存储从后端服务器接收的响应头。

- **语法**：`proxy_buffer_size <size>;`

- **示例**：

  nginx

  复制

  ```
  server {
      location / {
          proxy_pass http://backend;
          proxy_buffer_size 4k;
      }
  }
  ```

- **说明**：

  - `4k`：缓冲区大小为4KB。
  - 如果响应头较大，可以适当增加缓冲区大小。

#### 2.2 `proxy_buffers`

- **作用**：设置用于存储从后端服务器接收的响应体的缓冲区数量和大小。

- **语法**：`proxy_buffers <number> <size>;`

- **示例**：

  nginx

  复制

  ```
  server {
      location / {
          proxy_pass http://backend;
          proxy_buffers 8 4k;
      }
  }
  ```

- **说明**：

  - `8`：缓冲区的数量为8个。
  - `4k`：每个缓冲区的大小为4KB。
  - 适用于响应体较大的场景。

#### 2.3 `proxy_busy_buffers_size`

- **作用**：设置忙碌缓冲区的大小，用于存储正在发送给客户端的响应数据。

- **语法**：`proxy_busy_buffers_size <size>;`

- **示例**：

  nginx

  复制

  ```
  server {
      location / {
          proxy_pass http://backend;
          proxy_busy_buffers_size 8k;
      }
  }
  ```

- **说明**：

  - `8k`：忙碌缓冲区的大小为8KB。
  - 适用于高并发场景，确保响应数据能够及时发送。

#### 2.4 `proxy_connect_timeout`

- **作用**：设置与后端服务器建立连接的超时时间。

- **语法**：`proxy_connect_timeout <time>;`

- **示例**：

  nginx

  复制

  ```
  server {
      location / {
          proxy_pass http://backend;
          proxy_connect_timeout 5s;
      }
  }
  ```

- **说明**：

  - `5s`：连接超时时间为5秒。
  - 如果后端服务器响应较慢，可以适当增加超时时间。

#### 2.5 `proxy_read_timeout`

- **作用**：设置从后端服务器读取数据的超时时间。

- **语法**：`proxy_read_timeout <time>;`

- **示例**：

  nginx

  复制

  ```
  server {
      location / {
          proxy_pass http://backend;
          proxy_read_timeout 60s;
      }
  }
  ```

- **说明**：

  - `60s`：读取超时时间为60秒。
  - 适用于响应时间较长的场景。

#### 2.6 `proxy_send_timeout`

- **作用**：设置向后端服务器发送数据的超时时间。

- **语法**：`proxy_send_timeout <time>;`

- **示例**：

  nginx

  复制

  ```
  server {
      location / {
          proxy_pass http://backend;
          proxy_send_timeout 60s;
      }
  }
  ```

- **说明**：

  - `60s`：发送超时时间为60秒。
  - 适用于请求体较大的场景。

------

### 3. 全局优化参数

#### 3.1 `worker_processes`

- **作用**：设置Nginx工作进程的数量。

- **语法**：`worker_processes <number>;`

- **示例**：

  nginx

  复制

  ```
  worker_processes auto;
  ```

- **说明**：

  - `auto`：根据CPU核心数自动设置工作进程数量。
  - 通常设置为CPU核心数或核心数的2倍。

#### 3.2 `worker_connections`

- **作用**：设置每个工作进程的最大连接数。

- **语法**：`worker_connections <number>;`

- **示例**：

  nginx

  复制

  ```
  events {
      worker_connections 1024;
  }
  ```

- **说明**：

  - `1024`：每个工作进程最多可以处理1024个连接。
  - 适用于高并发场景。

#### 3.3 `multi_accept`

- **作用**：设置每个工作进程是否一次性接受所有新连接。

- **语法**：`multi_accept on|off;`

- **示例**：

  nginx

  复制

  ```
  events {
      multi_accept on;
  }
  ```

- **说明**：

  - `on`：一次性接受所有新连接，适用于高并发场景。

------

### 

4. 算法比较

   | `least_conn`（最少连接算法） | 1. **动态分配合理**：会将新请求分配给当前连接数最少的服务器，能更好地应对不同服务器处理请求能力的差异。例如，当部分服务器处理请求速度较快，连接数会相对较少，新请求就会优先分配给这些服务器，充分利用服务器资源。 2. **适应请求处理时间差异**：对于处理时间差异较大的请求，该算法能避免请求集中在处理慢的服务器上，提高整体性能。比如，某些请求需要进行复杂的数据库查询，处理时间长，而其他请求处理时间短，使用该算法可均衡负载。 | 1. **计算开销大**：需要实时跟踪每个服务器的连接数，会带来一定的计算开销，尤其是在服务器数量较多或请求频繁的情况下，可能会影响 Nginx 的性能。 2. **对短期波动不敏感**：当服务器的负载突然发生变化时，由于算法主要基于连接数进行分配，可能无法迅速适应这种短期波动，导致部分服务器过载，而其他服务器资源未充分利用。 | 1. 请求处理时间差异较大的场景，如包含复杂业务逻辑处理、大量数据库查询等的应用。 2. 服务器硬件性能差异较大的环境，能让性能好的服务器处理更多请求。 |
   | ---------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
   | `round - robin`（轮询算法）  | 1. **简单高效**：算法实现简单，不需要记录服务器的状态信息，对 Nginx 的性能影响极小。它按照固定顺序依次将请求分配给各个服务器，处理速度快。 2. **公平分配**：每个服务器都会依次收到请求，保证了服务器之间的相对公平性，避免某个服务器被过度使用。 | 1. **未考虑服务器性能差异**：不考虑服务器的实际处理能力和负载情况，即使某些服务器性能较差或已经过载，仍会按照顺序分配请求，可能导致部分服务器性能瓶颈。 2. **不适应请求处理时间差异**：对于处理时间差异较大的请求，可能会使处理慢的服务器积压大量请求，而处理快的服务器空闲，无法充分发挥服务器的整体性能。 | 1. 请求处理时间较为均匀的场景，如静态资源服务，每个请求的处理时间大致相同。 2. 服务器硬件性能相近的环境，各服务器能够较好地处理相同数量的请求。 |

5. 总结

6. 案例解释


      **配置文件的可读性和可维护性得到了提高。每个 upstream 块都有清晰的注释，说明了其用途和配置含义。同时，整体格式也更加统一，便于后续的修改和扩展**

       location /api/ {
       # 设置代理读取超时时间为 300 秒
       proxy_read_timeout 300;
        # 设置代理发送超时时间为 300 秒
       proxy_send_timeout 300;
       # 将请求代理到 http://center_api/
       proxy_pass http://center_api/;
       # 使用 HTTP/1.1 协议进行代理
       proxy_http_version 1.1;
       # 设置请求头 Host 为客户端请求的主机名
       proxy_set_header Host $http_host;
       # 设置请求头 X-Real-IP 为客户端的真实 IP 地址
       proxy_set_header X-Real-IP $remote_addr;
       
       # 根据 $http_upgrade 动态设置 Connection 头
       proxy_set_header Connection $http_upgrade;
       # 如果是 WebSocket 升级请求，不使用缓存
       proxy_cache_bypass $http_upgrade;
       
       # 设置 X-Forwarded-For 头，追加客户端 IP 地址链
       proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
       # 设置 X-Forwarded-Proto 头为客户端请求的协议（http 或 https）
       proxy_set_header X-Forwarded-Proto $scheme;
       # 设置 X-Forwarded-Prefix 头为 api
       proxy_set_header X-Forwarded-Prefix api;
       
       # 传递上游服务器的 X-Response-Time 头
       proxy_pass_header X-Response-Time;
       
       # 使用名为 my_cache 的缓存
       proxy_cache my_cache;
       # 缓存 200、302 状态码的响应 10 分钟
       proxy_cache_valid 200 302 10m;
       # 缓存 404 状态码的响应 1 分钟
       proxy_cache_valid 404 1m;
       }
   

通过添加和调整上述参数，可以显著优化Nginx的性能。以下是一些建议：

- 使用`zone`共享上游服务器状态信息。
- 配置`keepalive`和`keepalive_requests`优化长连接。
- 调整`proxy_buffer_size`和`proxy_buffers`优化代理性能。
- 设置合理的超时时间（如`proxy_connect_timeout`、`proxy_read_timeout`）。
- 根据服务器硬件配置调整`worker_processes`和`worker_connections`。

根据实际需求选择合适的参数，并定期监控和优化配置，可以确保Nginx在高并发场景下稳定运行。