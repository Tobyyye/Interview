三次握手（three times handshake；three-way handshake）所谓的“三次握手”即对每次发送的数据量是怎样跟踪进行协商使数据段的发送和接收同步，根据所接收到的数据量而确定的数据确认数及数据发送、接收完毕后何时撤消联系，并建立虚连接。

为了提供可靠的传送，TCP在发送新的数据之前，以特定的顺序将数据包的序号，并需要这些包传送给目标机之后的确认消息。TCP总是用来发送大批量的数据。当应用程序在收到数据后要做出确认时也要用到TCP。

![img](https:////upload-images.jianshu.io/upload_images/14326004-88338b1326863318?imageMogr2/auto-orient/strip|imageView2/2/w/769/format/webp)

image

第一次握手：建立连接时，客户端发送syn包（syn=j）到服务器，并进入SYN_SENT状态，等待服务器确认；SYN：同步序列编号（Synchronize Sequence Numbers）。

第二次握手：服务器收到syn包，必须确认客户的SYN（ack=j+1），同时自己也发送一个SYN包（syn=k），即SYN+ACK包，此时服务器进入SYN_RECV状态；

第三次握手：客户端收到服务器的SYN+ACK包，向服务器发送确认包ACK(ack=k+1），此包发送完毕，客户端和服务器进入ESTABLISHED（TCP连接成功）状态，完成三次握手。





https://baijiahao.baidu.com/s?id=1654225744653405133&wfr=spider&for=pc



TCP connection

![img](https://pics1.baidu.com/feed/4afbfbedab64034f641e67dffe1d813408551d7f.jpeg?token=341c4a88025d5441f6efeadd184f0968&s=0A007B220D6A452093CDFDC8030070B1)

![img](https://pics4.baidu.com/feed/5ab5c9ea15ce36d35970c9af6c2dc282e850b119.png?token=7bee1e859605f019d0de2a3912fc6882&s=D1B0147246C8FF2429DA4D53020010FA)

客户端与服务器之间数据的发送和返回的过程当中需要创建一个叫TCP connection的东西；

由于TCP不存在连接的概念，只存在请求和响应，请求和响应都是数据包，它们之间都是经过由TCP创建的一个从客户端发起，服务器接收的类似连接的通道，这个连接可以一直保持，http请求是在这个连接的基础上发送的；

在一个TCP连接上是可以发送多个http请求的，不同的版本这个模式不一样。

在HTTP/1.0中这个TCP连接是在http请求创建的时候同步创建的，http请求发送到服务器端，服务器端响应了之后，这个TCP连接就关闭了；

HTTP/1.1中可以以某种方式声明这个连接一直保持，一个请求传输完之后，另一个请求可以接着传输。这样的好处是：在创建一个TCP连接的过程中需要“三次握手”的消耗，“三次握手”代表有三次网络传输。

如果TCP连接保持，第二个请求发送就没有这“三次握手”的消耗。HTTP/2中同一个TCP连接里还可以并发地传输http请求。

TCP报文格式简介

![img](https://pics3.baidu.com/feed/64380cd7912397ddb480a4110c5c4ab2d1a28709.jpeg?token=45d9d76830f1e8052a5f5394378e459a&s=048A5F31C60E77495EC7C14D0300C0E2)

其中比较重要的字段有：

（1）序号（sequence number）：Seq序号，占32位，用来标识从TCP源端向目的端发送的字节流，发起方发送数据时对此进行标记。

（2）确认号（acknowledgement number）：Ack序号，占32位，只有ACK标志位为1时，确认序号字段才有效，Ack=Seq+1。

（3）标志位（Flags）：共6个，即URG、ACK、PSH、RST、SYN、FIN等。具体含义如下：

URG：紧急指针（urgent pointer）有效。ACK：确认序号有效。PSH：接收方应该尽快将这个报文交给应用层。RST：重置连接。SYN：发起一个新连接。FIN：释放一个连接。

需要注意的是：

不要将确认序号Ack与标志位中的ACK搞混了。确认方Ack=发起方Seq+1，两端配对。

TCP的三次握手（Three-Way Handshake）

1.”三次握手”的详解

所谓的三次握手即TCP连接的建立。这个连接必须是一方主动打开，另一方被动打开的。以下为客户端主动发起连接的图解：

![img](https://pics1.baidu.com/feed/d8f9d72a6059252d20d93b0a6645fb3e59b5b9d2.jpeg?token=c86d4509157378798ebbccbe843486d1&s=9746F8123F5754CA48D574DA0300D0B2)

握手之前主动打开连接的客户端结束CLOSED阶段，被动打开的服务器端也结束CLOSED阶段，并进入LISTEN阶段。随后开始“三次握手”：

（1）首先客户端向服务器端发送一段TCP报文，其中：

标记位为SYN，表示“请求建立新连接”;序号为Seq=X（X一般为1）；随后客户端进入SYN-SENT阶段。（2）服务器端接收到来自客户端的TCP报文之后，结束LISTEN阶段。并返回一段TCP报文，其中：

标志位为SYN和ACK，表示“确认客户端的报文Seq序号有效，服务器能正常接收客户端发送的数据，并同意创建新连接”（即告诉客户端，服务器收到了你的数据）；序号为Seq=y；确认号为Ack=x+1，表示收到客户端的序号Seq并将其值加1作为自己确认号Ack的值；随后服务器端进入SYN-RCVD阶段。（3）客户端接收到来自服务器端的确认收到数据的TCP报文之后，明确了从客户端到服务器的数据传输是正常的，结束SYN-SENT阶段。并返回最后一段TCP报文。其中：

标志位为ACK，表示“确认收到服务器端同意连接的信号”（即告诉服务器，我知道你收到我发的数据了）；序号为Seq=x+1，表示收到服务器端的确认号Ack，并将其值作为自己的序号值；确认号为Ack=y+1，表示收到服务器端序号Seq，并将其值加1作为自己的确认号Ack的值；随后客户端进入ESTABLISHED阶段。服务器收到来自客户端的“确认收到服务器数据”的TCP报文之后，明确了从服务器到客户端的数据传输是正常的。结束SYN-SENT阶段，进入ESTABLISHED阶段。

在客户端与服务器端传输的TCP报文中，双方的确认号Ack和序号Seq的值，都是在彼此Ack和Seq值的基础上进行计算的，这样做保证了TCP报文传输的连贯性。一旦出现某一方发出的TCP报文丢失，便无法继续"握手"，以此确保了"三次握手"的顺利完成。

此后客户端和服务器端进行正常的数据传输。这就是“三次握手”的过程。

