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

# `Netty的线程模型` 

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

# `Netty的高性能表现在哪些方面`



# `TCP 粘包/拆包的原因及解决方法` 

# `Netty 黏包/拆包`

TCP作为传输层只是负责传输数据，具体业务含义。 比如客户端发送了4个包，但是在通讯过程中，这4个包合成了一个包发送，所以我们更具通信协议会定义什么才算一个完整的包(比如分隔符)。在netty中会把tcp收到的数据读取到缓冲区中，然后netty定义了一些拆包器，比如固定长度的拆包器 FixedLengthFrameDecoder、行拆包器 LineBasedFrameDecoder等，根据您这些拆包器，netty会自动帮我们读取到一个完整的包，而不用我们去手动写代码去做(当然你也可以根据需求自己拓展)。

