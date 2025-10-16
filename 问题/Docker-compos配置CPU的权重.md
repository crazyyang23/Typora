在 **Docker Compose** 中，可以通过 `cpu_shares`（权重）和 `cpus`（硬限制）结合使用，实现更精细的 **CPU 资源控制**。

对于你的 `docker-compose-version.yml` 文件，我们可以根据不同服务的 **CPU 需求** 进行优化配置，确保关键服务获得更多资源，同时限制低优先级服务的 CPU 使用。

------

## **1. `cpu_shares`（权重） vs. `cpus`（硬限制）**

| 参数             | 作用                                           | 适用场景                            | 示例                                    |
| :--------------- | :--------------------------------------------- | :---------------------------------- | :-------------------------------------- |
| **`cpu_shares`** | 设置 **CPU 资源分配的相对权重**（默认 `1024`） | 适用于需要动态调整 CPU 优先级的服务 | `cpu_shares: 2048`（权重是默认的 2 倍） |
| **`cpus`**       | 限制容器 **最多能使用的 CPU 核数**             | 适用于需要严格限制 CPU 使用的服务   | `cpus: 1.5`（最多使用 1.5 个 CPU 核心） |

### **最佳实践**

- **关键服务（如 `skjmesmesservice`）**：提高 `cpu_shares`（如 `2048`），并设置合理的 `cpus`（如 `2.0`）。
- **次要服务（如 `skjmesplanservice`）**：降低 `cpu_shares`（如 `512`），并限制 `cpus`（如 `0.5`）。
- **默认服务**：保持 `cpu_shares: 1024`，`cpus` 根据需求设定。

------

## **2. 优化配置示例**

### **(1) 关键服务（高 CPU 需求）**

yaml

复制

```
$(SystemID)skjmesmesservice:
  image: harbor.kaifa.cn/center/skj.mes.messervice:$(MesService_Version)
  cpu_shares: 2048  # 权重是默认的 2 倍（高优先级）
  cpus: 2.0        # 最多使用 2 个 CPU 核心
  # 其他配置...
```

### **(2) 次要服务（低 CPU 需求）**

yaml

复制

```
$(SystemID)skjmesplanservice:
  image: harbor.kaifa.cn/center/skj.mes.planservice:$(PlanService_Version)
  cpu_shares: 512  # 权重是默认的一半（低优先级）
  cpus: 0.5        # 最多使用 0.5 个 CPU 核心
  # 其他配置...
```

### **(3) 默认服务（中等 CPU 需求）**

yaml

复制

```
$(SystemID)skjmesbasservice:
  image: harbor.kaifa.cn/center/skj.mes.basservice:$(BasService_Version)
  cpu_shares: 1024  # 默认权重（中等优先级）
  cpus: 1.0         # 最多使用 1 个 CPU 核心
  # 其他配置...
```

------

## **3. 完整优化方案（推荐配置）**

以下是针对你的 `docker-compose-version.yml` 的优化建议：

| 服务名称               | `cpu_shares`（权重） | `cpus`（最大核数） | 说明                       |
| :--------------------- | :------------------- | :----------------- | :------------------------- |
| `skjmesmesservice`     | `2048`               | `2.0`              | **核心服务，高优先级**     |
| `skjmesmesservice1`    | `2048`               | `2.0`              | **核心服务，高优先级**     |
| `skjmesmesservice2`    | `2048`               | `2.0`              | **核心服务，高优先级**     |
| `skjmesmesservice3`    | `2048`               | `2.0`              | **核心服务，高优先级**     |
| `skjmesmesservice4`    | `2048`               | `2.0`              | **核心服务，高优先级**     |
| `skjmesaiservice`      | `1536`               | `1.5`              | **AI 服务，较高优先级**    |
| `skjmesmescoreservice` | `1536`               | `1.5`              | **核心服务，较高优先级**   |
| `skjmeswmscoreservice` | `1536`               | `1.5`              | **核心服务，较高优先级**   |
| `skjmesbasservice`     | `1024`               | `1.0`              | **基础服务，默认优先级**   |
| `skjmeseasservice`     | `1024`               | `1.0`              | **基础服务，默认优先级**   |
| `skjmesplanservice`    | `512`                | `0.5`              | **计划服务，低优先级**     |
| `skjmesqmsservice`     | `512`                | `0.5`              | **质量管理服务，低优先级** |
| `skjmesreportservice`  | `512`                | `0.5`              | **报表服务，低优先级**     |
| `k8s-vue`              | `512`                | `0.3`              | **前端服务，低优先级**     |

------

## **4. 如何验证配置是否生效？**

### **(1) 启动服务**

bash

复制

```
docker-compose -f docker-compose-version.yml up -d
```

### **(2) 查看容器资源限制**

bash

复制

```
docker inspect <container_name> | grep -i "CpuShares\|NanoCpus"
```

- `CpuShares` 显示权重（如 `2048`）。
- `NanoCpus` 显示 `cpus` 限制（如 `2000000000` 表示 `2.0` 核）。

### **(3) 实时监控 CPU 使用**

bash

复制

```
docker stats
```

查看各容器的 CPU 占用情况，确保资源分配符合预期。

------

## **5. 总结**

- **`cpu_shares`（权重）**：调整不同服务的 CPU 优先级（默认 `1024`）。
- **`cpus`（硬限制）**：防止某个服务占用过多 CPU 资源。
- **优化策略**：
  - **关键服务**（如 `messervice`）提高权重（`2048`），并放宽 CPU 限制（`2.0`）。
  - **次要服务**（如 `planservice`）降低权重（`512`），并严格限制 CPU（`0.5`）。
  - **默认服务**（如 `basservice`）保持 `cpu_shares: 1024`，`cpus: 1.0`。

这样配置后，系统在高负载时仍能保证核心服务稳定运行，同时避免低优先级服务占用过多资源。 