# SQL Server 2019 高可用集群部署手册

**版本：** V2.1
**适用系统：** CentOS 7/8
**最后更新：** 2023年8月

------

## 目录

1. 部署准备
2. SQL Server安装
3. AlwaysOn配置
4. Pacemaker集群
5. 维护与监控
6. 附录

------

## 1. 部署准备

### 1.1 硬件要求

| 组件 | 最低要求  | 生产环境推荐 |
| :--- | :-------- | :----------- |
| CPU  | 4核       | 8核+         |
| 内存 | 8GB       | 16GB+        |
| 存储 | 100GB SSD | RAID 10 NVMe |
| 网络 | 1Gbps     | 10Gbps+      |

### 1.2 节点规划

| 主机名 | IP地址      | 角色     |
| :----- | :---------- | :------- |
| node1  | 192.168.1.1 | 主节点   |
| node2  | 192.168.1.2 | 备用节点 |
| VIP    | 192.168.1.3 | 虚拟IP   |

### 1.3 系统配置（所有节点执行）

bash



复制



下载

```
# 禁用SELinux
sudo setenforce 0
sudo sed -i 's/^SELINUX=.*/SELINUX=disabled/g' /etc/selinux/config

# 时间同步
sudo yum install -y chrony
sudo systemctl enable chronyd --now
```

------

## 2. SQL Server安装

### 2.1 安装主程序

bash



复制



下载

```
# 添加微软仓库
sudo curl -o /etc/yum.repos.d/mssql-server.repo https://packages.microsoft.com/config/rhel/8/mssql-server-2019.repo

# 安装
sudo yum install -y mssql-server

# 运行配置
sudo /opt/mssql/bin/mssql-conf setup
```

> 注意：SA密码需包含大小写字母、数字和特殊字符

### 2.2 验证安装

bash



复制



下载

```
systemctl status mssql-server
sqlcmd -S localhost -U SA -P '密码' -Q "SELECT @@VERSION"
```

------

## 3. AlwaysOn配置

### 3.1 启用HADR功能

bash



复制



下载

```
sudo /opt/mssql/bin/mssql-conf set hadr.hadrenabled 1
sudo systemctl restart mssql-server
```

### 3.2 创建可用性组（主节点执行）

sql



复制



下载

```
-- 创建端点
CREATE ENDPOINT [Hadr_Endpoint] 
    STATE = STARTED
    AS TCP (LISTENER_PORT = 5022)
    FOR DATABASE_MIRRORING (ROLE = ALL);

-- 创建可用性组
CREATE AVAILABILITY GROUP [ag1]
    WITH (CLUSTER_TYPE = EXTERNAL)
    FOR REPLICA ON 
        N'node1' WITH (ENDPOINT_URL = 'tcp://node1:5022'),
        N'node2' WITH (ENDPOINT_URL = 'tcp://node2:5022');
```

------

## 4. Pacemaker集群

### 4.1 初始化集群

bash



复制



下载

```
# 安装组件
sudo yum install -y pacemaker pcs fence-agents-all

# 设置hacluster密码
echo "P@ssw0rd!" | sudo passwd --stdin hacluster

# 启动服务
sudo systemctl enable --now pcsd pacemaker
```

### 4.2 配置资源

bash



复制



下载

```
# 创建AG资源
sudo pcs resource create ag_cluster ocf:mssql:ag ag_name=ag1 \
    meta failure-timeout=60s --master meta notify=true

# 创建VIP资源
sudo pcs resource create virtualip ocf:heartbeat:IPaddr2 \
    ip=192.168.1.3 cidr_netmask=24

# 设置约束
sudo pcs constraint colocation add virtualip ag_cluster-master INFINITY
sudo pcs constraint order promote ag_cluster-master then start virtualip
```

------

## 5. 维护与监控

### 5.1 常用命令

| 命令                | 用途         |
| :------------------ | :----------- |
| `pcs status`        | 查看集群状态 |
| `pcs resource move` | 手动故障转移 |
| `crm_mon -Afr`      | 实时集群监控 |

### 5.2 监控指标

sql



复制



下载

```
-- 检查AG状态
SELECT ar.replica_server_name, 
       ars.connected_state_desc,
       ars.synchronization_health_desc
FROM sys.availability_replicas ar
JOIN sys.dm_hadr_availability_replica_states ars 
    ON ar.replica_id = ars.replica_id;
```

------

## 6. 附录

### 6.1 日志位置

| 组件       | 路径                          |
| :--------- | :---------------------------- |
| SQL Server | /var/opt/mssql/log/errorlog   |
| Pacemaker  | /var/log/cluster/corosync.log |

### 6.2 故障排查流程

1. 检查服务状态：`systemctl status mssql-server`
2. 查看错误日志：`sudo tail -100 /var/opt/mssql/log/errorlog`
3. 验证网络连接：`telnet node2 5022`

------

**修订记录**

| 版本 | 日期       | 修改说明             |
| :--- | :--------- | :------------------- |
| 2.1  | 2023-08-15 | 增加CentOS 8适配说明 |