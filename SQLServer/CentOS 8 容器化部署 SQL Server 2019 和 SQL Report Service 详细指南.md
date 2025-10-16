### CentOS 8 容器化部署 SQL Server 2019 和 SQL Report Service 详细指南

以下是完整的部署步骤，包含环境准备、依赖安装、容器配置和服务验证等关键环节：

### 一、环境准备与系统配置

#### 1. 系统要求检查

bash











```bash
# 检查内核版本 (需 >= 3.10)
uname -r

# 检查内存 (至少 2GB)
free -h

# 关闭SELinux (临时)
setenforce 0

# 永久关闭SELinux
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config

# 关闭防火墙 (测试环境)
systemctl stop firewalld
systemctl disable firewalld

# 或配置防火墙规则 (生产环境)
firewall-cmd --zone=public --add-port=1433/tcp --permanent
firewall-cmd --zone=public --add-port=8080/tcp --permanent
firewall-cmd --zone=public --add-port=8081/tcp --permanent
firewall-cmd --reload
```

#### 2. 安装 Docker 和 Docker Compose

bash











```bash
# 添加 Docker 仓库
sudo dnf config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo

# 安装 Docker CE
sudo dnf install -y docker-ce docker-ce-cli containerd.io

# 启动并设置开机自启
sudo systemctl start docker
sudo systemctl enable docker

# 验证安装
docker --version

# 安装 Docker Compose
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose

# 验证安装
docker-compose --version
```

### 二、创建数据持久化目录

bash











```bash
# 创建挂载目录
sudo mkdir -p /mssql/data
sudo mkdir -p /mssql/log
sudo mkdir -p /mssql/secrets
sudo mkdir -p /mssql/report_data

# 设置权限
sudo chmod -R 777 /mssql

# 查看目录结构
tree /mssql
```

### 三、配置 Docker Compose 文件

在 `/mssql` 目录下创建 `docker-compose.yml`：



yaml











```yaml
version: '3.7'

services:
  sqlserver:
    image: mcr.microsoft.com/mssql/server:2019-latest
    container_name: sqlserver
    ports:
      - "1433:1433"
    environment:
      - ACCEPT_EULA=Y
      - SA_PASSWORD=YourStrong@Passw0rd  # 请修改为强密码
      - MSSQL_PID=Enterprise
      - MSSQL_TCP_PORT=1433
    volumes:
      - /mssql/data:/var/opt/mssql/data
      - /mssql/log:/var/opt/mssql/log
      - /mssql/secrets:/var/opt/mssql/secrets
    restart: always
    healthcheck:
      test: ["CMD", "/opt/mssql-tools/bin/sqlcmd", "-S", "localhost", "-U", "sa", "-P", "YourStrong@Passw0rd", "-Q", "SELECT 1", "||", "exit 1"]
      interval: 10s
      retries: 5
      start_period: 30s

  reportservice:
    image: mcr.microsoft.com/mssql/reporting-services:2019-latest
    container_name: reportservice
    ports:
      - "8080:80"   # HTTP 访问
      - "8081:443"  # HTTPS 访问
    environment:
      - SQL_SERVER=sqlserver
      - SQL_USER=sa
      - SQL_PASSWORD=YourStrong@Passw0rd  # 与 SQL Server 密码一致
      - SQL_DB=ReportServer
      - SQL_REPORTING_DB=ReportServerTempDB
      - ACCEPT_EULA=Y
    volumes:
      - /mssql/report_data:/var/opt/mssql-reporting-services
    depends_on:
      sqlserver:
        condition: service_healthy
    restart: always
```

### 四、启动并验证容器

bash











```bash
# 进入工作目录
cd /mssql

# 启动容器 (后台运行)
docker-compose up -d

# 查看容器状态
docker ps

# 查看容器日志
docker logs sqlserver
docker logs reportservice

# 验证 SQL Server 连接
docker exec -it sqlserver /opt/mssql-tools/bin/sqlcmd -S localhost -U SA -P 'YourStrong@Passw0rd' -Q 'SELECT @@VERSION'

# 输出示例:
# Microsoft SQL Server 2019 (RTM-CU18) (KB5021522) - 15.0.4261.1 (X64)
#	May 11 2023 16:42:56
#	Copyright (C) 2019 Microsoft Corporation
#	Enterprise Edition (64-bit) on Linux (CentOS Linux 8.5.2111) <X64>
```

### 五、配置 SQL Server Reporting Services

#### 1. 访问 Reporting Services Web 门户

打开浏览器访问：



- **报表管理器**：`http://<服务器IP>:8080/Reports`
- **配置管理器**：`http://<服务器IP>:8080/Reports`

#### 2. 初始配置步骤

1. 首次访问时，系统会提示配置报表服务器数据库
2. 选择 "创建新的报表服务器数据库"
3. 输入 SQL Server 连接信息：
   - 服务器名称：`sqlserver`
   - 身份验证：SQL Server 身份验证
   - 用户名：`sa`
   - 密码：`YourStrong@Passw0rd`
4. 完成数据库创建向导
5. 验证配置成功后，进入报表管理器

#### 3. 创建示例报表

1. 点击 "新建数据源"
2. 配置连接信息指向本地 SQL Server
3. 点击 "新建报表"
4. 使用报表生成器创建简单报表
5. 发布报表到服务器

### 六、验证数据持久性

bash











```bash
# 测试重启容器后数据是否保留
docker-compose restart

# 重新连接 SQL Server 查看数据
docker exec -it sqlserver /opt/mssql-tools/bin/sqlcmd -S localhost -U SA -P 'YourStrong@Passw0rd' -Q 'CREATE DATABASE TestDB; SELECT name FROM sys.databases;'

# 重启整个系统
sudo reboot

# 系统重启后验证容器自动启动
docker ps

# 再次验证数据库是否存在
docker exec -it sqlserver /opt/mssql-tools/bin/sqlcmd -S localhost -U SA -P 'YourStrong@Passw0rd' -Q 'SELECT name FROM sys.databases;'
```

### 七、日常管理与维护

#### 1. 查看容器日志

bash











```bash
docker logs sqlserver
docker logs reportservice
```

#### 2. 停止 / 启动服务

bash











```bash
docker-compose stop
docker-compose start
```

#### 3. 更新服务

bash











```bash
# 拉取最新镜像
docker-compose pull

# 重新创建容器
docker-compose up -d
```

#### 4. 备份数据

bash











```bash
# 备份 SQL Server 数据
docker exec -it sqlserver /opt/mssql-tools/bin/sqlcmd -S localhost -U SA -P 'YourStrong@Passw0rd' -Q 'BACKUP DATABASE TestDB TO DISK = "/var/opt/mssql/data/TestDB.bak"'

# 物理备份数据目录 (建议定期执行)
tar -czvf mssql_backup_$(date +%Y%m%d).tar.gz /mssql
```

### 八、故障排除

#### 常见问题及解决方案：

1. **SQL Server 无法启动**
   - 检查密码复杂度是否符合要求
   - 查看日志：`docker logs sqlserver`
   - 确保挂载目录权限正确
2. **报表服务无法连接 SQL Server**
   - 确认 SQL Server 容器正在运行
   - 验证 SA 密码是否正确
   - 尝试手动连接测试：`docker exec -it reportservice ping sqlserver`
3. **Web 门户无法访问**
   - 检查防火墙设置
   - 验证容器端口映射是否正确
   - 查看报表服务日志：`docker logs reportservice`



通过以上步骤，你可以在 CentOS 8 上成功部署 SQL Server 2019 和 SQL Reporting Services，并确保数据持久性和服务可用性。



分享



生成的yml文件内容

提供一份部署完成后的验证步骤

如何保障容器化部署的安全性？