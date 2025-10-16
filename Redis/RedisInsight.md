You can install Redis Insight using one of the options described below.

1. If you do not want to persist your Redis Insight data:

```bash
docker run -d --name redisinsight -p 5540:5540 redis/redisinsight:latest
```



1. If you want to persist your Redis Insight data, first attach the Docker volume to the `/data` path and then run the following command:

```bash
docker run -d --name redisinsight -p 5540:5540 redis/redisinsight:latest -v redisinsight:/data
```



If the previous command returns a permission error, ensure that the user with `ID = 1000` has the necessary permissions to access the volume provided (`redisinsight` in the command above).

Next, point your browser to `http://localhost:5540`.

Redis Insight also provides a health check endpoint at `http://localhost:5540/api/health/` to monitor the health of the running container.

如果你不想持久化 Redis Insight 的数据：
docker run -d --name redisinsight -p 5540:5540 redis/redisinsight:latest



如果你想持久化 Redis Insight 的数据，先将 Docker 卷挂载到 /data 路径，然后执行以下命令：
docker run -d --name redisinsight -p 5540:5540 redis/redisinsight:latest -v redisinsight:/data



如果前面的命令返回权限错误，请确保 ID 为 1000 的用户拥有访问所提供卷（上述命令中的 redisinsight）的必要权限。



接下来，将浏览器指向[http://localhost:5540](http://localhost:5540/)。



Redis Insight 还提供了一个健康检查端点http://localhost:5540/api/health/，用于监控运行中容器的健康状态。