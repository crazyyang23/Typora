# Jenkins 配置 .NET 环境手册

## 1. 问题概述

在 Jenkins 中执行 `.NET` 命令时常见以下问题：

- `dotnet: command not found`
- `Permission denied` 错误
- 环境变量配置无效

## 2. 解决方案总览

### 2.1 推荐方案（⭐⭐⭐⭐⭐）

- 系统级安装 .NET SDK 到 `/usr/bin`
- 无需特殊权限配置
- 长期稳定

### 2.2 替代方案

- 修改权限让 Jenkins 访问 `/root/.dotnet`
- 使用 Docker 容器执行构建

## 3. 详细操作指南

### 3.1 系统级安装 .NET SDK（推荐）

#### 3.1.1 CentOS/RHEL 系统

bash



复制



下载

```
# 添加微软仓库
sudo rpm -Uvh https://packages.microsoft.com/config/centos/7/packages-microsoft-prod.rpm

# 安装 SDK
sudo yum install -y dotnet-sdk-7.0

# 验证安装
which dotnet  # 应显示 /usr/bin/dotnet
dotnet --version
```

#### 3.1.2 Ubuntu/Debian 系统

bash



复制



下载

```
# 添加微软仓库
wget https://packages.microsoft.com/config/ubuntu/$(lsb_release -rs)/packages-microsoft-prod.deb
sudo dpkg -i packages-microsoft-prod.deb
sudo apt update

# 安装 SDK
sudo apt install -y dotnet-sdk-7.0
```

### 3.2 Jenkins 任务配置

#### 3.2.1 自由风格项目配置

1. 新建自由风格项目
2. 在构建步骤中选择"执行 shell"
3. 添加以下命令：

bash



复制



下载

```
#!/bin/bash
dotnet --version
dotnet build
```

#### 3.2.2 管道项目配置

groovy



复制



下载

```
pipeline {
    agent any
    
    stages {
        stage('Build') {
            steps {
                sh 'dotnet --version'
                sh 'dotnet build'
            }
        }
    }
}
```

### 3.3 权限问题处理（替代方案）

#### 3.3.1 临时解决方案

bash



复制



下载

```
# 开放权限（不推荐生产环境）
sudo chmod -R 755 /root/.dotnet
sudo chown -R jenkins:jenkins /root/.dotnet

# Jenkins 任务中
export PATH=$PATH:/root/.dotnet
```

#### 3.3.2 使用 sudo（不推荐）

bash



复制



下载

```
# 配置 sudo 权限
echo "jenkins ALL=(root) NOPASSWD: /root/.dotnet/dotnet" | sudo tee /etc/sudoers.d/jenkins-dotnet

# Jenkins 任务中
sudo /root/.dotnet/dotnet --version
```

### 3.4 Docker 方案（推荐用于 CI/CD）

#### 3.4.1 Jenkins 任务配置

bash



复制



下载

```
docker run --rm -v $(pwd):/app -w /app mcr.microsoft.com/dotnet/sdk:7.0 dotnet build
```

#### 3.4.2 管道配置

groovy



复制



下载

```
pipeline {
    agent {
        docker {
            image 'mcr.microsoft.com/dotnet/sdk:7.0'
        }
    }
    stages {
        stage('Build') {
            steps {
                sh 'dotnet build'
            }
        }
    }
}
```

## 4. 验证与调试

### 4.1 环境检查脚本

bash



复制



下载

```
#!/bin/bash
echo "=== Environment Check ==="
echo "1. PATH: $PATH"
echo "2. dotnet path: $(which dotnet || echo 'Not found')"
echo "3. dotnet version: $(dotnet --version || echo 'Command failed')"
echo "4. Current user: $(whoami)"
echo "5. /root/.dotnet permissions: $(ls -ld /root/.dotnet 2>/dev/null || echo 'No access')"
```

### 4.2 常见错误处理

| 错误现象                    | 解决方案                      |
| :-------------------------- | :---------------------------- |
| `dotnet: command not found` | 确保系统级安装或正确设置 PATH |
| `Permission denied`         | 使用系统级安装或调整权限      |
| 版本不匹配                  | 使用 `global.json` 指定版本   |

## 5. 最佳实践

1. **环境隔离**：推荐使用 Docker 方案实现环境隔离
2. **版本控制**：在项目中添加 `global.json` 文件锁定 .NET 版本
3. **权限最小化**：避免使用 root 权限运行 Jenkins 任务
4. **日志记录**：在 Jenkins 任务中添加环境检查步骤

## 6. 附录

### 6.1 常用命令参考

bash



复制



下载

```
# 列出已安装 SDK
dotnet --list-sdks

# 创建 global.json
dotnet new globaljson --sdk-version 7.0.203

# 清理 NuGet 缓存
dotnet nuget locals all --clear
```

### 6.2 参考文档

- [.NET 官方安装文档](https://docs.microsoft.com/dotnet/core/install/)
- [Jenkins 官方文档](https://www.jenkins.io/doc/)

------

本手册最后更新日期：2023年8月
适用版本：.NET 6/7/8，Jenkins 2.3+
维护建议：定期检查 .NET 和 Jenkins 的版本兼容性