
# `Dubbo是什么？解决了什么问题？`
Dubbo是阿里巴巴开源的基于 Java 的高性能 RPC（一种远程调用） 分布式服务框架（SOA），致力于提供高性能和透明化的RPC远程服务调用方案，以及SOA服务治理方案。具有性能高、已使用和可拓展性强等特点。

**dubbo中的各种角色**     
<img src="https://s2.ax1x.com/2019/10/14/uzy3DO.jpg"  width="500" height="300" />

1. provide： 暴露服务的提供方
2. consumer：调用服务的服务消费放
3. registry： 服务注册与发现的注册中心。
4. monitor：统计服务调用次数和调用时间的监控中心。
5. container：服务运行容器。

**`dubbo支持的协议：`**
1. dubbo://
2. http://
3. rest://
4. redis://
5. memcached://
6. multicast://
7. nacos://

# `Dubbo的底层实现原理和机制` 
![](https://img-blog.csdnimg.cn/2019050810072764.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2VuX2pva2Vy,size_16,color_FFFFFF,t_70)

1. client一个线程调用远程接口，生成一个唯一的ID（比如一段随机字符串，UUID等），Dubbo是使用AtomicLong从0开始累计数字的
2. 将打包的方法调用信息（如调用的接口名称，方法名称，参数值列表等），和处理结果的回调对象callback，全部封装在一起，组成一个对象object
3. 向专门存放调用信息的全局ConcurrentHashMap里面put(ID, object)
    将ID和打包的方法调用信息封装成一对象connRequest，使用IoSession.write(connRequest)异步发送出去
4. 当前线程再使用callback的get()方法试图获取远程返回的结果，在get()内部，则使用synchronized获取回调对象callback的锁， 再先检测是否已经获取到结果，如果没有，然后调用callback的wait()方法，释放callback上的锁，让当前线程处于等待状态。
5. 服务端接收到请求并处理后，将结果（此结果中包含了前面的ID，即回传）发送给客户端，客户端socket连接上专门监听消息的线程收到消息，分析结果，取到ID，再从前面的ConcurrentHashMap里面get(ID)，从而找到callback，将方法调用结果设置到callback对象里。
6. 监听线程接着使用synchronized获取回调对象callback的锁（因为前面调用过wait()，那个线程已释放callback的锁了），再notifyAll()，唤醒前面处于等待状态的线程继续执行（callback的get()方法继续执行就能拿到调用结果了），至此，整个过程结束。


# `Dubbo的SPI机制` 

# `Dubbo有哪些配置项？`
| **标签** | **用途** | **解释** | 
|:----|:----|:----|
| &lt;dubbo:application/>   | 公共 | 用于配置当前应用信息，不管该应用是提供者还是消费者 | 
| &lt;dubbo:registry/> | 公共 | 用于配置连接注册中心相关信息 | 
| &lt;dubbo:protocol/> | 服务 | 用于配置提供服务的协议信息，协议由提供方指定，消费方被动接受 | 
| &lt;dubbo:service/> | 服务 | 用于暴露一个服务，定义服务的元信息，一个服务可以用多个协议暴露，一个服务也可以注册到多个注册中心 | 
| &lt;dubbo:provider/> | 服务 | 当 ProtocolConfig 和 ServiceConfig 某属性没有配置时，采用此缺省值，可选 | 
| &lt;dubbo:consumer/> | 引用 | 当 ReferenceConfig 某属性没有配置时，采用此缺省值，可选 | 
| &lt;dubbo:reference/> | 引用 | 用于创建一个远程服务代理，一个引用可以指向多个注册中心 | 
| &lt;dubbo:method/> | 公共 | 用于 ServiceConfig 和 ReferenceConfig 指定方法级的配置信息 | 
| &lt;dubbo:argument/> | 公共 | 用于指定方法参数配置 | 


# `Dubbo超时时间和重试配置？配置优先级问题`
dubbo官方推荐：`在 Provider 端尽量多配置 Consumer 端属性`
**重要的配置项**
```
 <dubbo:method name="findAllPerson" timeout="10000" retries="9" loadbalance="leastactive" actives="5" />

```
- timeout: 方法调用超时时间
- retries： 失败重试次数，缺省是2。表示加上第一次调用，会调用 3 次
- loadbalance：负载均衡算法，缺省是随机random。可以配置轮询 roundrobin、最不活跃优先 [4] leastactive 和一致性哈希 consistenthash 、RoundRobin LoadBalance（权重轮询均衡算法）。
- 消费者端的最大并发调用限制，即当 Consumer 对一个服务的并发调用到上限后，新调用会阻塞直到超时，在方法上配置 dubbo:method 则针对该方法进行并发限制，在接口上配置 dubbo:service，则针对该服务进行并发限制

# `Dubbo的服务请求失败怎么处理`

# `重试机制会不会造成错误` 
在幂等的情况下不会。

# `Zookeeper的用途，选举的原理是什么？` 
ZooKeeper 的核心是原子广播，这个机制保证了各个 Server 之间的同步。实现这个机制的协议叫做 Zab 协议。Zab 协议有两种模式，它们分别是恢复模式（选主）和广播模式（同步）：
1. 选举线程由当前 Server 发起选举的线程担任，其主要功能是对投票结果进行统计，并选出推荐的 Server 。
2. 选举线程首先向所有 Server 发起一次询问(包括自己)。
3. 选举线程收到回复后，验证是否是自己发起的询问(验证 zxid 是否一致)，然后获取对方的 id(myid)，并存储到当前询问对象列表中，最后获取对方提议的 Leader相关信息(id，zxid)，并将这些信息存储到当次选举的投票记录表中。
4. 收到所有 Server 回复以后，就计算出 zxid 最大的那个 Server ，并将这个 Server 相关信息设置成下一次要投票的 Server 。
5. 线程将当前 zxid 最大的 Server 设置为当前 Server 要推荐的 Leader ，如果此时获胜的 Server 获得 n/2+1 的 Server 票数，设置当前推荐的 Leader 为获胜的 Server ，将根据获胜的 Server 相关信息设置自己的状态，否则，继续这个过程，直到 Leader 被选举出来。



# `数据的垂直拆分水平拆分。` 

# `zookeeper特性`
ZooKeeper 是一个开放源码的分布式协调服务，它是集群的管理者，监视着集群中各个节点的状态根据节点提交的反馈进行下一步合理操作。最终，将简单易用的接口和性能高效、功能稳定的系统提供给用户。     
分布式应用程序可以基于 Zookeeper 实现诸如数据发布/订阅、负载均衡、命名服务、分布式协调/通知、集群管理、Master 选举、分布式锁和分布式队列等功能。
Zookeeper 具有如下特性：
1. 顺序一致性(有序性)
2. 原子性
3. 单一视图
4. 可靠性
5. 实时性 


# `Zookeeper 对于读写请求有所不同`
- 客户端的读请求可以被集群中的任意一台机器处理，如果读请求在节点上注册了监听器，这个监听器也是由所连接的 Zookeeper 机器来处理。
- 对于写请求，这些请求会同时发给其他 Zookeeper 机器并且达成一致后，请求才会返回成功。因此，随着 Zookeeper 的集群机器增多，读请求的吞吐会提高但是写请求的吞吐会下降。


# `zookeeper原理和适用场景 `

# `zookeeper watch机制` 

# `redis/zk节点宕机如何处理 `

# `Zk的同步流程？`
选完 Leader 以后，Zookeeper 就进入状态同步过程。
1. Leader 等待 Server 连接。
2. Follower 连接 Leader ，将最大的 zxid 发送给 Leader 。
3. Leader 根据 Follower 的 zxid 确定同步点。
1. 完成同步后通知 Follower 已经成为 update 状态。
2. Follower 收到 update 消息后，又可以重新接受 Client 的请求进行服务了。


# `讲讲Paxo算法和Zab协议`
todo