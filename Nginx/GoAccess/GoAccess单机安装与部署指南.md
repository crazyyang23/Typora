# **GoAccess 日志分析报告部署手册**

**（使用 Python HTTP 服务器）**

------

## **1. 准备工作**

### **1.1 确认 GoAccess 报告已生成**

确保已通过以下命令生成 `report.html` 文件：

bash



复制



下载

```
goaccess detailed-2025-07-21.log \
  --log-format='%h %^[%d:%t %^];%^;%^;"%m %U %H" %s %b;%^;%^;Request_Length:%^;Response_Length:%^;Request_Time:%T;%^;%^;%^;%^;%^;%^;%^;%^;%^;%^;%^;%^;%^;%^;%^;%^;%^;%^;%^;%^;%^;%^;%^' \
  --date-format='%d/%b/%Y' \
  --time-format='%H:%M:%S' \
  -o /var/www/html/report.html
```

### **1.2 检查文件权限**

确保报告文件可被读取：

bash



复制



下载

```
chmod 644 /var/www/html/report.html
```

------

## **2. 启动 Python HTTP 服务器**

### **2.1 进入报告所在目录**

bash



复制



下载

```
cd /var/www/html/
```

### **2.2 启动 Python3 临时 Web 服务器**

bash



复制



下载

```
python3 -m http.server 8000
```

- **默认端口**：`8000`

- **监听地址**：`0.0.0.0`（所有网络接口）

- **仅本地访问**（安全推荐）：

  bash

  

  复制

  

  下载

  ```
  python3 -m http.server 8000 --bind 127.0.0.1
  ```

### **2.3 保持服务运行（可选）**

使用 `nohup` 让服务在后台运行：

bash



复制



下载

```
nohup python3 -m http.server 8000 &
```

查看日志：

bash



复制



下载

```
tail -f nohup.out
```

------

## **3. 访问报告**

### **3.1 获取服务器 IP**

bash



复制



下载

```
curl ifconfig.me  # 获取公网IP
hostname -I      # 获取内网IP
```

### **3.2 浏览器访问**

text



复制



下载

```
http://<服务器IP>:8000/report.html
```

- 替换 `<服务器IP>` 为你的服务器实际 IP。

------

## **4. 防火墙配置**

### **4.1 开放端口（如需公网访问）**

#### **Ubuntu/Debian**

bash



复制



下载

```
sudo ufw allow 8000/tcp
sudo ufw enable
```

#### **CentOS/RHEL**

bash



复制



下载

```
sudo firewall-cmd --add-port=8000/tcp --permanent
sudo firewall-cmd --reload
```

### **4.2 验证端口是否开放**

bash



复制



下载

```
nc -zv <服务器IP> 8000
```

------

## **5. 停止服务**

### **5.1 查找进程 ID**

bash



复制



下载

```
ps aux | grep "python3 -m http.server"
```

### **5.2 终止进程**

bash



复制



下载

```
kill <PID>  # 替换<PID>为实际进程ID
```

------

## **6. 自动化脚本（可选）**

创建 `start_report.sh` 脚本：

bash



复制



下载

```
#!/bin/bash
cd /var/www/html/
nohup python3 -m http.server 8000 &
echo "GoAccess 报告已发布：http://$(hostname -I | awk '{print $1}'):8000/report.html"
```

赋予执行权限：

bash



复制



下载

```
chmod +x start_report.sh
```

运行脚本：

bash



复制



下载

```
./start_report.sh
```

------

## **7. 注意事项**

1. **仅限临时使用**：Python HTTP 服务器不适合高并发生产环境。
2. **安全建议**：
   - 使用 `--bind 127.0.0.1` 限制本地访问。
   - 如需公网访问，建议搭配 Nginx/Apache 并启用 HTTPS。
3. **实时更新**：
   - 若需实时数据，改用 GoAccess 的 `--real-time-html` 模式（需开放额外端口）。

------

## **8. 附录**

### **常用命令速查**

| 功能     | 命令                                        |
| :------- | :------------------------------------------ |
| 启动服务 | `python3 -m http.server 8000`               |
| 后台运行 | `nohup python3 -m http.server 8000 &`       |
| 检查端口 | `netstat -tulnp | grep 8000`                |
| 关闭服务 | `kill $(pgrep -f "python3 -m http.server")` |

------

**✅ 部署完成！** 现在可通过 `http://<IP>:8000/report.html` 访问日志分析报告。