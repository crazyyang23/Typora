### **. 通过 INFO 命令获取统计信息**

`INFO` 命令可以获取 Redis 的详细统计信息，包括每个数据库的键数量和内存使用：



```bash
redis-cli INFO keyspace
# 输出示例：
# db0:keys=100,expires=5,avg_ttl=1000000
# db2:keys=50,expires=0,avg_ttl=0
```

### **总结**

- **快速判断**：使用 `INFO keyspace` 查看非空数据库。
- **精确检查**：遍历所有数据库并执行 `dbsize` 命令。
- **注意**：Redis 的数据库是逻辑隔离的，所有数据库共享内存资源，删除空数据库不会释放内存（除非使用 `FLUSHALL`）。