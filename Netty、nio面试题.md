# `BIO、NIO和AIO`
**IO模型主要分类**    
 同步(synchronous) IO   
 异步(asynchronous) IO阻塞(blocking) IO   
 非阻塞(non-blocking)IO同步阻塞(blocking-IO)简称BIO  
 同步非阻塞(non-blocking-IO)简称NIO    
 异步非阻塞(synchronous-non-blocking-IO)简称AIO  

 **Bio**    
 同步阻塞IO，程序的读取和写入必须阻塞在一个线程中知道IO操作完成。
 **NIO**    
 NIO：同步非阻塞IO，但是这说法其实也不对，因为NIO在linux底层是使用的epoll，epoll其实也是同步阻塞IO，只是他采用事件模型，通过上层程序逻辑来规避了阻塞。
 一般会有通过Selector监听各个连接(Channel)的事件，NIO中定义了4个事件：    

 1. OP_ACCEPT: 接收就绪，
 2. serviceSocketChannel使用的OP_READ: 读取就绪，
 3. socketChannel使用OP_WRITE: 写入就绪，
 4. socketChannel使用OP_CONNECT: 连接就绪，socketChannel使用

在使用时，我们只需要监听我们感兴趣的事件就行，不用去等待IO就绪了，大大提高了吞吐量。当然我们一般不直接使用NIO的API，因为比较繁琐，一般使用Netty等框架。   
NIo组件:缓冲区Buffer、通道Channel、多路复用器Selector
**AIO**   
完全的异步非阻塞IO，实际开发比较少用，不过是未来的趋势，和NIO的区别是，不需要同Selector去监听，有系统来进行回调。

# `Netty 的各大组件 `

- Channel

  Netty网络抽象类，包含了基本的I/O操作。子类有SocketChannel和ServerSocketChannel。

- ChannelFuture

  Netty为异步非阻塞的，所有IO操作都为异步的，因此我们不能知道消息是否被成功处理了，Netty提供了ChannelFuture几口，通过该接口addListener()来监听。

- EventLoop

  Event主要为Channel处理I/O操作。一个EventLoopGroup包含一个或多个EventLoop。一个EventLoop可被分配至一个或多个Channel。

- ChannelHandler

  可以让我们来处理各种事件的，比如连接、数据接收、异常等。是我们主要的编码。

- ChannelPipeline

  一个管道，管道上可以提供多个ChannelHandler来进行链式处理。

> [Netty的核心组件](https://blog.csdn.net/chenssy/article/details/78703551)

# `Netty的线程模型` 
更加优雅的 Reactor 模式实现、灵活的线程模型、利用 EventLoop 等创新性的机制，可以非常高效地管理成百上千的 Channel 。Reactor模式也有三种模型：单线程、多线程、主从多线程模型。

**单线程模型**

用一个线程来处理所有的事情，包括客户端的连接以及所有连接产生的读写时间。优点是一个线程的利用率非常高，同时也简单。缺点是：对于多核心CPU比较浪费资源。一个线程同时处理多个连接，管理比较复杂，同时连接数很多的情况下，性能无法得到保证，同时也使得单核心负载过重。

**多线程模型**

有一组NIO线程来处理IO操作。接受连接和处理请求作为两部分分离了。Accpter使用单独的线程来接受请求，网络IO操作等由一个NIO线程池负责。这些线程池负责消息的读取、解码、编码和发送。

**主从多线程模型**

多线程模型其实已经很好的解决大部分情况，但是Accptor存在瓶颈，比如大量的认证请求(Https)。 工作流程主要为：从主线程池随机找一个Reactor线程作为Accptor线程，用了接受客户端连接。Accptor线程接收到客户端连接请求后创建SocketChannel，将其注册到主线程池的其他Reator线程上，有器负责接入认证、IP过滤、握手等操作。之后连接正式建立后，会从主线程的Reactor线程多路复用器上摘除，重新注册到SUb线程池的线程上，用于处理I/O的读写操作。

**Netty线程模型**

Netty的线程模型并不是一成不变的，他取决于用户的代码和参数配置。Netty支持3种。一般常用的是服务端监听线程和IO线程分离(两个EventGroup)。

![](https://static001.infoq.cn/resource/image/e5/63/e5da0c60482a1b0f48374ad571822a63.png)

> [Netty 系列之 Netty 线程模型](https://www.infoq.cn/article/netty-threading-model)
>
> [Netty线程模型及EventLoop详解](https://www.jianshu.com/p/128ddc36e713)


# `了解哪几种序列化协议？包括使用场景和如何去选择` 

# `Netty的零拷贝实现` 

**传统意义的拷贝**
是在发送数据的时候，传统的实现方式是：

```
    File.read(bytes)
    Socket.send(bytes)
```
这种方式需要四次数据拷贝和四次上下文切换：
1. 数据从磁盘读取到内核的read buffer
2. 数据从内核缓冲区拷贝到用户缓冲区
3. 数据从用户缓冲区拷贝到内核的socket buffer
4. 数据从内核的socket buffer拷贝到网卡接口（硬件）的缓冲区
**零拷贝**
明显上面的第二步和第三步是没有必要的，通过java的FileChannel.transferTo方法，可以避免上面两次多余的拷贝（当然这需要底层操作系统支持）
1. 调用transferTo,数据从文件由DMA引擎拷贝到内核read buffer
2. 接着DMA从内核read buffer将数据拷贝到网卡接口buffer
上面的两次操作都不需要CPU参与，所以就达到了零拷贝。

**netty中的零拷贝***

1、bytebuffer

Netty发送和接收消息主要使用bytebuffer，bytebuffer使用对外内存（DirectMemory）直接进行Socket读写。

原因：如果使用传统的堆内存进行Socket读写，JVM会将堆内存buffer拷贝一份到直接内存中然后再写入socket，多了一次缓冲区的内存拷贝。DirectMemory中可以直接通过DMA发送到网卡接口

2、Composite Buffers

传统的ByteBuffer，如果需要将两个ByteBuffer中的数据组合到一起，我们需要首先创建一个size=size1+size2大小的新的数组，然后将两个数组中的数据拷贝到新的数组中。但是使用Netty提供的组合ByteBuf，就可以避免这样的操作，因为CompositeByteBuf并没有真正将多个Buffer组合起来，而是保存了它们的引用，从而避免了数据的拷贝，实现了零拷贝。

3、对于FileChannel.transferTo的使用

Netty中使用了FileChannel的transferTo方法，该方法依赖于操作系统实现零拷贝。

**`答案2：`**    
Netty 的零拷贝实现，是体现在多方面的，主要如下：     
【重点】Netty 的接收和发送 ByteBuffer 采用堆外直接内存 Direct Buffer 。
- 使用堆外直接内存进行 Socket 读写，不需要进行字节缓冲区的二次拷贝；使用堆内内存会多了一次内存拷贝，JVM 会将堆内存 Buffer 拷贝一份到直接内存中，然后才写入 Socket 中。
-  Netty 创建的 ByteBuffer 类型，由 ChannelConfig 配置。而 ChannelConfig 配置的 ByteBufAllocator 默认创建 Direct Buffer 类型。

CompositeByteBuf 类，可以将多个 ByteBuf 合并为一个逻辑上的 ByteBuf ，避免了传统通过内存拷贝的方式将几个小 Buffer 合并成一个大的 Buffer 。
- #addComponents(...) 方法，可将 header 与 body 合并为一个逻辑上的 ByteBuf 。这两个 ByteBuf 在CompositeByteBuf 内部都是单独存在的，即 CompositeByteBuf 只是逻辑上是一个整体。

通过 FileRegion 包装的 FileChannel 。
- #tranferTo(...) 方法，实现文件传输, 可以直接将文件缓冲区的数据发送到目标 Channel ，避免了传统通过循环 write 方式，导致的内存拷贝问题。
    通过 wrap 方法, 我们可以将 byte[] 数组、ByteBuf、ByteBuffer 等包装成一个 Netty ByteBuf 对象, 进而避免了拷贝操作。



# `Netty的高性能表现在哪些方面`



# `TCP 粘包/拆包的原因及解决方法` 

# `Netty 黏包/拆包`

TCP作为传输层只是负责传输数据，具体业务含义。 比如客户端发送了4个包，但是在通讯过程中，这4个包合成了一个包发送，所以我们更具通信协议会定义什么才算一个完整的包(比如分隔符)。在netty中会把tcp收到的数据读取到缓冲区中，然后netty定义了一些拆包器，比如固定长度的拆包器 FixedLengthFrameDecoder、行拆包器 LineBasedFrameDecoder等，根据您这些拆包器，netty会自动帮我们读取到一个完整的包，而不用我们去手动写代码去做(当然你也可以根据需求自己拓展)。

