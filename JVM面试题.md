# `#1. JVM`
## ## `JVM由几部分组成\ JVM内存结构`
> JVM = 类加载器(classloader) + 执行引擎(execution engine) + 运行时数据区域(runtime data area)

**内存区域**
![avatar](https://s2.ax1x.com/2019/10/12/uLmlfe.jpg)

## ## `OOM错误，stackoverflow错误，permgen space错误`
## ## `讲讲什么情况下回出现内存溢出，内存泄漏？` 
**虚拟机栈**
- 如果线程请求的栈深度大于虚拟机所允许的深度，会抛出StackOverflowError异常。
- 当虚拟机动态拓展时无法申请到足够的内存，会抛出OutOfMemoryError异常。（大部分虚拟机可以动态拓展，只不过虚拟机规范中允许固定长度的虚拟机栈）    

**堆**
如果堆无法完成实例分配，同时堆也无法拓展时，将会抛出OutOfMemoryError异常

**元数据区**
但是如果动态创建太多类的话，还是造成该区域的内存溢出。

## ## `说说Java线程栈，帧栈理由存放了那些内容？ `
栈是线程私有，生命周期与线程相同。   
虚拟机栈描述的是Java方法执行的内存模型：每个方法在执行的时候都会创建一个帧栈（Stack Frame）用于存储局部变量表、操作数栈、动态链接、方法出口等信息。
每一个方法从调用到执行完成的过程，就对应着一个帧栈在虚拟机栈中入栈到出栈的信息。  
局部变量表存放了编译器可知的各种基本数据类型（boolean、byte、short、char、int、long、float、double），对象引用（reference类型，不等于对象本身，可能是指向对象的地址指针等）,在编译期完成内存分配。


## ## `java对象头`
![avatar][base64str]
![avatar](https://s2.ax1x.com/2019/10/12/uLnbvj.jpg)
> https://s2.ax1x.com/2019/10/11/uqmkCQ.jpg

## ##  `堆内存设置的参数是什么？`
-Xmx 设置堆的最大空间大小
-Xms 设置堆的最小空间大小

## ## `Perm Space中保存什么数据？会引起OutOfMemory吗？`
加载class文件。
会引起，出现异常可以设置 -XX:PermSize 的大小。JDK 1.8后，字符串常量不存放在永久带，而是在堆内存中，JDK8以后没有永久代概念，而是用元空间替代，元空间不存在虚拟机中，二是使用本地内存。
详细查看Java8内存模型—永久代(PermGen)和元空间(Metaspace)

## ## `你有没有生产环境遇到过OutOfMemory问题？你是怎么来处理这个问题的？处理 过程中有哪些收获？`
permgen space、heap space 错误。
常见的原因
内存加载的数据量太大：一次性从数据库取太多数据；
集合类中有对对象的引用，使用后未清空，GC不能进行回收；
代码中存在循环产生过多的重复对象；
启动参数堆内存值小。
详见 Java 内存溢出（java.lang.OutOfMemoryError）的常见情况和处理方式总结。

##  ## `JDK 1.8之后Perm Space有哪些变动? MetaSpace⼤⼩默认是⽆限的么? 还是你们会通过什么⽅式来指定⼤⼩?`
JDK 1.8后用元空间替代了 Perm Space；字符串常量存放到堆内存中。
MetaSpace大小默认没有限制，一般根据系统内存的大小。JVM会动态改变此值。
- -XX:MetaspaceSize：分配给类元数据空间（以字节计）的初始大小（Oracle逻辑存储上的初始高水位，the initial high-water-mark）。此值为估计值，MetaspaceSize的值设置的过大会延长垃圾回收时间。垃圾回收过后，引起下一次垃圾回收的类元数据空间的大小可能会变大。
- -XX:MaxMetaspaceSize：分配给类元数据空间的最大值，超过此值就会触发Full GC，此值默认没有限制，但应取决于系统内存的大小。JVM会动态地改变此值。

## ## `StackOverflow异常有没有遇到过？⼀般你猜测会在什么情况下被触发？如何指定⼀个线程的堆栈⼤⼩？⼀般你们写多少？`
栈内存溢出，一般由栈内存的局部变量过爆了，导致内存溢出。出现在递归方法，参数个数过多，递归过深，递归没有出口。或者调用链太深。

# `#2. GC`
## ## `如何判断一个对象已死？`
一般有两种算法来判断一个对象是否死亡
- 引用分析算法：此算法简单易懂，每当有一个地方引用，引用计数就加1，失去引用就减1。引用计数为0时就代表这个对象是垃圾对象了。但是缺点是不太好处理循环引用。
- 可达性分析算法：通过一些被称为`GC ROOT`的节点出发进行向下搜索，最终形成一条引用链，而没在引用链上的对象就是垃圾对象。

在java中标记一个对象真正是已经执行了`finalize()方法(失去引用的对象可以借助此方法复活))`的。   
`能作为GC ROOT的有`
- 虚拟机栈（帧栈的本地变量表）中引用的对象。
- 元数据区中类静态属性引用的对象。
- 元数据区常量引用的对象。
- 本地方法栈（Native方法）引用的对象。

## ## `各种回收算法及其特点`
**`标记-清除算法`**   
分为标记和清除两个阶段。首先标记所有需要回收的对象，在标记完成后统一回收所有被标记的对象。
主要不足：
- 效率问题：标记和清除两个过程效率都不高。
- 标记清除后会产生大量不连续的内存碎片。碎片多会导致程序运行分配大对象时，无法找到足够的连续内存导致分配失败，而且会导致提前触发另一次GC。    

**`复制算法`**
将内存分为2块大小相等的区域，每次只使用一块，当一块满了，就把活着的对象移动到另一块，移动完成清空原来那块就行了。   
优点：实现简单、效率高。  
缺点：代价太高，每次只使用一般内存。     
主要在新生代使用。  
**`标记-整理（标记-压缩）算法`**
标记过程和标记-清除一样,但后续步骤不是直接对可回收对象进行清理，而是让所有存活对象都向一端移动，然后直接清理掉端边界以外的内存。    
此算法主要为老年代使用。
**`分代收集算法`**
现在主流虚拟机都采用分代算法。按对象生命周期分为老年代和新生代，老年代使用`标记-整理`算法，新生代使用`复制`算法。    
**`分区算法`**
todo

## ## `HotSpot为什么要分为新生代和老年代？`
参见👆的`分代收集算法`。

## ## `常见的垃圾回收器有那些？`
![avatar](https://s2.ax1x.com/2019/10/12/uLu3qI.jpg)    
**Serial （串行）收集器**
**ParNew收集器**
**Parallel Scavenge 收集器**
**Serial Old 收集器**
**Parallel Old 收集器**
**CMS 收集器**
**G1 收集器**



## ## `JVM 年轻代到年老代的晋升过程的判断条件是什么呢？`
虚拟机给每个对象定义一个对象年龄计数器。如果对象在Eden出生并经过第一次Minor GC后仍然存活，并且能被Survivor容纳的话，将被移动到Survivor空间中，并且对象年龄设为1。对象在Survivor区中每“熬过”一次Minor GC，年龄就增加1岁，当它的年龄增加到一定程度（默认为15岁），就将会被晋升到老年代中。对象晋升老年代的年龄阈值，可以通过参数-XX:MaxTenuringThreshold设置。            
还有一种方式是动态对象年龄判定。为了适应不同程序的内存状况，虚拟机并不是永远地要求对象的年龄必须达到看MaxTenuringThreshold才能晋升老年代，如果Survivor空间中相同年龄所有对象大小的总和大于Survivor空间的一半，年龄大于或等于该年龄的对象就可以直接进入老年代，无须等到MaxTenuringThreshold中要求的年龄。

## ## `JVM 出现 fullGC 很频繁，怎么去线上排查问题？` 
一般fullGC基本都是以下问题
1. 老年代空间不足。
2. 永生代或者元数据空间不足。
3. System.gc()方法调用。
4. CMS GC时出现promotion failed和concurrent mode failure
5. YoungGC时晋升老年代的内存平均值大于老年代剩余空间
6. 有连续的大对象需要分配

然后根据👆的情况尽情具体排查。如果是老年代空间不足可以通过图形化界面或者命令查看大概情况，必要在老年代快要满时，通过jmap进行dump内存(注意dump内存时是无法对外正常提供服务的，需要注意)，然后使用`MAT`等可视化工具分析。找出其中的大量对象是撒，然后寻找是那产生的，进行分析原因。    
`连续的大对象需要分配`也是一样的步骤。    
资料：
> 记一次因为短命大对象导致fullGC的问题 https://my.oschina.net/u/2315110/blog/3026538
> 记一次异常FullGC的问题排查 https://www.jianshu.com/p/6831d5448065
> 线上FullGC频繁的排查 https://blog.51cto.com/3270430/2141151?source=dra

## ## `JVM垃圾回收机制，何时触发MinorGC等操作` 
Minor GC是新生代GC。当Eden区域满的时候回进行一次Minor GC，
比如s0有对象数据，s1没有。当Eden满的时候，把Eden+s0还存活的数据复制到s1，并清空s0。


## ## `JVM 中一次完整的 GC 流程（从 ygc 到 fgc）是怎样的 `
FullGC：这个和垃圾收集有关。Serial Old 收集器、Parallel Old 收集器是在老年满的时候进行垃圾收集，会发生STW(采用了标记整理算法)。    
**CMS**
而CMS是一款最求最短停顿事件的收集器，他不是在满的时候收集，当老年代到达一个阈值的时候就开始收集了。在默认设置下，CMS收集器在老年代使用了68%的空间时就会被激活(可配置)。CMS采用的标记清除算法，存在内存碎片的问题，当fullGC后也无法进行对象分配的话，会触发另一次FullGC来标记整理。当然CMS可以配置在多少次FullGC后加一个碎片整理过程。

## ## `CMS GC过程`
![avatar](https://s2.ax1x.com/2019/10/12/uLKOA0.jpg)    
初始标记仅仅只是标记出GC ROOTS能直接关联到的对象，速度很快，并发标记阶段是进行GC ROOTS 根搜索算法阶段，会判定对象是否存活。而重新标记阶段则是为了修正并发标记期间，因用户程序继续运行而导致标记产生变动的那一部分对象的标记记录



## ## `G1 GC过程`

## ## `各种回收器，各自优缺点，重点CMS、G1 `
**`CMS`**    
CMS收集器的优点：并发收集、低停顿，但是CMS还远远达不到完美，主要有三个显著缺点： 
- CMS收集器对CPU资源非常敏感。在并发阶段，虽然不会导致用户线程停顿，但是会占用CPU资源而导致引用程序变慢，总吞吐量下降。CMS默认启动的回收线程数是：(CPU数量+3) / 4。 
- 浮动垃圾： 因为清理时，用户线程还是在继续运行的，会继续产生垃圾，但是本次清理也无法清理这种垃圾。虽然CMS是到达一定阈值就进行过GC，但是在极端情况下，还没清理完。老年代被占满的话，无法继续分配内存的话会造成JVM临时启用Serial Old收集器来重新进行老年代的垃圾收集。这样造成长时间STW。
- 内存碎片：上面个问题已经说了。    

**`G1`**

## ## `你知道哪些或者你们线上使⽤什么GC策略？它有什么优势，适⽤于什么场景？`
G1

## ## `做GC时，⼀个对象在内存各个Space中被移动的顺序是什么？`
标记清除法，复制算法，标记整理、分代算法。
新生代一般采用复制算法 GC，老年代使用标记整理算法。
垃圾收集器：串行新生代收集器、串行老生代收集器、并行新生代收集器、并行老年代收集器。
CMS（Current Mark Sweep）收集器是一种以获取最短回收停顿时间为目标的收集器，它是一种并发收集器，采用的是Mark-Sweep算法。
详见 Java GC机制。

## ## `Minor Gc和Full GC 有什么不同呢？`
Minor Gc只是发生在年轻代
FullGC发生在老年代满的情况，会发生STW。在CMS、G1等新型垃圾收集器中已经不存在full GC的概念了(极端情况还会存在)

# `#3. JVM工具`
## ## `JVM线上调优？`
https://mp.weixin.qq.com/s?__biz=MzIwMzY1OTU1NQ==&mid=2247486951&idx=1&sn=e0bdd989691142cf9e175fd7130fbab1&chksm=96cd4daba1bac4bdc52e80f0af1cc6f5d655b3f03da953c4f14b27ee219aac7564a8ae582eaa&scene=27#wechat_redirect

## ## `线上CPU飙升至100%，什么排查问题。`
此问题在并发编程题中有，不重复解答。

## ## `jstack 是⼲什么的? jstat 呢？如果线上程序周期性地出现卡顿，你怀疑可 能是 GC 导致的，你会怎么来排查这个问题？线程⽇志⼀般你会看其中的什么 部分？`
jstack 用来查询 Java 进程的堆栈信息。
jvisualvm 监控内存泄露，跟踪垃圾回收、执行时内存、cpu分析、线程分析。
详见Java jvisualvm简要说明，可参考 线上FullGC频繁的排查。

## ## `虚拟机性能监控和故障处理工具`
- jmap：它可以生成 java 程序的 dump 文件， 也可以查看堆内对象示例的统计信息、查看 ClassLoader 的信息以及 finalizer 队列。
- jstack： dump java栈的。
- jstat：jstat是一个用来监控虚拟机资源和性能的命令行工具
- jinfo：可以用来查看正在运行的java运用程序的扩展参数，甚至支持在运行时动态地更改部分参数。可以查看垃圾回收到信息。
-  jps：列出所有java 进程的PID。

# `#4. 类加载、Class字节码相关`
## ## `对类加载器有了解吗？`
虚拟机类加载是把类的Class文件加载到虚拟内存，类加载主要为3大步骤：加载，链接，初始化。很多语言是在编译时链接的，而Java是在程序运行期间完成的，这样Java启动时可能会慢一些，但是这样提供了高度的灵活性，可以在运行期间动态加载和动态链接。

## ## `什么是双亲委派模型？类加载为什么要使用双亲委派模式，有没有什么场景是打破了这个模式？` 
SPI机制打破了这个模式，比如加载数据库驱动。
JDK提供了一些Sql Driver接口,是由BootstrapClassLoader去加载，但是第三的具体实现是由AppClassLoader来加载的。问题主要出现在`DriverManager`去获取接口实现时，由于它也是
BootstrapClassLoader加载的，也就无法”看见“第三方的具体实现。这时候JDK给出的解决方法是使用上下文加载器来进行加载(也就是调用者的类加载器))))。
还有 Class.forName("")也是，它也是BootstrapClassLoader去加载，也就是做它只能看见由BootstrapClassLoader加载的类，所以也是通过上下文类加载器来解决。

> SPI是去每个包的从META-INF/services/xxxxx  文件得到实现此类的自定义实现。
> SPI机制的解决方式是：用Thread.currentThread().getContextClassLoader()来加载实现类，实现在核心包里的基础类调用用户代码。(上下文类加载器)

## ## `类的实例化顺序 `
1.首先是父类的静态变量和静态代码块（看两者的书写顺序）；
2.第二执行子类的静态变量和静态代码块（看两者的书写顺序）；
3.第三执行父类的成员变量赋值
4.第四执行父类类的构造代码块
5.第五执行父类的构造方法（）
6.执行子类的构造代码块
7.第七执行子类的构造方法（）；
总结，也就是说虽然客户端代码是new 的构造方法，但是构造方法确实是在整个实例创建中的最后一个调用。切记切记！！！
**先是父类，再是子类； 
先是类静态变量和静态代码块，再是对象的成员变量和构造代码块–》构造方法。**
记住，构造方法最后调用！！！！成员变量优先构造代码块优先构造方法！！

## ## `简单说说类加载过程，里面执行了哪些操作？`
加载、连接(验证，准备，解析)、初始化、使用、卸载。

## ## `Java类加载器包括⼏种？它们之间的⽗⼦关系是怎么样的？双亲委派机制是什么意思？有什么好处？`
启动Bootstrap类加载、扩展Extension类加载、系统System类加载。
父子关系如下：
启动类加载器 ，由C++ 实现，没有父类；
扩展类加载器，由Java语言实现，父类加载器为null；
系统类加载器，由Java语言实现，父类加载器为扩展类加载器；
自定义类加载器，父类加载器肯定为AppClassLoader。
双亲委派机制：类加载器收到类加载请求，自己不加载，向上委托给父类加载，父类加载不了，再自己加载。
优势避免Java核心API篡改。详细查看：深入理解Java类加载器(ClassLoader)

## ## `如何⾃定义⼀个类加载器？你使⽤过哪些或者你在什么场景下需要⼀个⾃定义的类加载器吗？`
自定义类加载的意义：
加载特定路径的class文件
加载一个加密的网络class文件
热部署加载class文件

## ## `简单介绍一下Class类文件结构（常量池主要存放的是那两大常量？Class文件的继承关系是如何确定的？字段表、方法表、属性表主要包含那些信息？）`
1. 魔数与Class文件版本
2. 常量池
3. 访问标志
4. 类索引、父类索引、接口索引
5. 字段表集合
6. 方法表集合
7. 属性表集合


# # `5. 其他`
## ## `对象的访问定位的两种方式。`   
- 句柄访问：Java堆中将会划分出一块内存来作为句柄池，reference中存储的就是对象的句柄地址，而句柄中包含了对象实例数据与类型数据各自的具体地址信息
![avatar](https://s2.ax1x.com/2019/10/12/uLgF4P.jpg)
- 直接指针访问：Java堆对象的布局中就必须考虑如何放置访问类型数据的相关信息，而reference中存储的直接就是对象地址。
![avatar](https://s2.ax1x.com/2019/10/12/uLg3CV.jpg)

## ## `Java分配内存有几种方式？`
1. 指针碰撞：假设Java堆中的内存是绝对规整的，吧内存分为两块，左边块为使用的内存，右边为空闲的内存，中间是一个指针为分界点的指示器，那分配内存就是把指针向空闲空间挪动等同于对象大小的距离。
2. 空闲列表：如果堆中的内存不是规整的，已使用的内存和空闲的内存相互交错，这时就无法使用指针碰撞，虚拟机必须维护一个列表，记录那些内存块是可用的，在分配的时候从列表中找到一块足够大的空间划分给对象实例，并更新列表上的记录。

## ## `简单的介绍一下强引用、软引用、弱引用、虚引用（虚引用与软引用和弱引用的区别、使用软引用能带来的好处）。`
- 强引用(Strong Reference)：就是指在程序代码之中普遍存在的，类似“Object obj = new Object()”这类的引用，只要强引用还存在，垃圾收集器永远不会回收掉被引用的对象。
- 软引用(Soft Reference)：用来描述一些还有用但并非必须的对象。在系统将要发生内存溢出异常之前，将会把这些对象列进回收范围之中进行第二次回收。
- 弱引用(Weak Reference)：用户描述非必须对象的。被弱引用关联的对象只能生存到下一次垃圾收集发生之前。当垃圾收集器工作时，无论当前内存是否足够，都会回收掉只被弱引用关联的对象。
- 虚引用(Phantom Reference)：一个对象是否有虚引用存在，完全不会对其生存时间构成影响，也无法通过虚引用来取得一个对象实例。为一个对象设置虚引用的唯一目的就是能在这个对象被收集器回收时刻得到一个系统通知。

## ## `jvm根引用中那些可以被称作根`
>JVM对那些没有根引用的对象进行来及回收，也就是无法从根对象中追述的对象。
>JVM垃圾回收的根对象的范围有以下几种：
>1. 栈中引用的对象，引用是在栈帧中的本地变量表中的，真正的对象在堆中
>2. 方法区perm中的类静态属性引用的对象，以及常量引用的对象
>3. 本地方法栈中JNI（Native方法）的引用的对象
>https://blog.csdn.net/yaozhifeng123456/article/details/48375115

## ## ` jvm 内存设成32g以上和32g以下有啥区别 - 鹅厂面试题`
JVM 在内存小于 32 GB 的时候会采用一个内存对象指针压缩技术。       
在 Java 中，所有的对象都分配在堆上，并通过一个指针进行引用。 普通对象指针（OOP）指向这些对象，通常为 CPU 字长 的大小：32 位或 64 位，取决于你的处理器。指针引用的就是这个 OOP 值的字节位置。   
对于 32 位的系统，意味着堆内存大小最大为 4 GB。对于 64 位的系统， 可以使用更大的内存，但是 64 位的指针意味着更大的浪费，因为你的指针本身大了。更糟糕的是， 更大的指针在主内存和各级缓存（例如 LLC，L1 等）之间移动数据的时候，会占用更多的带宽。    
Java 使用一个叫作 内存指针压缩（compressed oops）的技术来解决这个问题。 它的指针不再表示对象在内存中的精确位置，而是表示 偏移量 。这意味着 32 位的指针可以引用 40 亿个 对象 ， 而不是 40 亿个字节。最终， 也就是说堆内存增长到 32 GB 的物理内存，也可以用 32 位的指针表示。   
一旦你越过那个神奇的 ~32 GB 的边界，指针就会切回普通对象的指针。 每个对象的指针都变长了，就会使用更多的 CPU 内存带宽，也就是说你实际上失去了更多的内存。事实上，当内存到达 40–50 GB 的时候，有效内存才相当于使用内存对象指针压缩技术时候的 32 GB 内存。   
这段描述的意思就是说：即便你有足够的内存，也尽量不要 超过 32 GB。因为它浪费了内存，降低了 CPU 的性能，还要让 GC 应对大内存。   
> https://www.cnblogs.com/wanhua-wu/p/9305372.html
> https://blog.csdn.net/zjerryj/article/details/77206928












# `Base 64图片`
[base64str]:data:image/jpeg;base64,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
