# **Docker 容器 CPU & 内存问题处理手册**

**适用场景**：容器性能排查、资源限制调整、OOM Killer 问题、CPU 争用分析

------

## **1. 快速检查容器资源使用情况**

### **(1) 查看容器实时资源占用**

bash

复制

```
docker stats <容器名或ID> --no-stream
```

**输出示例**：

复制

```
CONTAINER   CPU %   MEM USAGE / LIMIT   MEM %   NET I/O     BLOCK I/O   PIDS
my-app     25.5%   450MiB / 2GiB       22.5%   1.2MB/800KB 0B/0B       10
```

**关键指标**：

- `CPU %` > 80% → 可能 CPU 不足
- `MEM %` > 90% → 可能触发 OOM Killer

------

### **(2) 查看容器资源限制**

bash

复制

```
docker inspect <容器名> --format '
=== CPU 限制 ===
- 权重 (Shares): {{.HostConfig.CpuShares}} (默认 1024)
- 周期 (Period): {{.HostConfig.CpuPeriod}} (默认 100000μs)
- 配额 (Quota): {{.HostConfig.CpuQuota}} (100000=1核)
- 可用 CPU 核心: {{.HostConfig.CpusetCpus}}

=== 内存限制 ===
- 内存限制: {{.HostConfig.Memory}} bytes
- 内存+Swap: {{.HostConfig.MemorySwap}}
- OOM Killer: {{.HostConfig.OomKillDisable}}
'
```

**关键点**：

- `CpuQuota=-1` → 无限制
- `Memory=0` → 无限制

------

## **2. 深入分析 CPU 问题**

### **(1) 检查 cgroup CPU 使用**

bash

复制

```
CONTAINER_ID=$(docker inspect -f '{{.Id}}' <容器名>)
cat /sys/fs/cgroup/cpu,cpuacct/docker/$CONTAINER_ID/cpu.stat
```

**关键字段**：

- `nr_periods`：CPU 周期数
- `nr_throttled`：被限制的次数（若 >0，说明 CPU 配额不足）
- `throttled_time`：被限制的总时间

### **(2) 调整 CPU 限制**

bash

复制

```
# 限制容器使用 1.5 核 CPU
docker update --cpus="1.5" <容器名>

# 绑定到特定 CPU 核心
docker update --cpuset-cpus="0,1" <容器名>
```

------

## **3. 深入分析内存问题**

### **(1) 检查 cgroup 内存使用**

bash

复制

```
cat /sys/fs/cgroup/memory/docker/$CONTAINER_ID/memory.stat
```

**关键字段**：

- `rss`：进程实际占用内存
- `cache`：缓存占用内存
- `oom_control`：OOM 状态

### **(2) 模拟 OOM 测试**

bash

复制

```
# 在容器内触发内存分配
docker exec -it <容器名> bash -c "tail /dev/zero | head -c 2G"
```

**观察是否被 OOM Killer 杀死**：

bash

复制

```
dmesg | grep -i "killed process"
```

### **(3) 调整内存限制**

bash

复制

```
# 限制内存为 1GB，并禁用 OOM Killer
docker update --memory="1g" --memory-swap="-1" --oom-kill-disable=true <容器名>
```

------

## **4. 高级工具**

### **(1) 使用 `perf` 分析 CPU 热点**

bash

复制

```
# 在宿主机采样容器进程
PID=$(docker inspect -f '{{.State.Pid}}' <容器名>)
perf top -p $PID
```

### **(2) 使用 `htop` 查看进程**

bash

复制

```
docker exec -it <容器名> htop
```

### **(3) 检查内核日志**

bash

复制

```
journalctl -xe | grep -i "oom\|kill"
dmesg | grep -i "docker"
```

------

## **5. 常见问题处理**

### **问题 1：CPU 100% 占用**

**可能原因**：

- 单线程应用卡死
- 死循环或高计算任务

**解决方案**：

1. 使用 `docker exec -it <容器名> top` 找到高 CPU 进程
2. 用 `perf` 或 `strace` 分析调用栈
3. 调整 `--cpus` 限制或优化代码

------

### **问题 2：容器被 OOM Killer 杀死**

**可能原因**：

- 内存泄漏
- 限制过低

**解决方案**：

1. 检查 `docker inspect` 内存限制
2. 用 `docker stats` 观察内存增长趋势
3. 调整 `--memory` 或修复内存泄漏

------

### **问题 3：CPU 争用导致性能下降**

**可能原因**：

- 多个容器竞争 CPU
- 未绑定 CPU 核心

**解决方案**：

1. 用 `docker stats` 检查各容器 CPU 使用
2. 通过 `--cpuset-cpus` 绑定核心
3. 调整 `--cpu-shares` 优先级

------

## **6. 附录：常用命令速查**

| **场景**         | **命令**                                                |
| :--------------- | :------------------------------------------------------ |
| 查看实时资源占用 | `docker stats <容器名>`                                 |
| 检查容器资源限制 | `docker inspect --format '...' <容器名>`                |
| 调整 CPU 限制    | `docker update --cpus="2" <容器名>`                     |
| 调整内存限制     | `docker update --memory="1g" <容器名>`                  |
| 禁用 OOM Killer  | `docker update --oom-kill-disable=true <容器名>`        |
| 分析高 CPU 进程  | `docker exec -it <容器名> top`                          |
| 检查 cgroup 内存 | `cat /sys/fs/cgroup/memory/docker/<容器ID>/memory.stat` |
| 查看 OOM 事件    | `dmesg | grep -i "killed process"`                      |

------

**总结**：本手册覆盖了从 **基础监控 → 高级分析 → 问题修复** 的全流程，适用于大多数 Docker 资源问题排查场景。🚀