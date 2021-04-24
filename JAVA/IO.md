https://www.cnblogs.com/lanqingzhou/p/13609317.html

### 1. Java中有几种类型的流？

1. 字符流和字节流。
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190418185002235.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3ODc1NTg1,size_16,color_FFFFFF,t_70)
2. 字节流继承inputStream和OutputStream
3. 字符流继承自InputSteamReader和OutputStreamWriter
4. 总体结构图
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190418184716728.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3ODc1NTg1,size_16,color_FFFFFF,t_70)

### 2.字节流和字符流哪个好？怎么选择？

1. 大多数情况下使用字节流会更好，因为大多数时候 IO 操作都是直接操作磁盘文件，所以这些流在传输时都是以字节的方式进行的（图片等都是按字节存储的）
2. 如果对于操作需要通过 IO 在内存中频繁处理字符串的情况使用字符流会好些，因为字符流具备缓冲区，提高了性能

### 3. 什么是缓冲区？有什么作用？

1. 缓冲区就是一段特殊的内存区域，很多情况下当程序需要频繁地操作一个资源（如文件或数据库）则性能会很低，所以为了提升性能就可以将一部分数据暂时读写到缓存区，以后直接从此区域中读写数据即可，这样就显著提升了性。
2. 对于 Java 字符流的操作都是在缓冲区操作的，所以如果我们想在字符流操作中主动将缓冲区刷新到文件则可以使用 flush() 方法操作。

### 4. 字符流和字节流有什么区别？

字符流和字节流的使用非常相似，但是实际上字节流的操作不会经过缓冲区（内存）而是直接操作文本本身的，而字符流的操作会先经过缓冲区（内存）然后通过缓冲区再操作文件

### 5. 什么是Java序列化，如何实现Java序列化？

1. 序列化就是一种用来处理对象流的机制，将对象的内容进行流化。可以对流化后的对象进行读写操作，可以将流化后的对象传输于网络之间。序列化是为了解决在对象流读写操作时所引发的问题
2. 序列化的实现：将需要被序列化的类实现Serialize接口，没有需要实现的方法，此接口只是为了标注对象可被序列化的，然后使用一个输出流（如：FileOutputStream）来构造一个ObjectOutputStream(对象流)对象，再使用ObjectOutputStream对象的write(Object obj)方法就可以将参数obj的对象写出

### 6. PrintStream、BufferedWriter、PrintWriter的比较?

1. PrintStream类的输出功能非常强大，通常如果需要输出文本内容，都应该将输出流包装成PrintStream后进行输出。它还提供其他两项功能。与其他输出流不同，PrintStream 永远不会抛出 IOException；而是，异常情况仅设置可通过 checkError 方法测试的内部标志。另外，为了自动刷新，可以创建一个 PrintStream
2. BufferedWriter:将文本写入字符输出流，缓冲各个字符从而提供单个字符，数组和字符串的高效写入。通过write()方法可以将获取到的字符输出，然后通过newLine()进行换行操作。BufferedWriter中的字符流必须通过调用flush方法才能将其刷出去。并且BufferedWriter只能对字符流进行操作。如果要对字节流操作，则使用BufferedInputStream
3. PrintWriter的println方法自动添加换行，不会抛异常，若关心异常，需要调用checkError方法看是否有异常发生，PrintWriter构造方法可指定参数，实现自动刷新缓存（autoflush）

### 7. BufferedReader属于哪种流,它主要是用来做什么的,它里面有那些经典的方法？

属于处理流中的缓冲流，可以将读取的内容存在内存里面，有readLine()方法，它，用来读取一行

### 8. 什么是节点流,什么是处理流,它们各有什么用处,处理流的创建有什么特征？

1. 节点流 直接与数据源相连，用于输入或者输出
2. 处理流：在节点流的基础上对之进行加工，进行一些功能的扩展
3. 处理流的构造器必须要 传入节点流的子类

### 9.流一般需要不需要关闭,如果关闭的话在用什么方法,一般要在那个代码块里面关闭比较好，处理流是怎么关闭的，如果有多个流互相调用传入是怎么关闭的？

1. 流一旦打开就必须关闭，使用close方法
2. 放入finally语句块中（finally 语句一定会执行）
3. 调用的处理流就关闭处理流
4. 多个流互相调用只关闭最外层的流

### 10. InputStream里的read()返回的是什么,read(byte[] data)是什么意思,返回的是什么值？

1. 返回的是所读取的字节的int型（范围0-255）
2. read（byte [ ] data）将读取的字节储存在这个数组。返回的就是传入数组参数个数

### 11. OutputStream里面的write()是什么意思,write(byte b[], int off, int len)这个方法里面的三个参数分别是什么意思？

1. write将指定字节传入数据源
2. Byte b[ ]是byte数组
3. b[off]是传入的第一个字符、b[off+len-1]是传入的最后的一个字符 、len是实际长度






![这里写图片描述](https://img-blog.csdn.net/20180127210359151)

![这里写图片描述](https://img-blog.csdn.net/20180127210410630)







（1）java中有几种类型的流？
字符流和字节流。字节流继承inputStream和OutputStream,字符流继承自InputSteamReader和OutputStreamWriter。

（2）谈谈Java IO里面的常见类，字节流，字符流、接口、实现类、方法阻塞
答：输入流就是从外部文件输入到内存，输出流主要是从内存输出到文件。
IO里面常见的类，第一印象就只知道IO流中有很多类，IO流主要分为字符流和字节流。字符流中有抽象类InputStream和OutputStream，它们的子类FileInputStream，FileOutputStream,BufferedOutputStream等。字符流BufferedReader和Writer等。都实现了Closeable, Flushable, Appendable这些接口。程序中的输入输出都是以流的形式保存的，流中保存的实际上全都是字节文件。
java中的阻塞式方法是指在程序调用改方法时，必须等待输入数据可用或者检测到输入结束或者抛出异常，否则程序会一直停留在该语句上，不会执行下面的语句。比如read()和readLine()方法。

（3）字符流和字节流有什么区别？
要把一片二进制数据数据逐一输出到某个设备中，或者从某个设备中逐一读取一片二进制数据，不管输入输出设备是什么，我们要用统一的方式来完成这些操作，用一种抽象的方式进行描述，这个抽象描述方式起名为IO流，对应的抽象类为OutputStream和InputStream ，不同的实现类就代表不同的输入和输出设备，它们都是针对字节进行操作的。

在应用中，经常要完全是字符的一段文本输出去或读进来，用字节流可以吗？
计算机中的一切最终都是二进制的字节形式存在。对于“中国”这些字符，首先要得到其对应的字节，然后将字节写入到输出流。读取时，首先读到的是字节，可是我们要把它显示为字符，我们需要将字节转换成字符。由于这样的需求很广泛，人家专门提供了字符流的包装类。

底层设备永远只接受字节数据，有时候要写字符串到底层设备，需要将字符串转成字节再进行写入。字符流是字节流的包装，字符流则是直接接受字符串，它内部将串转成字节，再写入底层设备，这为我们向IO设别写入或读取字符串提供了一点点方便。
4）讲讲NIO
答：看了一些文章，传统的IO流是阻塞式的，会一直监听一个ServerSocket，在调用read等方法时，他会一直等到数据到来或者缓冲区已满时才返回。调用accept也是一直阻塞到有客户端连接才会返回。每个客户端连接过来后，服务端都会启动一个线程去处理该客户端的请求。并且多线程处理多个连接。每个线程拥有自己的栈空间并且占用一些 CPU 时间。每个线程遇到外部未准备好的时候，都会阻塞掉。阻塞的结果就是会带来大量的进程上下文切换。
对于NIO，它是非阻塞式，核心类：
1.Buffer为所有的原始类型提供 (Buffer)缓存支持。
2.Charset字符集编码解码解决方案
3.Channel一个新的原始 I/O抽象，用于读写Buffer类型，通道可以认为是一种连接，可以是到特定设备，程序或者是网络的连接。

### （5）递归读取文件夹的文件





Netty的使用场景

https://blog.csdn.net/trecn001/article/details/105382814

