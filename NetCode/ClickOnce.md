# CentOS 系统下.NET 4.5 ClickOnce 生成详细操作手册

## 一、引言

本手册旨在指导用户在 CentOS 操作系统下实现.NET 4.5 ClickOnce 应用的生成与部署。ClickOnce 是一种方便的应用程序部署技术，允许用户通过 Web 轻松安装和更新应用。在 CentOS 环境中，需要借助 Mono 和 Wine 等工具来模拟实现这一过程。本手册将详细介绍从环境搭建、应用创建、清单生成到最终部署的全流程操作，并对可能遇到的问题提供解决方案与优化建议。

## 二、环境准备

### 2.1 安装 Mono

Mono 是一个开源的.NET 运行时环境，能够编译和运行大部分.NET 代码。在 CentOS 上安装 Mono 的步骤如下：

1. 导入 Mono 项目的 GPG 密钥，以确保软件包的完整性和安全性：

```
sudo rpm --import "https://keyserver.ubuntu.com/pks/lookup?op=get&search=0x3FA7E0328081BFF6A14DA29AA6A19B38D3D831"
```

1. 添加 Mono 官方的 yum 仓库：

```
sudo yum-config-manager --add-repo https://download.mono-project.com/repo/centos/
```

1. 安装完整的 Mono 开发环境，该环境包含编译器、运行时和常用类库：

```
sudo yum install mono-complete
```

### 2.2 安装 Wine

Wine 是一个能够在 Linux 系统上运行 Windows 应用程序的兼容层，用于运行 ClickOnce 发布工具。在 CentOS 上安装 Wine 的命令如下：

```
sudo yum install wine
```

Wine 不是虚拟机，它直接将 Windows API 调用转换为 Linux 系统调用，在运行 Windows 程序时具有较好的性能表现。

## 三、创建.NET 4.5 应用

### 3.1 创建项目目录

在 CentOS 系统中，创建一个新的目录用于存放应用项目文件：

```
mkdir MyClickOnceApp && cd MyClickOnceApp
```

### 3.2 编写应用代码

使用文本编辑器创建一个名为Program.cs的文件，并编写以下简单的控制台应用代码：

```
using System;

namespace MyClickOnceApp
{
    class Program
    {
        static void Main(string[] args)
        {
            Console.WriteLine("Hello from ClickOnce application!");
            Console.ReadLine();
        }
    }
}
```

上述代码实现了一个简单的功能，即输出欢迎信息并等待用户输入。

### 3.3 编译应用

使用 Mono 的mcs编译器将代码编译为可执行文件，具体命令如下：

```
mcs -target:exe -out:MyClickOnceApp.exe -langversion:4 Program.cs
```

参数说明：

- -target:exe：指定编译目标为可执行文件；

- -out:：指定输出的可执行文件名称；

- -langversion:4：指定使用 C# 4.0 语言版本（对应.NET 4.5）。

## 四、准备 ClickOnce 发布工具

ClickOnce 发布依赖特定的工具生成相关清单文件，由于 Mono 不直接支持 ClickOnce 发布，因此需要使用 Wine 运行 Microsoft 的发布工具。这里使用dotnetpublish.exe，它是一个开源的 ClickOnce 发布工具替代方案，原是 SharpDevelop IDE 的一部分。

1. 下载dotnetpublish.exe工具：

```
wget https://github.com/icsharpcode/SharpDevelop/raw/master/Source/AddIns/Setup%20Packager/PublishFiles/bin/Release/dotnetpublish.exe
```

1. 该工具用于生成 ClickOnce 发布所需的应用清单和部署清单文件。

## 五、生成 ClickOnce 发布文件

ClickOnce 发布依赖两个核心清单文件：应用清单（.exe.manifest）和部署清单（.application）。

### 5.1 生成应用清单

应用清单描述了应用程序本身的信息，包括程序集引用、版本等。使用以下命令生成应用清单：

```
wine dotnetpublish.exe /generate-manifest MyClickOnceApp.exe /out:publish/MyClickOnceApp.exe.manifest /targetversion:v4.5 /platform:x86
```

参数说明：

- /generate-manifest：指定生成应用清单；

- /out:：指定应用清单的输出路径和文件名；

- /targetversion:v4.5：指定应用目标版本为.NET 4.5；

- /platform:x86：指定应用运行平台为 x86 架构。

### 5.2 生成部署清单

部署清单描述了应用程序的部署元数据，如更新频率、依赖项等。使用以下命令生成部署清单：

```
wine dotnetpublish.exe /generate-deployment MyClickOnceApp.exe.manifest /out:publish/MyClickOnceApp.application /product:"My ClickOnce App" /version:1.0.0.0
```

参数说明：

- /generate-deployment：指定生成部署清单；

- /out:：指定部署清单的输出路径和文件名；

- /product:：指定应用程序名称；

- /version:：指定应用程序版本。

### 5.3 复制应用文件

将生成的可执行文件复制到发布目录中：

```
cp MyClickOnceApp.exe publish/
```

## 六、部署到 Web 服务器

本手册选择 Nginx 作为 Web 服务器，因其轻量级、高性能且配置简单，能够很好地支持 ClickOnce 发布文件的静态服务。

### 6.1 安装 Nginx

在 CentOS 上安装 Nginx 的命令如下：

```
sudo yum install nginx
```

### 6.2 启动 Nginx

安装完成后，启动 Nginx 服务：

```
sudo systemctl start nginx
```

### 6.3 复制发布文件

将包含应用清单、部署清单和可执行文件的发布目录内容复制到 Nginx 的默认网站目录中：

```
sudo cp -r publish/* /usr/share/nginx/html/
```

## 七、配置 MIME 类型

ClickOnce 发布依赖特定的 MIME 类型才能正常工作，需要在 Web 服务器中进行配置。

### 7.1 编辑 Nginx 配置文件

打开 Nginx 的主配置文件：

```
sudo nano /etc/nginx/nginx.conf
```

### 7.2 添加 MIME 类型

在配置文件中添加以下 MIME 类型配置：

```
types {
    application/x-ms-application        application;
    application/x-ms-manifest           manifest;
    application/octet-stream            deploy;
    application/x-ms-vsto               vsto;
    application/xaml+xml                xaml;
}
```

这些 MIME 类型用于告诉浏览器如何处理 ClickOnce 相关文件，例如，.application文件会被识别为应用部署清单，.manifest文件会被识别为应用程序清单。

### 7.3 重启 Nginx

保存配置文件后，重启 Nginx 服务使配置生效：

```
sudo systemctl restart nginx
```

## 八、问题与解决方案

### 8.1 Wine 兼容性问题

在使用 Wine 运行 ClickOnce 发布工具时，可能会遇到运行不稳定的情况。解决方案如下：

- 尝试安装不同版本的 Wine，可通过 WineHQ 安装最新开发版；

- 考虑使用商业解决方案，如 CrossOver；

- 若工具运行异常导致清单文件生成错误，可手动编辑 XML 格式的清单文件，但需要熟悉 ClickOnce 清单格式。

### 8.2 .NET 兼容性问题

Mono 对.NET 4.5 的支持并非完全兼容，可能遇到以下问题：

- **WPF 应用**：Mono 不支持 WPF，若应用使用了 WPF 技术，需要改用 Windows Forms 或[ASP.NET](https://ASP.NET) Core；

- **特定框架特性**：某些.NET 4.5 特有的 API 在 Mono 中可能不被支持，需要对相关代码进行改写或寻找替代方案。

### 8.3 安全与权限问题

为确保 Web 服务器能够正常访问发布文件，需要设置适当的文件权限：

```
sudo chmod -R 755 /usr/share/nginx/html/
```

确保 Web 服务器用户（通常是nginx用户）对发布文件具有访问权限，否则可能会导致 403 错误。

## 九、高级优化建议

### 9.1 使用 Docker 封装环境

使用 Docker 可以创建一个可复用的 ClickOnce 发布环境，方便在不同系统中快速部署。以下是一个简单的 Dockerfile 示例：

```
# Dockerfile示例
FROM centos:7

# 安装Mono和Wine
RUN rpm --import "https://keyserver.ubuntu.com/pks/lookup?op=get&search=0x3FA7E0328081BFF6A14DA29AA6A19B38D3D831" && \
    yum-config-manager --add-repo https://download.mono-project.com/repo/centos/ && \
    yum install -y mono-complete wine nginx && \
    yum clean all

# 设置工作目录
WORKDIR /app

# 复制应用和发布工具
COPY. /app

# 暴露Nginx端口
EXPOSE 80

# 启动Nginx
CMD ["nginx", "-g", "daemon off;"]
```

通过构建和运行该 Docker 镜像，可以快速搭建好 ClickOnce 发布环境。

### 9.2 自动化发布脚本

编写自动化脚本可以简化从编译到发布的整个流程。以下是一个示例脚本[publish-clickonce.sh](http://publish-clickonce.sh)：

```
#!/bin/bash
# publish-clickonce.sh

# 编译应用
mcs -target:exe -out:MyClickOnceApp.exe -langversion:4 Program.cs

# 创建发布目录
mkdir -p publish

# 生成清单文件
wine dotnetpublish.exe /generate-manifest MyClickOnceApp.exe /out:publish/MyClickOnceApp.exe.manifest /targetversion:v4.5 /platform:x86
wine dotnetpublish.exe /generate-deployment MyClickOnceApp.exe.manifest /out:publish/MyClickOnceApp.application /product:"My ClickOnce App" /version:1.0.0.0

# 复制文件到Web服务器
cp MyClickOnceApp.exe publish/
sudo cp -r publish/* /usr/share/nginx/html/

# 重启Nginx
sudo systemctl restart nginx

echo "ClickOnce应用发布成功！"
```

将上述脚本保存后，赋予执行权限chmod +x [publish-clickonce.sh](http://publish-clickonce.sh)，即可通过运行该脚本一键完成应用的发布过程。

## 十、验证方法

在 Windows 客户端上验证 ClickOnce 应用的安装和运行：

1. 打开浏览器，访问http://your-server-ip/MyClickOnceApp.application，其中your-server-ip为 CentOS 服务器的 IP 地址；

1. 浏览器会提示安装应用，按照提示完成安装过程；

1. 安装完成后，应用会出现在 “添加 / 删除程序” 列表中；

1. 启动应用，应该能够看到 “Hello from ClickOnce application!” 的输出信息。

如果在验证过程中遇到问题，可以通过以下方式排查：

- 查看浏览器开发者工具中的网络请求，检查是否存在 404 或 403 等错误；

- 查看 Web 服务器日志，Nginx 的访问日志通常位于/var/log/nginx/access.log，错误日志位于/var/log/nginx/error.log；

- 查看 ClickOnce 安装日志，Windows 系统中 ClickOnce 安装日志文件位于 % TEMP% 目录下（可通过在文件资源管理器地址栏输入%TEMP%快速访问）。

这份手册涵盖了 CentOS 下生成.NET 4.5 ClickOnce 应用的全流程。若在操作中还有疑问，或遇到其他问题，欢迎随时和我交流。