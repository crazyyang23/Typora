# **容器性能优化：cgroup 内核参数详细调优指南**

在容器环境中，即使 CPU 和内存使用率未达到限制值，配置资源限制仍可能导致性能下降。这通常与 **cgroup（控制组）的内核调度策略** 相关。本指南详细说明如何调整关键的 **cgroup 内核参数**，以优化容器性能。

------

## **1. CPU 子系统调优**

### **关键参数路径**

bash



复制



下载

```
/sys/fs/cgroup/cpu/<cgroup>/
/sys/fs/cgroup/cpuacct/<cgroup>/
```

### **1.1 `cpu.cfs_period_us` & `cpu.cfs_quota_us`**

- **作用**：控制 CPU 时间分配（CFS 调度器）。

- **默认值**：

  - `cpu.cfs_period_us` = 100000（100ms）
  - `cpu.cfs_quota_us` = -1（无限制）

- **优化建议**：

  - **缩短周期（提高响应性）**：

    bash

    

    复制

    

    下载

    ```
    echo 50000 > /sys/fs/cgroup/cpu/<cgroup>/cpu.cfs_period_us  # 50ms
    echo 25000 > /sys/fs/cgroup/cpu/<cgroup>/cpu.cfs_quota_us   # 50% CPU
    ```

  - **放宽限制（减少调度延迟）**：

    bash

    

    复制

    

    下载

    ```
    echo 100000 > /sys/fs/cgroup/cpu/<cgroup>/cpu.cfs_period_us
    echo 80000 > /sys/fs/cgroup/cpu/<cgroup>/cpu.cfs_quota_us   # 80% CPU
    ```

  - **完全解除限制（测试用）**：

    bash

    

    复制

    

    下载

    ```
    echo -1 > /sys/fs/cgroup/cpu/<cgroup>/cpu.cfs_quota_us
    ```

### **1.2 `cpu.shares`**

- **作用**：在 CPU 资源竞争时，控制权重（相对值）。

- **默认值**：1024

- **优化建议**：

  - 提高关键容器的权重（如 2048）：

    bash

    

    复制

    

    下载

    ```
    echo 2048 > /sys/fs/cgroup/cpu/<cgroup>/cpu.shares
    ```

  - 降低低优先级容器的权重（如 512）：

    bash

    

    复制

    

    下载

    ```
    echo 512 > /sys/fs/cgroup/cpu/<cgroup>/cpu.shares
    ```

### **1.3 `cpu.rt_runtime_us`（实时任务调度）**

- **作用**：控制实时任务（RT）的 CPU 时间。

- **默认值**：950000（950ms）

- **优化建议**：

  - 如果容器运行实时任务（如音视频处理），可适当增加：

    bash

    

    复制

    

    下载

    ```
    echo 1000000 > /sys/fs/cgroup/cpu/<cgroup>/cpu.rt_runtime_us  # 1s
    ```

------

## **2. 内存子系统调优**

### **关键参数路径**

bash



复制



下载

```
/sys/fs/cgroup/memory/<cgroup>/
```

### **2.1 `memory.swappiness`**

- **作用**：控制内核使用 swap 的倾向（0-100）。

- **默认值**：60

- **优化建议**：

  - **降低 swap 使用（减少 I/O 延迟）**：

    bash

    

    复制

    

    下载

    ```
    echo 10 > /sys/fs/cgroup/memory/<cgroup>/memory.swappiness
    ```

  - **完全禁用 swap（适用于内存密集型应用）**：

    bash

    

    复制

    

    下载

    ```
    echo 0 > /sys/fs/cgroup/memory/<cgroup>/memory.swappiness
    ```

### **2.2 `memory.oom_control`**

- **作用**：控制 OOM Killer 行为。

- **默认值**：0（启用 OOM Killer）

- **优化建议**：

  - **禁用 OOM Killer（防止关键容器被杀死）**：

    bash

    

    复制

    

    下载

    ```
    echo 1 > /sys/fs/cgroup/memory/<cgroup>/memory.oom_control
    ```

### **2.3 `memory.use_hierarchy`**

- **作用**：是否启用层级统计（影响内存回收策略）。

- **默认值**：1（启用）

- **优化建议**：

  - **禁用层级统计（减少开销）**：

    bash

    

    复制

    

    下载

    ```
    echo 0 > /sys/fs/cgroup/memory/<cgroup>/memory.use_hierarchy
    ```

------

## **3. I/O 子系统调优**

### **关键参数路径**

bash



复制



下载

```
/sys/fs/cgroup/blkio/<cgroup>/
```

### **3.1 `blkio.weight`**

- **作用**：控制块设备 I/O 权重（100-1000）。

- **默认值**：500

- **优化建议**：

  - **提高 I/O 优先级（适用于数据库容器）**：

    bash

    

    复制

    

    下载

    ```
    echo 800 > /sys/fs/cgroup/blkio/<cgroup>/blkio.weight
    ```

### **3.2 `blkio.throttle.read_bps_device` & `blkio.throttle.write_bps_device`**

- **作用**：限制设备的读写带宽（单位：bytes/sec）。

- **默认值**：无限制

- **优化建议**：

  - **解除不必要的限制**：

    bash

    

    复制

    

    下载

    ```
    echo "8:0 0" > /sys/fs/cgroup/blkio/<cgroup>/blkio.throttle.read_bps_device
    echo "8:0 0" > /sys/fs/cgroup/blkio/<cgroup>/blkio.throttle.write_bps_device
    ```

  - **限制低优先级容器的 I/O**：

    bash

    

    复制

    

    下载

    ```
    echo "8:0 1048576" > /sys/fs/cgroup/blkio/<cgroup>/blkio.throttle.read_bps_device  # 1MB/s
    ```

------

## **4. 系统全局调优**

### **关键参数路径**

bash



复制



下载

```
/proc/sys/vm/
/proc/sys/kernel/
```

### **4.1 `vm.dirty_ratio` & `vm.dirty_background_ratio`**

- **作用**：控制脏页（未写入磁盘的数据）比例。

- **默认值**：

  - `vm.dirty_ratio` = 20（系统内存的 20%）
  - `vm.dirty_background_ratio` = 10（后台刷脏页阈值）

- **优化建议**：

  - **减少 I/O 突发（适用于数据库/高吞吐应用）**：

    bash

    

    复制

    

    下载

    ```
    echo 10 > /proc/sys/vm/dirty_ratio
    echo 5 > /proc/sys/vm/dirty_background_ratio
    ```

### **4.2 `vm.swappiness`**

- **作用**：全局控制 swap 使用倾向。

- **默认值**：60

- **优化建议**：

  - **降低 swap 使用**：

    bash

    

    复制

    

    下载

    ```
    echo 10 > /proc/sys/vm/swappiness
    ```

### **4.3 `kernel.sched_migration_cost_ns`**

- **作用**：控制任务迁移成本（减少不必要的 CPU 核心切换）。

- **默认值**：500000（0.5ms）

- **优化建议**：

  - **增加迁移成本（减少上下文切换）**：

    bash

    

    复制

    

    下载

    ```
    echo 1000000 > /proc/sys/kernel/sched_migration_cost_ns  # 1ms
    ```

------

## **5. 调优实施建议**

1. **基准测试**：

   - 使用 `sysbench`、`stress-ng` 测试调整前后的性能差异。

   - 监控工具：

     bash

     

     复制

     

     下载

     ```
     cat /proc/pressure/cpu  # CPU 压力
     cat /proc/pressure/memory  # 内存压力
     systemd-cgtop  # cgroup 资源使用情况
     ```

2. **容器运行时调整**：

   - **Docker**：

     bash

     

     复制

     

     下载

     ```
     docker run --cpu-shares=2048 --memory-swappiness=10 ...
     ```

   - **Kubernetes**：

     yaml

     

     复制

     

     下载

     ```
     resources:
       limits:
         cpu: "2"
         memory: "4Gi"
       requests:
         cpu: "1"
         memory: "2Gi"
     ```

3. **注意事项**：

   - **不要过度调优**，某些参数可能影响系统稳定性。
   - **生产环境先在测试集群验证**。
   - **不同内核版本参数可能不同**，参考 `man cgroups` 和内核文档。

------

## **总结**

通过调整 **cgroup 内核参数**，可以显著优化容器性能，即使未达到资源限制。关键方向包括：
✅ **CPU 调度优化**（`cpu.cfs_period_us`, `cpu.shares`）
✅ **内存回收策略**（`memory.swappiness`, `oom_control`）
✅ **I/O 优先级**（`blkio.weight`, `throttle`）
✅ **系统全局调优**（`vm.swappiness`, `sched_migration_cost_ns`）

建议结合监控工具逐步调整，找到最适合业务场景的配置。