# Docker容器磁盘占用监控处理手册

## 1. 快速查看命令

### 基本查看命令

bash

复制

```
# 查看运行中容器的磁盘占用（名称、ID、大小）
docker ps --size --format "table {{.Names}}\t{{.ID}}\t{{.Size}}"

# 查看所有容器（包括停止的）的磁盘占用
docker ps -a --size --format "table {{.Names}}\t{{.ID}}\t{{.Size}}"
```

### 详细磁盘分析

bash

复制

```
# 显示Docker磁盘使用概况
docker system df

# 显示详细存储使用情况（包括每个容器）
docker system df -v
```

## 2. 实时监控命令

### 单次快照模式

bash

复制

```
docker container stats --no-stream --format "table {{.Name}}\t{{.ID}}\t{{.MemUsage}}\t{{.BlockIO}}"
```

### 持续监控模式（实时刷新）

bash

复制

```
docker container stats --format "table {{.Name}}\t{{.ID}}\t{{.MemUsage}}\t{{.BlockIO}}"
```

(按Ctrl+C退出监控)

## 3. 高级分析脚本

### 容器磁盘使用详细分析脚本

bash

复制

```
#!/bin/bash
echo -e "容器名称\t容器ID\t磁盘占用"
docker ps -q | xargs docker inspect --format '{{.Name}} {{.Id}} {{.GraphDriver.Data.WorkDir}}' | while read name id path; do
    name=${name#/}  # 移除名称前的/
    size=$(sudo du -sh "$path" 2>/dev/null | cut -f1 || echo "N/A")
    echo -e "$name\t${id:0:12}\t$size"
done
```

使用方法：

1. 将上述内容保存为`docker-disk-usage.sh`
2. 添加执行权限：`chmod +x docker-disk-usage.sh`
3. 运行脚本：`./docker-disk-usage.sh`

## 4. 数据解读与处理建议

### 输出列说明

- **容器名称**：Docker容器的名称
- **容器ID**：容器的唯一标识符（通常显示前12字符）
- **磁盘占用**：容器使用的磁盘空间大小

### 常见问题处理

#### 磁盘占用过大处理步骤：

1. 识别大容器：

   bash

   复制

   ```
   docker ps -a --format "table {{.Names}}\t{{.ID}}\t{{.Size}}" | sort -k3 -h
   ```

2. 进入容器检查大文件：

   bash

   复制

   ```
   docker exec -it <容器ID> bash
   du -sh /  # 在容器内执行
   exit
   ```

3. 清理不需要的容器：

   bash

   复制

   ```
   docker rm <容器ID>  # 删除停止的容器
   docker system prune  # 清理所有无用对象（谨慎使用）
   ```

4. 清理容器日志：

   bash

   复制

   ```
   truncate -s 0 $(docker inspect --format='{{.LogPath}}' <容器ID>)
   ```

## 5. 定时监控方案

### 设置定时任务（每1小时记录一次）

bash

复制

```
(crontab -l 2>/dev/null; echo "0 * * * * docker ps -a --format 'table {{.Names}}\t{{.ID}}\t{{.Size}}' >> /var/log/docker_disk.log") | crontab -
```

### 日志轮转配置（可选）

在`/etc/logrotate.d/docker_disk`中添加：

复制

```
/var/log/docker_disk.log {
    weekly
    rotate 4
    compress
    missingok
    notifempty
}
```

## 6. 注意事项

1. 部分命令需要root权限才能获取完整信息
2. 磁盘占用统计可能有几分钟延迟
3. `docker system prune`会删除所有停止的容器、未使用的网络和悬空镜像（谨慎使用）
4. 生产环境中建议使用专门的监控工具（如Prometheus+Granfana）进行长期监控

## 7. 附录：常用清理命令

bash

复制

```
# 删除所有停止的容器
docker container prune

# 删除所有悬空镜像
docker image prune

# 删除所有未使用的镜像
docker image prune -a

# 全面清理（容器、网络、镜像）
docker system prune -a
```

------

本手册提供了从快速检查到深入分析的各种方法，可根据实际需求选择合适的命令。对于生产环境，建议将关键命令设置为定时任务并记录日志，以便长期监控磁盘使用趋势。