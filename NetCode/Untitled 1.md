### ClickOnce 应用程序跨平台部署操作手册

#### 方案一：使用 Nginx 在 Linux 上部署 ClickOnce 应用

##### 准备工作

1. **环境准备**
   - 确保 Linux 服务器已安装 Nginx（可通过 `sudo apt-get install nginx` 或 `sudo yum install nginx` 安装）
   - 准备好 ClickOnce 应用的发布文件夹（包含 `.application`、`.manifest`、`.dll`、`.exe` 等文件）
   - 确保服务器开放了 80 端口（HTTP）或 443 端口（HTTPS）
2. **文件准备**
   - 在 Windows 开发环境中，使用 Visual Studio 发布 ClickOnce 应用
   - 复制发布文件夹到 Linux 服务器（例如 `/var/www/clickonce`）

##### 配置 Nginx

1. **创建 Nginx 配置文件**

   bash

   

   

   

   

   

   ```bash
   sudo nano /etc/nginx/sites-available/clickonce
   ```

2. **添加以下配置内容**

   

   clickonce-linux-nginxNginx 配置 ClickOnce 支持

   V2

   ```
   server {
       listen 80;
       server_name your-domain.com;  # 替换为你的域名或服务器IP
       
       # 根目录设置为 ClickOnce 应用发布目录
       root /var/www/clickonce;
       index index.html;
       
       # 设置 ClickOnce 所需的 MIME 类型
       types {
           application/x-ms-application application;
           application/x-ms-manifest manifest;
           application/octet-stream deploy;
           application/octet-stream dll;
           application/octet-stream exe;
           text/html html;
           text/plain txt;
       }
       
       # 确保 manifest 文件不被缓存
       location ~* \.(application|manifest)$ {
           add_header Cache-Control "no-cache, no-store, must-revalidate";
           add_header Pragma "no-cache";
           add_header Expires "0";
       }
       
       # 允许访问所有文件
       location / {
           autoindex on;  # 开启目录浏览，方便测试
       }
   }
   
   ```

   生成 nginx.conf

3. **激活配置**

   bash

   ```bash
   sudo ln -s /etc/nginx/sites-available/clickonce /etc/nginx/sites-enabled/
   sudo nginx -t  # 测试配置是否正确
   sudo systemctl restart nginx  # 重启 Nginx 服务
   ```

##### 测试部署

1. **验证文件访问**
   - 在浏览器中访问 `http://your-server-ip/` 或 `http://your-domain.com/`
   - 应该能看到 ClickOnce 应用的文件列表
2. **测试 ClickOnce 应用**
   - 在 Windows 客户端，打开浏览器并访问 `.application` 文件的 URL（例如 `http://your-server-ip/YourApp.application`）
   - 应该会弹出 ClickOnce 安装对话框

##### 配置 HTTPS（可选但推荐）

1. **安装 Certbot**

   bash

   ```bash
   sudo apt-get install certbot python3-certbot-nginx
   ```

2. **获取 SSL 证书**

   bash

   ```bash
   sudo certbot --nginx -d your-domain.com
   ```

3. **更新 Nginx 配置**

   - Certbot 会自动更新 Nginx 配置，添加 HTTPS 支持
   - 重启 Nginx：`sudo systemctl restart nginx`

#### 方案二：使用 .NET Core 自托管 Web 服务器

##### 准备工作

1. **环境准备**
   - 确保 Linux 服务器已安装 .NET Core SDK（版本 3.1 或更高）
   - 安装 Git（可选，用于克隆代码）：`sudo apt-get install git`
2. **创建项目**
   - 在 Linux 服务器上创建一个新目录：`mkdir ClickOnceServer && cd ClickOnceServer`
   - 创建一个新的 .NET Core Web 应用：`dotnet new web`

##### 配置项目

1. **编辑 Program.cs 文件**

   

   clickonce-self-hosted.NET Core 自托管 ClickOnce 服务器

   V2

   ```
   using Microsoft.AspNetCore.Builder;
   using Microsoft.AspNetCore.Hosting;
   using Microsoft.AspNetCore.Http;
   using Microsoft.Extensions.DependencyInjection;
   using Microsoft.Extensions.FileProviders;
   using Microsoft.Extensions.Hosting;
   using System.IO;
   
   public class Program
   {
       public static void Main(string[] args)
       {
           CreateHostBuilder(args).Build().Run();
       }
   
       public static IHostBuilder CreateHostBuilder(string[] args) =>
           Host.CreateDefaultBuilder(args)
               .ConfigureWebHostDefaults(webBuilder =>
               {
                   webBuilder.ConfigureServices(services =>
                   {
                       services.AddControllers();
                   });
                   
                   webBuilder.Configure(app =>
                   {
                       if (app.Environment.IsDevelopment())
                       {
                           app.UseDeveloperExceptionPage();
                       }
                       
                       // 添加 ClickOnce 所需的 MIME 类型
                       var provider = new FileExtensionContentTypeProvider();
                       provider.Mappings[".application"] = "application/x-ms-application";
                       provider.Mappings[".manifest"] = "application/x-ms-manifest";
                       provider.Mappings[".deploy"] = "application/octet-stream";
                       provider.Mappings[".dll"] = "application/octet-stream";
                       provider.Mappings[".exe"] = "application/octet-stream";
                       
                       // 创建发布目录
                       var publishDir = Path.Combine(Directory.GetCurrentDirectory(), "Publish");
                       if (!Directory.Exists(publishDir))
                       {
                           Directory.CreateDirectory(publishDir);
                       }
                       
                       // 设置静态文件服务
                       app.UseStaticFiles(new StaticFileOptions
                       {
                           FileProvider = new PhysicalFileProvider(publishDir),
                           RequestPath = "/",
                           ContentTypeProvider = provider
                       });
                       
                       // 确保 manifest 文件不被缓存
                       app.Use(async (context, next) =>
                       {
                           var path = context.Request.Path.Value;
                           if (path.EndsWith(".application") || path.EndsWith(".manifest"))
                           {
                               context.Response.Headers.Add("Cache-Control", "no-cache, no-store, must-revalidate");
                               context.Response.Headers.Add("Pragma", "no-cache");
                               context.Response.Headers.Add("Expires", "0");
                           }
                           await next();
                       });
                       
                       app.UseRouting();
                       app.UseEndpoints(endpoints =>
                       {
                           endpoints.MapControllers();
                       });
                   });
               });
   }
   
   ```

   生成 Program.cs

2. **复制 ClickOnce 应用文件**

   - 将 ClickOnce 应用的发布文件夹内容复制到项目的 `Publish` 目录中

3. **构建并运行应用**

   bash

   ```bash
   dotnet build
   dotnet run --urls=http://0.0.0.0:5000
   ```

##### 设置为服务

1. **创建服务文件**

   bash

   ```bash
   sudo nano /etc/systemd/system/clickonce-server.service
   ```

2. **添加以下内容**

   ini

   ```ini
   [Unit]
   Description=ClickOnce Web Server
   After=network.target
   
   [Service]
   WorkingDirectory=/path/to/ClickOnceServer
   ExecStart=/usr/bin/dotnet /path/to/ClickOnceServer/bin/Debug/net5.0/ClickOnceServer.dll
   Restart=always
   # Restart service after 10 seconds if the dotnet service crashes:
   RestartSec=10
   KillSignal=SIGINT
   SyslogIdentifier=dotnet-clickonce
   User=www-data
   Environment=ASPNETCORE_ENVIRONMENT=Production
   Environment=DOTNET_PRINT_TELEMETRY_MESSAGE=false
   
   [Install]
   WantedBy=multi-user.target
   ```

3. **启用并启动服务**

   bash

   ```bash
   sudo systemctl enable clickonce-server
   sudo systemctl start clickonce-server
   ```

##### 配置反向代理（可选）

1. **安装 Nginx**

   bash

   ```bash
   sudo apt-get install nginx
   ```

2. **创建 Nginx 配置**

   bash

   ```bash
   sudo nano /etc/nginx/sites-available/clickonce-proxy
   ```

3. **添加以下配置**

   nginx

   ```nginx
   server {
       listen 80;
       server_name your-domain.com;
   
       location / {
           proxy_pass http://localhost:5000;
           proxy_http_version 1.1;
           proxy_set_header Upgrade $http_upgrade;
           proxy_set_header Connection keep-alive;
           proxy_set_header Host $host;
           proxy_cache_bypass $http_upgrade;
       }
   }
   ```

4. **激活配置并重启 Nginx**

   bash

   ```bash
   sudo ln -s /etc/nginx/sites-available/clickonce-proxy /etc/nginx/sites-enabled/
   sudo nginx -t
   sudo systemctl restart nginx
   ```

#### 方案三：使用 Docker 容器化 Windows 环境

##### 准备工作

1. **环境准备**

   - 确保 Linux 服务器已安装 Docker 和 Docker Compose

   - 安装 Docker CE：

     bash

     ```bash
     sudo apt-get update
     sudo apt-get install docker-ce docker-ce-cli containerd.io
     ```

   - 安装 Docker Compose：

     bash

     ```bash
     sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
     sudo chmod +x /usr/local/bin/docker-compose
     ```

2. **创建项目目录**

   bash

   ```bash
   mkdir clickonce-docker && cd clickonce-docker
   ```

##### 配置 Docker

1. **创建 Dockerfile**

   

   clickonce-dockerDocker 配置 ClickOnce 部署

   ```
   # 使用 Windows Server Core 作为基础镜像
   FROM mcr.microsoft.com/windows/servercore/iis:windowsservercore-ltsc2019
   
   # 安装 .NET Framework
   RUN powershell -Command Add-WindowsFeature NET-Framework-45-ASPNET; \
       powershell -Command Add-WindowsFeature Web-Asp-Net45
   
   # 配置 ClickOnce MIME 类型
   RUN powershell -Command Add-Type -AssemblyName System.Web; \
       $mimeMap = New-Object System.Web.Configuration.MimeMapping; \
       $mimeMap.AddMimeType(".application", "application/x-ms-application"); \
       $mimeMap.AddMimeType(".manifest", "application/x-ms-manifest"); \
       $mimeMap.AddMimeType(".deploy", "application/octet-stream"); \
       $mimeMap.UpdateMimeMappings();
   
   # 清理 IIS 默认网站
   RUN Remove-Item -Recurse C:\inetpub\wwwroot\*
   
   # 复制 ClickOnce 应用文件到网站目录
   COPY ./Publish/ C:/inetpub/wwwroot/
   
   # 暴露 HTTP 端口
   EXPOSE 80
   
   # 启动 IIS
   CMD ["C:\\ServiceMonitor.exe", "w3svc"]
   
   ```

   生成 Dockerfile

2. **创建 Docker Compose 文件**

   bash

   ```bash
   nano docker-compose.yml
   ```

3. **添加以下内容**

   yaml

   

   

   

   

   

   ```yaml
   version: '3'
   services:
     clickonce:
       build: .
       ports:
         - "80:80"
       volumes:
         - ./publish:/inetpub/wwwroot
   ```

4. **创建发布目录**

   bash

   

   

   

   

   

   ```bash
   mkdir publish
   ```

##### 构建并运行容器

1. **复制 ClickOnce 应用文件**

   - 将 ClickOnce 应用的发布文件夹内容复制到 `publish` 目录中

2. **构建 Docker 镜像**

   bash

   

   

   

   

   

   ```bash
   docker-compose build
   ```

3. **启动容器**

   bash

   

   

   

   

   

   ```bash
   docker-compose up -d
   ```

##### 验证部署

1. **检查容器状态**

   bash

   

   

   

   

   

   ```bash
   docker-compose ps
   ```

2. **测试访问**

   - 在浏览器中访问 `http://your-server-ip/`
   - 应该能看到 ClickOnce 应用的文件列表

#### 客户端访问指南

1. **使用浏览器访问**
   - 打开 Windows 客户端的浏览器
   - 输入 ClickOnce 应用的 `.application` 文件 URL（例如 `http://your-server-ip/YourApp.application`）
2. **处理安全提示**
   - 如果是第一次访问，可能会看到安全警告
   - 点击 "运行" 或 "安装" 继续
3. **验证应用安装**
   - 应用安装完成后，应该可以在开始菜单或桌面上找到应用图标
   - 启动应用并验证功能正常

#### 故障排除

1. **检查日志**
   - Nginx 日志：`/var/log/nginx/error.log`
   - .NET Core 应用日志：查看控制台输出或系统日志
   - Docker 容器日志：`docker logs container-id`
2. **常见问题**
   - **MIME 类型问题**：确保所有必要的 MIME 类型都已正确配置
   - **证书问题**：如果使用 HTTPS，确保证书有效且被客户端信任
   - **文件权限问题**：确保服务器有权限访问发布文件夹
   - **网络问题**：检查防火墙设置，确保客户端可以访问服务器端口
3. **客户端日志**
   - ClickOnce 客户端日志通常位于：`%LOCALAPPDATA%\Apps\2.0\`
   - 可以使用 `rundll32 dfshim CleanOnlineAppCache` 清理 ClickOnce 缓存



通过以上步骤，你应该能够在非 Windows 环境中成功部署 ClickOnce 应用，并让 Windows 客户端正常访问和安装这些应用。根据你的具体需求和环境，可以选择最适合的方案进行实施。