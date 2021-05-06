cookie session 介绍一下 html 页面，怎么与后端交互？流程是什么？涉及到哪些组件？

Cookies 和 Session 的区别

    Session 在服务器端，Cookie 在客户端（浏览器）。
    
        Session 默认被存在在服务器的一个文件里（不是内存）。
    
    Session 的运行依赖 sessionid ，而 sessionid 是存在 Cookie 中的，也就是说，如果浏览器禁用了 Cookie ，同时 session 也会失效。但是，可以通过其它方式实现，比如在 url 参数中传递 sessionid 。
    
    Session 可以放在文件、数据库、或内存中都可以。
    
    【关键】用户验证这种场合一般会用 Session 。

参照：https://blog.csdn.net/striveb/article/details/82469253#Session%E4%B8%8ECookie%E5%8C%BA%E5%88%AB



 cookie的工作流程：
客户端访问服务器，服务器调用response.addCookie()方法，产生响应时，会产生set-cookie响应头，将cookie文本发送给客户端，客户端会将cookie文本保存起来，当客户端再次请求服务器时，会产生cookie请求头，将之前服务器发送的cookie信息，再发送给服务器，服务器就可以根据cookie信息跟踪客户端的状态。 



https://www.cnblogs.com/little-ab/articles/7123836.html



 session的工作流程：
客户端访问服务器，服务器调用request.getSession()方法，产生session对象，用于跟踪用户的
状态，同时，给session对象分配一个唯一标识sessionId。为了管理session对象，以sessionId为键，以session对象为值，封装成Map集合。产生响应时，将sessionId以cookie方式发送给客户端，存放在客户端浏览器的缓存中。当客户端再次请求服务器，会将sessionId以cookie请求头的方式发送给服务器，服务器得到sessionId后，从Map集合中，得到session对象，从而跟踪状态。 





https://www.cnblogs.com/shiy/p/6628613.html



在java servelt里的一些组件

https://www.cnblogs.com/zblwyj/p/10869575.html



#  [     java中的Cookie和Session         ](https://www.cnblogs.com/zblwyj/p/10869575.html) 

在讲解cookie和session之前，先理解下"状态管理"的定义：

　　1、什么是状态管理？

> ```
> 将浏览器与web服务器之间多次交互当做一个整体来看待（即为了完成某个业务，需要多次交互，比如购物），并且将多次交互所涉及的数据（即状态）保存下来。
> ```

　　2、如何进行状态管理

> 　客户端：利用cookie技术进行管理``
>
>   服务端：利用session技术进行管理

推荐一篇文章，讲解了HTML5新特性的localStorage和sessionStorage：https://www.cnblogs.com/st-leslie/p/5617130.html

下面开始进入正题，简单介绍下java中的cookie和Session：

## 一、Cookie

### 　1、什么是cookie？

> 服务器临时存放在浏览器的少量数据

### 　2、cookie的工作原理？

> 浏览器访问服务器时，服务器将少量数据以set-cookie消息头的方式发送给浏览器；浏览器会将这些数据临时保存下来，当浏览器再次访问服务器时，会将这些数据以cookie消息头的方式发送给服务器。

###  3、如何创建一个cookie？

>  Cookie c = new Cookie(String name,String value); name为cookie的名称，value为cookie的值
>
>  response.addCookie(c); 需要将生成的cookie添加到HttpServletResponse的对象中才能起作用

###  4、如何获取Cookie？

> Cookie[] arr = request.getCookies(); 当不存在cookie的时候此方法会返回null值
>
> arr[i].getName();  获取cookie的名称
>
> arr[i].getValue();　 获取cookie的值

###  5、cookie的生存时间？

>  默认情况下，浏览器会将cookie保存在内存里面，浏览器不关闭，cookie一直在），可以调用 cookie.setMaxAge(int seconds)方法来设置cookie的生存时间。
>
> ​    注意：
>
> ​      a.单位是秒
>
> ​      b.当 seconds > 0,浏览器会将cookie保存在硬盘上，
>
> ​      超过指定的时间，浏览器会销毁该cookie。
>
> ​      c.当 seconds < 0,保存在内存里面（缺省值）。
>
> ​      d.当 seconds = 0,浏览器会立即删除该cookie。
>
> ​        比如，要删除一个名称为"cart"的cookie。
>
> ​        Cookie c = new Cookie("cart","");
>
> ​        c.setMaxAge(0);
>
> 　　　　 response.addCookie(c);

###  6、cookie的编码问题？

>  cookie只能保存合法的ascii字符。中文显示不属于ascii字符，需要将中文进行编码处理（也就是说，将中文转换成相应的ascii字符串的形式）。
>
>  　String URLEncoder.encode(String str,String charset)
>
>  　String URLDecoder.decode(String str,String charset)
>
>   注：保存cookie时，尽量都编码处理。

###  7、cookie的路径问题？

> a.什么是cookie的路径问题?
>
> ​    浏览器访问服务器上的某个地址时，会比较该地址是否与cookie的路径匹配，只有匹配的cookie才会被发送。
>
> ​    匹配规则：要访问的地址（路径）必须等于cookie的路径或者是其子路径。
>
>  b.默认路径
>
> ​    默认路径等于添加该cookie的组件的路径。
>
> ​    比如： /servlet-day07-2/app01/addCookie.jsp添加了一个cookie,则该cookie默认的路径是"/servlet-day07-2/app01"
>
>   c.如何修改cookie的路径?
>
> ​    cookie.setPath(String path);

###  8、cookie的缺点？

>  a. cookie是可以被用户禁止的。
>
>  b. cookie只能保存少量的数据。（大约是4k左右）
>
>  c. 浏览器通常只允许保存几百个cookie。
>
>  d. cookie不安全。（如果需要将敏感数据，比如帐号密码以cookie的方式保存在浏览器端，一定需要加密）。

## 二、Session（会话）

### 　1、什么是session？

> 服务段为保存状态而创建的一个特殊对象

### 　2、session的工作原理？

> 浏览器访问服务器时，服务器会创建一个session对象（该对象有一个唯一的id,一般称之为sessionId）,服务器会将这个sessionId以cookie的形式发送给浏览器，浏览器会保存下来。当浏览器再次访问服务器时，会将sessionId以cookie的形式发发送给服务器，服务器会依据sessionId找到对应的session对象。

###  3、如何创建一个session对象？

> 方式一：HttpSession s = request.getSession(boolean flag)
>
>  　flag==true: 先查看请求当中是否有sessionId,如果没有，则创建session对象。如果有sessionId,则依据sessionId去查找对应的session对象，找到了，则返回；如果找不到，则创建一个新的session对象。
>
> 　　 flag==false：先查看请求当中是否有sessionId,如果没有，会返回null。如果有sessionId,则依据sessionId去查找对应的session对象，找到了，则返回；如果找不到，返回null。
>
> 方式二：HttpSession s = request.getSession(); 和flag==true一致

###  4、session如何绑定数据？

> 绑订数据：setAttribute(String name,Object obj)
>
> 依据绑订名获得绑订值：Object getAttribute(String name)
>
> 解除绑订：removeAttribute(String name)
>
> 删除session:session.invalidate() 

###  5、session超时？

> 1)什么是session超时?
>
> 　服务器会将空闲时间过长的session对象删除掉。
>
> 注：服务器默认的空闲时间一般是半个小时（可以修改服务器的配置）。
>
>   <session-config>
>
> ​    <session-timeout>30</session-timeout>
>
>   </session-config>
>
> 2) setMaxInactiveInterval(int seconds)

###  6、Session的使用场景？

> Session一般使用在用户登录时，保存用户的登录相关信息，以及项目中进行Session验证：
>
>  step1. 在登录成功以后，在session对象上绑订一些数据。比如： session.setAttribute("user",user);
>
> step2. 对于需要登录之后才能访问的地址，进行session验证：
>
> ​    Object obj = session.getAttribute("user");
>
> ​    if(obj == null){
>
> ​      response.sendRedirect("login.jsp");
>
> ​    }  

 给大家推荐个好点的博客链接，里面讲述的更完整：https://www.cnblogs.com/shiy/p/6628613.html







