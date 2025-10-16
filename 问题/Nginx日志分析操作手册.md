以下是为您整理的 **Nginx日志分析操作手册**，包含全部统计场景和命令，按功能分类说明：

------

# Nginx日志分析操作手册

**适用场景**：统计客户端请求、后端服务分发、响应状态码、upstream响应时间等

------

## 一、基础字段确认

### 1. 查看日志格式

bash

复制

```
# 查看日志前5行结构
head -n 5 /var/log/nginx/upstream-access-2025-04-17.log

# 确认字段数量（NF）和状态码位置
awk '{print "Line "NR": "$0; print "Fields: "NF; print "Status: "$8}' /var/log/nginx/upstream-access-2025-04-17.log | head -n 3
```

### 2. 检查客户端IP分布

bash

复制

```
awk '{print $1}' /var/log/nginx/upstream-access-2025-04-17.log | sort | uniq -c | sort -nr | head -n 10
```

------

## 二、按客户端IP统计

### 1. 统计特定IP的请求分布

bash

复制

```
awk '$1 == "10.2.255.43" {
    backend = $3;
    status = $8;
    if (status ~ /^[23]/) success[backend]++;
    else if (status ~ /^[45]/) fail[backend]++;
    total[backend]++;
}
END {
    printf "%-20s %-10s %-10s %-10s\n", "Backend", "Success", "Fail", "Total";
    print "------------------------------------------";
    for (svr in total) {
        printf "%-20s %-10s %-10s %-10s\n", svr, success[svr]+0, fail[svr]+0, total[svr]+0;
    }
}' /var/log/nginx/upstream-access-2025-04-17.log
```

### 2. 带响应时间的完整统计

bash

复制

```
awk '$1 == "10.2.255.43" {
    split($(NF), uptime_arr, "=");
    uptime = uptime_arr[2];
    backend = $3;
    status = $8;

    # 成功请求统计
    if (status ~ /^[23]/) {
        succ[backend]++;
        succ_time[backend] += uptime;
        if (uptime > max_succ[backend] || !max_succ[backend]) max_succ[backend] = uptime;
        if (uptime < min_succ[backend] || !min_succ[backend]) min_succ[backend] = uptime;
    }
    # 失败请求统计
    else if (status ~ /^[45]/) {
        fail[backend]++;
        fail_time[backend] += uptime;
        if (uptime > max_fail[backend] || !max_fail[backend]) max_fail[backend] = uptime;
    }
    total[backend]++;
    total_time[backend] += uptime;
}
END {
    # 打印表头
    printf "%-20s %-8s %-8s %-8s %-10s %-10s %-10s %-10s\n", 
           "Backend", "Success", "Fail", "Total", "Avg(ms)", "Min(ms)", "Max(ms)", "MaxFail(ms)";
    print "--------------------------------------------------------------------------------";
    
    # 打印数据
    for (svr in total) {
        avg = total_time[svr] * 1000 / total[svr];  # 转换为毫秒
        printf "%-20s %-8s %-8s %-8s %-10.2f %-10.2f %-10.2f %-10.2f\n", 
               svr, succ[svr]+0, fail[svr]+0, total[svr]+0,
               avg, min_succ[svr]*1000, max_succ[svr]*1000, 
               (fail[svr]>0 ? max_fail[svr]*1000 : 0);
    }
}' /var/log/nginx/upstream-access-2025-04-17.log
```

------

## 三、全局统计分析

### 1. 所有后端服务请求分布

bash

复制

```
awk '{
    split($3, backend, ":");
    port = backend[2];
    status = $8;
    
    if (status ~ /^[23]/) success[port]++;
    else if (status ~ /^[45]/) fail[port]++;
    total[port]++;
}
END {
    print "Port | Success | Fail | Total";
    print "-----------------------------";
    for (p in total) {
        printf "%4s | %7s | %4s | %5s\n", p, success[p]+0, fail[p]+0, total[p]+0;
    }
}' /var/log/nginx/upstream-access-2025-04-17.log | sort -n
```

### 2. 异常请求分析（4xx/5xx）

bash

复制

```
awk '$8 ~ /^[45]/ {
    print $1, $3, $8, $6;
}' /var/log/nginx/upstream-access-2025-04-17.log | head -n 20
```

------

## 四、高级分析技巧

### 1. 按时间段统计（需日志含时间戳）

bash

复制

```
awk '$4 >= "[17/Apr/2025:08:00:00" && $4 <= "[17/Apr/2025:09:00:00" {
    # 在此插入统计逻辑
}' /var/log/nginx/upstream-access-2025-04-17.log
```

### 2. 生成CSV格式报告

bash

复制

```
awk 'BEGIN {print "Client,Backend,Status,ResponseTime"} 
$1 == "10.2.255.43" {
    split($(NF), uptime, "=");
    print $1 "," $3 "," $8 "," uptime[2];
}' /var/log/nginx/upstream-access-2025-04-17.log > report.csv
```

------

## 五、常见问题处理

### 1. 字段错位处理

bash

复制

```
# 使用sub清理多余字符
awk '{gsub(/"/, "", $8); print $8}' access.log | head
```

### 2. 多行日志识别

bash

复制

```
awk 'BEGIN {RS="\r?\n"; FS=" "} 
NF < 10 {print "Incomplete line at NR:", NR}' access.log
```

------

## 六、可视化建议

1. **文本报告**：通过`column -t`对齐输出

   bash

   复制

   ```
   awk '{...}' logfile | column -t
   ```

2. **图形化**：生成CSV后用Excel/Python可视化

3. **监控集成**：将统计结果导入Prometheus+Grafana

------

**附：日志字段参考**

复制

```
$1: 客户端IP
$3: 后端服务地址(IP:Port)
$8: HTTP状态码
$(NF): upstream_response_time=0.123
```

将此手册保存为`nginx_log_analysis_guide.sh`即可快速调用所有命令。