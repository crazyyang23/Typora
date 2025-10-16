作者：江小北
链接：https://www.zhihu.com/question/11850080815/answer/129326845387
来源：知乎
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。



没错，反向代理 [WebSocket](https://zhida.zhihu.com/search?content_id=719230772&content_type=Answer&match_order=1&q=WebSocket&zhida_source=entity) 一度是很多开发者的噩梦，大家都会像追寻失落的神器一样去寻找它的真相。

简直像是考试中的背水一战——你明明知道自己能解出来，但总是差点什么。

### 一、你遇到的问题到底是什么？

大概情况是这样的：**你用 Nginx 配置反向代理 WebSocket，结果总是连接不上，直接访问 WebSocket 端口又没问题**。

嗯，看到这个问题，常见的原因大致可以分为两大类：

1. **WebSocket 和 [HTTP 协议](https://zhida.zhihu.com/search?content_id=719230772&content_type=Answer&match_order=1&q=HTTP+协议&zhida_source=entity)的区别**：WebSocket 是一个全双工通信协议，建立连接后就不再是传统的 HTTP 请求——因此，它需要特别的处理。
    
2. **Nginx 配置问题**：反向代理需要特殊的配置来正确处理 WebSocket，简单的配置可能会忽略 WebSocket 特有的头信息，导致连接失败。
    

### 二、问题背后的原因

我们先来剖析下背后的技术原理，再逐步解答你的困惑。

### 1. **WebSocket 协议的特性**

WebSocket 和 HTTP 的不同，别再傻傻地认为它只是“HTTP 协议的一个兄弟”，其实它是完全不同的一个连接。在客户端发起连接时，它会通过 HTTP 协议发送一个 **Upgrade 请求**，请求服务器升级协议为 WebSocket。所以，如果我们通过 Nginx 反向代理，必须正确处理这个“[协议升级](https://zhida.zhihu.com/search?content_id=719230772&content_type=Answer&match_order=1&q=协议升级&zhida_source=entity)”过程。

### 2. **Nginx 配置的痛点：头部处理**

Nginx 作为反向代理服务器，在处理 HTTP 请求时，它并不会默认转发所有 HTTP 头，尤其是 WebSocket 特有的头部字段。比如：

- `Upgrade`
- `Connection`

这两个头部非常重要。缺少其中任何一个，WebSocket 连接都无法建立成功。

### 三、解决方案：一步一步弄懂 Nginx 配置

接下来进入解决方案，手把手带你配置好 Nginx，确保 WebSocket 能成功反向代理。

### 1. **Nginx 配置示例**

这里假设你的 WebSocket 服务运行在 `ws://127.0.0.1:8080`，客户端通过 Nginx 访问 WebSocket 服务。你可以参考以下配置：

```text
server {
    listen 80;
    server_name yourdomain.com;

    location /ws/ {
        proxy_pass http://127.0.0.1:8080;
        
        # 必须传递这两个重要的 HTTP 头
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        
        # 允许超长连接
        proxy_read_timeout 86400;
    }
}
```

### 2. **配置解释**

- `proxy_http_version 1.1;`：WebSocket 必须通过 HTTP/1.1 协议建立连接。如果你没有这条配置，Nginx 会默认使用 HTTP/1.0 协议，这会导致 WebSocket 连接失败。
   
- `proxy_set_header Upgrade $http_upgrade;`：WebSocket 在连接时会发送一个 `Upgrade` 请求头，表示希望从 HTTP 协议升级到 WebSocket 协议。Nginx 必须转发这个头部。
   
- `proxy_set_header Connection 'upgrade';`：这表示要保持连接并升级为 WebSocket 协议。如果没有这个配置，Nginx 可能就会中断连接，导致连接无法建立。
   
- `proxy_read_timeout 86400;`：WebSocket 连接通常会持续较长时间。这个配置确保 Nginx 在长时间连接下不会关闭连接。
   

### 3. **常见的坑**

- **端口问题**：确保 WebSocket 服务监听的是正确的端口，并且该端口在防火墙上开放。如果你直接用端口号能连通，而反向代理不行，检查 Nginx 是否配置正确。
   
- **跨域问题**：WebSocket 与浏览器的跨域问题也是一种常见问题，尤其是在你跨不同的域访问时。确保 Nginx 配置了正确的 [CORS 头部](https://zhida.zhihu.com/search?content_id=719230772&content_type=Answer&match_order=1&q=CORS+头部&zhida_source=entity)。一般来说，WebSocket 本身不受同源策略的限制，但如果有跨域请求，可以通过设置 `Access-Control-Allow-Origin` 等头部来解决。
   

```text
add_header 'Access-Control-Allow-Origin' '*';
add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS';
add_header 'Access-Control-Allow-Headers' 'Origin, X-Requested-With, Content-Type, Accept, Authorization';
```

### 四、总结：配置完成后的验证与调试

配置好 Nginx 后，记得重启 Nginx 服务：

```text
sudo systemctl restart nginx
```

验证 WebSocket 是否正常连接，你可以用浏览器的开发者工具 (F12) 来查看网络请求，确保 WebSocket 请求的 `Upgrade` 头部正常传递。

如果还是无法连接，建议使用 `curl` 来模拟 WebSocket 连接：

```text
curl -i -X GET http://yourdomain.com/ws/
```

你应该看到类似以下的响应头（如果一切正常）：

```text
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
```

### 五、额外思考：Nginx 的高级配置

你可以进一步优化 Nginx 的性能，特别是当 WebSocket 连接数很多时，考虑调整以下参数：

- `worker_connections`：提高最大连接数，避免因连接数过多导致 Nginx 无法处理新的请求。
- `keepalive_timeout`：调整连接超时时间，避免 WebSocket 连接过早被关闭。

```text
worker_connections  10240;  # 设置最大连接数
keepalive_timeout  65;       # 设置连接超时时间
```

### 结语：一步一步，渐入佳境

看到这里，应该能理解为什么 WebSocket 和 Nginx 配置之间有那么多细节。如果你照着以上步骤走，基本上能解决大部分的问题。如果依然不行，那就检查一下 Nginx 的日志文件，可能是其他配置影响了 WebSocket 连接。

总之，WebSocket 在 Nginx 中的配置并不复杂，只要弄懂了背后的原理，配置就是小菜一碟。希望这篇文章能帮你解决问题，遇到啥技术难题，别忘了来找我聊聊。
