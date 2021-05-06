- nginx 达到上限了怎么办？怎么对 nginx 负载均衡？dns？
- nginx 负载均衡有哪些算法？各自有什么优缺点？



nginx 达到上限了怎么办？

1.是否可以增大限制 更具机器情况

# Nginx 限制并发



在Nginx服务器上进行一些常规设置，来限制其并发数及会话空间等。

nginx限制ip并发数，也是说限制同一个ip同时连接服务器的数量

**1，添加limit_conn_zone**

这个变量只能在http使用



```
http{
  ...
  #定义一个名为one的limit_zone,大小10M内存来存储session，
  #以$binary_remote_addr 为key
  #nginx 1.18以后用limit_conn_zone替换了limit_conn
  #且只能放在http作用域
  limit_conn_zone $binary_remote_addr zone=one:10m;
```



**2，添加limit_conn**

这个变量可以在http, server, location使用
只限制一个站点，所以添加到server里面

```
server{
    ...
    location {
      ...
       limit_conn one 20;		  #连接数限制
       #带宽限制,对单个连接限数，如果一个ip两个连接，就是500x2k
       limit_rate 500k;		 
      ...
    }
    ...
  }
```

**3，重启nginx**

# Nginx 限制IP带宽占用

2018-10-07 11:14 更新

nginx 限速模块



在Nginx服务器上进行一些常规设置，来限制其并发数及会话空间等。

nginx限制ip并发数，也是说限制同一个ip同时连接服务器的数量；

通过配合限制并发下的流量限制，可以一定程度上限制单ip带宽占用

**1，添加limit_conn_zone**

这个变量只能在http使用



```
http{
  ...
  #定义一个名为one的limit_zone,大小10M内存来存储session，
  #以$binary_remote_addr 为key
  #nginx 1.18以后用limit_conn_zone替换了limit_conn
  #且只能放在http作用域
  limit_conn_zone $binary_remote_addr zone=one:10m;
```



**2，添加limit_conn，limit_rate**

这两个变量可以在http, server, location使用
只限制一个站点，所以添加到server里面

limit_conn one 2; #限制每个IP只能发起两个并发连接。

limit_rate 300k; #对每个连接限速300k。

注意，这里是对连接限速，而不是对IP限速。
如果一个IP允许两个并发连接，那么这个IP就是限速limit_rate×limit_conn。比如 300k × 2 就是对ip的流量带宽控制

示例：

```
server{
    ...
    location {
      ...
       limit_conn one 2;		  #连接数限制
       #带宽限制,对单个连接限数，如果一个ip两个连接，就是300x2 k
       limit_rate 300k;		 
      ...
    }
    ...
  }
```







2.在不能增加的情况下 现有的设计是否满足大流量

https://www.cnblogs.com/kakatadage/p/9995578.html

### NGINX分布式集群

 如果有多台NGINX想实现负载均衡的话，

1、每台nginx都有公网地址，在域名处设置同个域名多个指向，最简单实现轮洵。但故障切负会慢一点。
2、一台公网nginx通过upstream功能，轮洵、ip、url多方式分发到内网多台nginx。但公网的nginx如果down机的话，内网全段。
3、一对公网nginx加三个公网ip，通过keepalive实现高可用，再upstream到内网(就是我们刚刚上一节讲的主从备份)。

一般来说，上面1、2、3种方法基本可以解决，建议用2或3；

如果并发量真的巨大的话，一般就要借助硬件F5等设备做负载均衡，跟DNS、CDN等服务商合作做域名解析转发、缓存配置，这也是目前大多数大厂的架构配置。







负载均衡：把请求均匀的分摊到多个服务器上处理

DNS负载均衡
DNS负载均衡是通过DNS服务器实现的，主要用于把请求均匀的分布到nginx服务器上，真实情况可能是根据区域区分请求，但是一个地域中请求还是需要均匀的分配到nginx服务器上
实现原理：DNS服务器为同一个主机名配置多个IP地址，在应答DNS查询时，DNS服务器对每个查询将以DNS文件中主机记录的IP地址按顺序返回不同的解析结果，将客户端的访问引导到不同你的机器上，使得不同的客户端访问不同的服务器，从而达到负载均衡目的
缺点：
无法区分服务器是否挂掉，即使某个ngnix服务器挂掉，DNS仍然会分配
DNS缓存，用户访问网站，dns解析出来的ip一般会在客户端进行缓存。下次访问时会直接从缓存中拿，无法达到真正的均匀

Nginx负载均衡
ngnix是目前流行的、优秀的反向代理服务器，其作为反向代理服务器，主要责任是请求均匀的分摊到应用服务器中，为了达到均匀，ngnix有5种负载均衡策略

1.轮询：请求依次轮流往每个应用服务器上进行分配
缺点：不均匀，可能会出现某些服务器接受的请求较重，负载压力大，不可控；服务器之间需要session同步
2.权重轮询：在轮询的基础上给每个服务器一定的权重，权重大的可以多分配几个请求
优点：可控
缺点：仍需要session同步
3.IP-hash
优点：无需进行session同步，固定IP会访问固定访问一台服务器
缺点：恶意攻击，会造成某台服务器压垮；提供的服务不同，面向的地区不同，ip可能会出现集中，造成不均匀
4.fair：会根据服务器处理请求的速度进行负载均衡分配
5.URL-hash：根据URL进行hash






nginx负载均衡的五种算法
一、Nginx负载均衡算法
1、轮询（默认）

每个请求按时间顺序逐一分配到不同的后端服务，如果后端某台服务器死机，自动剔除故障系统，使用户访问不受影响。

例如：

upstream bakend {  
    server 192.168.0.1;    
    server 192.168.0.2;  
}
1
2
3
4
2、weight（轮询权值）

weight的值越大分配到的访问概率越高，主要用于后端每台服务器性能不均衡的情况下。或者仅仅为在主从的情况下设置不同的权值，达到合理有效的地利用主机资源。

指定轮询几率，weight和访问比率成正比，用于后端服务器性能不均的情况。
例如：

upstream bakend {  
    server 192.168.0.1 weight=10;  
    server 192.168.0.2 weight=10;  
}
1
2
3
4
3、ip_hash

每个请求按访问IP的哈希结果分配，使来自同一个IP的访客固定访问一台后端服务器，并且可以有效解决动态网页存在的session共享问题。

每个请求按访问ip的hash结果分配，这样每个访客固定访问一个后端服务器，可以解决session的问题。
例如：

upstream bakend {  
    ip_hash;  
    server 192.168.0.1:88;  
    server 192.168.0.2:80;  
} 
1
2
3
4
5
4、fair（第三方）

比 weight、ip_hash更加智能的负载均衡算法，fair算法可以根据页面大小和加载时间长短智能地进行负载均衡，也就是根据后端服务器的响应时间 来分配请求，响应时间短的优先分配。Nginx本身不支持fair，如果需要这种调度算法，则必须安装upstream_fair模块。

按后端服务器的响应时间来分配请求，响应时间短的优先分配。
例如：

upstream backend {  
    server 192.168.0.1:88;  
    server 192.168.0.2:80;  
    fair;  
}
1
2
3
4
5
5、url_hash（第三方）

按访问的URL的哈希结果来分配请求，使每个URL定向到一台后端服务器，可以进一步提高后端缓存服务器的效率。Nginx本身不支持url_hash，如果需要这种调度算法，则必须安装Nginx的hash软件包。

按访问url的hash结果来分配请求，使每个url定向到同一个后端服务器，后端服务器为缓存时比较有效。

注意：在upstream中加入hash语句，server语句中不能写入weight等其他的参数，hash_method是使用的hash算法。

例如：

upstream backend {  
    server 192.168.0.1:88;  
    server 192.168.0.2:80;  
    hash $request_uri;  
    hash_method crc32;  
}
1
2
3
4
5
6
二、Nginx负载均衡调度状态
在Nginx upstream模块中，可以设定每台后端服务器在负载均衡调度中的状态，常用的状态有：

down，表示当前的server暂时不参与负载均衡
weight 默认为1，weight越大，负载的权重就越大。
backup，预留的备份机器。当其他所有的非backup机器出现故障或者忙的时候，才会请求backup机器，因此这台机器的访问压力最低
max_fails，允许请求失败的次数，默认为1，当超过最大次数时，返回proxy_next_upstream模块定义的错误。
fail_timeout，请求失败超时时间，在经历了max_fails次失败后，暂停服务的时间。max_fails和fail_timeout可以一起使用。
例如：

upstream bakend{ 
      ip_hash; 
      server 192.168.0.1:90 down; 
      server 192.168.0.1:80 weight=2; 
      server 192.168.0.2:90; 
      server 192.168.0.2:80 backup; 
}
