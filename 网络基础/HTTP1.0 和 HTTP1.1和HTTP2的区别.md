HTTP1.0 和 HTTP1.1的区别

主要是如下 8 点：

    1、可扩展性
    
    2、缓存
    
    3、带宽优化
    
        带来了分块传输
    
        HTTP/1.1中在请求消息中引入了range头域，它允许只请求资源的某个部分。在响应消息中Content-Range头域声明了返回的这部分对象的偏移值和长度。如果服务器相应地返回了对象所请求范围的内容，则响应码为206（Partial Content），它可以防止Cache将响应误以为是完整的一个对象。
    
        HTTP/1.1加入了一个新的状态码100（Continue）。客户端事先发送一个只带头域的请求，如果服务器因为权限拒绝了请求，就回送响应码401（Unauthorized）；如果服务器接收此请求就回送响应码100，客户端就可以继续发送带实体的完整请求了。注意，HTTP/1.0的客户端不支持100响应码。但可以让客户端在请求消息中加入Expect头域，并将它的值设置为100-continue。
    
    4、长连接
    
        HTTP 1.1支持长连接（PersistentConnection）和请求的流水线（Pipelining）处理，在一个TCP连接上可以传送多个HTTP请求和响应，减少了建立和关闭连接的消耗和延迟。例如：一个包含有许多图像的网页文件的多个请求和应答可以在一个连接中传输，但每个单独的网页文件的请求和应答仍然需要使用各自的连接。
    
        HTTP 1.1还允许客户端不用等待上一次请求结果返回，就可以发出下一次请求，但服务器端必须按照接收到客户端请求的先后顺序依次回送响应结果，以保证客户端能够区分出每次请求的响应内容，这样也显著地减少了整个下载过程所需要的时间。
    
        在HTTP/1.0中，要建立长连接，可以在请求消息中包含Connection: Keep-Alive头域，如果服务器愿意维持这条连接，在响应消息中也会包含一个Connection: Keep-Alive的头域。同时，可以加入一些指令描述该长连接的属性，如max，timeout等。
    
    5、消息传递
    
    6、Host 头域
    
    7、错误提示
    
    8、内容协商

详细的每一点的说明，可以看 《HTTP1.0 与 HTTP1.1 的区别》 文章，特别是第 4 点【长连接】。

    HTTP1.1 支持长连接（PersistentConnection）和请求的流水线（Pipelining）。
    
        长连接（PersistentConnection）：处理在一个 TCP 连接上可以传送多个 HTTP 请求和响应，减少了建立和关闭连接的消耗和延迟。在 HTTP1.1中 默认开启Connection：keep-alive ，一定程度上弥补了 HTTP1.0 每次请求都要创建连接的缺点。
    
        请求的流水线（Pipelining）：HTTP1.1 还允许客户端不用等待上一次请求结果返回，就可以发出下一次请求，但服务器端必须按照接收到客户端请求的先后顺序依次回送响应结果，以保证客户端能够区分出每次请求的响应内容，这样也显著地减少了整个下载过程所需要的时间。
    
    推荐，在看看 《HTTP Keep-Alive 是什么？如何工作？》 文章。
    
    关于这一点，可能演变的问题有：
    
        HTTP 的长连接是什么意思？
    
        HTTP Keep-Alive 机制是什么？
    
        HTTP Keep-Alive 机制和 TCP Keep-Alive 有什么区别？




 ![img](http://mmbiz.qpic.cn/mmbiz_png/cmOLumrNib1cfBOtIMQ6JfSibJdd6QkQribgQuEeJaevuN9LRgQ9WR85hRiaVISeia7SDz1aU9hAAgO33XFaJ3FhmhQ/0?wx_fmt=png) 







# HTTP2 ![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9yYXcuZ2l0aHVidXNlcmNvbnRlbnQuY29tL0pEYXduRi9sZWFybmluZ19ub3RlL21hc3Rlci9pbWFnZXMvNDgyYTEyN2Q4ZmQwNzk0MGQ4ZjI1NTYxMGM3MTY3OTIuanBlZw?x-oss-process=image/format,png) 

