# HTTP 缓存机制详解

HTTP Cache
什么是 HTTP Cache

    我们知道通过网络获取资源缓慢且耗时，需要三次握手等协议与远程服务器建立通信，对于大点的数据需要多次往返通信大大增加了时间开销，并且当今流量依旧没有理想的快速与便宜。对于开发者来说，长久缓存复用重复不变的资源是性能优化的重要组成部分。
    HTTP 缓存机制就是，配置服务器响应头来告诉浏览器是否应该缓存资源、是否强制校验缓存、缓存多长时间；浏览器非首次请求根据响应头是否应该取缓存、缓存过期发送请求头验证缓存是否可用还是重新获取资源的过程。下面我们就来结合简单的 node 服务器代码(文末)来介绍其中原理。

关键字
响应头	(常用)值	说明
Cache-Control	no-cache, no-store, must-revalidate, max-age, public, private	控制浏览器是否可以缓存资源、强制缓存校验、缓存时间
ETag	文件指纹（hash码、时间戳等可以标识文件是否更新）	强校验，根据文件内容生成精确
Last-Modified	请求的资源最近更新时间	弱校验， 根据文件修改时间，可能内容未变，不精确
Expires	资源缓存过期时间	与响应头中的 Date 对比
请求头	值	说明
If-None-Match	缓存响应头中的 ETag 值	发送给服务器比对文件是否更新（精确）
If-Modified-Since	缓存响应头中的 Last-Modified 值	发送给服务器比对文件是否更新（不精确）
简单流程图


 ![这里写图片描述](https://img-blog.csdn.net/20180818175055500?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2d1ZHV5aWJlaXpp/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70) 

304状态码

| 304（未修改） | 自从上次请求后，请求的网页未修改过。服务器返回此响应时，不会返回网页内容。 如果网页自请求者上次请求后再也没有更改过，您应将服务器配置为返回此响应（称为 If-Modified-Since HTTP 标头）。服务器可以告诉 Googlebot 自从上次抓取后网页没有变更，进而节省带宽和开销。 |
| ------------- | ------------------------------------------------------------ |
|               |                                                              |





header 中涉及到缓存的字段有哪些



浏览网页打开F12打开调试器，可以查看请求的详细情况（有些字段是没有的）。
强缓存：Expires，Cache-control

强缓存定义: 在缓存未失效时候，浏览器向服务端发起请求，直接从缓存中获取数据，
Expires 是http1.0的东西。
Cache-control 是http1.1的东西。cache-control:max-age=取代了expires。不过向下兼容expires。
expires

Expires的值为web服务器返回的到期时间(GMT 格林威治时间),浏览器下次请求时间小于服务器返回的时间则浏览器直接从缓存中获取数据，而不用再次发送请求。
cache-control

cache-control常见的取值：private, public, nocache, max-age, nostore
private : 客户端可以缓存
public : 客户端和代理服务器都可缓存
max-age= ： 缓存存储的最大周期，超过这个时间缓存被认为过期（单位秒）
no-cache : 并不是不缓存，是会被缓存的，只不过每次在向浏览器提供相应数据时，浏览器每次都要向服务器发送请求，有服务器来决策评估缓存的有效性。
no-store : 所有内容都不缓存。
对比缓存（协商缓存，弱缓存）: Last-Modified，Etag
Last-Modified

或者if-Modified-Since
服务器响应浏览器请求，告诉浏览器资源的最后修改时间
Etag

ETag是实体标签（Entity Tag）的缩写， 根据实体内容生成的一段hash字符串（类似于MD5或者SHA1之后的结果），可以标识资源的状态。可以理解为一个资源的唯一标识符，只要文件发生变化Etag的值也变化。
Last-Modified 和Etag区别：

Last-Modified 和Etag都是标识文件是否修改的。而Etag比Last-Modified更加严谨。

    一些文件也许会周期性的更改，但是他的内容并不改变(仅仅改变的修改时间)，这个时候我们并不希望客户端认为这个文件被修改了
    某些文件修改非常频繁，比如在秒以下的时间内进行修改，(比方说1s内修改了N次)，If-Modified-Since能检查到的粒度是s级的，这种修改无法判断
    如需要对动态生成的内容做缓存，那就可以用etag来控制缓存了
    注意：如果同时有Last-Modified和Etag存在，在发送请求时，浏览器会一次性的将这两个值都发给服务器，没有优先级，服务器是都比较，还是只比较一个，不同的web服务器可能比较逻辑不一样吧。

对比缓存定义：服务器对比判断文件是否修改，告诉浏览器是否可以使用本地缓存。对比生效时，服务器返回给浏览器的http code 为304，服务器只返回http header信息，并无响应正文。浏览器收到304的返回，知道本地缓存并无修改，直接使用本地缓存。
