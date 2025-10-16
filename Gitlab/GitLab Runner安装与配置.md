### GitLab Runner 安装与配置手册

#### **一、概述**

GitLab Runner 是 GitLab CI/CD 的执行组件，负责运行由 `.gitlab-ci.yml` 定义的作业。本手册详细介绍在 Linux 系统上安装、配置和使用 GitLab Runner 的步骤。

#### **二、系统要求**

- **操作系统**：Ubuntu/Debian/CentOS/RHEL 等。
- **内存**：至少 1GB RAM（生产环境建议 2GB+）。
- **磁盘空间**：至少 1GB 可用空间。
- **网络**：需访问 GitLab 服务器（如 `https://gitlab.com`）。

#### **三、安装步骤**

##### **1. 安装前准备**

bash

```bash
# Ubuntu/Debian
sudo apt-get update
sudo apt-get install -y curl gnupg2 ca-certificates apt-transport-https

# CentOS/RHEL
sudo yum install -y curl policycoreutils-python openssl-devel
```

##### **2. 添加 GitLab 官方仓库**

bash

```bash
# Ubuntu/Debian
curl -L "https://packages.gitlab.com/gpg.key" | sudo apt-key add -
sudo curl -L --output /etc/apt/sources.list.d/runner_gitlab-runner.list \
  "https://packages.gitlab.com/runner/gitlab-runner/debian/config_file.list?os=debian&dist=bullseye"
sudo apt-get update

# CentOS/RHEL
sudo tee /etc/yum.repos.d/gitlab-runner.repo <<EOF
[gitlab-runner]
name=GitLab Runner
baseurl=https://packages.gitlab.com/runner/gitlab-runner/el/7/\$basearch
repo_gpgcheck=1
gpgcheck=1
enabled=1
gpgkey=https://packages.gitlab.com/gpg.key https://gitlab.com/gitlab-org/omnibus-gitlab/raw/master/docker/images/gitlab.gpg
sslverify=1
sslcacert=/etc/pki/tls/certs/ca-bundle.crt
metadata_expire=300
EOF
```

##### **3. 安装 GitLab Runner**

bash

```bash
# 安装最新稳定版
# Ubuntu/Debian
sudo apt-get install gitlab-runner

# CentOS/RHEL
sudo yum install gitlab-runner

# 安装指定版本（例如 15.11.0）
sudo apt-get install gitlab-runner=15.11.0  # Ubuntu/Debian
sudo yum install gitlab-runner-15.11.0  # CentOS/RHEL
```

#### **四、配置步骤**

##### **1. 启动并启用服务**

bash

```bash
sudo systemctl start gitlab-runner
sudo systemctl enable gitlab-runner
sudo systemctl status gitlab-runner  # 验证状态
```

##### **2. 获取注册令牌**

1. 登录 GitLab → 项目主页 → Settings → CI/CD → Runners → Expand。
2. 复制 `Registration token`。

##### **3. 注册 Runner**

bash

```bash
sudo gitlab-runner register
```



按提示输入：



- GitLab instance URL（如 `https://gitlab.com`）
- Registration token
- Runner 描述
- Tags（如 `docker,build`）
- 是否执行未标记作业（`true`/`false`）
- 是否锁定 Runner（`true`/`false`）
- 执行器（如 `docker`）
- Docker 镜像（如 `alpine:latest`）

##### **4. 高级配置（`config.toml`）**

bash

```bash
sudo nano /etc/gitlab-runner/config.toml
```



**示例配置**：



toml

```toml
concurrent = 2
check_interval = 0

[[runners]]
  name = "docker-runner"
  url = "https://gitlab.com"
  token = "YOUR_RUNNER_TOKEN"
  executor = "docker"
  [runners.docker]
    image = "docker:20.10.18"
    privileged = false
    volumes = ["/cache", "/var/run/docker.sock:/var/run/docker.sock"]
  [runners.cache]
    Type = "s3"
    Path = "cache"
    Shared = true
    [runners.cache.s3]
      ServerAddress = "s3.amazonaws.com"
      AccessKey = "YOUR_ACCESS_KEY"
      SecretKey = "YOUR_SECRET_KEY"
      BucketName = "gitlab-runner-cache"
```

#### **五、验证与使用**

##### **1. 验证安装**

bash

```bash
sudo gitlab-runner list  # 查看已注册 Runner
sudo gitlab-runner verify  # 验证连接
```

##### **2. 在 GitLab 界面检查状态**

项目主页 → Settings → CI/CD → Runners → 确认状态为 **green**。

##### **3. 在 `.gitlab-ci.yml` 中使用**

yaml

```yaml
build:
  stage: build
  tags:
    - docker  # 匹配 Runner 标签
  script:
    - echo "Building..."
```

#### **六、常见问题处理**

| 问题                   | 解决方案                                                     |
| ---------------------- | ------------------------------------------------------------ |
| Runner 无法连接 GitLab | 检查网络、防火墙（开放 443 端口）、SELinux（临时禁用：`sudo setenforce 0`） |
| Docker 权限错误        | `sudo usermod -aG docker gitlab-runner` 并重启服务           |
| 查看日志               | `sudo journalctl -u gitlab-runner -f` 或 `sudo gitlab-runner exec docker job-name` |

#### **七、维护与升级**

bash

```bash
# 升级到最新版本
sudo apt-get update && sudo apt-get install --only-upgrade gitlab-runner  # Ubuntu/Debian
sudo yum update gitlab-runner  # CentOS/RHEL

# 卸载 Runner
sudo systemctl stop gitlab-runner
sudo apt-get remove gitlab-runner  # Ubuntu/Debian
sudo yum remove gitlab-runner  # CentOS/RHEL
```

#### **八、附录：关键参数说明**

| 参数         | 描述                                         |
| ------------ | -------------------------------------------- |
| `concurrent` | 全局并发作业数                               |
| `image`      | 默认 Docker 镜像                             |
| `privileged` | 是否以特权模式运行容器                       |
| `volumes`    | 挂载主机目录到容器                           |
| `shm_size`   | 共享内存大小（字节）                         |
| `cache`      | 配置缓存存储（支持 S3、GCS、本地文件系统等） |



**手册结束**
如有疑问，请参考 GitLab 官方文档：https://docs.gitlab.com/runner/

### GitLab Runner 安装与配置常见问题及解决方案

#### **一、安装阶段问题**

##### **1. 仓库添加失败**

**问题表现**：



- 添加 GitLab 官方仓库时出现 GPG 密钥验证失败。
- `apt-get update` 或 `yum update` 报错。



**解决方案**：

```bash
# 手动导入 GPG 密钥
curl -L "https://packages.gitlab.com/gpg.key" | sudo gpg --dearmor -o /usr/share/keyrings/gitlab-runner-archive-keyring.gpg

# Ubuntu/Debian 重新添加仓库
echo "deb [signed-by=/usr/share/keyrings/gitlab-runner-archive-keyring.gpg] https://packages.gitlab.com/runner/gitlab-runner/debian bullseye main" | sudo tee /etc/apt/sources.list.d/runner_gitlab-runner.list

# CentOS/RHEL 检查 repo 文件权限
sudo chmod 644 /etc/yum.repos.d/gitlab-runner.repo
```

##### **2. 版本冲突**

**问题表现**：



- 无法安装指定版本的 Runner。
- 提示依赖冲突（如 `docker` 版本不兼容）。



**解决方案**：

```bash
# 查看可用版本
apt-cache madison gitlab-runner  # Ubuntu/Debian
yum list gitlab-runner --showduplicates  # CentOS/RHEL

# 安装指定版本（例如 15.11.0）
sudo apt-get install gitlab-runner=15.11.0-1  # 指定完整版本号
```

#### **二、注册与配置阶段问题**

##### **1. 注册令牌无效**

**问题表现**：



- `gitlab-runner register` 时提示 `ERROR: Registering runner... forbidden (check registration token)`。



**解决方案**：



1. 确认令牌是否过期（项目设置 → CI/CD → Runners → 刷新令牌）。
2. 检查是否有多余空格或特殊字符。
3. 尝试使用全局注册令牌（管理员权限 → Admin Area → Runners）。

##### **2. 无法连接到 GitLab 服务器**

**问题表现**：



- 注册时提示 `ERROR: Registering runner... connection refused`。



**解决方案**：

```bash
# 检查网络连通性
ping gitlab.com  # 或私有 GitLab 域名
traceroute gitlab.com

# 检查防火墙规则
sudo ufw allow 443/tcp  # Ubuntu/Debian
sudo firewall-cmd --add-service=https --permanent  # CentOS/RHEL
sudo firewall-cmd --reload

# 检查 DNS 设置
cat /etc/resolv.conf
```

##### **3. 配置文件权限错误**

**问题表现**：



- Runner 启动失败，日志显示 `ERROR: Failed to load config stat /etc/gitlab-runner/config.toml: permission denied`。



**解决方案**：

```bash
# 修复权限
sudo chown gitlab-runner:gitlab-runner /etc/gitlab-runner/config.toml
sudo chmod 600 /etc/gitlab-runner/config.toml
```

#### **三、执行器相关问题**

##### **1. Docker 执行器权限问题**

**问题表现**：



- 作业失败，错误信息为 `Cannot connect to the Docker daemon`。



**解决方案**：

```bash
# 将 gitlab-runner 用户添加到 docker 组
sudo usermod -aG docker gitlab-runner

# 重启服务
sudo systemctl restart gitlab-runner

# 验证权限（无需 sudo 运行 docker）
su - gitlab-runner -c "docker info"
```

##### **2. 容器无法访问网络**

**问题表现**：



- 容器内无法访问外部网络（如 `ping google.com` 失败）。



**解决方案**：

```bash
# 检查 Docker 网络配置
docker network inspect bridge

# 重启 Docker 服务
sudo systemctl restart docker

# 修改 Docker 网络 MTU（如果是 VPN 环境）
sudo tee /etc/docker/daemon.json <<EOF
{
  "mtu": 1400
}
EOF
sudo systemctl restart docker
```

#### **四、服务与性能问题**

##### **1. Runner 服务无法启动**

**问题表现**：



- `systemctl start gitlab-runner` 失败。



**解决方案**：

```bash
# 查看详细错误日志
sudo journalctl -u gitlab-runner -xe

# 检查进程是否已运行
ps aux | grep gitlab-runner

# 手动启动排查
sudo gitlab-runner run --user=gitlab-runner --working-directory=/home/gitlab-runner
```

##### **2. 并发作业数限制**

**问题表现**：



- 多个作业排队，未并行执行。



**解决方案**：

```bash
# 修改 config.toml 增加并发数
sudo nano /etc/gitlab-runner/config.toml
concurrent = 4  # 根据服务器资源调整

# 重启服务
sudo systemctl restart gitlab-runner
```

#### **五、缓存与存储问题**

##### **1. 缓存不生效**

**问题表现**：



- 作业每次都重新下载依赖（如 npm、pip）。



**解决方案**：

```toml
# 检查 config.toml 缓存配置
[runners.cache]
  Type = "s3"  # 或 filesystem
  Path = "cache"
  Shared = true
  [runners.cache.s3]
    ServerAddress = "s3.amazonaws.com"
    AccessKey = "YOUR_KEY"
    SecretKey = "YOUR_SECRET"
    BucketName = "gitlab-runner-cache"
```

##### **2. 磁盘空间不足**

**问题表现**：



- 作业因磁盘空间不足失败。



**解决方案**：

```bash
# 清理 Docker 缓存
docker system prune -a --volumes

# 调整缓存大小
sudo nano /etc/gitlab-runner/config.toml
[runners.docker]
  disable_cache = false
  volumes = ["/cache:rw"]  # 挂载更大的磁盘
```

#### **六、高级问题处理**

##### **1. SELinux 限制**

**问题表现**：



- 在 CentOS/RHEL 上，Runner 无法访问某些资源。



**解决方案**：

```bash
# 临时禁用 SELinux
sudo setenforce 0

# 永久修改（编辑 /etc/selinux/config）
SELINUX=permissive

# 添加 SELinux 规则（示例：允许访问 Docker 套接字）
sudo semanage fcontext -a -t container_runtime_exec_t "/usr/bin/docker(/.*)?"
sudo restorecon -Rv /usr/bin/docker
```

##### **2. 自定义 CA 证书**

**问题表现**：



- 连接私有 GitLab 时 SSL 验证失败。



**解决方案**：

```bash
# 添加自定义 CA 证书
sudo mkdir -p /usr/local/share/ca-certificates/
sudo cp your-ca.crt /usr/local/share/ca-certificates/
sudo update-ca-certificates

# 重启 Runner
sudo systemctl restart gitlab-runner
```



#### **七、验证与测试

```bash
# 验证 Runner 配置
sudo gitlab-runner verify

# 本地执行作业测试
sudo gitlab-runner exec docker job-name  # 使用 docker 执行器测试

# 查看详细日志
sudo gitlab-runner --debug run
```



#### **八、参考资源**

- GitLab Runner 官方文档：https://docs.gitlab.com/runner/
- Docker 网络配置：https://docs.docker.com/network/
- SELinux 管理指南：https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/using_selinux/index



### 通过以上解决方案，你可以解决 GitLab Runner 安装与使用过程中的常见问题。如仍有疑问，请提供具体错误日志以便进一步排查

### 验证 GitLab Runner 安装成功的步骤

#### **一、检查服务状态**

##### **1. 查看系统服务状态

```bash
sudo systemctl status gitlab-runner
```



**预期输出**：

```plaintext
● gitlab-runner.service - GitLab Runner
   Loaded: loaded (/lib/systemd/system/gitlab-runner.service; enabled; vendor preset: enabled)
   Active: active (running) since Mon 2025-05-22 10:00:00 UTC; 1h ago
```



- 确保状态为 **active (running)**。

##### **2. 验证进程是否运行

```bash
ps aux | grep gitlab-runner
```



**预期输出**：

```plaintext
gitlab+  1234  0.1  2.0 123456 7890 ?        Ssl  10:00   0:01 /usr/bin/gitlab-runner run --working-directory /home/gitlab-runner --config /etc/gitlab-runner/config.toml --service gitlab-runner --syslog --user gitlab-runner
```

#### **二、检查 Runner 版本

```bash
gitlab-runner --version
```



**预期输出**：

```plaintext
Version:      15.11.0
Git revision: abcdef12
Git branch:   15-11-stable
GO version:   go1.20.5
Built:        2025-01-01T00:00:00Z
OS/Arch:      linux/amd64
```

#### **三、验证注册状态**

##### **1. 查看已注册的 Runner

```bash
sudo gitlab-runner list
```



**预期输出**：

```plaintext
Runtime platform                                    arch=amd64 os=linux pid=1234 revision=abcdef12 version=15.11.0
- executor = docker
  name = my-docker-runner
  url = https://gitlab.com
  token = ****************************************
```

##### **2. 在 GitLab 界面确认**

1. 登录 GitLab → 项目主页 → Settings → CI/CD → Runners。
2. 在 **Runners activated for this project** 部分，查看 Runner 是否显示为 **green**（已激活）。

#### **四、测试 Runner 连接

```bash
sudo gitlab-runner verify
```



**预期输出**：



plaintext

```plaintext
Runtime platform                                    arch=amd64 os=linux pid=1234 revision=abcdef12 version=15.11.0
Verifying runner... is alive                        runner=abcdefg
```



- 若提示

   

  ```
  FAILED
  ```

  ，检查：

  - GitLab URL 是否正确（`config.toml` 中的 `url`）。
  - Runner 令牌是否有效（可在 GitLab 界面重置）。
  - 网络连接和防火墙设置。

#### **五、执行简单 CI/CD 作业**

##### **1. 在项目中添加 `.gitlab-ci.yml

```yaml
# .gitlab-ci.yml
test_installation:
  stage: test
  script:
    - echo "GitLab Runner is working!"
    - gitlab-runner --version
```

##### **2. 推送代码触发流水线

```bash
git add .gitlab-ci.yml
git commit -m "Add CI test"
git push origin main
```

##### **3. 在 GitLab 界面查看结果**

1. 登录 GitLab → 项目主页 → CI/CD → Pipelines。
2. 点击最新的流水线记录，查看作业是否成功运行。

#### **六、常见问题排查**

##### **1. Runner 未显示在 GitLab 界面**

- 检查 `config.toml` 中的 `url` 和 `token` 是否正确。
- 重启 Runner 服务：`sudo systemctl restart gitlab-runner`。

##### **2. 作业无法启动**

```bash
# 查看 Runner 日志
sudo journalctl -u gitlab-runner -f

# 检查磁盘空间
df -h

# 验证 Docker 权限（如果使用 Docker 执行器）
su - gitlab-runner -c "docker info"
```



#### **七、总结**

通过以下步骤可确认 GitLab Runner 安装成功：



1. 服务状态为 **active (running)**。
2. 版本号正常显示。
3. `gitlab-runner list` 显示已注册的 Runner。
4. `verify` 命令返回成功。
5. 测试 CI/CD 作业能正常执行。



若遇到问题，请参考 GitLab 官方文档或提供具体错误日志进一步排查。