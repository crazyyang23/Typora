### 在 Linux 上部署 SQL Server 三节点群集方案

#### 一、环境准备

##### 1. 推荐的 Linux 版本

根据 Microsoft 官方支持，以下 Linux 版本适用于 SQL Server 群集部署：



- Red Hat Enterprise Linux (RHEL) 8/9
- SUSE Linux Enterprise Server (SLES) 15
- Ubuntu Server 20.04/22.04



本方案以 RHEL 8 为例进行说明。

##### 2. 硬件要求（单节点）

- CPU：至少 4 核
- 内存：至少 16GB
- 存储：系统盘 50GB+，数据盘根据需求配置
- 网络：双网卡（管理网络和群集心跳网络）

##### 3. 节点规划

| 节点名称      | IP 地址（管理网络） | IP 地址（心跳网络） | 角色        |
| ------------- | ------------------- | ------------------- | ----------- |
| sql-cluster-1 | 192.168.1.101       | 10.0.0.101          | 群集节点    |
| sql-cluster-2 | 192.168.1.102       | 10.0.0.102          | 群集节点    |
| sql-cluster-3 | 192.168.1.103       | 10.0.0.103          | 群集节点    |
| 虚拟 IP       | 192.168.1.100       | 不适用              | 群集服务 IP |

#### 二、操作系统配置

##### 1. 安装基础系统

在三个节点上安装 RHEL 8，选择 "Server with GUI" 或 "Minimal Install"（推荐 Minimal）。

##### 2. 配置主机名和 DNS

在每个节点上执行：



```bash
# 设置主机名
hostnamectl set-hostname sql-cluster-1
# 编辑hosts文件
vi /etc/hosts
192.168.1.101 sql-cluster-1
192.168.1.102 sql-cluster-2
192.168.1.103 sql-cluster-3
192.168.1.100 sql-cluster-vip
```

##### 3. 配置网络

为管理网络和心跳网络配置静态 IP（以 nmcli 为例）：



```bash
# 管理网络配置
nmcli connection modify enp0s3 ipv4.method manual ipv4.addresses 192.168.1.101/24 ipv4.gateway 192.168.1.1
nmcli connection modify enp0s3 ipv4.dns 8.8.8.8

# 心跳网络配置
nmcli connection add type ethernet ifname enp0s8 con-name cluster-heartbeat
nmcli connection modify cluster-heartbeat ipv4.method manual ipv4.addresses 10.0.0.101/24
nmcli connection up cluster-heartbeat
```

##### 4. 关闭防火墙和 SELinux（测试环境）

bash

```bash
# 关闭防火墙
systemctl stop firewalld
systemctl disable firewalld

# 禁用SELinux
setenforce 0
sed -i 's/^SELINUX=enforcing$/SELINUX=disabled/' /etc/selinux/config
```

##### 5. 配置 NTP 同步

```bash
yum install -y ntp
systemctl enable ntpd
systemctl start ntpd
```

#### 三、安装 SQL Server

##### 1. 配置 SQL Server 仓库

在每个节点上执行：

```bash
# 下载并注册Microsoft SQL Server Red Hat仓库
curl -o /etc/yum.repos.d/mssql-server.repo https://packages.microsoft.com/config/rhel/8/mssql-server-2022.repo

# 安装SQL Server
yum install -y mssql-server

# 配置SQL Server
sudo /opt/mssql/bin/mssql-conf setup
# 选择版本（如Enterprise），设置SA密码，配置TCP端口（默认1433）
```

##### 2. 验证 SQL Server 安装

bash

```bash
systemctl status mssql-server
systemctl enable mssql-server
```

#### 四、配置群集基础架构

##### 1. 安装群集组件

在所有节点上安装：



```bash
yum install -y pacemaker corosync pcs resource-agents
```

##### 2. 配置 Pacemaker 认证

bash

```bash
# 在第一个节点上生成认证密钥
corosync-keygen

# 复制配置文件到所有节点
scp /etc/corosync/corosync.conf sql-cluster-2:/etc/corosync/
scp /etc/corosync/authkey sql-cluster-2:/etc/corosync/
scp /etc/corosync/corosync.conf sql-cluster-3:/etc/corosync/
scp /etc/corosync/authkey sql-cluster-3:/etc/corosync/
```

##### 3. 配置 Corosync

编辑`/etc/corosync/corosync.conf`，添加以下内容（根据实际环境修改）：



yaml

```yaml
totem {
    version: 2
    secauth: on
    cluster_name: sql_cluster
    transport: udpu
    token: 10000
}

nodelist {
    node {
        ring0_addr: sql-cluster-1
        nodeid: 1
    }
    node {
        ring0_addr: sql-cluster-2
        nodeid: 2
    }
    node {
        ring0_addr: sql-cluster-3
        nodeid: 3
    }
}

quorum {
    provider: corosync_votequorum
    two_node: 1
}

logging {
    to_logfile: yes
    logfile: /var/log/corosync/corosync.log
    to_syslog: yes
}
```

##### 4. 启动群集服务

在所有节点上启动服务：



bash

```bash
systemctl start corosync
systemctl start pacemaker
systemctl enable corosync
systemctl enable pacemaker
```

##### 5. 配置 Pacemaker 管理

bash

```bash
# 初始化pcsd服务并设置密码
systemctl start pcsd
systemctl enable pcsd
pcs cluster auth sql-cluster-1 sql-cluster-2 sql-cluster-3 -u admin -p password
pcs cluster setup --name sql-cluster sql-cluster-1 sql-cluster-2 sql-cluster-3
pcs cluster start --all
pcs cluster enable --all
```

##### 6. 配置群集属性

bash

```bash
# 设置群集属性
pcs property set cluster-name=sql-cluster
pcs property set no-quorum-policy=ignore  # 测试环境，生产环境建议设置为auto或freeze
pcs property set stonith-enabled=false  # 禁用STONITH，生产环境根据需要配置
```

### 4.1 创建数据库主密钥（所有副本）

在主副本和所有辅助副本的 SQL Server 中，分别执行：



sql

```sql
CREATE MASTER KEY ENCRYPTION BY PASSWORD = 'Master@Key123';
GO
```



**注意**：密码需使用强密码，并妥善保管，各节点密码保持一致。

### 4.2 在主副本上创建证书

在主副本的 SQL Server 中执行：



sql

```sql
CREATE CERTIFICATE AG_Certificate 
WITH SUBJECT = 'AlwaysOn Endpoint Certificate',
EXPIRY_DATE = '2030-12-31';
GO

-- 备份证书（含私钥）
BACKUP CERTIFICATE AG_Certificate 
TO FILE = '/var/opt/mssql/data/AG_Certificate.crt'
WITH PRIVATE KEY (
    FILE = '/var/opt/mssql/data/AG_Certificate.key',
    ENCRYPTION BY PASSWORD = 'Backup@Key123'
);
GO
```



**注意**：记录好证书备份时设置的密码，后续导入需使用。

### 4.3 复制证书文件到辅助副本

在主副本的 Linux 系统中，通过 SSH 执行以下命令（将 `辅助副本IP` 替换为实际 IP 地址）：



bash

```bash
# 在主副本上执行（替换IP地址为辅助副本的实际IP）
scp /var/opt/mssql/data/AG_Certificate.* mssql@辅助副本IP:/var/opt/mssql/data/
```

### 4.4 在辅助副本上导入证书

在每个辅助副本的 SQL Server 中执行：



sql

```sql
CREATE CERTIFICATE AG_Certificate 
FROM FILE = '/var/opt/mssql/data/AG_Certificate.crt'
WITH PRIVATE KEY (
    FILE = '/var/opt/mssql/data/AG_Certificate.key',
    DECRYPTION BY PASSWORD = 'Backup@Key123'
);
GO
```

### 4.5 在所有副本上创建端点

在主副本和所有辅助副本的 SQL Server 中，执行：



sql

```sql
CREATE ENDPOINT [AG_Endpoint] 
STATE = STARTED
AS TCP (LISTENER_PORT = 5022)
FOR DATABASE_MIRRORING (
    AUTHENTICATION = CERTIFICATE AG_Certificate,
    ENCRYPTION = REQUIRED ALGORITHM AES,
    ROLE = ALL
);
GO
```

## 五、验证配置

### 5.1 检查证书状态（所有副本）

在各节点执行以下 SQL 语句：



```sql
SELECT 
    name, 
    pvt_key_encryption_type_desc, 
    start_date, 
    expiry_date
FROM sys.certificates
WHERE name = 'AG_Certificate';
```



**预期结果**：`pvt_key_encryption_type_desc` 应为 `ENCRYPTED_BY_MASTER_KEY`，且 `expiry_date` 在未来。若不符合预期，需重新检查证书创建及导入步骤。

### 5.2 检查端点状态（所有副本）

执行：



sql

```sql
SELECT 
    name, 
    state_desc, 
    port, 
    role_desc
FROM sys.database_mirroring_endpoints
WHERE name = 'AG_Endpoint';
```

**预期结果**：`state_desc` 应为 `STARTED`，`port` 为 `5022`，`role_desc` 为 `ALL`。若端点状态异常，需检查端点创建步骤及端口配置。

#### 1. **确认环境类型**

首先确认你的 SQL Server 实例是否运行在 Linux 上：



sql

```sql
SELECT @@VERSION;
```



如果输出包含 "Linux"，则说明是 Linux 环境，需要使用 Pacemaker 模式的 AG 配置。

#### 2. **使用正确的 AG 创建命令**

如果你要在 Linux 上创建 / 加入可用性组，应使用以下步骤：



**步骤 1：在主副本上创建 AG（示例）**

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



```sql
CREATE AVAILABILITY GROUP [AG_Kylin_Cluster]
WITH (CLUSTER_TYPE = EXTERNAL)
FOR REPLICA ON 
N'szmessqltest01' WITH (
   ENDPOINT_URL = N'tcp://szmessqltest01:5022',
   AVAILABILITY_MODE = SYNCHRONOUS_COMMIT,
   FAILOVER_MODE = EXTERNAL,
   SEEDING_MODE = AUTOMATIC
),
N'szmessqltest02' WITH (
   ENDPOINT_URL = N'tcp://szmessqltest02:5022',
   AVAILABILITY_MODE = SYNCHRONOUS_COMMIT,
   FAILOVER_MODE = EXTERNAL,
   SEEDING_MODE = AUTOMATIC
),
N'szmessqltest03' WITH (
   ENDPOINT_URL = N'tcp://szmessqltest03:5022',
   AVAILABILITY_MODE = ASYNCHRONOUS_COMMIT,  -- 第三节点可设为异步
   FAILOVER_MODE = EXTERNAL,
   SEEDING_MODE = AUTOMATIC
);
GO
```



**步骤 2：在辅助副本上加入 AG**



```sql
ALTER AVAILABILITY GROUP [AG_Kylin_Cluster] JOIN
WITH (CLUSTER_TYPE = EXTERNAL);
GO
```



**关键参数说明**：



- `CLUSTER_TYPE = EXTERNAL`：表示使用外部集群管理器（如 Pacemaker）
- `FAILOVER_MODE = EXTERNAL`：依赖外部集群进行故障转移