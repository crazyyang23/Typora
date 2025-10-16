# **CentOS 7.9 离线安装 MSSQL-Tools 18 操作手册**

## **1. 环境准备**

- **操作系统**：CentOS 7.9
- **目标软件**：
  - `msodbcsql18`（ODBC 驱动）
  - `mssql-tools18`（`sqlcmd` 和 `bcp`）
- **依赖项**：
  - `unixODBC >= 2.3.1`
  - `unixODBC-devel`
  - `libatomic`（可选，部分环境需要）

------

## **2. 在线环境下载 RPM 包**

### **(1) 添加 Microsoft 仓库**

bash



复制



下载

```
# 添加 Microsoft SQL Server 仓库
sudo curl -o /etc/yum.repos.d/mssql-server.repo https://packages.microsoft.com/config/rhel/7/mssql-server-2019.repo
sudo curl -o /etc/yum.repos.d/msprod.repo https://packages.microsoft.com/config/rhel/7/prod.repo

# 更新 yum 缓存
sudo yum clean all
sudo yum makecache
```

### **(2) 下载 RPM 包**

bash



复制



下载

```
mkdir -p ~/mssql-offline
cd ~/mssql-offline

# 下载核心组件
yum install --downloadonly --downloaddir=./ msodbcsql18 mssql-tools18

# 下载依赖项
yum install --downloadonly --downloaddir=./ unixODBC unixODBC-devel libatomic

# 如果 `yum downloadonly` 失败，手动下载
wget https://packages.microsoft.com/rhel/7/prod/msodbcsql18-18.3.2.1-1.x86_64.rpm
wget https://packages.microsoft.com/rhel/7/prod/mssql-tools18-18.2.1.1-1.x86_64.rpm
wget http://mirror.centos.org/centos/7/os/x86_64/Packages/unixODBC-2.3.1-14.el7.x86_64.rpm
wget http://mirror.centos.org/centos/7/os/x86_64/Packages/unixODBC-devel-2.3.1-14.el7.x86_64.rpm
wget http://mirror.centos.org/centos/7/os/x86_64/Packages/libatomic-4.8.5-44.el7.x86_64.rpm
```

### **(3) 打包 RPM 文件**

bash



复制



下载

```
tar -czvf mssql-offline-rpms.tar.gz *.rpm
```

------

## **3. 离线环境安装**

### **(1) 传输 RPM 包到目标机器

```
scp mssql-offline-rpms.tar.gz user@offline-server:/tmp/
```

或使用 U 盘复制。

### **(2) 解压并安装

```
cd /tmp
tar -xzvf mssql-offline-rpms.tar.gz

# 安装依赖项
sudo rpm -ivh unixODBC-2.3.1-14.el7.x86_64.rpm
sudo rpm -ivh unixODBC-devel-2.3.1-14.el7.x86_64.rpm
sudo rpm -ivh libatomic-*.rpm  # 如果存在

# 安装 MSSQL 组件（必须接受 EULA）
sudo ACCEPT_EULA=Y rpm -ivh msodbcsql18-18.3.2.1-1.x86_64.rpm
sudo ACCEPT_EULA=Y rpm -ivh mssql-tools18-18.2.1.1-1.x86_64.rpm
```

### **(3) 配置环境变量

```
echo 'export PATH="$PATH:/opt/mssql-tools/bin"' >> ~/.bashrc
source ~/.bashrc
```

------

## **4. 验证安装

```
# 检查 sqlcmd 和 bcp
sqlcmd -?
bcp -v

# 检查 ODBC 驱动
odbcinst -q -d -n "ODBC Driver 18 for SQL Server"
```

------

## **5. 常见问题解决**

| **问题**                      | **解决方案**                   |
| :---------------------------- | :----------------------------- |
| `unixODBC >= 2.3.1 is needed` | 确保先安装 `unixODBC-2.3.1`    |
| `libatomic.so.1 is missing`   | 安装 `libatomic-*.rpm`         |
| `ACCEPT_EULA not set`         | 添加 `ACCEPT_EULA=Y` 参数      |
| `file conflicts`              | 使用 `rpm -ivh --replacefiles` |

------

## **6. 卸载方法**

bash



复制



下载

```
sudo rpm -e msodbcsql18 mssql-tools18 unixODBC-devel unixODBC
```

------

## **7. 附录**

### **(1) 关键文件路径**

- `sqlcmd`：`/opt/mssql-tools/bin/sqlcmd`
- `bcp`：`/opt/mssql-tools/bin/bcp`
- ODBC 配置：`/etc/odbcinst.ini`

### **(2) 离线安装逻辑**



```
graph TD
    A[在线环境] -->|下载 RPM| B[打包传输]
    B --> C[离线环境]
    C --> D[安装依赖]
    D --> E[安装 MSSQL Tools]
    E --> F[验证]
```





下载

下载 RPM在线环境打包传输离线环境安装依赖安装 MSSQL Tools验证

------

**完成！** 您现在可以在 CentOS 7.9 上使用 `sqlcmd` 和 `bcp` 了。🚀