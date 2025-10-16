# Jenkins 安装与部署手册

## 一、环境要求

| 组件     | 版本要求                | 说明                                          |
| -------- | ----------------------- | --------------------------------------------- |
| 操作系统 | CentOS 8+/Ubuntu 20.04+ | 推荐使用 64 位系统                            |
| Java     | JDK 17/21               | Jenkins 自 2023 年起逐步淘汰对 Java 11 的支持 |
| 内存     | ≥2GB                    | 建议至少 2GB 内存，否则可能影响性能           |
| 端口     | 8080（默认）            | 需确保端口未被占用                            |

## 二、安装步骤（以 CentOS 为例）

### 2.1 安装 Java 17

bash











```bash
# 添加EPEL仓库（若未安装）
sudo yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm

# 安装OpenJDK 17
sudo yum install -y java-17-openjdk-devel

# 切换默认Java版本（若存在多版本）
sudo alternatives --config java
# 选择Java 17对应的路径（编号通常为2）
```

### 2.2 安装 Jenkins

bash











```bash
# 添加Jenkins官方仓库
sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo

# 导入GPG密钥
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key

# 安装Jenkins
sudo yum install -y jenkins
```

### 2.3 启动 Jenkins 服务

bash











```bash
# 启动服务
sudo systemctl start jenkins

# 检查服务状态（确保Active为running）
sudo systemctl status jenkins

# 设置开机自启（可选）
sudo systemctl enable jenkins
```

## 三、首次初始化配置

### 3.1 获取初始管理员密码

bash











```bash
# 查看密码
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

### 3.2 访问 Web 界面

- 打开浏览器访问 `http://服务器IP:8080`
- 输入密码后，选择「安装推荐的插件」或自定义插件安装
- 创建管理员用户并完成配置

## 四、常见问题及处理方案

### 4.1 问题：Jenkins 服务启动失败（Java 版本不兼容）

#### 现象

log











```log
Supported Java versions are: [17, 21]
jenkins.service: Main process exited, code=exited, status=1/FAILURE
```

#### 原因

当前 Java 版本低于 17，Jenkins 不支持。

#### 解决步骤

1. 安装 Java 17/21（参考 2.1 节）

2. 修改 Jenkins 配置文件指定 Java 路径：

   bash

   

   

   

   

   

   ```bash
   sudo nano /etc/default/jenkins
   # 添加或修改：JAVA_HOME=$(readlink -f /usr/bin/java | sed "s:bin/java::")
   ```

3. 重启服务：`sudo systemctl restart jenkins`

### 4.2 问题：端口冲突（8080 被占用）

#### 现象

log











```log
Address already in use: Jenkins (HTTP port 8080)
```

#### 解决步骤

1. 查看端口占用：`sudo lsof -i :8080`

2. 杀死占用进程（示例）：`sudo kill $(lsof -t -i :8080)`

3. 或修改 Jenkins 端口：

   bash

   

   

   

   

   

   ```bash
   sudo nano /etc/default/jenkins
   # 修改HTTP_PORT=8081（自定义未占用端口）
   ```

### 4.3 问题：文件权限不足

#### 现象

log











```log
Permission denied: /var/lib/jenkins/secrets/initialAdminPassword
```

#### 解决步骤

bash











```bash
# 重置Jenkins目录权限
sudo chown -R jenkins:jenkins /var/lib/jenkins
sudo chown -R jenkins:jenkins /var/log/jenkins
sudo chown -R jenkins:jenkins /var/cache/jenkins
```

### 4.4 问题：服务启动频繁失败（触发 systemd 保护机制）

#### 现象

log











```log
Start request repeated too quickly
jenkins.service: Failed with result 'exit-code'
```

#### 解决步骤

1. 重置失败计数器：`sudo systemctl reset-failed jenkins`

2. 手动启动并查看详细日志：

   bash

   

   

   

   

   

   ```bash
   sudo systemctl start jenkins
   sudo journalctl -u jenkins -n 100 --no-pager
   ```

## 五、日志查看指南

| 日志类型     | 路径                                            | 说明                        |
| ------------ | ----------------------------------------------- | --------------------------- |
| 系统服务日志 | `sudo journalctl -u jenkins`                    | 查看 systemd 管理的启动日志 |
| 应用运行日志 | `/var/log/jenkins/jenkins.log`                  | 记录 Jenkins 核心运行日志   |
| 初始化密码   | `/var/lib/jenkins/secrets/initialAdminPassword` | 首次启动生成的管理员密码    |

## 六、维护命令

| 操作         | 命令                             |
| ------------ | -------------------------------- |
| 启动服务     | `sudo systemctl start jenkins`   |
| 停止服务     | `sudo systemctl stop jenkins`    |
| 重启服务     | `sudo systemctl restart jenkins` |
| 查看状态     | `sudo systemctl status jenkins`  |
| 设置开机自启 | `sudo systemctl enable jenkins`  |
| 关闭开机自启 | `sudo systemctl disable jenkins` |

## 七、升级指南

1. 备份数据：

   bash

   ```bash
sudo cp -r /var/lib/jenkins /var/lib/jenkins_backup_$(date +%Y%m%d)
   ```

2. 更新 Jenkins：

   bash

   ```bash
sudo yum update jenkins
   ```

3. 重启服务：`sudo systemctl restart jenkins`

## 八、安全建议

1. 防火墙配置

   ：

   bash

   ```bash
   
   ```
# CentOS（Firewalld）
   sudo firewall-cmd --permanent --add-port=8080/tcp
sudo firewall-cmd --reload

# Ubuntu（UFW）
   sudo ufw allow 8080
```
   

   
2. **禁用匿名访问**：Jenkins 系统设置→全局安全配置→勾选「启用安全」并配置权限

3. **定期更新插件和系统**：避免使用过时组件带来的安全风险

## 九、参考链接

- Jenkins 官方文档：https://www.jenkins.io/doc/
- Java 支持政策：https://jenkins.io/redirect/java-support/
- 插件市场：https://plugins.jenkins.io/



**注**：部署过程中如遇特殊问题，可通过 `jenkins.log` 或官方社区获取进一步支持。





# Jenkins 安装与部署手册（Ubuntu 20.04/22.04）

## 一、环境要求

| 组件     | 版本要求      | 说明                                          |
| -------- | ------------- | --------------------------------------------- |
| 操作系统 | Ubuntu 20.04+ | 推荐使用 64 位系统                            |
| Java     | JDK 17/21     | Jenkins 自 2023 年起逐步淘汰对 Java 11 的支持 |
| 内存     | ≥2GB          | 建议至少 2GB 内存，否则可能影响性能           |
| 端口     | 8080（默认）  | 需确保端口未被占用                            |

## 二、安装步骤（以 Ubuntu 22.04 为例）

### 2.1 安装 Java 17

bash











​```bash
# 更新系统包列表
sudo apt update

# 安装 OpenJDK 17（推荐 Adoptium 版本）
sudo apt install -y openjdk-17-jdk

# 验证 Java 版本
java -version
# 应输出：openjdk version "17.x.x"
```

### 2.2 安装 Jenkins

#### 方法 1：使用官方 APT 源（推荐）

bash











```bash
# 导入 Jenkins GPG 密钥
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo tee /usr/share/keyrings/jenkins-keyring.asc > /dev/null

# 添加 Jenkins 官方源
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/ | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null

# 更新系统包列表并安装 Jenkins
sudo apt update
sudo apt install -y jenkins
```

#### 方法 2：使用 WAR 包安装（可选）

bash











```bash
# 下载最新 WAR 包（查看官网获取最新版本）
wget https://get.jenkins.io/war-stable/latest/jenkins.war -O /usr/share/java/jenkins.war

# 创建 Jenkins 服务文件（可选，用于系统管理）
sudo nano /etc/systemd/system/jenkins.service
```



ini











```ini
[Unit]
Description=Jenkins Continuous Integration Server
After=network.target

[Service]
User=jenkins
Group=jenkins
Environment="JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64"
ExecStart=/usr/bin/java -Djava.awt.headless=true -jar /usr/share/java/jenkins.war --webroot=/var/cache/jenkins/war --httpPort=8080
Restart=always
LimitNOFILE=1000000

[Install]
WantedBy=multi-user.target
```

### 2.3 启动 Jenkins 服务

bash











```bash
# 启动服务
sudo systemctl start jenkins

# 检查服务状态（确保 Active 为 running）
sudo systemctl status jenkins

# 设置开机自启（可选）
sudo systemctl enable jenkins
```

## 三、首次初始化配置

### 3.1 获取初始管理员密码

bash

```bash
# 查看密码（首次启动后生成）
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

### 3.2 访问 Web 界面

- 打开浏览器访问 `http://服务器IP:8080`
- 输入密码后，选择「安装推荐的插件」或自定义插件安装
- 创建管理员用户并完成配置

## 四、常见问题及处理方案

### 4.1 问题：Java 版本不兼容

#### 现象

log

```log
Supported Java versions are: [17, 21]
jenkins.service: Main process exited, code=exited, status=1/FAILURE
```

#### 原因

当前 Java 版本低于 17，Jenkins 不支持。

#### 解决步骤

1. 卸载旧版本 Java：

   bash

   ```bash
sudo apt remove -y openjdk-11*
   ```

2. 安装 Java 17（参考 2.1 节）

3. 确认 Jenkins 配置文件指向 Java 17：

   bash

   ```bash
sudo nano /etc/default/jenkins
   # 确保 JAVA_HOME 指向 Java 17 路径（如：/usr/lib/jvm/java-17-openjdk-amd64）
   ```
```

### 4.2 问题：端口冲突（8080 被占用）

#### 现象

log

​```log
Address already in use: Jenkins (HTTP port 8080)
```

#### 解决步骤

1. 查看端口占用：

   bash

   ```bash
sudo lsof -i :8080
   ```

2. 杀死占用进程（示例）：

   bash

   ```bash
sudo kill $(lsof -t -i :8080)
   ```

3. 或修改 Jenkins 端口：

   bash

   ```bash
sudo nano /etc/default/jenkins
   # 修改 HTTP_PORT=8081（自定义未占用端口）
   ```
```

### 4.3 问题：文件权限不足

#### 现象

log

​```log
Permission denied: /var/lib/jenkins/secrets/initialAdminPassword
```

#### 解决步骤

bash

```bash
# 重置 Jenkins 目录权限
sudo chown -R jenkins:jenkins /var/lib/jenkins
sudo chown -R jenkins:jenkins /var/log/jenkins
sudo chown -R jenkins:jenkins /var/cache/jenkins
```

### 4.4 问题：服务启动失败（systemd 保护机制触发）

#### 现象

log

```log
Start request repeated too quickly
jenkins.service: Failed with result 'exit-code'
```

#### 解决步骤

1. 重置失败计数器：

   bash

   ```bash
sudo systemctl reset-failed jenkins
   ```

2. 手动启动并查看详细日志：

   bash

   ```bash
sudo systemctl start jenkins
   sudo journalctl -u jenkins -n 100 --no-pager
   ```
```

## 五、日志查看指南

| 日志类型     | 路径                                            | 说明                        |
| ------------ | ----------------------------------------------- | --------------------------- |
| 系统服务日志 | `sudo journalctl -u jenkins`                    | 查看 systemd 管理的启动日志 |
| 应用运行日志 | `/var/log/jenkins/jenkins.log`                  | 记录 Jenkins 核心运行日志   |
| 初始化密码   | `/var/lib/jenkins/secrets/initialAdminPassword` | 首次启动生成的管理员密码    |

## 六、维护命令

| 操作         | 命令                             |
| ------------ | -------------------------------- |
| 启动服务     | `sudo systemctl start jenkins`   |
| 停止服务     | `sudo systemctl stop jenkins`    |
| 重启服务     | `sudo systemctl restart jenkins` |
| 查看状态     | `sudo systemctl status jenkins`  |
| 设置开机自启 | `sudo systemctl enable jenkins`  |
| 关闭开机自启 | `sudo systemctl disable jenkins` |

## 七、升级指南

1. 备份数据

   ：

   bash

   ```bash
sudo cp -r /var/lib/jenkins /var/lib/jenkins_backup_$(date +%Y%m%d)
```

2. 更新 Jenkins

   ：

   bash

   ```bash
sudo apt update
   sudo apt upgrade -y jenkins
   ```
```
   
3. 重启服务

   ：

   bash

   ```bash
sudo systemctl restart jenkins
```

## 八、安全建议

1. 防火墙配置

   ：

   bash

   ```bash
   
   ```
# 允许 8080 端口（UFW 防火墙）
   sudo ufw allow 8080/tcp
sudo ufw reload
   ```

   

2. **禁用匿名访问**：
   进入 Jenkins 系统设置 → 全局安全配置 → 勾选「启用安全」并配置权限。

3. **定期更新插件和系统**：
   避免使用过时组件带来的安全风险。

## 九、参考链接

- Jenkins 官方文档：https://www.jenkins.io/doc/
- Java 支持政策：https://jenkins.io/redirect/java-support/
- 插件市场：https://plugins.jenkins.io/

sudo usermod -aG docker jenkins 

**注**：部署过程中如遇特殊问题，可通过 `jenkins.log` 或官方社区获取进一步支持。
   ```