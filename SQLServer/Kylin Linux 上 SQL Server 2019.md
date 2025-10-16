# Kylin Linux 上 SQL Server 2019 Always On 高可用性配置手册

## 环境说明

- **节点1**: szmessqltest01 (主节点初始角色)
- **节点2**: szmessqltest02
- **节点3**: szmessqltest03
- **操作系统**: Kylin Linux (基于 CentOS)
- **数据库**: SQL Server 2019
- **AG 名称**: AG_Kylin_Cluster
- **监听器名称**: AG_Listener
- **监听器 IP**: 192.168.100.100 (示例，请替换为实际可用IP)
- **子网掩码**: 255.255.255.0
- **端口**: 5022 (镜像端点), 1433 (监听器)

## 第一部分：基础环境准备

### 1.1 所有节点系统配置

```
# 1. 设置主机名解析（所有节点执行）
sudo vi /etc/hosts
# 添加以下内容（IP地址替换为实际地址）
192.168.100.1 szmessqltest01
192.168.100.2 szmessqltest02
192.168.100.3 szmessqltest03

# 2. 关闭防火墙或开放端口（生产环境建议开放端口）
sudo systemctl stop firewalld
sudo systemctl disable firewalld

# 或者开放必要端口
sudo firewall-cmd --permanent --add-port=1433/tcp
sudo firewall-cmd --permanent --add-port=5022/tcp
sudo firewall-cmd --permanent --add-port=2224/tcp
sudo firewall-cmd --permanent --add-port=3121/tcp
sudo firewall-cmd --permanent --add-port=21064/tcp
sudo firewall-cmd --reload

# 3. 禁用SELinux
sudo setenforce 0
sudo sed -i 's/^SELINUX=.*/SELINUX=disabled/g' /etc/selinux/config

# 4. 安装必要依赖
sudo yum install -y policycoreutils-python-utils libseccomp-devel curl
```

### 1.2 所有节点安装 SQL Server 2019

```
# 1. 导入Microsoft GPG密钥
sudo rpm --import https://packages.microsoft.com/keys/microsoft.asc

# 2. 添加SQL Server存储库
sudo curl -o /etc/yum.repos.d/mssql-server.repo https://packages.microsoft.com/config/rhel/8/mssql-server-2019.repo

# 3. 安装SQL Server
sudo yum install -y mssql-server

# 4. 运行配置
sudo /opt/mssql/bin/mssql-conf setup
# 选择版本：2 (Developer) 或根据实际选择
# 设置强密码：例如 Kylin@SQL2019!

# 5. 验证安装
systemctl status mssql-server
sudo /opt/mssql/bin/mssql-conf validate
```

### 1.3 安装SQL Server工具（可选）

```
sudo curl -o /etc/yum.repos.d/msprod.repo https://packages.microsoft.com/config/rhel/8/prod.repo
sudo yum install -y mssql-tools unixODBC-devel
echo 'export PATH="$PATH:/opt/mssql-tools/bin"' >> ~/.bash_profile
source ~/.bash_profile
```

## 第二部分：配置Always On可用性组

### 2.1 所有节点启用Always On功能

```
# 1. 启用HADR功能
sudo /opt/mssql/bin/mssql-conf set hadr.hadrenabled 1
sudo /opt/mssql/bin/mssql-conf set sqlagent.enabled true
sudo systemctl restart mssql-server

# 2. 验证HADR状态
sqlcmd -S localhost -U SA -P 'mes_2011' -Q "SELECT SERVERPROPERTY('IsHadrEnabled')"
```

### 2.2 主节点(szmessqltest01)配置

sql

```
-- 1. 创建主密钥和证书
USE master;
GO
CREATE MASTER KEY ENCRYPTION BY PASSWORD = 'Master@Key123';
GO
CREATE CERTIFICATE AG_Certificate WITH SUBJECT = 'AG_Certificate_Kylin';
GO

-- 2. 创建镜像端点
CREATE ENDPOINT [AG_Endpoint] 
STATE = STARTED
AS TCP (LISTENER_PORT = 5022)
FOR DATABASE_MIRRORING (
   AUTHENTICATION = CERTIFICATE AG_Certificate,
   ENCRYPTION = REQUIRED ALGORITHM AES,
   ROLE = ALL
);
GO

-- 3. 备份证书
BACKUP CERTIFICATE AG_Certificate TO FILE = '/var/opt/mssql/data/AG_Certificate.cer';
GO
BACKUP MASTER KEY TO FILE = '/var/opt/mssql/data/AG_MasterKey.key' 
ENCRYPTION BY PASSWORD = 'Backup@Key123';
GO

-- 4. 创建登录名并授权
CREATE LOGIN AG_Login WITH PASSWORD = 'AGLogin@123';
GO
CREATE USER AG_User FOR LOGIN AG_Login;
GO
GRANT CONNECT ON ENDPOINT::AG_Endpoint TO AG_Login;
GO
```

### 2.3 将证书复制到辅助节点

在主节点执行：

```
sudo scp /var/opt/mssql/data/AG_Certificate.cer szmessqltest02:/var/opt/mssql/data/
sudo scp /var/opt/mssql/data/AG_Certificate.cer szmessqltest03:/var/opt/mssql/data/
sudo scp /var/opt/mssql/data/AG_MasterKey.key szmessqltest02:/var/opt/mssql/data/
sudo scp /var/opt/mssql/data/AG_MasterKey.key szmessqltest03:/var/opt/mssql/data/
```

### 2.4 辅助节点配置(szmessqltest02和szmessqltest03)

```
chown mssql:mssql /var/opt/mssql/data/AG_MasterKey.key
chmod 600 /var/opt/mssql/data/AG_MasterKey.key
```



```
-- 1. 恢复主密钥和证书
USE master;
GO
RESTORE MASTER KEY FROM FILE = '/var/opt/mssql/data/AG_MasterKey.key'
DECRYPTION BY PASSWORD = 'Backup@Key123'
ENCRYPTION BY PASSWORD = 'Master@Key123';
GO

-- 2. 从文件创建证书
CREATE CERTIFICATE AG_Certificate FROM FILE = '/var/opt/mssql/data/AG_Certificate.cer';
GO

-- 3. 创建镜像端点（与主节点相同）
CREATE ENDPOINT [AG_Endpoint] 
STATE = STARTED
AS TCP (LISTENER_PORT = 5022)
FOR DATABASE_MIRRORING (
   AUTHENTICATION = CERTIFICATE AG_Certificate,
   ENCRYPTION = REQUIRED ALGORITHM AES,
   ROLE = ALL
);
GO

-- 4. 创建相同的登录名并授权
CREATE LOGIN AG_Login WITH PASSWORD = 'AGLogin@123';
GO
CREATE USER AG_User FOR LOGIN AG_Login;
GO
GRANT CONNECT ON ENDPOINT::AG_Endpoint TO AG_Login;
GO
```

## 第三部分：创建可用性组

### 3.1 在主节点(szmessqltest01)准备数据库

```
-- 1. 创建测试数据库
CREATE DATABASE AG_TestDB;
GO
ALTER DATABASE AG_TestDB SET RECOVERY FULL;
GO

-- 2. 创建完整备份和日志备份
BACKUP DATABASE AG_TestDB TO DISK = '/var/opt/mssql/data/AG_TestDB.bak' WITH COMPRESSION;
GO
BACKUP LOG AG_TestDB TO DISK = '/var/opt/mssql/data/AG_TestDB.trn' WITH COMPRESSION;
GO
```

### 3.2 创建可用性组

在主节点执行：

```
-- 创建可用性组
CREATE AVAILABILITY GROUP [AG_Kylin_Cluster]
WITH (CLUSTER_TYPE = NONE)
FOR REPLICA ON 
N'szmessqltest01' WITH (
   ENDPOINT_URL = N'tcp://szmessqltest01:5022',
   AVAILABILITY_MODE = SYNCHRONOUS_COMMIT,
   FAILOVER_MODE = MANUAL,
   SEEDING_MODE = AUTOMATIC,
   SECONDARY_ROLE (ALLOW_CONNECTIONS = ALL)
),
N'szmessqltest02' WITH (
   ENDPOINT_URL = N'tcp://szmessqltest02:5022',
   AVAILABILITY_MODE = SYNCHRONOUS_COMMIT,
   FAILOVER_MODE = MANUAL,
   SEEDING_MODE = AUTOMATIC,
   SECONDARY_ROLE (ALLOW_CONNECTIONS = ALL)
),
N'szmessqltest03' WITH (
   ENDPOINT_URL = N'tcp://szmessqltest03:5022',
   AVAILABILITY_MODE = ASYNCHRONOUS_COMMIT,  -- 第三节点可设为异步
   FAILOVER_MODE = MANUAL,
   SEEDING_MODE = AUTOMATIC,
   SECONDARY_ROLE (ALLOW_CONNECTIONS = ALL)
);
GO

-- 将数据库加入可用性组
ALTER AVAILABILITY GROUP [AG_Kylin_Cluster] ADD DATABASE [AG_TestDB];
GO
```

### 3.3 辅助节点加入可用性组

在szmessqltest02和szmessqltest03上执行：

```
-- 加入可用性组
ALTER AVAILABILITY GROUP [AG_Kylin_Cluster] JOIN;
GO

-- 授予创建数据库权限
ALTER AVAILABILITY GROUP [AG_Kylin_Cluster] GRANT CREATE ANY DATABASE;
GO
```

## 第四部分：配置监听器和验证

### 4.1 创建可用性组监听器

在主节点执行：

```
-- 创建监听器（IP地址替换为实际可用IP）
ALTER AVAILABILITY GROUP [AG_Kylin_Cluster]
ADD LISTENER 'AG_Listener' (
   WITH IP ((N'192.168.100.100', N'255.255.255.0')),
   PORT=1433);
GO
```

### 4.2 验证配置

```
-- 1. 检查可用性组状态
SELECT ag.name AS [AG Name], 
       ar.replica_server_name AS [Replica],
       ars.connected_state_desc AS [Connection State],
       ars.synchronization_health_desc AS [Sync Health],
       ars.operational_state_desc AS [Operational State]
FROM sys.availability_groups ag
JOIN sys.availability_replicas ar ON ag.group_id = ar.group_id
JOIN sys.dm_hadr_availability_replica_states ars ON ar.replica_id = ars.replica_id;
GO

-- 2. 检查数据库同步状态
SELECT dc.database_name AS [Database],
       drs.synchronization_state_desc AS [Sync State],
       drs.synchronization_health_desc AS [Sync Health],
       drs.last_hardened_time AS [Last Hardened Time]
FROM sys.dm_hadr_database_replica_states drs
JOIN sys.availability_databases_cluster dc ON drs.group_database_id = dc.group_database_id;
GO

-- 3. 检查监听器状态
SELECT listener_id, ip_address, port, is_conformant, state_desc
FROM sys.availability_group_listeners;
GO
```

## 第五部分：Pacemaker集群配置（自动故障转移）

### 5.1 所有节点安装Pacemaker

```
# 1. 安装必要软件包
sudo yum install -y pacemaker pcs fence-agents-all resource-agents

# 2. 设置hacluster用户密码（所有节点相同）
sudo passwd hacluster
# 设置密码，例如：Pacemaker@123

# 3. 启用服务
sudo systemctl enable pcsd
sudo systemctl start pcsd
sudo systemctl enable pacemaker
```

### 5.2 主节点配置集群

```
# 1. 认证集群节点
sudo pcs cluster auth szmessqltest01 szmessqltest02 szmessqltest03 -u hacluster -p Pacemaker@123

# 2. 创建集群
sudo pcs cluster setup --name ag_cluster szmessqltest01 szmessqltest02 szmessqltest03

# 3. 启动集群
sudo pcs cluster start --all
sudo pcs cluster enable --all

# 4. 禁用STONITH（测试环境）
sudo pcs property set stonith-enabled=false
sudo pcs property set no-quorum-policy=ignore
```

### 5.3 安装SQL Server资源代理

```
sudo yum install -y mssql-server-ha
```

### 5.4 创建Pacemaker资源

```
# 1. 创建可用性组资源
sudo pcs resource create ag_cluster ocf:mssql:ag ag_name=AG_Kylin_Cluster meta failure-timeout=60s --group ag_group

# 2. 创建虚拟IP资源
sudo pcs resource create virtualip ocf:heartbeat:IPaddr2 ip=192.168.100.100 cidr_netmask=24 op monitor interval=30s --group ag_group

# 3. 验证配置
sudo pcs status
sudo pcs resource show
```

## 第六部分：维护操作

### 6.1 添加新数据库到可用性组

```
-- 在主节点执行
CREATE DATABASE NewAppDB;
GO
ALTER DATABASE NewAppDB SET RECOVERY FULL;
GO
BACKUP DATABASE NewAppDB TO DISK = '/var/opt/mssql/data/NewAppDB.bak' WITH COMPRESSION;
GO
BACKUP LOG NewAppDB TO DISK = '/var/opt/mssql/data/NewAppDB.trn' WITH COMPRESSION;
GO

ALTER AVAILABILITY GROUP [AG_Kylin_Cluster] ADD DATABASE [NewAppDB];
GO
```

### 6.2 手动故障转移

```
-- 在主节点执行（将主角色转移到szmessqltest02）
ALTER AVAILABILITY GROUP [AG_Kylin_Cluster] FAILOVER TO 'szmessqltest02';
GO
```

### 6.3 监控命令

```
-- 监控同步状态
SELECT ar.replica_server_name, 
       db_name(drs.database_id) as database_name,
       drs.synchronization_state_desc,
       drs.synchronization_health_desc,
       drs.last_hardened_time
FROM sys.dm_hadr_database_replica_states drs
JOIN sys.availability_replicas ar ON drs.replica_id = ar.replica_id;
GO

-- 监控性能计数器
SELECT * FROM sys.dm_os_performance_counters
WHERE counter_name LIKE '%Hadr%' OR object_name LIKE '%Availability Replica%';
GO
```

## 常见问题处理

1. **端点连接问题**：
   - 验证端口5022是否开放
   - 检查证书是否正确复制到所有节点
   - 验证端点状态：`SELECT * FROM sys.database_mirroring_endpoints`
2. **同步延迟问题**：
   - 检查网络带宽和延迟
   - 监控日志发送队列：`SELECT log_send_queue_size FROM sys.dm_hadr_database_replica_states`
   - 考虑将第三节点设置为异步提交模式
3. **Pacemaker资源故障**：
   - 检查日志：`journalctl -u pacemaker -n 100 -f`
   - 验证资源约束：`sudo pcs constraint show`
   - 可能需要清理并重新创建资源
4. **证书过期问题**：
   - 定期检查证书有效期：`SELECT name, expiry_date FROM sys.certificates`
   - 提前规划证书更新流程

本手册提供了在Kylin Linux三节点环境上配置SQL Server 2019 Always On可用性组的完整流程。生产环境中请根据实际网络配置、安全要求和性能需求进行调整。