Redis 的 **内存淘汰策略（Eviction Policy）** 用于当内存使用达到 `maxmemory` 限制时，决定哪些键会被自动删除以释放内存。你提到的 `allkeys-lru` 是其中一种常用策略，下面详细说明：

### **1. allkeys-lru 策略的含义**

- **LRU（Least Recently Used）**：最近最少使用。

- **allkeys-lru**：从 **所有键** 中，优先删除 **最近最少被访问过** 的键。

  无论键是否设置了过期时间，只要内存达到上限，就会按照 “最近最少使用” 的规则淘汰键。

### **2. 如何配置 allkeys-lru**

需要在 Redis 配置文件（`redis.conf`）中或通过命令行设置，具体步骤：

#### **方法 1：修改 redis.conf 配置文件**

```
# 基础配置
bind 0.0.0.0
protected-mode no
port 6379
tcp-backlog 511
timeout 300
tcp-keepalive 300

# 内存管理
maxmemory 2gb
maxmemory-policy allkeys-lru
maxmemory-samples 5
maxclients 10000

# 持久化优化 (根据业务需求选择)
save ""  # 禁用RDB快照

# 性能优化
daemonize no
supervised no
lazyfree-lazy-eviction yes
lazyfree-lazy-expire yes
lazyfree-lazy-server-del yes
replica-lazy-flush yes

# 内核参数
oom-score-adj no
disable-thp yes

# 高级网络配置
repl-backlog-size 256mb
repl-backlog-ttl 3600

# 多线程配置 (Redis 6+)
io-threads 4
io-threads-do-reads yes

# 慢查询日志
slowlog-log-slower-than 10000
slowlog-max-len 128

# 客户端限制
client-output-buffer-limit normal 0 0 0
client-output-buffer-limit replica 512mb 128mb 300
client-output-buffer-limit pubsub 32mb 8mb 60

# 监控
slowlog-log-slower-than 10000
slowlog-max-len 128
latency-monitor-threshold 100
```

找到 `maxmemory-policy` 配置项，设置为：



```conf
maxmemory 512mb  # 先设置最大内存限制（必须配置，否则淘汰策略不生效）
maxmemory-policy allkeys-lru  # 设置淘汰策略为 allkeys-lru
```

#### **方法 2：通过 redis-cli 动态生效（无需重启）**

bash

```bash
# 先设置最大内存（例如 512MB）
127.0.0.1:6379> config set maxmemory 512mb

# 再设置淘汰策略为 allkeys-lru
127.0.0.1:6379> config set maxmemory-policy allkeys-lru
```

### **3. 验证配置是否生效**

设置后可以通过以下命令查看当前策略：



```bash
127.0.0.1:6379> config get maxmemory-policy
1) "maxmemory-policy"
2) "allkeys-lru"  # 显示此结果说明配置成功
```



同时确认 `maxmemory` 已正确设置（不为 0）：



```bash
127.0.0.1:6379> config get maxmemory
1) "maxmemory"
2) "536870912"  # 512MB 的字节数（512*1024*1024=536870912）
```

### **4. 其他常见淘汰策略对比**

除了 `allkeys-lru`，Redis 还有其他策略，可根据业务场景选择：



| 策略              | 说明                                                         |
| ----------------- | ------------------------------------------------------------ |
| `volatile-lru`    | 仅从 **设置了过期时间的键** 中，淘汰最近最少使用的键         |
| `allkeys-random`  | 从所有键中 **随机淘汰** 键                                   |
| `volatile-random` | 从设置了过期时间的键中随机淘汰                               |
| `volatile-ttl`    | 从设置了过期时间的键中，优先淘汰 **剩余生存时间（TTL）最短** 的键 |
| `noeviction`      | （默认策略）不淘汰任何键，内存满时拒绝新写入操作（可能导致错误） |

### **注意事项**

1. **`maxmemory` 必须设置**：淘汰策略仅在 `maxmemory` 不为 0 时生效（即限制了最大内存）。
2. **LRU 近似实现**：Redis 的 LRU 并非严格意义上的 “最近最少使用”，而是通过采样（默认采样 5 个键）来近似，平衡性能和准确性（可通过 `maxmemory-samples` 调整采样数）。
3. **业务适配**：`allkeys-lru` 适合 **所有键优先级相同** 的场景（如缓存通用数据）；如果部分键需要长期保留，可使用 `volatile-lru` 并为重要键不设置过期时间。



如果需要进一步调整或验证，可通过 `config` 命令动态修改并观察内存使用情况。