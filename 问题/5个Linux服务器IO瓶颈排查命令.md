# 5个Linux服务器IO瓶颈排查命令，让你的系统运维效率提升10倍

2025-08-14 19:55·[Echo](https://www.toutiao.com/c/user/token/MS4wLjABAAAAmYDSHuFGXZv4kW2Kv-dUXUkch04kUY8y9WNqAYjuhi0/?source=tuwen_detail)

作为Linux服务器运维工程师，你是否遇到过这样的情况：服务器明明配置不低，却频繁出现响应缓慢、服务卡顿，甚至数据库查询超时？90%的概率是遇到了**IO瓶颈**！磁盘IO是系统性能的"隐形杀手"，但只要用好这5个命令，就能像"CT扫描"一样精准定位问题根源。

# 一、iostat：系统级IO"体检仪"

**一句话概括**：快速查看磁盘整体负载，判断IO是否饱和的"第一道防线"

# 为什么需要它？

当用户抱怨"网站打开慢"时，第一步就是用iostat判断是不是磁盘在"偷懒"。它能告诉你磁盘每秒读写了多少次、响应速度如何，是不是已经忙到100%。

# 核心用法

```
# 每秒刷新1次，共5次，显示详细IO统计
iostat -x 1 5
```

# 关键指标解读（重点看这3个）

- **%util**：磁盘忙碌时间占比（**超过80%就要警惕**，接近100%表示IO饱和，像高速公路堵车）
- **await**：平均IO响应时间（正常应低于5ms，超过20ms说明磁盘"反应迟钝"）
- **rkB/s/wkB/s**：每秒读写数据量（结合业务判断是否超过磁盘性能上限）

# 真实案例

某电商网站在促销活动时页面加载慢，运维工程师用iostat -x 1发现%util持续95%，await高达30ms，最终定位到数据库日志写入过于频繁，优化后响应速度提升3倍（案例来源：51CTO博客）。

# 注意事项

- 首次输出是系统启动以来的累计数据，建议加-y参数跳过：iostat -xy 1
- 需安装sysstat包：yum install sysstat（CentOS）或apt install sysstat（Ubuntu）

# 二、iotop：进程级IO"追踪器"

**一句话概括**：找出"吃IO"的"元凶进程"，像任务管理器一样直观

# 为什么需要它？

iostat告诉你"磁盘很忙"，iotop告诉你"谁在忙"。比如服务器IO高，但不知道是MySQL还是日志程序导致，用它一秒定位。

# 核心用法

```
# 只显示正在进行IO的进程，每2秒刷新
iotop -o -d 2
```

# 界面解读（重点关注2列）

- **DISK READ/WRITE**：进程每秒读写速度（单位KB/s，谁数值大谁就是"肇事者"）
- **IO>**：IO等待时间占比（超过60%说明该进程被IO严重拖累）

# 实战技巧

按o键只显示活动IO进程，按p键切换显示PID，找到高IO进程后用kill -STOP [PID]临时暂停，观察系统是否恢复（案例来源：阿里云帮助中心）。

# 注意事项

- 需要root权限运行（sudo iotop）
- 安装命令：yum install iotop或apt install iotop

# 三、pidstat：进程IO"显微镜"

**一句话概括**：精准统计单个进程的IO细节，适合深入分析

# 为什么需要它？

iotop能看到实时IO，但想知道某个进程"一整天读了多少数据"，pidstat能给出精确到秒的统计。

# 核心用法

```
# 监控进程IO，每秒1次，共10次
pidstat -d 1 10
```

# 关键指标

- **kB_rd/s**：进程每秒读取的KB数（长期高于10MB/s可能有问题）
- **kB_wr/s**：进程每秒写入的KB数（比如日志程序频繁写小文件会导致该值高）
- **Command**：进程名称（结合PID定位具体服务，如mysqld、java等）

# 案例场景

某数据库服务器IO高，用pidstat -d -p [mysql_pid] 1发现kB_wr/s持续20MB/s，排查后发现是某定时任务频繁写入临时表，优化SQL后写入量降为5MB/s。

# 四、vmstat：系统"全景监控"工具

**一句话概括**：不仅看IO，还能看CPU、内存，适合初步判断瓶颈类型

# 为什么需要它？

有时候系统慢，分不清是CPU、内存还是IO问题，vmstat能"一站式"给出系统整体状态。

# 核心用法

```
# 每秒1次，共5次，显示磁盘IO相关指标
vmstat -d 1 5
```

# 关键IO指标（重点2个）

- **bi/bo**：每秒从磁盘读/写的块数（单位：块/秒，1块=512字节）
- **wa**：CPU等待IO的时间占比（超过20%说明IO是瓶颈，CPU在"摸鱼等磁盘"）

![img](https://p3-sign.toutiaoimg.com/tos-cn-i-axegupay5k/3d06801aa0234e02a74d82544a046c73~tplv-tt-origin-web:gif.jpeg?_iz=58558&from=article.pc_detail&lk3s=953192f4&x-expires=1756534456&x-signature=weQjkgJ9Bn%2BpxotxocOOtm3xNeM%3D)



# 解读技巧

如果wa高但bi/bo低，可能是磁盘碎片或RAID卡问题；如果bi/bo高且wa高，才是业务IO量大导致的瓶颈。

# 五、dstat：全能型"监控仪表盘"

**一句话概括**：整合CPU、内存、IO、网络监控，适合快速 overview

# 为什么需要它？

运维时需要同时看IO和CPU，dstat能在一个界面展示多维度数据，像汽车仪表盘一样直观。

# 核心用法

```
# 显示CPU、磁盘、网络和高IO进程
dstat -cdng --top-io
```

# 界面亮点

- **磁盘行**：read/writ显示读写速度，util显示磁盘利用率
- **--top-io**：自动显示最耗IO的进程（不用手动找）

![img](https://p3-sign.toutiaoimg.com/tos-cn-i-6w9my0ksvp/8646300aa4b04511aaebd267d9f80563~tplv-tt-origin-web:gif.jpeg?_iz=58558&from=article.pc_detail&lk3s=953192f4&x-expires=1756534456&x-signature=RISPUBLXp4TQbvUTBMG1IyT6Zmo%3D)



# 优势场景

做压力测试时，用dstat -cdng 1实时观察"CPU升高时IO是否跟着高"，快速判断瓶颈关联性。

# 排查流程总结：3步定位IO瓶颈

1. **初步判断**：iostat -x 1看%util和await，确认是否IO瓶颈
2. **定位进程**：iotop -o找出高IO进程，记录PID
3. **深入分析**：pidstat -d -p [PID] 1看具体读写量，结合业务优化

**小提醒**：IO瓶颈不一定是磁盘慢，也可能是"小文件太多""缓存没用好"，排查时记得结合业务日志哦！

> 你在运维中遇到过哪些"奇葩"IO问题？用这些命令解决过吗？欢迎在评论区分享你的经验～