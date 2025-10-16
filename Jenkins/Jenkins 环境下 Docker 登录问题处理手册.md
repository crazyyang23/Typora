# **Jenkins 环境下 Docker 登录问题处理手册**

# **Jenkins 环境下 Docker 登录问题处理手册**

## **问题描述**

在 Jenkins 环境中执行 `docker login` 时遇到以下错误：

text



复制



下载

```
time="2025-07-30T17:35:14+08:00" level=info msg="Error logging in to endpoint, trying next endpoint" error="Get \"https://10.2.233.125/v2/\": dial tcp 10.2.233.125:443: connect: connection refused"
```

或：

text



复制



下载

```
sudo: 读取密码需要一个终端；请使用 -S 选项以从标准输入进行读取，或者配置一个 askpass 助手程序
sudo: 需要密码
```

------

## **1. 问题分析**

### **1.1 主要错误原因**

| 错误现象                         | 可能原因                                     | 影响                |
| :------------------------------- | :------------------------------------------- | :------------------ |
| `connection refused` (HTTPS 443) | Docker 默认尝试 HTTPS，但 Harbor 仅支持 HTTP | Docker 无法连接仓库 |
| `sudo: 需要密码`                 | Jenkins 非交互模式无法输入 `sudo` 密码       | 命令执行失败        |
| `unauthorized`                   | 用户名/密码错误或权限不足                    | 无法登录 Harbor     |

### **1.2 解决思路**

1. **允许 Docker 使用 HTTP**（关键）
2. **避免 `sudo` 或配置 `sudo` 免密码**
3. **确保网络和 Harbor 服务正常**

------

## **2. 解决方案**

### **2.1 允许 Docker 使用 HTTP**

在 **所有运行 Docker 的机器（包括 Jenkins 节点）** 上执行：

bash

```
# 配置 Docker 允许 HTTP 仓库
sudo tee /etc/docker/daemon.json <<EOF
{
  "insecure-registries": ["10.2.233.125"]
}
EOF

# 重启 Docker
sudo systemctl restart docker

# 验证配置
docker info | grep "Insecure Registries"
```

**预期输出**：`Insecure Registries: 10.2.233.125`

------

### **2.2 修改 `docker login` 命令**

在 Jenkins 脚本中使用以下格式（避免 `http://` 前缀）：

bash

```
echo "Harbor@12345" | docker login 10.2.233.125 --username=admin --password-stdin
```

或显式指定协议：

bash

```
echo "Harbor@12345" | docker login --username=admin --password-stdin http://10.2.233.125
```

------

### **2.3 避免 `sudo`（推荐）**

将 Jenkins 用户加入 `docker` 组：

bash

```
sudo usermod -aG docker jenkins
sudo systemctl restart jenkins
```

**验证**：

bash

```
groups jenkins  # 应包含 "docker"
```

------

### **2.4 配置 `sudo` 免密码（如需）**

如果必须使用 `sudo`：

bash

```
sudo visudo
```

添加以下行（按需调整命令路径）：

text

```
jenkins ALL=(ALL) NOPASSWD: /usr/bin/docker, /bin/systemctl restart docker
```

**保存后验证**：

bash

```
sudo -U jenkins -l
```

------

### **2.5 检查 Harbor 服务状态**

在 Harbor 服务器上执行：

bash

```
# 检查 Harbor 是否运行
docker-compose -f /path/to/harbor/docker-compose.yml ps

# 检查 HTTP 端口
netstat -tuln | grep 80
```

------

## **3. 验证步骤**

### **3.1 手动测试 Docker 登录**

bash

```
docker login 10.2.233.125 --username=admin --password="Harbor@12345"
```

### **3.2 测试镜像推送**

bash

```
docker pull hello-world
docker tag hello-world 10.2.233.125/library/hello-world
docker push 10.2.233.125/library/hello-world
```

### **3.3 检查网络连通性**

bash

```
curl -v http://10.2.233.125/v2/_catalog
telnet 10.2.233.125 80
```

------

## **4. 常见问题排查**

| 问题                 | 检查命令                                  | 解决方案                |
| :------------------- | :---------------------------------------- | :---------------------- |
| `connection refused` | `telnet 10.2.233.125 80`                  | 检查 Harbor 服务/防火墙 |
| `unauthorized`       | `curl -v http://10.2.233.125/v2/_catalog` | 验证用户名/密码         |
| `sudo: 需要密码`     | `sudo -U jenkins -l`                      | 配置 `visudo` 免密码    |
| `permission denied`  | `groups jenkins`                          | 将用户加入 `docker` 组  |

------

## **5. 附录**

### **5.1 安全注意事项**

- 禁止在脚本中硬编码密码！
- 使用 `visudo` 时仅授权必要命令。
- 优先通过 `docker` 组权限替代 `sudo`。

### **5.2 相关配置文件**

| 文件                         | 作用                  |
| :--------------------------- | :-------------------- |
| `/etc/docker/daemon.json`    | 配置 Docker 允许 HTTP |
| `/etc/sudoers`               | 管理 `sudo` 权限      |
| `/path/to/harbor/harbor.yml` | Harbor 服务配置       |

------

**问题仍未解决？** 请提供以下信息：

1. `docker info` 输出
2. `sudo -U jenkins -l` 完整错误
3. `curl -v http://10.2.233.125/v2/_catalog` 结果

------

**文档版本**：1.0
**最后更新**：2025-08-01
**适用环境**：Jenkins + Docker + Harbor HTTP 模式