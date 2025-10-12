# **Docker 容器存储目录排查操作手册**

**版本：1.0**
**适用场景**：当发现 `/var/lib/docker/overlay2/` 下某个目录（如 `c438dddf...`）占用过大，需确认其所属容器时使用。

------

## **1. 问题描述**

在 `/var/lib/docker/overlay2/` 中发现某个目录（如 `c438dddf56f1b1eddbaaa86064895104568dd27ac9594edbf403412bb7434c69`）占用大量磁盘空间，需确认该目录属于哪个容器。

------

## **2. 操作步骤**

### **方法1：通过 `docker inspect` 直接关联容器**

1. **安装 `jq` 工具（可选）**

   bash

   复制

   ```
   sudo apt install jq  # Debian/Ubuntu
   sudo yum install jq  # CentOS/RHEL
   ```

2. **查找匹配的容器ID**

   bash

   复制

   ```
   docker inspect $(docker ps -aq) | jq -r '.[] | select(.GraphDriver.Data.MergedDir | contains("c438dddf56f1b1eddbaaa86064895104568dd27ac9594edbf403412bb7434c69")) | .Id'
   ```

   - 如果输出结果为空，说明该目录可能属于已删除的容器。

3. **查看容器详情**

   bash

   复制

   ```
   docker inspect <容器ID>
   ```

------

### **方法2：对比 `overlay2` 目录名与容器ID**

1. **显示所有容器的完整ID**

   bash

   复制

   ```
   docker ps -a --no-trunc
   ```

   - 观察 `CONTAINER ID` 列，检查是否有与 `c438dddf...` 前缀匹配的ID。

2. **如果找到匹配的容器**

   bash

   复制

   ```
   docker inspect <容器ID> | grep -A 5 "GraphDriver"
   ```

   - 确认 `MergedDir` 或 `UpperDir` 是否包含目标 `overlay2` 目录。

------

### **方法3：遍历所有容器存储占用**

1. **列出所有容器及其存储占用**

   bash

   复制

   ```
   docker ps -q | xargs docker inspect --format '{{.Id}} {{.Name}}' | while read id name; do
     size=$(sudo du -sh "/var/lib/docker/overlay2/$id" 2>/dev/null | awk '{print $1}');
     echo "$name ($id): $size";
   done | grep -v "0B"
   ```

   - 在输出中查找目标目录名或占用异常的容器。

------

### **方法4：检查已删除容器的残留数据**

如果目录无关联容器，可能是残留数据：

1. **查看所有 Docker 存储层**

   bash

   复制

   ```
   sudo ls -lh /var/lib/docker/overlay2/
   ```

2. **清理无用数据**

   bash

   复制

   ```
   docker system prune -a  # 清理未使用的镜像、容器、卷
   ```

   - **谨慎操作**：此命令会删除所有未被引用的数据。

------

## **3. 常见问题**

### **Q1：`docker inspect` 找不到容器，但目录仍存在？**

- 可能是容器已删除但数据未清理，尝试方法4。

### **Q2：如何防止 `/var/lib/docker` 占用过大？**

- 定期清理无用容器和镜像：

  bash

  复制

  ```
  docker system df      # 查看存储使用情况
  docker system prune   # 清理临时文件
  ```

### **Q3：`overlay2` 目录名与容器ID无关？**

- Docker 的存储驱动可能会生成随机哈希目录名，需通过 `docker inspect` 或存储驱动元数据关联。

------

## **4. 附录**

### **命令速查表**

| 用途             | 命令                                                         |
| :--------------- | :----------------------------------------------------------- |
| 查找目录关联容器 | `docker inspect $(docker ps -aq) | jq -r '.[] | select(.GraphDriver.Data.MergedDir | contains("DIR_NAME")) | .Id'` |
| 查看容器存储占用 | `docker ps -q | xargs docker inspect --format '{{.Id}} {{.Name}}' | while read id name; do size=$(sudo du -sh "/var/lib/docker/overlay2/$id" 2>/dev/null | awk '{print $1}'); echo "$name ($id): $size"; done` |
| 清理无用数据     | `docker system prune -a`                                     |

------

**维护建议**：定期监控 `/var/lib/docker` 目录大小，避免单个容器占用过多磁盘空间。



要使用 `du` 命令查看根目录下所有文件和目录的大小，并且只显示占用大于1G的部分，可以结合 `du`、`sort` 和 `awk` 命令来实现。以下是几种方法：

### 方法1：使用 `du` + `awk` 过滤

bash

复制

```
du -h --max-depth=1 / 2>/dev/null | awk '$1 ~ /G/ && $1+0 > 1 {print}'
```

- `du -h --max-depth=1 /`：查看根目录下第一级子目录和文件的大小（人类可读格式）。
- `2>/dev/null`：忽略权限拒绝等错误信息。
- `awk '$1 ~ /G/ && $1+0 > 1 {print}'`：筛选出以 `G` 为单位且数值大于1的行。

### 方法2：直接使用 `du` 的 `--threshold` 选项（部分系统支持）

bash

复制

```
du -h --threshold=1G --max-depth=1 / 2>/dev/null
```

- `--threshold=1G`：只显示大于等于1G的项目（部分较新版本的 `du` 支持）。

### 方法3：使用 `find` + `du`（更精确）

bash

复制

```
find / -maxdepth 1 -type d -exec du -h {} + 2>/dev/null | awk '$1 ~ /G/ && $1+0 > 1 {print}'
```

- `find / -maxdepth 1 -type d`：查找根目录下的所有一级目录。
- `du -h {} +`：计算这些目录的大小。

### 注意事项

1. 需要 `root` 权限才能无错误地扫描所有目录（如 `/sys`、`/proc` 等会报权限拒绝）。

   bash

   复制

   ```
   sudo du -h --max-depth=1 / 2>/dev/null | awk '$1 ~ /G/ && $1+0 > 1 {print}'
   ```

2. 如果 `awk` 不兼容，可以用 `grep` 简单过滤（但精度较低）：

   bash

   复制

   ```
   du -h --max-depth=1 / 2>/dev/null | grep -E '^[0-9\.]+G'
   ```