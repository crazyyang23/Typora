# 麒麟系统部署SQL Server前期准备工作手册

## 一、硬件环境准备

### 1.1 服务器硬件要求

**最低配置要求**：

- **CPU**：x86_64架构，至少4核（建议8核以上）
- **内存**：至少8GB（建议16GB以上，每1GB内存可支持约5-10个并发连接）
- **存储**：
  - 系统分区：至少50GB
  - 数据分区：根据数据库规模规划，建议100GB起步
  - 日志分区：单独分区，建议数据空间的25%
- **网络**：千兆网卡（生产环境建议万兆）

**推荐生产环境配置**：

markdown



复制



下载

```
| 组件        | 小型环境       | 中型环境        | 大型环境        |
|-------------|---------------|----------------|----------------|
| CPU         | 8核           | 16核           | 32核+          |
| 内存        | 32GB          | 64GB           | 128GB+         |
| 存储类型    | SSD           | NVMe SSD       | 高性能存储阵列 |
| 网络带宽    | 1Gbps         | 10Gbps         | 10Gbps+        |
```

### 1.2 存储规划建议

1. **分区方案**：

   ```
/ (根分区)       50GB
   /var/opt/mssql  剩余空间的70%  # 数据文件
/var/log        剩余空间的20%  # 日志文件
   swap分区        内存大小的1-2倍
   ```
```
   
2. **文件系统配置**：

```
# 查看现有分区
   lsblk -f

   # 对新增磁盘分区并格式化为XFS（推荐）
sudo mkfs.xfs /dev/sdb1 -f
   sudo mkdir /var/opt/mssql
sudo mount /dev/sdb1 /var/opt/mssql

# 永久挂载配置
   echo "/dev/sdb1 /var/opt/mssql xfs defaults 0 0" | sudo tee -a /etc/fstab
   ```

## 二、操作系统准备

### 2.1 系统安装与基础配置

1. **麒麟系统版本确认**：

   ```
# 查看系统版本
   cat /etc/kylin-release
uname -a

# 要求：麒麟V10 SP1及以上，内核4.19+
   ```

2. **基础软件包安装**：

   ```
sudo apt-get update
   sudo apt-get install -y \
  curl \
     chrony \
  net-tools \
     telnet \
  traceroute \
     lsof \
  sysstat \
     iotop \
     htop \
     ncdu
   ```

### 2.2 系统安全加固

1. **防火墙配置**：

   ```
sudo systemctl enable firewalld
   sudo systemctl start firewalld

   # 预先开放必要端口
sudo firewall-cmd --permanent --add-port=1433/tcp  # SQL Server默认端口
   sudo firewall-cmd --permanent --add-port=5022/tcp  # AlwaysOn端点
sudo firewall-cmd --permanent --add-port=2224/tcp  # Pacemaker
   sudo firewall-cmd --permanent --add-port=5405/udp  # Corosync
sudo firewall-cmd --reload
   ```
   
2. **SELinux策略调整**：

   ```
# 查看当前状态
   getenforce

   # 临时设置为宽松模式
sudo setenforce 0

# 永久修改（需重启）
   sudo sed -i 's/SELINUX=enforcing/SELINUX=permissive/g' /etc/selinux/config
```

### 2.3 内核参数优化

编辑`/etc/sysctl.d/99-mssql.conf`：

```
# 内存相关
vm.swappiness = 1
vm.dirty_ratio = 80
vm.dirty_background_ratio = 5
vm.dirty_expire_centisecs = 12000

# 网络相关
net.core.somaxconn = 32768
net.ipv4.tcp_max_syn_backlog = 8096
net.ipv4.tcp_fin_timeout = 30

# 文件系统
fs.file-max = 6815744
fs.aio-max-nr = 1048576

# SQL Server专用
kernel.numa_balancing = 0
```

应用配置：

```
sudo sysctl -p /etc/sysctl.d/99-mssql.conf
```

## 三、软件依赖准备

### 3.1 Microsoft存储库配置

1. **信任微软GPG密钥**：

```
sudo curl -o /etc/apt/trusted.gpg.d/microsoft.asc https://packages.microsoft.com/keys/microsoft.asc
   ```

2. **添加SQL Server仓库**：

   ```
sudo curl -o /etc/apt/sources.list.d/mssql-server-2019.list \
     https://packages.microsoft.com/config/ubuntu/16.04/mssql-server-2019.list
```
   
3. **更新软件包索引**：

```
sudo apt-get update
   ```

### 3.2 依赖软件安装

   ```
# 基础依赖
sudo apt-get install -y \
  openssl \
  libssl-dev \
  libunwind8 \
  libicu60

# 高可用组件依赖
sudo apt-get install -y \
  pacemaker \
  corosync \
  fence-agents \
  resource-agents
```

## 四、环境验证

### 4.1 系统环境检查

1. **时间和时区设置**：

```
# 设置时区为上海
   sudo timedatectl set-timezone Asia/Shanghai

   # 启用NTP同步
sudo systemctl enable chronyd
   sudo systemctl start chronyd

   # 验证时间同步
chronyc sources -v
   ```
   
2. **内存和交换空间检查**：

   ```
free -h
   swapon --show
```

### 4.2 存储性能测试

```
# 安装测试工具
sudo apt-get install -y fio

# 随机读写测试（在数据目录执行）
fio --name=randwrite --ioengine=libaio --iodepth=16 \
  --rw=randwrite --bs=4k --direct=1 --size=1G --numjobs=4 \
  --runtime=60 --group_reporting
```

**预期指标**：

- 随机读IOPS > 5000（SSD）
- 随机写IOPS > 2000（SSD）
- 延迟 < 10ms

## 五、部署规划

### 5.1 网络拓扑规划
应用连接同步复制心跳检测客户端VIP: 192.168.1.100主节点: node1备用节点: node2

### 5.2 安装流程时序

1. 主节点安装SQL Server
2. 备节点安装SQL Server
3. 配置AlwaysOn可用性组
4. 部署Pacemaker集群
5. 配置故障转移资源
6. 验证高可用功能

## 六、检查清单

### 6.1 预安装检查表

✅ 硬件配置符合最低要求
✅ 操作系统版本已验证兼容
✅ 存储分区已按规范划分
✅ 网络端口已预先开放
✅ 系统时间已同步
✅ 内核参数已优化
✅ 防火墙策略已配置
✅ 微软仓库已正确添加

### 6.2 注意事项

1. **数据目录权限**：

```
sudo chown -R mssql:mssql /var/opt/mssql
   sudo chmod 750 /var/opt/mssql
```
   
2. **备份策略**：

   - 提前规划备份存储位置
   - 测试备份恢复流程

3. **监控方案**：

   - 部署Zabbix/Prometheus监控
   - 配置关键指标告警（CPU、内存、磁盘空间等）

## 七、问题排查准备

### 7.1 常见问题收集

1. **依赖缺失错误**：

```
   # 查找缺失库
   ldd /opt/mssql/bin/sqlservr | grep not

   # 解决方案
   sudo apt-get install -y libssl1.0.0
   ```

2. **端口冲突处理**：

   bash

   

   复制

   

   下载

   ```
   # 查看端口占用
   sudo netstat -tulnp | grep 1433

   # 解决方案
   sudo /opt/mssql/bin/mssql-conf set network.tcpport <新端口>
   ```

### 7.2 日志文件位置

| 组件       | 日志路径                      |
| :--------- | :---------------------------- |
| SQL Server | /var/opt/mssql/log/errorlog   |
| Pacemaker  | /var/log/cluster/corosync.log |
| 系统日志   | /var/log/messages             |

通过完成以上准备工作，可确保后续SQL Server的安装和高可用配置顺利进行。建议在正式部署前，在测试环境完整演练所有步骤。
   ```