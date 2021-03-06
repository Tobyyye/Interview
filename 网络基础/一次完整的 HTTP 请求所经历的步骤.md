一次完整的 HTTP 请求所经历的步骤

    这里的客户端，更多指的是浏览器。
    
    1、DNS 解析(通过访问的域名找出其 IP 地址，递归搜索)。
    
        如用客户端浏览器请求这个页面:http://localhost.com:8080/index.htm 从中分解出协议名、主机名、 端口、对象路径等部分，对于我们的这个地址，解析得到的结果如下:
    
        协议名:http
    
        主机名:localhost.com
    
        端口:8080
    
        对象路径:/index.htm
    
        在这一步，需要域名系统 DNS 解析域名 localhost.com,得主机的 IP 地址。
    
    2、HTTP 请求，当输入一个请求时，建立一个 Socket 连接发起 TCP的 3 次握手。
    
        把以上部分结合本机自己的信息，封装成一个 HTTP 请求数据包。
    
        如果是 HTTPS 请求，会略微有不同。
    
    3、客户端发送请求
    
        3.1 客户端向服务器发送请求命令（一般是 GET 或 POST 请求）。
    
        客户端的网络层不用关心应用层或者传输层的东西，主要做的是通过查找路由表确定如何到达服务器，期间可能经过多个路由器，这些都是由路由器来完成的工作，无非就是通过查找路由表决定通过那个路径到达服务器。
    
        客户端的链路层，包通过链路层发送到路由器，通过邻居协议查找给定 IP 地址的 MAC 地址，然后发送 ARP 请求查找目的地址，如果得到回应后就可以使用 ARP 的请求应答交换的 IP 数据包现在就可以传输了，然后发送 IP 数据包到达服务器的地址。
    
        客户机发送请求命令:建立连接后，客户机发送一个请求给服务器，请求方式的格式为:统一资源标识符(URL)、协议版本号，后边是 MIME 信息包括请求修饰符、客户机信息和可内容。
    
        3.2客户端发送请求头信息和数据。
    
    4.1、服务器发送应答头信息。
    
        服务器接到请求后，给予相应的响应信息，其格式为一个状态行，包括信息的协议版本号、一个成功或错误的代码，后边是 MIME 信息包括服务器信息、实体信息和可能的内容。
    
    4.2、服务器向客户端发送数据。
    
    5、服务器关闭 TCP 连接（4次挥手）。
    
        这里是否关闭 TCP 连接，也根据 HTTP Keep-Alive 机制有关。同时，客户端也可以主动发起关闭 TCP 连接。
    
        如果浏览器或者服务器在其头信息加入了这行代码 Connection:keep-alive，TCP 连接在发送后将仍然保持打开状态，于是，浏览器可以继续通过相同的连接发送请求。保持连接节省了为每个请求建立新连接所需的时间，还节约了网络带宽。
    
    6、客户端根据返回的 HTML、CSS、JS 进行渲染。

如下是《图解HTTP》提供的图片：

 ![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9yYXcuZ2l0aHVidXNlcmNvbnRlbnQuY29tL0pEYXduRi9sZWFybmluZ19ub3RlL21hc3Rlci9pbWFnZXMvYmU1NjAzYjFiYjZmZjQ2YjY0MDkwOWI1Yjg5NzY4MjcuanBlZw?x-oss-process=image/format,png) 