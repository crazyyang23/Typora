# SQL Server 2019 AlwaysOn 高可用集群部署操作手册

**版本：** V1.0
**适用系统：** 麒麟服务器操作系统 V10 SP1+
**适用场景：** 生产环境高可用数据库集群部署

------

## 目录

1. 环境准备
2. 系统优化配置
3. SQL Server 安装部署
4. AlwaysOn 高可用配置
5. Pacemaker 集群配置
6. 验证与监控
7. 维护操作
8. 附录

------

## 1. 环境准备

### 1.1 硬件要求

| 组件 | 最低配置 | 生产环境推荐配置 |
| :--- | :------- | :--------------- |
| CPU  | 4核      | 8核及以上        |
| 内存 | 8GB      | 16GB及以上       |
| 存储 | 100GB    | 根据数据量规划   |
| 网络 | 1Gbps    | 10Gbps           |

### 1.2 软件要求

- **操作系统**：麒麟服务器 V10 SP1 x86_64

- **软件包**：

  bash

  

  复制

  

  下载

  ```
  # 所有节点均需安装
  sudo apt-get install -y curl chrony firewalld
  ```

### 1.3 网络规划

| 节点  | IP地址      | 角色                 |
| :---- | :---------- | :------------------- |
| node1 | 192.168.1.1 | 主节点               |
| node2 | 192.168.1.2 | 备用节点             |
| VIP   | 192.168.1.3 | 虚拟IP（客户端访问） |

------

## 2. 系统优化配置

### 2.1 内核参数优化



```
# 编辑配置文件
sudo vi /etc/sysctl.d/99-sqlserver.conf

# 添加以下内容
vm.swappiness = 1
vm.dirty_ratio = 80
fs.file-max = 6815744
net.core.somaxconn = 32768

# 生效配置
sudo sysctl -p /etc/sysctl.d/99-sqlserver.conf
```

### 2.2 磁盘调度策略



```
# 查看当前磁盘设备
lsblk

# 永久修改调度策略（以sda为例）
echo 'ACTION=="add|change", KERNEL=="sda", ATTR{queue/scheduler}="noop"' | sudo tee /etc/udev/rules.d/60-sqlserver-io.rules
sudo udevadm control --reload
```

### 2.3 禁用透明大页



```
echo never | sudo tee /sys/kernel/mm/transparent_hugepage/enabled
echo 'echo never > /sys/kernel/mm/transparent_hugepage/enabled' | sudo tee -a /etc/rc.local
```

------

## 3. SQL Server 安装部署

### 3.1 安装主程序



```
# 添加微软仓库
sudo curl -o /etc/apt/trusted.gpg.d/microsoft.asc https://packages.microsoft.com/keys/microsoft.asc
sudo curl -o /etc/apt/sources.list.d/mssql-server-2019.list https://packages.microsoft.com/config/ubuntu/16.04/mssql-server-2019.list

# 安装
sudo apt-get update
sudo apt-get install -y mssql-server

# 运行配置
sudo /opt/mssql/bin/mssql-conf setup
```

### 3.2 内存配置（每个节点）



```
# 限制SQL Server最大内存为12GB（根据实际调整）
sudo /opt/mssql/bin/mssql-conf set memory.memorylimitmb 12288
sudo systemctl restart mssql-server
```

------

## 4. AlwaysOn 高可用配置

### 4.1 启用AlwaysOn功能

```
sudo /opt/mssql/bin/mssql-conf set hadr.hadrenabled 1
sudo systemctl restart mssql-server
```

### 4.2 创建可用性组（主节点执行）

sql

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

## 5. Pacemaker 集群配置

### 5.1 安装集群组件

```
sudo apt-get install -y pacemaker corosync fence-agents
```

### 5.2 启动集群服务



```
sudo systemctl enable --now corosync pacemaker
```

### 5.3 配置资源

```
# 创建AG资源
sudo pcs resource create ag_cluster ocf:mssql:ag ag_name=ag1 \
    meta failure-timeout=60s --master meta notify=true

# 创建VIP资源
sudo pcs resource create virtualip ocf:heartbeat:IPaddr2 \
    ip=192.168.1.3 cidr_netmask=24 op monitor interval=30s
```

------

## 6. 验证与监控

### 6.1 检查集群状态

```
sudo pcs status

# 预期输出示例
Cluster name: sqlcluster
Stack: corosync
Current DC: node1 (version 2.0.5-9)
2 nodes configured
2 resources configured
```

### 6.2 数据库级别验证

```
SELECT 
    ag.name AS [AG Name], 
    ar.replica_server_name,
    ars.connected_state_desc
FROM sys.availability_groups ag
JOIN sys.availability_replicas ar ON ag.group_id = ar.group_id
JOIN sys.dm_hadr_availability_replica_states ars ON ar.replica_id = ars.replica_id;
```

------

## 7. 维护操作

### 7.1 手动故障转移



```
sudo pcs resource move ag_cluster-master node2 --master
```

### 7.2 维护模式

```
# 进入维护模式
sudo pcs property set maintenance-mode=true

# 退出维护模式
sudo pcs property set maintenance-mode=false
```

------

## 8. 附录

### 8.1 常用命令速查

| 命令                          | 用途             |
| :---------------------------- | :--------------- |
| `sudo pcs cluster stop --all` | 停止整个集群     |
| `sudo pcs resource cleanup`   | 清理资源状态     |
| `crm_mon -Afr`                | 实时监控集群状态 |

### 8.2 故障排查指南

1. **日志位置**：

   - SQL Server: `/var/opt/mssql/log/errorlog`
   - Pacemaker: `/var/log/cluster/corosync.log`

2. **网络检查**：

   ```
   
   ```
# 验证端点连通性
   telnet node2 5022
```

------

**文档修订记录**

| 版本 | 日期       | 修改说明 |
| :--- | :--------- | :------- |
| 1.0  | 2023-08-01 | 初始版本 |

**注意事项**

1. 生产环境部署前需在测试环境验证
2. 所有密码应使用符合复杂度要求的强密码
3. 建议配置定期备份和监控告警
```