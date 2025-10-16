# Docker Compose 命令手册

## 概述

`docker-compose` 是一个用于定义和运行多容器 Docker 应用程序的工具。它通过一个 YAML 文件（通常命名为 `docker-compose.yml`）来配置应用程序的服务、网络和卷等。本手册详细介绍了 `docker-compose` 的常用命令及其参数。

## 命令详解

### 1. `docker-compose up`

**用途**: 启动服务容器。

**常用参数**:

- `-d` 或 `--detach`: 在后台运行容器。
- `--build`: 在启动容器之前构建镜像。
- `--force-recreate`: 强制重新创建容器，即使配置没有更改。
- `--no-recreate`: 如果容器已经存在，则不重新创建。
- `--no-build`: 不构建镜像，即使镜像缺失。
- `--no-deps`: 不启动依赖的服务。
- `--abort-on-container-exit`: 如果有容器停止，则停止所有容器。
- `--remove-orphans`：这个选项告诉 `docker-compose` 在启动服务之前移除那些在配置文件中没有被定义的，但是以 `docker-compose` 项目名标记运行的容器。这些容器可能是之前版本的配置文件创建的，但在新的配置文件中不再需要。移除孤儿容器有助于清理不再需要的容器，并保持环境整洁。

**示例**:

bash

复制

```
docker-compose up -d
```

### 2. `docker-compose down`

**用途**: 停止并删除容器、网络、卷和镜像。

**常用参数**:

- `--volumes` 或 `-v`: 删除在 `docker-compose.yml` 中定义的匿名卷。
- `--rmi all`: 删除所有服务使用的镜像。
- `--rmi local`: 仅删除没有自定义标签的镜像。

**示例**:

bash

复制

```
docker-compose down --volumes
```

### 3. `docker-compose build`

**用途**: 构建或重新构建服务镜像。

**常用参数**:

- `--no-cache`: 构建镜像时不使用缓存。
- `--force-rm`: 总是删除中间容器。
- `--pull`: 始终尝试拉取更新的镜像版本。

**示例**:

bash

复制

```
docker-compose build --no-cache
```

### 4. `docker-compose start`

**用途**: 启动已经存在的服务容器。

**常用参数**:

- 无特定常用参数。

**示例**:

bash

复制

```
docker-compose start
```

### 5. `docker-compose stop`

**用途**: 停止运行中的服务容器。

**常用参数**:

- `-t` 或 `--timeout`: 设置停止容器的超时时间（默认为 10 秒）。

**示例**:

bash

复制

```
docker-compose stop -t 5
```

### 6. `docker-compose restart`

**用途**: 重启服务容器。

**常用参数**:

- `-t` 或 `--timeout`: 设置停止容器的超时时间（默认为 10 秒）。

**示例**:

bash

复制

```
docker-compose restart -t 5
```

### 7. `docker-compose pause`

**用途**: 暂停服务容器。

**常用参数**:

- 无特定常用参数。

**示例**:

bash

复制

```
docker-compose pause
```

### 8. `docker-compose unpause`

**用途**: 恢复暂停的服务容器。

**常用参数**:

- 无特定常用参数。

**示例**:

bash

复制

```
docker-compose unpause
```

### 9. `docker-compose logs`

**用途**: 查看服务容器的日志输出。

**常用参数**:

- `-f` 或 `--follow`: 实时跟踪日志输出。
- `--tail="all"`: 显示日志的最后几行（默认为全部）。
- `--no-color`: 禁用颜色输出。

**示例**:

bash

复制

```
docker-compose logs -f --tail=10
```

### 10. `docker-compose ps`

**用途**: 列出服务容器的状态。

**常用参数**:

- `-q` 或 `--quiet`: 仅显示容器 ID。
- `--services`: 列出服务名称。
- `--filter KEY=VAL`: 过滤输出。

**示例**:

bash

复制

```
docker-compose ps -q
```

### 11. `docker-compose exec`

**用途**: 在运行中的容器中执行命令。

**常用参数**:

- `-d` 或 `--detach`: 在后台运行命令。
- `--privileged`: 赋予命令特权。
- `-u` 或 `--user`: 以指定用户身份运行命令。

**示例**:

bash

复制

```
docker-compose exec web bash
```

### 12. `docker-compose pull`

**用途**: 拉取服务镜像。

**常用参数**:

- `--ignore-pull-failures`: 忽略拉取失败的镜像。
- `--parallel`: 并行拉取多个镜像。

**示例**:

bash

复制

```
docker-compose pull
```

### 13. `docker-compose config`

**用途**: 验证并查看 `docker-compose.yml` 文件的配置。

**常用参数**:

- `--services`: 列出服务名称。
- `--volumes`: 列出卷名称。
- `--resolve-image-digests`: 解析镜像摘要。

**示例**:

bash

复制

```
docker-compose config
```

### 14. `docker-compose scale`

**用途**: 设置服务的容器数量。

**常用参数**:

- 无特定常用参数。

**示例**:

bash

复制

```
docker-compose scale web=3
```

### 15. `docker-compose rm`

**用途**: 删除停止的服务容器。

**常用参数**:

- `-f` 或 `--force`: 强制删除，不提示确认。
- `-v`: 删除与容器关联的匿名卷。

**示例**:

bash

复制

```
docker-compose rm -fv
```

### 16. `docker-compose kill`

**用途**: 强制停止服务容器。

**常用参数**:

- `-s` 或 `--signal`: 发送指定的信号（默认为 `SIGKILL`）。

**示例**:

bash

复制

```
docker-compose kill -s SIGTERM
```

### 17. `docker-compose run`

**用途**: 在服务上运行一次性命令。

**常用参数**:

- `-d` 或 `--detach`: 在后台运行容器。
- `--name`: 为容器指定名称。
- `--entrypoint`: 覆盖默认的入口点。
- `-e` 或 `--env`: 设置环境变量。
- `--rm`: 运行后删除容器。

**示例**:

bash

复制

```
docker-compose run web echo "Hello, World!"
```

### 18. `docker-compose version`

**用途**: 显示 Docker Compose 版本信息。

**常用参数**:

- 无特定常用参数。

**示例**:

bash

复制

```
docker-compose version
```

### 19. `docker-compose top`

**用途**: 显示服务容器的运行进程。

**常用参数**:

- 无特定常用参数。

**示例**:

bash

复制

```
docker-compose top
```

### 20. `docker-compose events`

**用途**: 实时查看容器事件。

**常用参数**:

- `--json`: 以 JSON 格式输出事件。

**示例**:

bash

复制

```
docker-compose events --json
```

### 21. `docker-compose port`

**用途**: 显示服务容器的端口绑定。

**常用参数**:

- `--protocol`: 指定协议（`tcp` 或 `udp`）。
- `--index`: 指定容器索引。

**示例**:

bash

复制

```
docker-compose port web 80
```

### 22. `docker-compose push`

**用途**: 推送服务镜像到镜像仓库。

**常用参数**:

- `--ignore-push-failures`: 忽略推送失败的镜像。

**示例**:

bash

复制

```
docker-compose push
```

### 23. `docker-compose create`

**用途**: 创建服务容器但不启动。

**常用参数**:

- `--force-recreate`: 强制重新创建容器。
- `--no-recreate`: 如果容器已经存在，则不重新创建。
- `--build`: 在创建容器之前构建镜像。

**示例**:

bash

复制

```
docker-compose create
```

### 24. `docker-compose images`

**用途**: 列出服务容器使用的镜像。

**常用参数**:

- `-q` 或 `--quiet`: 仅显示镜像 ID。

**示例**:

bash

复制

```
docker-compose images
```

### 25. `docker-compose help`

**用途**: 显示帮助信息。

**常用参数**:

- 无特定常用参数。

**示例**:

bash

复制

```
docker-compose help
```

#### **`docker-compose config`**

**作用**: 验证并查看最终的 Compose 配置（合并所有配置后的结果）。
**参数**:

- `--services`: 显示服务列表。
- `--volumes`: 显示卷配置。

**示例**:

bash

复制

```
docker-compose config
```

------

### **3. 参数通用说明**

- **`-f` 或 `--file`**: 指定 Compose 文件（可叠加多个文件）。

  bash

  复制

  ```
  docker-compose -f docker-compose.yml -f override.yml up
  ```

- **`-p` 或 `--project-name`**: 指定项目名称（默认使用当前目录名）。

- **`--env-file`**: 指定环境变量文件路径（覆盖默认 `.env`）。



## 总结

`docker-compose` 提供了丰富的命令和参数来管理多容器应用程序。通过合理使用这些命令和参数，可以有效地构建、启动、停止和管理 Docker 容器。