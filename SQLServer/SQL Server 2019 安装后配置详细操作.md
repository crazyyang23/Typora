# SQL Server 2019 安装后配置详细操作指南

## 一、初始配置运行

### 1.1 启动配置向导

执行以下命令启动交互式配置：

bash

```
sudo /opt/mssql/bin/mssql-conf setup
```

### 1.2 交互式配置步骤

1. **选择版本**：

   ```
Choose an edition of SQL Server:
     1) Evaluation (free, no production use rights, 180-day limit)
  2) Developer (free, no production use rights)
     3) Express (free)
  4) Web (PAID)
     5) Standard (PAID)
  6) Enterprise (PAID)
     7) Enterprise Core (PAID)
     8) I bought a license through a retail sales channel and have a product key to enter.
   
   Details about editions can be found at
   https://go.microsoft.com/fwlink/?LinkId=852748&clcid=0x409
   
   Use of PAID editions of this software requires separate licensing through a
   Microsoft Volume Licensing program.
   By choosing a PAID edition, you are verifying that you have the appropriate
   number of licenses in place to install and run this software.
   
   Enter your edition(1-8): 
   ```
   
   - 生产环境选择对应许可版本(5/6/7)
   - 测试环境可选择2(Developer)
   
2. **接受许可条款**：

   ```
The license terms for this product can be found in
   /usr/share/doc/mssql-server or downloaded from:
https://go.microsoft.com/fwlink/?LinkId=855862&clcid=0x409
   
The privacy statement can be viewed at:
   https://go.microsoft.com/fwlink/?LinkId=853010&clcid=0x409

   Do you accept the license terms? [Yes/No]:
   ```
   
   输入`Yes`继续
   
3. **设置SA密码**：

   ```
Enter the SQL Server system administrator password: 
   Confirm the SQL Server system administrator password: 
```
   
- 密码必须包含：
     - 至少8个字符
  - 大写字母、小写字母、数字和符号中的三种
   - 示例强密码：`SQL_Server2019!`
   
4. **配置完成**：

   

   ```
Configuring SQL Server...
   
The configuration process was successful. SQL Server is now ready for use.
   ```

## 二、基础服务管理

### 2.1 服务状态检查

```
# 检查运行状态
sudo systemctl status mssql-server --no-pager

# 预期输出示例：
● mssql-server.service - Microsoft SQL Server Database Engine
   Loaded: loaded (/lib/systemd/system/mssql-server.service; enabled; vendor preset: enabled)
   Active: active (running) since Tue 2023-08-01 14:30:45 CST; 5min ago
```

### 2.2 服务控制命令

```
# 启动服务
sudo systemctl start mssql-server

# 停止服务
sudo systemctl stop mssql-server

# 重启服务
sudo systemctl restart mssql-server

# 设置开机自启
sudo systemctl enable mssql-server
```

## 三、网络配置

### 3.1 修改监听端口

```
sudo /opt/mssql/bin/mssql-conf set network.tcpport 14333
sudo systemctl restart mssql-server
```

### 3.2 配置监听IP地址

```
# 允许所有IP连接
sudo /opt/mssql/bin/mssql-conf set network.ipaddress 0.0.0.0

# 或指定特定IP
sudo /opt/mssql/bin/mssql-conf set network.ipaddress 192.168.1.100

sudo systemctl restart mssql-server
```

## 四、存储配置

### 4.1 更改数据文件目录

1. 停止SQL Server服务：

   ```
sudo systemctl stop mssql-server
   ```

2. 移动现有数据文件：

   ```
sudo mv /var/opt/mssql/data /new/data/path
   ```

3. 更新配置：

   ```
sudo /opt/mssql/bin/mssql-conf set filelocation.defaultdatadir /new/data/path/data
   sudo /opt/mssql/bin/mssql-conf set filelocation.defaultlogdir /new/data/path/log
sudo /opt/mssql/bin/mssql-conf set filelocation.errorlogfile /new/data/path/log/errorlog
   ```

4. 修复权限：

   ```
sudo chown -R mssql:mssql /new/data/path
   sudo chmod -R 750 /new/data/path
```
   
5. 重启服务：

   ```
sudo systemctl start mssql-server
   ```

## 五、内存配置

### 5.1 设置内存限制

```
# 设置为8GB（单位MB）
sudo /opt/mssql/bin/mssql-conf set memory.memorylimitmb 8192
sudo systemctl restart mssql-server
```

### 5.2 验证内存配置

sql



复制



下载

```
-- 使用sqlcmd连接
sqlcmd -S localhost -U SA -P 'YourPassword'

-- 执行查询
SELECT physical_memory_kb/1024 AS [Physical Memory (MB)],
       committed_target_kb/1024 AS [Committed Memory (MB)],
       max_server_memory/1024 AS [Max Server Memory (MB)]
FROM sys.dm_os_sys_info;
GO
```

## 六、安全配置

### 6.1 更改SA密码

bash



复制



下载

```
sudo systemctl stop mssql-server
sudo /opt/mssql/bin/mssql-conf set-sa-password
sudo systemctl start mssql-server
```

### 6.2 启用TLS加密

1. 准备证书：

   bash

   

   复制

   

   下载

   ```
   sudo mkdir -p /etc/mssql/tls
   sudo cp server.crt /etc/mssql/tls/
   sudo cp server.key /etc/mssql/tls/
   sudo chown mssql:mssql /etc/mssql/tls/*
   sudo chmod 600 /etc/mssql/tls/*
   ```

2. 配置TLS：

   bash

   

   复制

   

   下载

   ```
   sudo /opt/mssql/bin/mssql-conf set network.forceencryption 1
   sudo /opt/mssql/bin/mssql-conf set network.tlscert /etc/mssql/tls/server.crt
   sudo /opt/mssql/bin/mssql-conf set network.tlskey /etc/mssql/tls/server.key
   sudo /opt/mssql/bin/mssql-conf set network.tlsprotocols 1.2
   ```

3. 重启服务：

   bash

   

   复制

   

   下载

   ```
   sudo systemctl restart mssql-server
   ```

## 七、性能调优

### 7.1 配置最大并行度

sql



复制



下载

```
EXEC sp_configure 'show advanced options', 1;
RECONFIGURE;
EXEC sp_configure 'max degree of parallelism', 4;  -- 通常设为物理核心数的一半
RECONFIGURE;
```

### 7.2 优化TempDB配置

sql



复制



下载

```
-- 根据CPU核心数创建多个TempDB文件（推荐1:4比例）
ALTER DATABASE tempdb MODIFY FILE (NAME = 'tempdev', SIZE = 8GB, FILEGROWTH = 1GB);
GO

-- 添加额外的TempDB文件（示例添加3个）
ALTER DATABASE tempdb ADD FILE (NAME = 'tempdev2', SIZE = 8GB, FILEGROWTH = 1GB);
ALTER DATABASE tempdb ADD FILE (NAME = 'tempdev3', SIZE = 8GB, FILEGROWTH = 1GB);
ALTER DATABASE tempdb ADD FILE (NAME = 'tempdev4', SIZE = 8GB, FILEGROWTH = 1GB);
GO
```

## 八、远程连接配置

### 8.1 启用远程连接

sql



复制



下载

```
EXEC sp_configure 'remote access', 1;
RECONFIGURE;
```

### 8.2 创建防火墙规则

bash



复制



下载

```
sudo firewall-cmd --zone=public --add-port=1433/tcp --permanent
sudo firewall-cmd --reload
```

## 九、验证安装

### 9.1 基本连接测试

bash



复制



下载

```
sqlcmd -S localhost -U SA -P 'YourPassword' -Q "SELECT @@VERSION"
```

### 9.2 系统健康检查

sql



复制



下载

```
-- 检查数据库状态
SELECT name, state_desc FROM sys.databases;

-- 检查服务状态
SELECT servicename, status_desc FROM sys.dm_server_services;

-- 检查资源使用
SELECT
    (physical_memory_in_use_kb/1024) AS Memory_usedby_Sqlserver_MB,
    (locked_page_allocations_kb/1024) AS Locked_pages_used_Sqlserver_MB,
    (total_physical_memory_kb/1024) AS Physical_Memory_MB,
    (available_physical_memory_kb/1024) AS Available_Physical_Memory_MB
FROM sys.dm_os_process_memory;
```

## 十、故障排除

### 10.1 常见问题解决

**问题1：服务无法启动**

检查错误日志：

bash



复制



下载

```
sudo cat /var/opt/mssql/log/errorlog | grep -i error
```

**问题2：连接被拒绝**

验证服务监听：

bash



复制



下载

```
sudo netstat -tulnp | grep sqlservr
```

**问题3：性能问题**

检查资源使用：

bash



复制



下载

```
top -p $(pgrep sqlservr)
```

## 十一、配置备份

### 11.1 导出当前配置

bash



复制



下载

```
sudo /opt/mssql/bin/mssql-conf get all > /path/to/mssql-config-backup-$(date +%Y%m%d).txt
```

### 11.2 重要配置文件备份

bash



复制



下载

```
sudo tar -czvf /backup/mssql-config-$(date +%Y%m%d).tar.gz \
    /etc/systemd/system/mssql-server.service \
    /var/opt/mssql/mssql.conf \
    /etc/apt/sources.list.d/mssql-server-2019.list
```

通过以上详细步骤，您可以完成SQL Server 2019在麒麟系统上的全面配置。建议在生产环境部署前，在测试环境验证所有配置项。