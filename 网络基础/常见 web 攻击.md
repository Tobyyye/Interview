整理常见的Web攻击手段：

- XSS攻击
- CSRF攻击
- SQL注入攻击
- 文件上传漏洞
- DDoS攻击
- 其他攻击手



## XSS攻击

XSS（Cross Site Scripting）跨站脚本攻击，为了不与层叠样式表（CSS）混淆，故将跨站脚本攻击缩写为XSS。原理即在网页中嵌入恶意脚本，当用户打开网页时，恶意脚本便开始在用户浏览器上执行，以盗取客户端cookie、用户名、密码，甚至下载木马程式，危害可想而知。

以一个表单输入举例说明

```xml
<input type="text" name="firstname" value="">
1
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190303142658772.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xvb25nc2hhd24=,size_16,color_FFFFFF,t_70)
倘若用户在表单中输入恶意脚本，即对输入做些处理，如：

```xml
<input type="text" name="firstname" value=""/><script>alert("longlong")</script><!-- "/>
```

## 如何预防XSS

答案很简单，坚决不要相信用户的任何输入，并过滤掉输入中的所有特殊字符。这样就能消灭绝大部分的XSS攻击。

目前防御XSS主要有如下几种方式：

1. 过滤特殊字符
   避免XSS的方法之一主要是将用户所提供的内容进行过滤(如上面的`script`标签)。
2. 使用HTTP头指定类型
   `w.Header().Set("Content-Type","text/javascript")`
   这样就可以让浏览器解析javascript代码，而不会是html输出。



# SQL注入

## 什么是SQL注入

攻击者成功的向服务器提交恶意的SQL查询代码，程序在接收后错误的将攻击者的输入作为查询语句的一部分执行，导致原始的查询逻辑被改变，额外的执行了攻击者精心构造的恶意代码。

举例：`' OR '1'='1`

这是最常见的 SQL注入攻击，当我们输如用户名 admin ，然后密码输如`' OR '1'=1='1`的时候，我们在查询用户名和密码是否正确的时候，本来要执行的是`SELECT * FROM user WHERE username='' and password=''`,经过参数拼接后，会执行 SQL语句 `SELECT * FROM user WHERE username='' and password='' OR '1'='1'`，这个时候1=1是成立，自然就跳过验证了。
如下图所示：

![img](https://images.morethink.cn/69855b1538333659f26afc281feb4e30.png)

但是如果再严重一点，密码输如的是`';DROP TABLE user;--`，那么 SQL命令为`SELECT * FROM user WHERE username='admin' and password='';drop table user;--'` 这个时候我们就直接把这个表给删除了。

## 如何预防SQL注入

- 在Java中，我们可以使用预编译语句(PreparedStatement)，这样的话即使我们使用 SQL语句伪造成参数，到了服务端的时候，这个伪造 SQL语句的参数也只是简单的字符，并不能起到攻击的作用。
- 对进入数据库的特殊字符（`'"\尖括号&*`;等）进行转义处理，或编码转换。
- 在应用发布之前建议使用专业的SQL注入检测工具进行检测，以及时修补被发现的SQL注入漏洞。网上有很多这方面的开源工具，例如sqlmap、SQLninja等。
- 避免网站打印出SQL错误信息，比如类型错误、字段不匹配等，把代码里的SQL语句暴露出来，以防止攻击者利用这些错误信息进行SQL注入。

在上图展示中，使用了Java JDBC中的`PreparedStatement`预编译预防SQL注入，可以看到将所有输入都作为了字符串，避免执行恶意SQL。





# DDOS

## 什么是DDOS

DDOS：分布式拒绝服务攻击（Distributed Denial of Service），简单说就是发送大量请求是使服务器瘫痪。DDos攻击是在DOS攻击基础上的，可以通俗理解，dos是单挑，而ddos是群殴，因为现代技术的发展，dos攻击的杀伤力降低，所以出现了DDOS，攻击者借助公共网络，将大数量的计算机设备联合起来，向一个或多个目标进行攻击。

在技术角度上，DDoS攻击可以针对网络通讯协议的各层，手段大致有：TCP类的SYN Flood、ACK Flood，UDP类的Fraggle、Trinoo，DNS Query Flood，ICMP Flood，Slowloris类等等。一般会根据攻击目标的情况，针对性的把技术手法混合，以达到最低的成本最难防御的目的，并且可以进行合理的节奏控制，以及隐藏保护攻击资源。

下面介绍一下TCP协议中的SYN攻击。

## SYN攻击

在三次握手过程中，服务器发送 `SYN-ACK` 之后，收到客户端的 `ACK` 之前的 TCP 连接称为半连接(half-open connect)。此时服务器处于 `SYN_RCVD` 状态。当收到 ACK 后，服务器才能转入 `ESTABLISHED` 状态.

`SYN`攻击指的是，攻击客户端在短时间内伪造大量不存在的IP地址，向服务器不断地发送`SYN`包，服务器回复确认包，并等待客户的确认。由于源地址是不存在的，服务器需要不断的重发直至超时，这些伪造的`SYN`包将长时间占用未连接队列，正常的`SYN`请求被丢弃，导致目标系统运行缓慢，严重者会引起网络堵塞甚至系统瘫痪。

## 如何预防DDOS

阿里巴巴的安全团队在实战中发现，DDoS 防御产品的核心是检测技术和清洗技术。检测技术就是检测网站是否正在遭受 DDoS 攻击，而清洗技术就是清洗掉异常流量。而检测技术的核心在于对业务深刻的理解，才能快速精确判断出是否真的发生了 DDoS 攻击。清洗技术对检测来讲，不同的业务场景下要求的粒度不一样。





# CSRF

## 什么是CSRF

CSRF（Cross-site request forgery），中文名称：跨站请求伪造，也被称为：one click attack/session riding，缩写为：CSRF/XSRF。

你这可以这么理解CSRF攻击：攻击者盗用了你的身份，以你的名义发送恶意请求。CSRF能够做的事情包括：以你名义发送邮件，发消息，盗取你的账号，甚至于购买商品，虚拟货币转账......造成的问题包括：个人隐私泄露以及财产安全。

## CSRF的原理

下图简单阐述了CSRF攻击的思
![img](https://images.morethink.cn/138ad4f05b47533bf46904dc165167cc.png)

从上图可以看出，要完成一次CSRF攻击，受害者必须依次完成两个步骤：

1. 登录受信任网站A，并在本地生成Cookie。
2. 在不登出A的情况下，访问危险网站B。

看到这里，你也许会说：“如果我不满足以上两个条件中的一个，我就不会受到CSRF的攻击”。是的，确实如此，但你不能保证以下情况不会发生：

1. 你不能保证你登录了一个网站后，不再打开一个tab页面并访问另外的网站。
2. 你不能保证你关闭浏览器了后，你本地的Cookie立刻过期，你上次的会话已经结束。（事实上，关闭浏览器不能结束一个会话，但大多数人都会错误的认为关闭浏览器就等于退出登录/结束会话了......）
3. 上图中所谓的攻击网站，可能是一个存在其他漏洞的可信任的经常被人访问的网站。

下面讲一讲java解决CSRF攻击的方式。



**CSRF攻击防御：**

1、将cookie设置为HttpOnly

CSRF攻击很大程度是利用了浏览器的cookie，为了防止站内XSS漏洞，cookie设置HttpOnly属性，JS脚本就无法读取到cookie中的信息，避免攻击者伪造cookie的情况出现。

设置方式参考：https://www.cnblogs.com/relucent/p/4171478.html

2、增加token

CSRF攻击之所以成功，主要是攻击中伪造了用户请求，而用户请求的验证信息都在cookie中，攻击者就可以利用cookie伪造请求通过安全验证。因此抵御CSRF攻击的关键就是，在请求中放入攻击者不能伪造的信息，并且信息不在cookie中。

鉴于此，开发人员可以在http请求中以参数的形式加一个token，此token在服务端生成，也在服务端校验，服务端的每次会话都可以用同一个token。如果验证token不一致，则认为至CSRF攻击，拒绝请求。

表单中增加一个隐藏域：

```xml
<input type="hidden" name="_token" value="tokenvalue"/>
1
```

在服务端session中添加token：

```java
HttpSession session = request.getSession();
Object token = session.getAttribute("_token");
if(token == null || "".equals(_token){
	session.setAttribute("_token",UUID.randomUUID().toString());
}
12345
```

3、通过Referer识别

Http头中有一个字段Referer，它记录了Http请求来源地址。但是注意不要把Rerferer用在身份验证或者其他非常重要的检查上，因为Rerferer非常容易在客户端被改变。

（火狐的一个插件RefControl修改Referer引用）