## 1. 在 Linux 系统上安装

### 方法一：使用官方脚本安装（推荐）

bash

复制下载

```
# 下载最新版本的 Docker Compose
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

# 授予执行权限
sudo chmod +x /usr/local/bin/docker-compose

# 验证安装
docker-compose --version
```



### 方法二：使用包管理器安装

**Ubuntu/Debian:**

bash

复制下载

```
sudo apt update
sudo apt install docker-compose-plugin
```



**CentOS/RHEL/Fedora:**

bash

复制下载

```
# CentOS/RHEL
sudo yum install docker-compose-plugin

# Fedora
sudo dnf install docker-compose-plugin
```