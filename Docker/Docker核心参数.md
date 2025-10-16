docker命令的参数详细描述

### **核心参数**

#### 1. **容器运行模式**

- **`-d` / `--detach`**
  后台运行容器（脱离终端），常用于服务类容器。
  示例：`docker run -d nginx`
- **`-it`**
  交互式运行容器（分配伪终端并保持 STDIN 打开），常用于调试或交互式操作。
  示例：`docker run -it ubuntu /bin/bash`

------

#### 2. **容器命名与重启策略**

- **`--name`**
  指定容器名称（唯一标识），避免使用随机名称。
  示例：`docker run --name my-nginx nginx`
- **`--restart`**
  定义容器退出时的重启策略：
  - `no`：不重启（默认）
  - `on-failure[:max-retries]`：非正常退出时重启（可指定最大重试次数）
  - `always`：总是重启
  - `unless-stopped`：除非手动停止，否则总是重启
    示例：`docker run --restart unless-stopped nginx`

------

#### 3. **网络配置**

- **`-p` / `--publish`**
  映射宿主机端口到容器端口（端口绑定）：
  格式：`-p [宿主机IP:]宿主机端口:容器端口[/协议]`
  示例：

  bash

  复制

  ```
  -p 8080:80           # 宿主机8080 → 容器80 (TCP)
  -p 192.168.1.10::53  # 随机宿主机端口 → 容器53
  -p 3306:3306/udp     # 指定UDP协议
  ```

- **`-P` / `--publish-all`**
  自动绑定容器所有暴露的端口到宿主机随机端口。

- **`--network`**
  指定容器使用的网络模式：

  - `bridge`：默认桥接网络
  - `host`：共享宿主机网络栈
  - `none`：无网络
  - 自定义网络：如 `--network my-net`
    示例：`docker run --network host nginx`

------

#### 4. **文件系统挂载**

- **`-v` / `--volume`**
  挂载宿主机目录或卷到容器：
  格式：`-v 宿主机路径:容器内路径[:权限]`
  示例：

  bash

  复制

  ```
  -v /data:/app/data         # 挂载目录（读写）
  -v my-volume:/app/config   # 挂载命名卷
  -v /config.json:/app/config.json:ro  # 挂载单个文件（只读）
  ```

- **`--mount`**
  更灵活的挂载方式（支持详细参数）：

  bash

  复制

  ```
  --mount type=bind,source=/host/data,target=/container/data
  --mount type=volume,source=my-vol,target=/app/data
  ```

------

#### 5. **环境变量**

- **`-e` / `--env`**
  设置容器内的环境变量：

  bash

  复制

  ```
  -e MY_ENV=value            # 单个变量
  -e "JAVA_OPTS=-Xms512m"    # 带特殊字符的值
  ```

- **`--env-file`**
  从文件批量加载环境变量：

  bash

  复制

  ```
  --env-file /path/to/env.list
  ```

------

#### 6. **资源限制**

- **`--memory` / `-m`**
  限制容器内存使用：

  bash

  复制

  ```
  -m 512m        # 限制为512MB
  -m 2g          # 限制为2GB
  ```

- **`--cpus`**
  限制容器可用的 CPU 核心数：

  bash

  复制

  ```
  --cpus 1.5     # 使用1.5个CPU核心
  ```

- **`--cpu-shares`**
  设置 CPU 权重（默认1024）：

  bash

  复制

  ```
  --cpu-shares 512  # 优先级减半
  ```

------

#### 7. **容器元数据**

- **`--label`**
  为容器添加元数据标签：

  bash

  复制

  ```
  --label com.example.version="1.0"
  ```

- **`--rm`**
  容器退出后自动删除（适用于临时任务）：

  bash

  复制

  ```
  docker run --rm alpine echo "Hello"
  ```

------

### **完整示例**

bash

复制

```
docker run -d \
  --name my-webapp \
  -p 8080:80 \
  -v /opt/webapp/config:/app/config:ro \
  -v webapp-data:/app/data \
  -e DB_HOST=mysql.example.com \
  -e TZ=Asia/Shanghai \
  --network my-bridge-net \
  --restart unless-stopped \
  --memory 1g \
  --cpus 2 \
  dgharbor.kaifa.cn/dgtc/skj.task.web:latest
```

------

### **高级参数**

| 参数             | 说明                                               |
| :--------------- | :------------------------------------------------- |
| `--user`         | 指定运行容器的用户（如 `--user 1000:1000`）        |
| `--security-opt` | 安全配置（如 `--security-opt seccomp=unconfined`） |
| `--ulimit`       | 限制容器资源（如 `--ulimit nofile=1024:2048`）     |
| `--device`       | 挂载宿主机设备（如 `--device /dev/sda:/dev/xvda`） |
| `--log-driver`   | 指定日志驱动（如 `--log-driver json-file`）        |



### 默认情况

在Docker容器内部，默认的DNS服务器通常是宿主机的DNS设置。当你启动一个容器时，Docker会自动将宿主机的`/etc/resolv.conf`文件中的DNS服务器配置复制到容器中，除非你显式地指定了其他DNS服务器。

默认情况下，容器内部的DNS服务器通常是以下地址之一：

- 127.0.0.11：这是Docker分配给容器的内置DNS服务器，它用于解析其他容器的名字。 -宿主机的DNS服务器地址

- **使用容器运行时默认配置**：如果使用常见的容器运行时如 Docker，默认情况下，容器会使用由 Docker 网络驱动分配的 DNS 服务器地址。通常，Docker 会为容器网络创建一个虚拟的网桥（如`docker0`），并在该网络中运行一个 DNS 服务，容器内部的 DNS IP 会被设置为这个虚拟网络中的 DNS 服务地址，一般是容器网络的网关地址。例如，在默认的`bridge`网络模式下，可能是`172.17.0.1`，不同的环境和配置可能会有所不同。
- **使用宿主机的 DNS 配置**：有些容器运行时可能会默认将容器的 DNS 配置继承自宿主机，此时容器内部的 DNS IP 就是宿主机所使用的 DNS 服务器地址。可以通过查看宿主机的`/etc/resolv.conf`文件来确定，常见的有`114.114.114.114`、`8.8.8.8`等公共 DNS 服务器地址，或者是企业内部自定义的 DNS 服务器地址。

### 自定义情况



- **手动指定 DNS IP**：在启动容器时，可以通过命令行参数或配置文件来手动指定容器内部的 DNS IP。以 Docker 为例，使用`--dns`参数可以指定自定义的 DNS 服务器地址，如`docker run --dns=192.168.1.100 -it your_image_name`，这里将容器的 DNS IP 设置为`192.168.1.100`。在 Kubernetes 等容器编排系统中，也可以在 Pod 的配置文件中通过`dnsConfig`字段来指定自定义的 DNS 服务器地址。
- **在容器内修改配置文件**：进入容器后，也可以直接修改容器内的`/etc/resolv.conf`文件来更改 DNS IP。但这种方式在容器重启后可能会失效，因为容器的配置可能会被重新初始化。

### **注意事项**

1. **变量替换**：在 CI/CD 流水线中，`$(变量名)` 会被替换为实际值（如 `$(Build.BuildId)`）。
2. **路径规范**：始终使用绝对路径挂载目录（如 `/host/path` 而非 `./relative/path`）。
3. **权限问题**：确保容器进程对挂载目录有读写权限（可通过 `chmod` 或 `--user` 调整）。