# Dapr 与 gRPC 通信问题处理手册

## 一、概述

Dapr（分布式应用运行时）通过 Sidecar 模式为应用提供分布式能力，支持 HTTP 和 gRPC 两种通信协议。gRPC 因高性能、低延迟特性，常用于高吞吐量场景，但配置和调试相对复杂。本手册汇总了 Dapr 与 gRPC 通信的常见问题、分析思路及解决方案，适用于 Docker 部署环境。

## 二、核心概念与端口说明

### 1. Dapr gRPC 端口分类

| 端口类型       | 用途                     | 默认值 | 配置参数               | 日志特征                          |
| -------------- | ------------------------ | ------ | ---------------------- | --------------------------------- |
| 公共 gRPC 端口 | 应用与 Dapr Sidecar 通信 | 50001  | `--dapr-grpc-port`     | `gRPC server listening on port X` |
| 内部 gRPC 端口 | Dapr 组件间内部通信      | 随机   | `--internal-grpc-port` | `internal gRPC server on port X`  |
| 工作流调度端口 | Dapr Scheduler 服务通信  | 50007  | `--port`（scheduler）  | `scheduler listening on port X`   |

### 2. 关键特征

- 公共 gRPC 端口是应用调用 Dapr 服务的核心端口，需重点关注。
- 端口可能因 Dapr 版本、配置参数或环境变量偏离默认值（如 50002）。

## 三、常见问题及解决方案

### 问题 1：grpcurl 提示 “missing port in address”

#### 现象

执行 `grpcurl -plaintext <service-name> dapr.proto.runtime.v1.Dapr/ListComponents` 时报错：
`connection error: desc = "transport: error while dialing: dial tcp: address <service-name>: missing port in address"`

#### 原因

gRPC 通信需同时指定**服务名（或 IP）和端口**，命令中遗漏了 Dapr 监听的 gRPC 端口。

#### 解决方案

1. **确认 Dapr 实际监听的 gRPC 端口**（通过日志）：

   bash

   

   

   

   

   

   ```bash
   docker logs <dapr-container-id> | grep "gRPC server listening on port"
   ```

   示例输出：`gRPC server listening on port 50002`（端口为 50002）。

2. **补充端口重新执行**：

   bash

   

   

   

   

   

   ```bash
   grpcurl -plaintext <service-name>:<实际端口> dapr.proto.runtime.v1.Dapr/ListComponents
   ```

   示例：`grpcurl -plaintext centerskjmesmescoreservice-dapr:50002 dapr.proto.runtime.v1.Dapr/ListComponents`

### 问题 2：无法连接到 Dapr gRPC 服务（connection refused）

#### 现象

执行 gRPC 命令时提示：`dial tcp <ip>:<port>: connect: connection refused`

#### 可能原因及排查步骤

1. **端口错误**

   - 重新检查日志确认 Dapr 监听的端口（非默认 50001 时需特别注意）。

2. **网络不通**

   - 若使用容器名访问，确认容器在同一 Docker 网络：

     bash

     

     

     

     

     

     ```bash
     docker network inspect <network-name> | grep <container-name>  # 检查网络成员
     ```

   - 若跨网络，需通过容器 IP 访问：

     bash

     

     

     

     

     

     ```bash
     # 获取容器 IP
     CONTAINER_IP=$(docker inspect -f '{{.NetworkSettings.IPAddress}}' <container-name>)
     # 测试连接
     grpcurl -plaintext $CONTAINER_IP:<port> ...
     ```

3. **端口未映射到宿主机**

   - 若需从宿主机访问，检查容器是否配置端口映射：

     bash

     

     

     

     

     

     ```bash
     docker inspect <container-name> | jq '.[0].NetworkSettings.Ports'
     ```

   - 若无映射，需重启容器并添加映射（示例）：

     bash

     

     

     

     

     

     ```bash
     docker run -d --name <container-name> -p <宿主机端口>:<容器端口> ...
     ```

### 问题 3：容器内无 netstat/ss 工具，无法查看端口

#### 现象

执行 `docker exec -it <container> netstat` 提示命令不存在。

#### 解决方案

1. **通过日志直接获取端口**（推荐）：

   bash

   

   

   

   

   

   ```bash
   docker logs <container-name> | grep "listening on port"
   ```

2. **使用 docker inspect 查看网络配置**：

   bash

   

   

   

   

   

   ```bash
   # 查看容器网络模式
   docker inspect <container-name> | jq '.[0].HostConfig.NetworkMode'
   # 查看容器 IP（bridge 模式下）
   docker inspect -f '{{.NetworkSettings.IPAddress}}' <container-name>
   ```

3. **通过宿主机 nsenter 工具查看**：

   bash

   ```bash
   # 获取容器 PID
   PID=$(docker inspect -f '{{.State.Pid}}' <container-name>)
   # 进入容器网络命名空间查看端口
   nsenter -t $PID -n netstat -tulpn | grep LISTEN
   ```

### 问题 4：应用代码无法通过 gRPC 调用 Dapr

#### 现象

应用日志提示 gRPC 连接失败（如 `failed to connect to localhost:50001`）。

#### 解决方案

1. **确认代码中端口与 Dapr 实际端口一致**
   示例（JavaScript）：

   javascript

   

   ```javascript
   const client = new DaprClient({
     daprHost: "localhost",
     daprPort: "50002",  // 与 Dapr 日志中的端口一致
     communicationProtocol: CommunicationProtocolEnum.GRPC,
   });
   ```

2. **检查容器网络模式**

   - 若 Dapr Sidecar 与应用共享网络（如 `network_mode: "service:app"`），应用可通过 `localhost:<port>` 访问。
   - 若为独立网络，需使用 Dapr 容器名或 IP 作为主机名。

## 四、gRPC 通信验证工具：grpcurl 详解

### 1. 安装方法

| 系统          | 命令                                                         |
| ------------- | ------------------------------------------------------------ |
| macOS         | `brew install grpcurl`                                       |
| Ubuntu/Debian | `sudo apt-get install -y grpcurl`                            |
| 手动安装      | 从 [GitHub 发布页](https://github.com/fullstorydev/grpcurl/releases) 下载二进制文件 |

### 2. 常用命令

| 功能               | 命令示例                                                     |
| ------------------ | ------------------------------------------------------------ |
| 列出所有 gRPC 服务 | `grpcurl -plaintext <host>:<port> list`                      |
| 查看服务方法       | `grpcurl -plaintext <host>:<port> list dapr.proto.runtime.v1.Dapr` |
| 调用保存状态方法   | `grpcurl -plaintext -d '{"storeName":"statestore","states":[{"key":"test","value":"dGVzdA=="}]}' <host>:<port> dapr.proto.runtime.v1.Dapr/SaveState` |
| 调用获取状态方法   | `grpcurl -plaintext -d '{"storeName":"statestore","key":"test"}' <host>:<port> dapr.proto.runtime.v1.Dapr/GetState` |

## 五、Docker 环境配置最佳实践

### 1. Docker Compose 配置示例（含 gRPC 端口）

yaml











```yaml
version: '3'
services:
  app:  # 应用服务
    build: .
    networks:
      - dapr-network

  app-dapr:  # Dapr Sidecar
    image: daprio/daprd:1.11.2
    command: [
      "./daprd",
      "--app-id", "app",
      "--app-port", "3000",
      "--dapr-grpc-port", "50002",  # 显式指定 gRPC 端口
      "--resources-path", "/components"
    ]
    volumes:
      - ./components:/components
    network_mode: "service:app"  # 与应用共享网络，支持 localhost 访问
    depends_on:
      - app

networks:
  dapr-network:
```

### 2. 网络访问规则

| 访问场景                 | 访问方式                   | 示例                                           |
| ------------------------ | -------------------------- | ---------------------------------------------- |
| 同一网络内容器间访问     | `容器名:端口`              | `app-dapr:50002`                               |
| 宿主机访问容器（无映射） | `容器IP:端口`              | `172.18.0.3:50002`                             |
| 宿主机访问容器（有映射） | `localhost:宿主机映射端口` | `localhost:50002`（映射配置 `-p 50002:50002`） |

## 六、排查流程总结

1. **确认 Dapr 实际 gRPC 端口**：通过容器日志提取 `gRPC server listening on port X` 中的 X。

2. 验证网络连通性

   ：

   - 容器内：`nc -zv <目标IP/容器名> <端口>`
   - 宿主机：`nc -zv <容器IP/localhost> <端口>`

3. **检查应用配置**：确保代码中 gRPC 端口与实际端口一致。

4. **使用 grpcurl 测试**：通过 `grpcurl -plaintext <host>:<port> ...` 验证服务可用性。

## 七、参考资料

- [Dapr gRPC 官方文档](https://docs.dapr.io/developing-applications/building-blocks/service-invocation/service-invocation-grpc/)
- [grpcurl 工具文档](https://github.com/fullstorydev/grpcurl)
- [Docker 网络配置指南](https://docs.docker.com/network/)