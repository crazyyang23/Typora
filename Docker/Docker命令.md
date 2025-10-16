在容器化环境中，维护和排查问题是日常运维的重要部分。本文档整理了常用的 Docker 命令，帮助您快速定位问题、分析资源使用情况、查看容器配置等。

------

## 1. 查看容器占用磁盘空间排序

查看容器占用的磁盘空间并按大小排序：

bash

复制

```
docker system df -v
```

查看单个容器的磁盘使用情况：

bash

复制

```
docker ps -s
```

------

## 2. 查看容器内存占用率

通过容器IP，查找对应的容器名称

```
# 替换 <目标IP> 为你要查询的容器IP（如 172.18.0.66）
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}} {{end}} {{.Name}}' $(docker ps -q) | grep "<目标IP>"
```

bash

复制

```
docker stats
```

查看特定容器的资源使用情况：

bash

复制

```
docker stats <container_id_or_name>
```

------

## 3. 查看容器 IP 地址

查看容器的 IP 地址：

bash

复制

```
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' <container_id_or_name>
```

------

## 4. 查看容器日志

查看容器日志：

bash

复制

```
docker logs <container_id_or_name>
```

实时查看日志输出：

bash

复制

```
docker logs -f <container_id_or_name>
```

------

## 5. 进入容器内部进行调试

进入容器内部：

bash

复制

```
docker exec -it <container_id_or_name> /bin/bash
```

如果容器内没有 `bash`，可以使用 `sh`：

bash

复制

```
docker exec -it <container_id_or_name> /bin/sh
```

------

## 6. 查看容器的资源限制

查看容器的 CPU 和内存限制：

bash

复制

```
docker inspect <container_id_or_name> | grep -i "memory\|cpu"
```

------

## 7. 查看容器的网络配置

查看容器的网络配置：

bash

复制

```
docker inspect <container_id_or_name> | grep -i "network"
```

------

## 8. 查看容器的启动命令

查看容器的启动命令：

bash

复制

```
docker inspect --format='{{.Config.Cmd}}' <container_id_or_name>
```

------

## 9. 查看容器的端口映射

查看容器的端口映射：

bash

复制

```
docker port <container_id_or_name>
```

------

## 10. 查看容器的环境变量

查看容器的环境变量：

bash

复制

```
docker inspect --format='{{.Config.Env}}' <container_id_or_name>
```

------

## 11. 查看容器的挂载卷

查看容器的挂载卷：

bash

复制

```
docker inspect --format='{{.Mounts}}' <container_id_or_name>
```

------

## 12. 查看容器的健康状态

查看容器的健康状态（需配置健康检查）：

bash

复制

```
docker inspect --format='{{.State.Health.Status}}' <container_id_or_name>
```

------

## 13. 查看容器的启动时间

查看容器的启动时间：

bash

复制

```
docker inspect --format='{{.State.StartedAt}}' <container_id_or_name>
```

------

## 14. 查看容器的退出状态

查看已停止容器的退出状态：

bash

复制

```
docker inspect --format='{{.State.ExitCode}}' <container_id_or_name>
```

------

## 15. 查看容器的资源使用历史

查看容器的资源使用历史：

bash

复制

```
docker stats --no-stream <container_id_or_name>
```

------

## 16. 查看容器的网络连接

查看容器内部的网络连接情况：

bash

复制

```
docker exec -it <container_id_or_name> netstat -tuln
```

或使用 `ss` 命令：

bash

复制

```
docker exec -it <container_id_or_name> ss -tuln
```

------

## 17. 查看容器的进程

查看容器内运行的进程：

bash

复制

```
docker top <container_id_or_name>
```

------

## 18. 查看容器的元数据

查看容器的所有元数据：

bash

复制

```
docker inspect <container_id_or_name>
```

------

## 19. 查看容器的镜像层

查看容器使用的镜像层：

bash

复制

```
docker history <image_name>
```

------

## 20. 查看容器的文件系统变化

查看容器文件系统的变化：

bash

复制

```
docker diff <container_id_or_name>
```

## 21. 查看容器占用磁盘空间排序



```
docker stats --no-stream --format "table {{.Name}}\t{{.Container}}\t{{.CPUPerc}}\t{{.MemUsage}}" | sort -k 4 -h
```

## 21. 容器占用磁盘空间排序



```
 docker ps -s --format "{{.ID}}\t{{.Names}}\t{{.Size}}" | sort -hrk 2
```



## 总结

本文档整理了 Docker 容器维护和问题排查中的常用命令，涵盖了资源监控、日志查看、网络配置、容器调试等方面。根据具体问题选择合适的命令进行排查和分析，能够有效提升运维效率。

------

**备注**：

- 替换 `<container_id_or_name>` 为实际的容器 ID 或名称。
- 部分命令可能需要管理员权限（如 `sudo`）。