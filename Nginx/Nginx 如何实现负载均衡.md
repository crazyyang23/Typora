作者：头像我女儿
链接：https://www.zhihu.com/question/571270100/answer/1933669212871713056
来源：知乎
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。



## **前言**

[Nginx](https://zhida.zhihu.com/search?content_id=739512181&content_type=Answer&match_order=1&q=Nginx&zd_token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJ6aGlkYV9zZXJ2ZXIiLCJleHAiOjE3NTYxMDI4MzIsInEiOiJOZ2lueCIsInpoaWRhX3NvdXJjZSI6ImVudGl0eSIsImNvbnRlbnRfaWQiOjczOTUxMjE4MSwiY29udGVudF90eXBlIjoiQW5zd2VyIiwibWF0Y2hfb3JkZXIiOjEsInpkX3Rva2VuIjpudWxsfQ.rDNE5pNNfJrpDAr8CxkYYMMXZrV8aAFEO-mr33hK-tc&zhida_source=entity)有[负载均衡](https://zhida.zhihu.com/search?content_id=739512181&content_type=Answer&match_order=1&q=负载均衡&zd_token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJ6aGlkYV9zZXJ2ZXIiLCJleHAiOjE3NTYxMDI4MzIsInEiOiLotJ_ovb3lnYfooaEiLCJ6aGlkYV9zb3VyY2UiOiJlbnRpdHkiLCJjb250ZW50X2lkIjo3Mzk1MTIxODEsImNvbnRlbnRfdHlwZSI6IkFuc3dlciIsIm1hdGNoX29yZGVyIjoxLCJ6ZF90b2tlbiI6bnVsbH0.wnOhQzl4U2oqfKmfvaMBRGDlZUjBS4aFXnAblQMi1ko&zhida_source=entity)功能，实验目的是搭建一个Nginx作为应用负载入口，Nginx后面不同的项目需要实现访问此ip能够负载到不同项目，并且在不同项目中做负载。

## **实验环境**

| ip              | 主机名      | 系统                                                         |
| --------------- | ----------- | ------------------------------------------------------------ |
| 192.168.198.106 | nginx-proxy | [Rocky Linux](https://zhida.zhihu.com/search?content_id=739512181&content_type=Answer&match_order=1&q=Rocky+Linux&zd_token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJ6aGlkYV9zZXJ2ZXIiLCJleHAiOjE3NTYxMDI4MzIsInEiOiJSb2NreSBMaW51eCIsInpoaWRhX3NvdXJjZSI6ImVudGl0eSIsImNvbnRlbnRfaWQiOjczOTUxMjE4MSwiY29udGVudF90eXBlIjoiQW5zd2VyIiwibWF0Y2hfb3JkZXIiOjEsInpkX3Rva2VuIjpudWxsfQ.4Nn7TDKvfMFdiH7oIC-Sk58MWEivMgJ2VKVx40eNDKk&zhida_source=entity) release 8.8 |
| 192.168.198.107 | p1a         | Rocky Linux release 8.8                                      |
| 192.168.198.108 | p1b         | Rocky Linux release 8.8                                      |
| 192.168.198.109 | p2a         | Rocky Linux release 8.8                                      |
| 192.168.198.110 | p2b         | Rocky Linux release 8.8                                      |

## **环境准备**

nginx-proxy，直接通过yum安装nginx，其他nginx安装方式请看《Nginx离线部署》文章

```bash
[root@nginx-proxy ~]# yum install -y nginx
#启动nginx
[root@nginx-proxy ~]# systemctl status nginx
● nginx.service - The nginx HTTP and reverse proxy server
   Loaded: loaded (/usr/lib/systemd/system/nginx.service; disabled; vendor preset: disabled)
   Active: inactive (dead)

11月 13 08:37:50 nginx-proxy systemd[1]: nginx.service: Unit cannot be reloaded because it is inactive
[root@nginx-proxy ~]# systemctl start nginx
[root@nginx-proxy ~]# systemctl status nginx
● nginx.service - The nginx HTTP and reverse proxy server
   Loaded: loaded (/usr/lib/systemd/system/nginx.service; disabled; vendor preset: disabled)
   Active: active (running) since Wed 2024-11-13 08:39:53 EST; 13s ago
  Process: 30632 ExecStart=/usr/sbin/nginx (code=exited, status=0/SUCCESS)
  Process: 30631 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=0/SUCCESS)
  Process: 30629 ExecStartPre=/usr/bin/rm -f /run/nginx.pid (code=exited, status=0/SUCCESS)
 Main PID: 30634 (nginx)
    Tasks: 2 (limit: 4497)
   Memory: 6.8M
   CGroup: /system.slice/nginx.service
           ├─30634 nginx: master process /usr/sbin/nginx
           └─30635 nginx: worker process

11月 13 08:39:52 nginx-proxy systemd[1]: Starting The nginx HTTP and reverse proxy server...
11月 13 08:39:52 nginx-proxy nginx[30631]: nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
11月 13 08:39:52 nginx-proxy nginx[30631]: nginx: configuration file /etc/nginx/nginx.conf test is successful
11月 13 08:39:53 nginx-proxy systemd[1]: nginx.service: Failed to parse PID from file /run/nginx.pid: Invalid argument
11月 13 08:39:53 nginx-proxy systemd[1]: Started The nginx HTTP and reverse proxy server.
#设置开机自动启动
[root@nginx-proxy ~]# systemctl enable nginx.service 
Created symlink /etc/systemd/system/multi-user.target.wants/nginx.service → /usr/lib/systemd/system/nginx.service.
```

配置四台主机，使用httpd服务搭建简单的http服务器，四台操作相同，就是在配置页面的时候不同

```bash
[root@p1a ~]# yum install httpd -y
#配置开机自启并启动
[root@p1b ~]# systemctl enable httpd --now
#关闭防火墙
[root@p1a html]# systemctl stop firewalld.service && systemctl disable firewalld.service
#关闭selinux，临时关闭
[root@nginx-proxy ~]# setenforce 0

#配置页面
[root@p2b ~]# echo "hello Im is `hostname`" > /var/www/html/index.html

#测试页面访问
[root@p2b ~]# curl http://192.168.198.107
```



## **配置Nginx**

```text
#进入到nginx配置文件夹下
[root@nginx-proxy conf.d]# cd /etc/nginx/conf.d/
#添加nginx配置
[root@nginx-proxy conf.d]# cat project_a.conf
upstream pa {
        server 192.168.198.107:80;
        server 192.168.198.108:80;
}
server {
        #端口要在1000以上，我测试用101，浏览器一直访问不了
        listen 1000;
        server_name  localhost;
        location / {
        proxy_pass http://pa;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
                                                                        }
}
[root@nginx-proxy conf.d]# cat project_b.conf 
upstream pb {
        server 192.168.198.109:80;
        server 192.168.198.110:80;
}
server {
        listen 192.168.198.106:1001;
        location / {
        proxy_pass http://pb;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
                                                                        }
}
[root@nginx-proxy conf.d]# nginx -t
[root@nginx-proxy conf.d]# nginx -s reload
```

![img](https://picx.zhimg.com/80/v2-dea71b5807f9a114daf685159f10521a_720w.webp?source=2c26e567)



验证功能实现