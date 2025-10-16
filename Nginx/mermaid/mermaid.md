graph TB

  User[/用户请求

  DSBU5.gwkf.cn/] --> |HTTP请求| Nginx[Nginx服务器]

  Nginx -->LoadBalancer{负载均衡器}

  

  

  LoadBalancer --> Backend1[应用服务器]

  LoadBalancer --> Backend2[backup服务器]

  

  Backend1 --> SQLServer[数据库服务器]

  Backend2 --> SQLServer[数据库服务器]

  

  Backend1 -->|响应数据| Nginx

  Backend2 -->|响应数据| Nginx

  

  Nginx -->|最终响应| User