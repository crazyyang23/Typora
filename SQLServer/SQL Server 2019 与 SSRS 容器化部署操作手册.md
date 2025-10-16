## 一、环境准备

### 1.1 系统要求

- **操作系统**：Linux (推荐 Ubuntu 20.04+) 或 Windows Server 2019+

- 硬件配置

  ：

  - CPU：至少 4 核
  - 内存：至少 16GB
  - 存储：至少 100GB 可用空间

- **网络**：开放 80、443、1433 端口

### 1.2 软件安装

1. **Docker Engine**

   ```bash
   # Ubuntu安装命令
   sudo apt-get update
   sudo apt-get install docker-ce docker-ce-cli containerd.io
   sudo systemctl enable docker
   sudo systemctl start docker
   ```

2. **Docker Compose**

   ```bash
   sudo curl -L "https://github.com/docker/compose/releases/download/v2.18.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
   sudo chmod +x /usr/local/bin/docker-compose
   ```

3. **验证安装**

   ```bash
   docker --version
   docker-compose --version
   ```

## 二、部署步骤

### 2.1 创建工作目录

```bash
mkdir sqlserver-ssrs
cd sqlserver-ssrs
```

### 2.2 编写 docker-compose.yml

创建文件`docker-compose.yml`，内容如下：

```yaml
version: '3.9'

services:
  sqlserver:
    image: mcr.microsoft.com/mssql/server:2019-latest
    container_name: sqlserver2019
    environment:
      - SA_PASSWORD=YourStrong!Passw0rd
      - ACCEPT_EULA=Y
      - MSSQL_PID=Enterprise
    ports:
      - "1433:1433"
    volumes:
      - mssql-data:/var/opt/mssql
      - mssql-backup:/var/opt/mssql/backup
    restart: always

  ssrs:
    image: mcr.microsoft.com/mssql/reporting-services:2019-latest
    container_name: ssrs2019
    environment:
      - SQLSERVER_HOST=sqlserver
      - SQLSERVER_USER=sa
      - SQLSERVER_PASSWORD=YourStrong!Passw0rd
      - SQLSERVER_DB=ReportServer
      - SQLSERVER_REPORT_DB=ReportServerTempDB
    ports:
      - "80:80"
      - "443:443"
    depends_on:
      - sqlserver
    volumes:
      - ssrs-data:/var/opt/mssql-reporting-services
    restart: always

volumes:
  mssql-data:
  mssql-backup:
  ssrs-data:
```

### 2.3 启动容器

```bash
docker-compose up -d
```

### 2.4 验证部署

1. 查看容器状态

   ```bash
   docker ps
   ```

   应看到两个运行中的容器：`sqlserver2019`和`ssrs2019`

2. 检查日志

   ```bash
   docker logs sqlserver2019
   docker logs ssrs2019
   ```

## 三、配置与访问

### 3.1 初始配置

1. **修改 SA 密码**

   ```bash
   docker exec -it sqlserver2019 /opt/mssql-tools/bin/sqlcmd -S localhost -U SA -P 'YourStrong!Passw0rd' -Q "ALTER LOGIN sa WITH PASSWORD='NewStrong!Passw0rd'"
   ```

2. **更新 docker-compose.yml**
   将`SA_PASSWORD`和`SQLSERVER_PASSWORD`修改为新密码

### 3.2 访问 SSRS

1. **报表管理器**
   访问：`http://localhost/reports`
2. **报表服务器配置管理器**
   访问：`http://localhost/ReportServer`

## 四、数据管理

### 4.1 数据持久化机制

- **SQL Server 数据**：存储在 Docker 卷`mssql-data`中
- **SSRS 配置**：存储在 Docker 卷`ssrs-data`中
- **备份文件**：存储在 Docker 卷`mssql-backup`中

### 4.2 手动备份数据库

```bash
# 创建数据库备份
docker exec -it sqlserver2019 /opt/mssql-tools/bin/sqlcmd -S localhost -U SA -P 'YourStrong!Passw0rd' -Q "BACKUP DATABASE ReportServer TO DISK='/var/opt/mssql/backup/ReportServer.bak'"

# 从容器复制备份到主机
docker cp sqlserver2019:/var/opt/mssql/backup/ReportServer.bak ./
```

### 4.3 恢复数据库

```bash
# 复制备份文件到容器
docker cp ReportServer.bak sqlserver2019:/var/opt/mssql/backup/

# 执行恢复命令
docker exec -it sqlserver2019 /opt/mssql-tools/bin/sqlcmd -S localhost -U SA -P 'YourStrong!Passw0rd' -Q "RESTORE DATABASE ReportServer FROM DISK='/var/opt/mssql/backup/ReportServer.bak' WITH REPLACE"
```

## 五、日常维护

### 5.1 服务重启

```bash
# 停止服务
docker-compose down

# 启动服务
docker-compose up -d
```

### 5.2 服务更新

```bash
# 拉取最新镜像
docker-compose pull

# 重启服务
docker-compose up -d
```

### 5.3 查看日志

```bash
# 实时查看日志
docker-compose logs -f
```

## 六、故障排除

### 6.1 常见问题

1. **SSRS 无法连接 SQL Server**
   - 检查 SQL Server 容器是否正常运行
   - 验证`SQLSERVER_PASSWORD`是否正确
   - 检查网络连接：`docker exec -it ssrs2019 ping sqlserver`
2. **端口冲突**
   - 修改`docker-compose.yml`中的端口映射
   - 检查主机上的端口占用情况：`netstat -tulpn | grep :1433`
3. **磁盘空间不足**
   - 清理无用容器和镜像：`docker system prune -a`
   - 扩展存储卷或迁移数据

## 七、YAML 配置详解

### 7.1 SQL Server 服务配置

```yaml
sqlserver:
  image: mcr.microsoft.com/mssql/server:2019-latest  # 使用官方SQL Server 2019镜像
  container_name: sqlserver2019  # 容器名称
  environment:  # 环境变量配置
    - SA_PASSWORD=YourStrong!Passw0rd  # SA账户密码
    - ACCEPT_EULA=Y  # 接受许可协议
    - MSSQL_PID=Enterprise  # 版本为企业版
  ports:  # 端口映射
    - "1433:1433"  # 主机1433端口映射到容器1433端口
  volumes:  # 数据卷挂载
    - mssql-data:/var/opt/mssql  # 数据目录
    - mssql-backup:/var/opt/mssql/backup  # 备份目录
  restart: always  # 自动重启策略
```

### 7.2 SSRS 服务配置

```yaml
ssrs:
  image: mcr.microsoft.com/mssql/reporting-services:2019-latest  # 官方SSRS 2019镜像
  container_name: ssrs2019  # 容器名称
  environment:  # 环境变量配置
    - SQLSERVER_HOST=sqlserver  # SQL Server主机名
    - SQLSERVER_USER=sa  # 数据库用户名
    - SQLSERVER_PASSWORD=YourStrong!Passw0rd  # 数据库密码
    - SQLSERVER_DB=ReportServer  # 报表主数据库
    - SQLSERVER_REPORT_DB=ReportServerTempDB  # 报表临时数据库
  ports:  # 端口映射
    - "80:80"  # HTTP端口
    - "443:443"  # HTTPS端口
  depends_on:  # 依赖关系
    - sqlserver  # 依赖SQL Server服务
  volumes:  # 数据卷挂载
    - ssrs-data:/var/opt/mssql-reporting-services  # SSRS配置和数据
  restart: always  # 自动重启策略
```

### 7.3 数据卷配置

```yaml
volumes:
  mssql-data:  # SQL Server数据卷
  mssql-backup:  # SQL Server备份卷
  ssrs-data:  # SSRS数据卷
```





## 八、安全注意事项

1. **密码管理**
   - 使用强密码并定期更换
   - 不要在版本控制系统中存储明文密码
2. **网络安全**
   - 配置防火墙限制对 1433、80、443 端口的访问
   - 仅允许授权 IP 访问服务器
3. **定期备份**
   - 建立每日备份策略
   - 测试备份恢复流程
4. **更新管理**
   - 定期更新 Docker 镜像
   - 关注官方安全公告