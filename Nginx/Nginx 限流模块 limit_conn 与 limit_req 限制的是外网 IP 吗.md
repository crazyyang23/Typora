一. 前言 　　本文针对 Nginx 的三个模块进行配置，并证实各自的功能特点： 　

（1）limit_conn_zone 模块 - 限制统一 IP 地址并发连接数； 　　

（2）limit_request 模块 - 限制同一 IP 某段时间的访问量； 　　

（3）core 模块提供 - limit_rate 限制同一 IP 流量。 　　

在 Nginx 中 以 LIMIT 开头的 配置项，都是做 限制 功能，以上三个功能都是 Nginx 编译后就有的功能，属于内置模块。 location  /vlovev/ {
            if ( $http_x_forwarded_for = 117.139.252.46 ) {
                	return 200;
	    }
            proxy_pass http://api.vlovev.cn;

            proxy_set_header Accept-Encoding "";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $http_x_forwarded_for;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      
        add_header From edu.vlovev.cn ;
    
        proxy_cookie_path /edu/ /;
        proxy_set_header Cookie $http_cookie;
        proxy_connect_timeout 60000; 
        proxy_read_timeout 60000;
        proxy_send_timeout 60000; 
} 二. limit_conn_zone 模块 通过 limit_zone 模块来达到限制用户的连接数的目的，即限制同一用户 IP 地址的并发连接数。 2.1 配置示例   2.2 指令 指令名称：limit_conn_zone（nginx 1.18以后用 limit_conn_zone 取代了 limit_conn）
语法：limit_conn_zone key zone=name:size;
默认：no
区域：http
功能：该指令定义一个 zone，该 zone 存储会话的状态。
例如：上面的例子中，$binary_remote_addr 是 获取客户端ip地址的变量，长度为 4 字节，会话信息的长度为 32 字节。 
 指令名称：limit_conn
语法：limit_conn zone number;
默认：no
区域：http、server、location
功能：该指令用于为一个会话设定最大并发连接数。如果并发请求超过这个限制，那么将返回预定错误（limit_conn_status ） 
 指令名称：limit_conn_status
语法：limit_conn_status code;
默认：limit_conn_status 503;
区域：http、server、location
功能：设置要返回的状态码以响应被拒绝的请求。 
 指令名称：limit_conn_log_level
语法：limit_conn_log_level info | notice | warn | error
默认值：error
区域：http、server、location
功能：该指令用于设置日志的错误级别，当达到连接限制时，将会产生错误日志。 上面的配置示例中，没有显式配置 limit_conn_status 、limit_conn_log_level ，如果没有配置，则启用它们的默认值。 
 2.3 测试 接下来，对 limit_conn_zone 模块的功能进行测试。 测试1：对页面访问的并发限制 首先，来尝试用户对页面访问的并发限制，配置如下：   通过配置，设定 location /download 这个区域，每个IP，同一时刻只存在一个连接。注意：并发的概念并不是说 每秒多少连接。 生成一个较大的 index.html 页面： [root@vlovev /usr/local/nginx/conf]#cd ../html/
[root@vlovev /usr/local/nginx/html]#mkdir download/
[root@vlovev /usr/local/nginx/html]#cd download/
[root@vlovev /usr/local/nginx/html/download]#for i in {1..10000}; do echo "hello, $i" >> index.html ; done
[root@vlovev /usr/local/nginx/html/download]#ll -sh index.html 
192K -rw-r--r-- 1 root root 117K Mar  8 21:16 index.htm 客户端主机：通过 ab 命令模拟并发访问： [root@client ~]#ab -n 10 -c 10 http://vlovev.cn/download/index.html
-n：总请求数：10
-c：单个时刻并发 10   通过上面的测试，只有 2 个连接错误。经过多次测试，这个问题依然存在，也查阅过一些资料，有人表示这是个 BUG 的。目前还是表示疑惑的，希望有人能够帮忙解惑。 测试2：对文件下载进行测试，或者是访问较大的文件 生成一个 100M 的文件，因为局域网 千兆网络，所以生成得比较大。 [root@vlovev /usr/local/nginx/html/download]#dd if=/dev/zero of=testfile bs=1M count=100 然后再用 ab 进行测试： [root@client ~]#ab -n 10 -c 10 http://vlovev.cn/download/testfile   经过这次的测试，完成了 10 个并发连接，其中 9 个返回的是 非200 的状态。继续查看 Nginx 日志：   10 个并发连接，其中只有 1 个返回 200 状态，其余 9 个都是 503 状态，这样的测试结果和设定的配置相符。 
 2.4 总结 通过上面两次测试结果来看：第一次测试结果，配置的并发请求为 1 的限制貌似没有起到作用。第二次测试结果，完全符合配置项的限制规则。而这两次测试唯一的不同就是：数据文件大小不同。 这里只能猜想：对于较小的文件或页面，在千兆网络的环境里，单个时刻请求数据的速度很快，并没有造成多个连接同时并发的情况产生，而对于较大的页面或文件来说，文件正在传输的时候处于 ESTABLISHED连接时间长，非常容易形成并发的效果，并发就会触发限制规则。这也只是简单的猜想，希望有大神能够给出具体的科学的解释。 三. limit_request 模块 使用 ngx_http_limit_req_module 模块可以 限制某一 IP 在一段时间内对服务器发起请求的连接数，该模块为内置模块。 3.1 配置示例   3.2 指令 指令名称：limit_req_zone
语法：limit_req_zone key zone=name:size rate= number r/s
默认值：no
区域：http
使用示例：limit_req_zone $binary_remote_addr zone=addr:10m rate=1r/s

对于上面的示例：
$binary_remote_addr ：表示通过remote_addr 这个标识用来做限制.

zone=addr:10m：表示生成一个 10M ，名字为 addr 的内存区域，用来存储访问的频次信息

rate=1r/s：表示允许相同标识的客户端的访问频次，这里限制的是每秒1次，即每秒只处理一个请求，还可以有比如 30r/m ， 即限制每 2秒 访问一次，即每 2秒 才处理一个请求。 
 指令名称：limit_req
语法：limit_req zone=name [burst=number] [nodelay | delay=number];
默认：no
区域：http、server、location
使用示例：limit_req zone=zone burst=5 nodelay;

zone=zone：设置使用哪个配置名来做限制，与上面 limit_req_zone 里的 name 对应

burst=5 ：这个配置的意思是设置一个大小为5的缓冲区，当有大量请求过来时，超过访问频次限制 rate=1r/s 的请求可以先放到这个缓冲区内等待，但是这个缓冲区只有5个位置，超过这个缓冲区的请求直接报503并返回。

nodelay：如果设置，会在瞬间提供处理（rate+burst）个请求的能力，请求超时（rat+burst）的时候直接返回503，永远不存在请求需要等待的情况。如果没有设置，则所有请求会依次等待排队； 
 指令名称：limit_req_status
语法：limit_req_status code;
默认：limit_req_status 503;
区域：http、server、location
功能：设置要返回的状态码以响应被拒绝的请求。 
 指令名称：limit_req_log_level（该指令出现在版本0.8.18中）
语法：limit_req_log_level info | notice | warn | error;
默认：limit_req_log_level error;
区域：http、server、location
功能：该指令用于设置日志的错误级别，当达到连接限制时，将会产生错误日志。 上面的示例内容没有显式写明 limit_req_status 、limit_req_log_level 两个配置项，所以采用默认值。 这里有几个不太好理解地方：   limit_req_zone 后面的 rate=1r/s limit_req burst=5
rate 为规定时间内连接请求的数量，单位（request/second）
burst 为爆咋的意思，这里可以理解为连接等待队列长度。 这里就使用到了 ‘漏斗算法（Leaky Bucket）’，该算法有两种处理方式 Traffic Shaping 和 Traffic Policing 在桶满了之后，常用的两种处理方式为： 1. 暂时拦截住上方水的向下流动，等待桶中的一部分水漏走后，再放行上方水；

2. 溢出的上方水直接抛弃。 举个栗子： 比如上面的配置： rate=1r/s burst=5
第一秒来了6个请求 - 1 个处理 - 5个等待；
第二秒来了2个请求 - 1个处理 - 5个等待 - 1 个丢弃；
上面总共8个请求，时间2秒，2秒处理了2个请求，还剩6个，但是缓冲队列只能存放5个请求，剩下1个丢弃。
或者
第一秒：来了10个请求 - 1个处理 - 5个等待 - 4个丢弃 就是按照这样的方式来进行计算的，也就是说，上面的配置，每秒最多只能保持 6 个请求，但是每秒最多只能处理 1 个请求。 
 3.3 测试 本模块测试分为三种不同的方式进行测试验证： 　　（1）不加 burst 和 不加 nodelay 　　（2）加 burst 和 不加 nodelay 　　（3）加burtst 和 加 nodelay 
 测试1 - 不加 burst 和 不加 nodelay 配置如下：   客户端主机： 通过 ab 命令模拟并发访问： [root@client ~]#ab -n 10 -c 10 http://vlovev.cn/index.html   通过 ab 测试后的结果，10 个并发连接，只有 1 个成功，剩余 9 个都返回 非200 状态，查看 Nginx 日志：   1秒钟的10次并发请求，只有 1 个返回 200 ，剩余全是 503 请求被拒绝。 不加 burst 和 不加 nodelay 的情况下，rate=1r/s 1 秒钟只能处理 1 个请求，剩余的所有请求都会直接返回 503
测试2 - 加 burst 和 不加 nodelay 配置如下：   客户端主机：client 通过 ab 命令模拟并发访问： [root@client ~]#ab -n 10 -c 10 http://vlovev.cn/index.html   处理了10个连接请求，其中失败了 4 个，成功了 6个。 在本次测试中，使用 burst = 5 建立了一个可以存放 5 个并发连接的缓冲区。根据上面的 漏斗算法 来进行分析： 第一秒：10个连接并发请求，1个处理 5个等待，4个丢弃；
第二秒：1个处理 4 个等待；
第三秒：1个处理 3 个等待；
第四秒：1个处理 2 个等待；
第五秒：1个处理 1 个等待；
第六秒：1个处理 完毕。 通过上面的推算，一共需要 6 秒才能处理完所有的请求，查看 Nginx 日志，验证推算是不是正确：   通过日志，可以看出，第一秒处理了 1 个请求，拒绝了 4 个连接，剩下的请求分别是每秒 1个连接请求的处理。 规则：rate=1r/s 、burst=5
一次来了 10 个连接并发请求，处理 1 个，缓存 5 个后续 1 秒一个的处理，其他的全部丢弃。
加 burst 和 不加 nodelay 的情况下，rate=1r/s burst=5 处理 1 个，缓存 5 个后续 1 秒一个的处理，其他的全部丢弃。 
 测试3 - 加 burst 和 加 nodelay 配置如下：   客户端主机：client 通过 ab 命令模拟并发访问： [root@client ~]#ab -n 10 -c 10 http://vlovev.cn/index.html   10 个并发连接请求，很快就直接返回了，其中 4 个非200错误， 6 个成功。查看 Nginx 日志：   这次的测试结果，同一秒钟，处理了 rate+burst 个请求，其他的全部返回连接请求拒绝。 3.4 总结 通过测试的三种情况，返回了不同的结果，因此有必要详细说明： （1）不加 burst 和 不加 nodelay 的情况：
按照 rate 设定的规则，严格执行。
例如：rate=1r/s ，则1秒只处理1个请求，其他的全部返回连接503

（2）加 burst 和 不加 nodelay 的情况：
首先按照 rate 规则处理，并且缓存 burst 个连接，剩余的全部返回503.
后续缓存的 burst 按照 rate 规则进行处理

（3）加 burst 和 nodelay 的情况：
第一次处理 rate+burst 个连接请求，剩余的请求全部返回 503 四. limit_rate 根据 ip 限制流量 对于提供下载的网站，肯定是要进行流量控制的。Nginx 通过 core模块的 limit_rate 等指令可以做到限流的目的。 4.1 示例   4.2 指令 指令名称：limit_rate
语法：limit_rate speed;
默认值：no
使用环境：http、server、location
示例： limit_rate 512k;
功能：该指令用于指定向客户端传输数据的速度，速度的单位是每秒传输的字节数。注意：该限制只是针对一个连接的设定，也就是说，如果同时有2个连接，那么它的速度将会是该指令设置的两倍。 
 指令名称：limit_rate_after
语法：limit_rate_after size;
默认值：limit_rate_after 1m;
使用环境：http、server、location
示例：limit_rate_after 3m;
功能：以最大的速度下载 size大小后，在进行 limit_rate speed 限速，例如：limit_rate_after 3m 解释为：以最大的速度下载3m后，再进行限速。 
 4.3 测试 测试前疑问：对于这个模块是通过什么来进行限速的呢？ 上面的两个模块都有声明 $binary_remote_addr 远端ip地址进行操作的，而 limit_rate 什么都没规定。针对这个问题，做以下测试： 配置如下：   正常速度下载 3m数据后，限速 512k 的速度下载。 生成一个较大的下载文件： [root@vlovev /usr/local/nginx/html]#dd if=/dev/zero of=testfile bs=1M count=50 这次通过，curl 来下载文件：   这里使用的是千兆网络，超过3m，速度被限制在512K 以下。这是一个连接下载，如果同时开启多个终端，都进行下载呢？ 连接-1   连接-2   因为 limit_rate 并没有声明以什么条件作为限制，所以同一个ip无论发起多少个请求，每个请求都会是 512k 下载。在迅雷等多种下载软件中，使用多线程的方式下载同一个文件，这个速度就翻倍了。 那这个限速就形同虚设。

 因此，如果要进行限速，可以和 limit_conn_zone 模块配合进行使用。配置如下：   



![image-20250425083607351](C:\Users\FanYang\AppData\Roaming\Typora\typora-user-images\image-20250425083607351.png)

同一时刻，只有一个连接请求。 再进行同主机多线程下载： 

连接-1  

![image-20250425083541041](C:\Users\FanYang\AppData\Roaming\Typora\typora-user-images\image-20250425083541041.png)

 连接-2 

![image-20250425083528688](C:\Users\FanYang\AppData\Roaming\Typora\typora-user-images\image-20250425083528688.png)

  当再次发起第二个连接的时候，服务器就直接返回连接拒绝 503，这样就达到了限速的目的。 
 4.4 总结 　　当需要进行限速操作时，需要 limit_rate 和 limit_conn 模块联合起来使用才能达到限速的效果。