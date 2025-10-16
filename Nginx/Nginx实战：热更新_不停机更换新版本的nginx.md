## 热部署 （不停机更换新版本的nginx 二进制文件）

- 查看nginx进程
- 模拟 上传新版本，老版本的ng备份
- 发送 USR2 信号给 原来的ng的 pid
- 再次查看ng进程，会发现多出来几个， 此时老的ng已经不再监听了，流量会切到新的ng上来
- 向老的ng master 发送信息，优雅的关闭它的 work线程 （需要等待老业务都处理完了）

```java
# 查看nginx进程  
[root@VM-0-7-centos artisan_ng]# ps -ef|grep nginx |grep -v grep
root      447751       1  0 12:23 ?        00:00:00 nginx: master process ./nginx -c /root/ng/artisan_ng/conf/nginx.conf
nobody    454927  447751  0 13:14 ?        00:00:00 nginx: worker process
[root@VM-0-7-centos artisan_ng]#

[root@VM-0-7-centos artisan_ng]# pwd
/root/ng/artisan_ng
[root@VM-0-7-centos artisan_ng]# ls
client_body_temp  conf  fastcgi_temp  html  logs  proxy_temp  sbin  scgi_temp  uwsgi_temp
[root@VM-0-7-centos artisan_ng]# cd sbin/
# 模拟 上传新版本，老版本的ng备份 
[root@VM-0-7-centos sbin]# cp nginx nginx_old
[root@VM-0-7-centos sbin]#
# 发送  USR2 信号给 原来的ng的 pid 
[root@VM-0-7-centos sbin]# kill -USR2 447751
[root@VM-0-7-centos sbin]#
# 再次查看ng进程，会发现多出来几个， 此时老的ng已经不再监听了，流量会切到新的ng上来 
[root@VM-0-7-centos sbin]# ps -ef|grep nginx
root      447751       1  0 12:23 ?        00:00:00 nginx: master process ./nginx -c /root/ng/artisan_ng/conf/nginx.conf
nobody    454927  447751  0 13:14 ?        00:00:00 nginx: worker process
root      455372  447751  0 13:17 ?        00:00:00 nginx: master process ./nginx -c /root/ng/artisan_ng/conf/nginx.conf
nobody    455373  455372  0 13:17 ?        00:00:00 nginx: worker process
root      455386  454554  0 13:17 pts/1    00:00:00 grep --color=auto nginx
[root@VM-0-7-centos sbin]#
[root@VM-0-7-centos sbin]#
# 向老的ng master 发送信息，优雅的关闭它的 work线程   （需要等待老业务都处理完了）
[root@VM-0-7-centos sbin]# kill -WINCH 447751
[root@VM-0-7-centos sbin]#
[root@VM-0-7-centos sbin]#

# 老的master进程 还是存在的， 已经没有work进程了 。 老的ngxin存在 便于我们进行版本回退 （可以给老的nginx 发送 reload命令）
[root@VM-0-7-centos sbin]# ps -ef|grep nginx |grep -v grep
root      447751       1  0 12:23 ?        00:00:00 nginx: master process ./nginx -c /root/ng/artisan_ng/conf/nginx.conf
root      455372  447751  0 13:17 ?        00:00:00 nginx: master process ./nginx -c /root/ng/artisan_ng/conf/nginx.conf
nobody    455373  455372  0 13:17 ?        00:00:00 nginx: worker process
[root@VM-0-7-centos sbin]#
[root@VM-0-7-centos sbin]#
```

------

## 热更新流程

**1、** 将旧Nginx件换成新Nginx件(注意备份)；
**2、** 向master进程发送USR2信号；
**3、** master进程修改pid件名,加后缀.oldbin；
**4、** master进程用新Nginx件启动新master进程；
**5、** 向老master进程发送WINCH号,关闭老worker；
**6、** 回滚:向老master送HUP,向新master送QUIT；

![*](https://cloud.cxykk.com/images/2024/2/2/1515/1706858126878.png)