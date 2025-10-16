# SQL Server Always On 可用性组端点及加密配置操作手册

## 一、适用场景

本手册适用于需要重新配置 SQL Server Always On 可用性组端点及加密功能，解决因证书、密钥配置问题导致端点创建失败的场景。

## 二、前提条件

1. 已安装并启动 SQL Server 服务，且各节点已配置为 Always On 可用性组的潜在成员。
2. 确保各节点间网络连通，端口 5022 未被占用且防火墙已开放此端口。
3. 具备数据库管理员权限，能够执行 SQL 语句及 Linux 系统命令。

## 三、清理现有配置

### 3.1 删除所有副本上的端点

在主副本和所有辅助副本的 SQL Server 中，依次执行以下 SQL 语句：



sql











```sql
IF EXISTS (SELECT * FROM sys.database_mirroring_endpoints WHERE name = 'AG_Endpoint')
    DROP ENDPOINT [AG_Endpoint];
GO
```

### 3.2 删除所有副本上的证书

同样在各节点执行：



sql











```sql
IF EXISTS (SELECT * FROM sys.certificates WHERE name = 'AG_Certificate')
    DROP CERTIFICATE AG_Certificate;
GO
```

### 3.3 删除主密钥（可选，但建议）

在各节点执行：



sql











```sql
IF EXISTS (SELECT * FROM sys.symmetric_keys WHERE name = '##MS_DatabaseMasterKey##')
    DROP MASTER KEY;
GO
```

### 3.4 删除物理文件（可选）

在 Linux 系统下，通过 SSH 登录各节点，执行以下命令删除旧的证书和密钥文件：



bash











```bash
rm -f /var/opt/mssql/data/AG_Certificate.*
rm -f /var/opt/mssql/data/AG_MasterKey.*
```

## 四、重新配置加密和端点

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



sql











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

## 六、后续步骤

1. 在主副本上备份数据库和事务日志。
2. 将备份文件还原到辅助副本（使用 `NORECOVERY` 选项）。
3. 使用 SQL Server Management Studio（SSMS）或 T-SQL 创建可用性组并添加副本。

## 七、常见问题及解决

1. **问题**：证书状态检查时，`pvt_key_encryption_type_desc` 显示为 `NO_PRIVATE_KEY`。
   **解决**：重新执行证书创建及备份步骤，确保证书包含私钥并正确备份和导入。
2. **问题**：端点创建失败，提示证书无效。
   **解决**：检查证书有效期、私钥状态，确保各节点证书配置一致，且文件权限正确（证书文件权限应为 `600`，属主为 `mssql`）。



若在操作过程中遇到其他问题，可查看 SQL Server 错误日志（路径：`/var/opt/mssql/log/errorlog`）获取详细报错信息，并针对性解决。