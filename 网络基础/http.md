https://blog.csdn.net/striveb/article/details/91585050



https://blog.csdn.net/striveb/article/details/91585050  **不错的总结**

一、HTTP

HTTP 协议，是 Hyper Text Transfer Protocol（超文本传输协议）的缩写，是用于从万维网（WWW:World Wide Web ）服务器传输超文本到本地浏览器的传送协议。

    HTTP 是一个无状态的协议。无状态是指客户机(Web 浏览器)和服务器之间不需要建立持久的连接，这意味着当一个客户端向服务器端发出请求，然后服务器返回响应(response)，连接就被关闭了，在服务器端不保留连接的有关信息。HTTP 遵循请求(Request)/应答(Response)模型。客户机(浏览器)向服务器发送请求，服务器处理请求并返回适当的应答。所有 HTTP 连接都被构造成一套请求和应答。

主要特点如下：

    简单快速：客户向服务器请求服务时，只需传送请求方法和路径。请求方法常用的有 GET、HEAD、POST 等等。每种方法规定了客户与服务器联系的类型不同。由于 HTTP 协议简单，使得 HTTP 服务器的程序规模小，因而通信速度很快。
    
    数据格式灵活：HTTP 允许传输任意类型的数据对象。正在传输的类型由Content-Type 加以标记。
    
    无连接：无连接的含义是限制每次连接只处理一个请求。服务器处理完客户的请求，并收到客户的应答后，即断开连接。采用这种方式可以节省传输时间。
    
        主要指的是不使用 Keep-Alive 机制的情况下。
    
    无状态：HTTP 协议是无状态协议。无状态，是指协议对于事务处理没有记忆能力。无状态意味着如果后续处理需要前面的信息，则它必须重传，这样可能导致每次连接传送的数据量增大。另一方面，在服务器不需要先前信息时它的应答就较快。
    
        无状态，所以更容易做服务的扩容，支撑更大的访问量。
    
    支持 B/S 及 C/S 模式。
    
        另外，HTTP 协议已经不仅仅使用在浏览器上。在前后端分离的架构中，又或者微服务架构的内部通信中，HTTP 因为其数据格式的通用性，和语言无关，被大规模使用。



HTTP 基本格式

    参照 《猫哥网络编程系列：详解 BAT 面试题》 

? HTTP 请求格式

    请求行：用来说明请求类型，要访问的资源以及所使用的 HTTP 版本。
    
    请求头部：紧接着请求行（即第一行）之后的部分，用来说明服务器要使用的附加信息从第二行起为请求头部。
    
        HOST ，将指出请求的目的地。
    
        User-Agent ，服务器端和客户端脚本都能访问它,它是浏览器类型检测逻辑的重要基础。该信息由你的浏览器来定义，并且在每个请求中自动发送等等
    
    空行：请求头部后面的空行是必须的。
    
    请求数据：也叫主体，可以添加任意的其他数据。

HTTP 响应格式

    状态行：由 HTTP 协议版本号、状态码、状态消息三部分组成。
    
    消息报头：用来说明客户端要使用的一些附加信息。
    
    空行：消息报头后面的空行是必须的。
    
    响应正文：服务器返回给客户端的文本信息。




HTTP 协议请求方法

    GET: 对服务器资源的简单请求。
    
    POST: 用于发送包含用户提交数据的请求。
    
    HEAD：类似于 GET 请求，不过返回的响应中没有具体内容，用于获取报头。
    
    PUT：传说中请求文档的一个版本。
    
    DELETE：发出一个删除指定文档的请求。
    
    TRACE：发送一个请求副本，以跟踪其处理进程。
    
    OPTIONS：返回所有可用的方法，检查服务器支持哪些方法。
    
    CONNECT：用于 SSL 隧道的基于代理的请求。

GET 和 POST 的区别
请求方式	数据位置	明文密文	数据安全	长度限制	应用场景
GET	HTTP 请求的 path 中	明文	不安全	长度较小，一般 2k	查询数据
POST	HTTP 请求 body 中	可明可密	安全	支持较大数据传输	修改数据

    GET在浏览器回退时是无害的，而POST会再次提交请求。
    
    GET产生的URL地址可以被添加到书签，而POST不可以。
    
    GET请求会被浏览器主动cache，而POST不会，除非手动设置。
    
    GET请求只能进行url编码，而POST支持多种编码方式。
    
    GET请求参数会被完整保留在浏览器历史记录里，而POST中的参数不会被保留。
    
    GET请求在URL中传送的参数是有长度限制的，而POST没有。

对参数的数据类型，GET只接受ASCII字符，而POST没有限制。

GET比POST更不安全，因为参数直接暴露在URL上，所以不能用来传递敏感信息。

GET参数通过URL传递，POST放在Request body中。

GET 请求可被缓存；POST 请求不会被缓存。

GET 请求可被收藏为书签；POST 不能被收藏为书签。

参见 《99%的人理解错 HTTP 中 GET 与 POST 的区别》

        对于 GET 方式的请求，浏览器会把 HTTP header 和 data 一并发送出去，服务器响应 200（返回数据）。
    
        而对于 POST，浏览器先发送 header ，服务器响应 100 continue ，浏览器再发送 data ，服务器响应 200 ok（返回数据）。
    
        本质上来说：get和post本质上都是基于TCP/IP的HTTP协议的请求方式，也就是说这两者本质上TCP连接。此外，要注意：GET产生一个TCP数据包；POST产生两个TCP数据包。
    
    也就是说，GET 只需要汽车跑一趟就把货送到了，而 POST得跑两趟，第一趟，先去和服务器打个招呼“嗨，我等下要送一批货来，你们打开门迎接我”，然后再回头把货送过去。
    
    ps：不过要注意，POST 具体发几次，也和浏览器的实现有关系。例如：Firefox 只发一次。 ps2：据研究，在网络环境好的情况下，发一次包的时间和发两次包的时间差别基本可以无视。而在网络环境差的情况下，两次包的 TCP 在验证数据包完整性上，有非常大的优点。



# HTTP 协议中 URI 和 URL 有什么区别？

URI 在于I(Identifier)是统一资源标示符，可以唯一标识一个资源。
URL在于Locater，一般来说（URL）统一资源定位符，可以提供找到该资源的路径，比如http://www.zhihu.com/question/21950864，但URL又是URI，因为它可以标识一个资源，所以URL又是URI的子集。
举个是个URI但不是URL的例子：urn:isbn:0-486-27557-4，这个是一本书的isbn，可以唯一标识这本书，更确切说这个是URN。
总的来说，**locators are also identifiers**, so every URL is also a URI, but there are URIs which are not URLs.





# [HTTP Keep-Alive是什么？如何工作？](https://www.cnblogs.com/zhangxian/articles/4747467.html)

#### HTTP Keep-Alive

在http早期，每个http请求都要求打开一个tpc socket连接，并且使用一次之后就断开这个tcp连接。

使用keep-alive可以改善这种状态，即在一次TCP连接中可以持续发送多份数据而不会断开连接。通过使用keep-alive机制，可以减少tcp连接建立次数，也意味着可以减少TIME_WAIT状态连接，以此提高性能和提高httpd服务器的吞吐率(更少的tcp连接意味着更少的系统内核调用,socket的accept()和close()调用)。

但是，[keep-alive](http://www.nowamagic.net/academy/tag/keep-alive)并不是免费的午餐,长时间的tcp连接容易导致系统资源无效占用。配置不当的keep-alive，有时比重复利用连接带来的损失还更大。所以，正确地设置keep-alive timeout时间非常重要。

#### keepalvie timeout

Httpd守护进程，一般都提供了keep-alive  timeout时间设置参数。比如nginx的keepalive_timeout，和Apache的KeepAliveTimeout。这个keepalive_timout时间值意味着：一个http产生的tcp连接在传送完最后一个响应后，还需要hold住keepalive_timeout秒后，才开始关闭这个连接。

当httpd守护进程发送完一个响应后，理应马上主动关闭相应的tcp连接，设置  keepalive_timeout后，httpd守护进程会想说：”再等等吧，看看浏览器还有没有请求过来”，这一等，便是keepalive_timeout时间。如果守护进程在这个等待的时间里，一直没有收到浏览发过来http请求，则关闭这个http连接。

#### keep-alive与TIME_WAIT

使用http keep-alvie，可以减少服务端TIME_WAIT数量(因为由服务端httpd守护进程主动关闭连接)。道理很简单，相较而言，启用keep-alive，建立的tcp连接更少了，自然要被关闭的tcp连接也相应更少了。

#### 最后

我想用一张示意图片来说明使用启用keepalive的不同。另外，http keepalive是客户端浏览器与服务端httpd守护进程协作的结果，所以，我们另外安排篇幅介绍不同浏览器的各种情况对keepalive的利用。

![img](http://www.nowamagic.net/librarys/images/201312/2013_12_20_02.png)





HTTP相关状态码

    1×× : 请求处理中，请求已被接受，正在处理
    
    2×× : 请求成功，请求被成功处理
    
        200 OK // 客户端请求成功
    
    3×× : 重定向，要完成请求必须进行进一步处理
    
        301 Moved Permanently // 永久重定向,使用域名跳转
    
        302 Found // 临时重定向,未登陆的用户访问用户中心重定向到登录页面
    
    4×× : 客户端错误，请求不合法
    
        400 Bad Request // 客户端请求有语法错误，不能被服务器所理解
    
        401 Unauthorized // 请求未经授权，这个状态代码必须和 WWW-Authenticate 报头域一起使用
    
        403 Forbidden // 服务器收到请求，但是拒绝提供服务
    
        404 Not Found // 请求资源不存在，eg：输入了错误的 URL
    
    5×× : 服务器端错误，服务器不能处理合法请求
    
        500 Internal Server Error // 服务器发生不可预期的错误
    
        502 Bad Gateway //作为网关或者代理工作的服务器尝试执行请求时，从上游服务器接收到无效的响应。
    
        503 Server Unavailable // 服务器当前不能处理客户端的请求，一段时间后可能恢复正常
    
        504 Gateway Time-out //网关超时，这个有时候Nginx会抛出的异常，主要原因是请求超时，比如你想导出下载某个文件，结果文件太大，就可能请求超时了。

