# `TCP`
##  `TCP 三次握手和四次挥手的流程，为什么断开连接要 4 次,如果握手只有两次，会出现什么。（这个问题很多变种：比如为什么需要3次，2次行不行。）`
<img src="https://s2.ax1x.com/2019/10/15/KPWQB9.png" style="zoom:50%;" />
<img src="https://s2.ax1x.com/2019/10/15/KPW3A1.png" style="zoom:50%;" />
<img src="https://s2.ax1x.com/2019/10/17/KkDrRg.png" style="zoom:70%;" />

## `TIME_WAIT 和 CLOSE_WAIT 的区别。`

## `什么是分块传送。`

## `当你用浏览器打开一个链接的时候，计算机做了哪些工作步骤。`

## `TCP/IP 如何保证可靠性， 说说 TCP 头的结构。`

## `tcp流量控制 和 拥塞控制`

## `tcp有keep-alive机制，为什么应用层还需要做心跳？`
`默认的tcp keep-alive超时时间太长，默认是7200秒，也就是2个小时。`
**socks proxy会让tcp keep-alive失效**
socks协议只管转发TCP层具体的数据包，而不会转发TCP协议内的实现细节的包（也做不到），参考socks_proxy。
所以，一个应用如果使用了socks代理，那么tcp keep-alive机制就失效了，所以应用要自己有心跳包。socks proxy只是一个例子，真实的网络很复杂，可能会有各种原因让tcp keep-alive失效。
**移动网络需要信令保活**
前两年，微信信令事件很火，搜索下“微信 信令”或者“移动网络 信令”可以查到很多相关文章。
> https://blog.csdn.net/znzxc/article/details/82054387

## `http的keep-alive和tcp的keepalive区别`
**tcp的keepalive**
链接建立之后，如果应用程序或者上层协议一直不发送数据，或者隔很长时间才发送一次数据，当链接很久没有数据报文传输时如何去确定对方还在线，到底是掉线了还是确实没有数据传输，链接还需不需要保持，这种情况在TCP协议设计中是需要考虑到的。
TCP协议通过一种巧妙的方式去解决这个问题，当超过一段时间之后，TCP自动发送一个数据为空的报文给对方，如果对方回应了这个报文，说明对方还在线，链接可以继续保持，如果对方没有报文返回，并且重试了多次之后则认为链接丢失，没有必要保持链接。
**http的keep-alive**
主要是为了保持tcp连接的打开，因为默认http是发送一次数据就关闭的，但是在大量请求的情况下，频繁建立连接时不必要且浪费资源的。http的keep-alive就是通道的重复使用。
在HTTP 1.1版本后，默认都开启Keep-Alive模式，只有加入加入` Connection: close`才关闭连接，当然也可以设置Keep-Alive模式的属性，例如` Keep-Alive: timeout=5, max=100`，表示这个TCP通道可以保持5秒，max=100，表示这个长连接最多接收100次请求就断开。

# `HTTP`
## ` 说一下HTTP协议`
http协议基于Tcp/ip的超文本传输协议。Http协议工作与客户端-服务端架构上。

HTTP使用统一资源标识符（Uniform Resource Identifiers, URI）来传输数据和建立连接。URL是一种特殊类型的URI，包含了用于查找某个资源的足够的信息。URI主要有7部分：

1. 协议头部分:`http:`，`//`只是作为分隔符。

2. 域名部分：`www.xxxx.com`
3. 端口部分：跟在域名后面的是端口，域名和端口之间使用`:`作为分隔符。端口不是必须的部分，如果省略端口部分，将采用默认端口。
4. 虚拟目录部分: 从域名后的第一个`/`开始到最后一个`/`为止，是虚拟目录部分。
5. 文件名部分

## `http1.0 和 http1.1 有什么区别。`

## `说说你知道的几种 HTTP 响应码，比如 200, 302, 404。`

## `如何避免浏览器缓存。`

比如js：在js 引用url后面加上一个随机数

```http
<script type=”text/javascript“ src=”/js/test.js?+Math.random()“></script> 
```

或者直接在chrome中设置。

## `如何理解 HTTP 协议的无状态性。`

无状态的官方解释：

1. 协议对于事务处理没有记忆能力
2. 对同一个url请求没有上下文关系
3. 每次的请求都是独立的，它的执行情况和结果与之前的请求和之后的请求是无直接关系的，它不会受前面的请求应答情况直接影响，也不会直接影响后面的请求应答情况
4. 服务器中没有保存客户端的状态，客户端必须每次带上自己的状态去请求服务器。

## `简述 Http 请求 get 和 post 的区别以及数据包格式。`

## `HTTP 有哪些 method `

## `简述 HTTP 请求的报文格式。`
分为请求行、请求头、空行和请求数据4部分组成。

1. 请求行，用来说明请求类型，要访问的资源以及所使用的HTTP版本。
2. 请求头部，紧接着请求行(即第一行)之后的部分，用来说明服务器要使用的附加信息。
3. 空行，`请求头部后面的空行是必须的`。
4. 请求数据也叫主体，可以添加任意的其他数据。

**Get**

```http
GET /562f25980001b1b106000338.jpg HTTP/1.1
Host    img.mukewang.com
User-Agent  Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/51.0.2704.106 Safari/537.36
Accept  image/webp,image/*,*/*;q=0.8
Referer http://www.imooc.com/
Accept-Encoding gzip, deflate, sdch
Accept-Language zh-CN,zh;q=0.8
                             // 此空行必须有
```

GET提交，请求的数据会附在URL之后（就是把数据放置在HTTP协议头中），所以空行后没有请去数据。

Post**

```http
POST / HTTP1.1
Host:www.wrox.com
User-Agent:Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1; SV1; .NET CLR 2.0.50727; .NET CLR 3.0.04506.648; .NET CLR 3.5.21022)
Content-Type:application/x-www-form-urlencoded
Content-Length:40
Connection: Keep-Alive
											// 空行不能省略
name=Professional%20Ajax&publisher=Wiley
```

## `HTTP 的长连接是什么意思。`

## `HTTPS 的加密方式是什么， 讲讲整个加密解密流程。`

## `Http 和 https 的三次握手有什么区别。`

## `Session 和 cookie 的区别`





























