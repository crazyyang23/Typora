1、只显示来自特定电脑转发服务的日志统计

```
awk '$1 == "10.2.255.43" {print $3}' /var/log/nginx/upstream-access-2025-04-17.log | sort | uniq -c | sort -nr
```

##2**统计成功（2xx/3xx）和失败（4xx/5xx）请求数**

```
grep '10.2.255.10' /var/log/nginx/upstream-access-2025-04-17.log | awk '{print $9}' | sort | uniq -c
```

```

awk '$1 == "10.2.255.43" {
    if ($9 ~ /^[23]/) success++;
    else if ($9 ~ /^[45]/) failure++
} 
END {
    print "From IP: 10.2.255.10";
    print "Success (2xx/3xx):", success;
    print "Failure (4xx/5xx):", failure;
    print "Total requests:", success + failure
}' /var/log/nginx/upstream-access-2025-04-17.log
```

### **按后端服务端口分类统计（成功/失败）**



```
awk '$1 == "10.2.255.43" {
    # 提取后端服务端口（$3 是 "127.0.0.1:30092"）
    split($3, backend, ":");
    port = backend[2];

    # 统计成功（2xx/3xx）和失败（4xx/5xx）
    if ($8 ~ /^[23]/) {
        success[port]++;
    } else if ($8 ~ /^[45]/) {
        failure[port]++;
    }
    total[port]++;  # 总请求数
} 
END {
    print "From IP: 10.2.255.10";
    print "Backend Port | Success (2xx/3xx) | Failure (4xx/5xx) | Total Requests";
    print "-------------|-------------------|-------------------|--------------";
    
    # 遍历所有端口并输出统计结果
    for (port in total) {
        printf "%-12s | %-17s | %-17s | %-12s\n", 
            port, 
            success[port] + 0,  # +0 避免未命中时显示空值
            failure[port] + 0, 
            total[port] + 0;
    }
}' /var/log/nginx/upstream-access-2025-04-17.log
```



#**日志格式解析（空格分隔字段）**

复制

```
$1 - 客户端IP
$2 - 保留字段（-）
$3 - 后端服务器地址（IP:Port）
$4 - 时间戳
$5 - 请求方法（带引号）
$6 - 请求路径（带引号）
$7 - HTTP协议（带引号）
$8 - 状态码
$9 - 响应大小
$10 - Referer（带引号）
$11 - User-Agent（带引号）
$12 - upstream响应时间
```

 

```
awk ' $1 == "10.2.255.43" {
    # 提取关键字段
    client_ip = $1
    backend = $3
    status = $9
    
    # 按客户端IP和后端服务统计
    if (status ~ /^[23]/) {
        success[client_ip][backend]++
    } 
    else if (status ~ /^[45]/) {
        fail[client_ip][backend]++
    }
    total[client_ip][backend]++
}
END {
    # 输出表头
    printf "%-15s %-20s %-10s %-10s %-10s\n", 
           "Client IP", "Backend", "Success", "Fail", "Total"
    print "----------------------------------------------------------------"
    
    # 输出统计结果
    for (ip in total) {
        for (server in total[ip]) {
            s = success[ip][server] + 0  # 处理可能不存在的key
            f = fail[ip][server] + 0
            t = total[ip][server] + 0
            
            printf "%-15s %-20s %-10s %-10s %-10s\n", 
                   ip, server, s, f, t
        }
    }
}' /var/log/nginx/upstream-access-2025-04-17.log
```



以下是增强版的统计命令，在原有基础上增加了 **upstream响应时间** 的统计（平均、最大、最小响应时间），并优化了输出格式：



```
awk '
$1 == "10.2.255.43" {
    # 提取关键字段
    client_ip = $1
    backend = $3
    status = $9
    split($(NF), uptime_arr, "=")  # 提取upstream_response_time=0.123中的数值
    uptime = uptime_arr[2] + 0     # 转换为数字

    # 按客户端IP和后端服务统计
    if (status ~ /^[23]/) {
        success[client_ip][backend]++
        succ_time[client_ip][backend] += uptime
        if (uptime > max_succ[client_ip][backend] || max_succ[client_ip][backend] == "") 
            max_succ[client_ip][backend] = uptime
        if (uptime < min_succ[client_ip][backend] || min_succ[client_ip][backend] == "") 
            min_succ[client_ip][backend] = uptime
    } 
    else if (status ~ /^[45]/) {
        fail[client_ip][backend]++
        fail_time[client_ip][backend] += uptime
        if (uptime > max_fail[client_ip][backend] || max_fail[client_ip][backend] == "") 
            max_fail[client_ip][backend] = uptime
        if (uptime < min_fail[client_ip][backend] || min_fail[client_ip][backend] == "") 
            min_fail[client_ip][backend] = uptime
    }
    total[client_ip][backend]++
    total_time[client_ip][backend] += uptime
}
END {
    # 输出表头
    printf "%-15s %-20s %-8s %-8s %-8s %-12s %-12s %-12s %-12s\n", 
           "Client", "Backend", "2xx/3xx", "4xx/5xx", "Total", 
           "Avg_time(s)", "Max_succ", "Min_succ", "Max_fail"
    print "====================================================================================="
    
    # 输出统计结果
    for (ip in total) {
        for (svr in total[ip]) {
            s = success[ip][svr] + 0
            f = fail[ip][svr] + 0
            t = total[ip][svr] + 0
            
            # 计算平均时间（保留3位小数）
            avg = (t > 0) ? sprintf("%.3f", total_time[ip][svr]/t) : "N/A"
            
            # 输出结果
            printf "%-15s %-20s %-8s %-8s %-8s %-12s %-12s %-12s %-12s\n", 
                   ip, svr, s, f, t, avg,
                   (s > 0) ? sprintf("%.3f", max_succ[ip][svr]) : "N/A",
                   (s > 0) ? sprintf("%.3f", min_succ[ip][svr]) : "N/A",
                   (f > 0) ? sprintf("%.3f", max_fail[ip][svr]) : "N/A"
        }
    }
}' /var/log/nginx/upstream-access-2025-04-17.log
```

------

### **输出示例**

复制

```
Client         Backend             2xx/3xx 4xx/5xx Total   Avg_time(s) Max_succ    Min_succ    Max_fail    
=====================================================================================
10.2.255.43    127.0.0.1:30092    850     23      873     0.045       0.210      0.001       0.500
10.2.255.43    127.0.0.1:8001     420     5       425     0.032       0.180      0.002       N/A
```

------

### **新增统计维度说明**

| 指标          | 说明                           | 计算逻辑                                         |
| :------------ | :----------------------------- | :----------------------------------------------- |
| `Avg_time(s)` | 所有请求的平均响应时间         | `总时间/总请求数`                                |
| `Max_succ`    | 成功请求的最大响应时间         | 记录所有2xx/3xx请求中的最大值                    |
| `Min_succ`    | 成功请求的最小响应时间         | 记录所有2xx/3xx请求中的最小值                    |
| `Max_fail`    | 失败请求的最大响应时间（如有） | 记录所有4xx/5xx请求中的最大值（无失败则显示N/A） |

------

### **注意事项**

1. **字段验证**：确认 `upstream_response_time` 确实是最后一个字段（`$(NF)`），如果不是请调整字段编号
2. **时间单位**：默认统计的是秒（如0.123s），如需毫秒可乘以1000
3. **零值处理**：当没有成功/失败请求时显示 `N/A` 避免除零错误

如果需要进一步调整统计维度，可以随时告诉我！



##1、直接清空日志文件（推荐方法）

### 1.1 使用 truncate 命令（保持文件描述符）



```
sudo truncate -s 0 /var/log/nginx/access.log
sudo truncate -s 0 /var/log/nginx/error.log
```

### 1.2 使用重定向空内容



```
sudo sh -c '> /var/log/nginx/access.log'
sudo sh -c '> /var/log/nginx/error.log'
```

## 2. 删除并重建日志文件



```
sudo rm /var/log/nginx/access.log
sudo rm /var/log/nginx/error.log
sudo touch /var/log/nginx/access.log
sudo touch /var/log/nginx/error.log
sudo chown www-data:adm /var/log/nginx/*.log  # 根据实际用户调整
sudo chmod 640 /var/log/nginx/*.log
```