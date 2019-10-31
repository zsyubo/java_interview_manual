> Ps: 至于什么是面向对象、三大特征 我相信每个人都有了自己的理解了，这里就不作为面试题了。

# `List 和 Set 的区别` 
List 是可重复集合，Set 是不可重复集合，这两个接口都实现了 Collection 父接口。     
Map 未继承 Collection，而是独立的接口，Map 是一种把键对象和值对象进行映射的集合，它的每一个元素都包含了一对键对象和值对象，Map 中存储的数据是没有顺序的， 其 key 是不能重复的，它的值是可以有重复的。

# `HashSet 是如何保证不重复的` 
Hash底层其实就是一个HashMap来承装元素，同时HashSet重写了equals和hashCode方法老保证set对象的唯一性，hashset：元素作为key，Object最为value。

# `HashMap 是线程安全的吗，为什么不是线程安全的（最好画图说明多线程环境下不安全）? `
不管哪个版本，都不建议在多线程的情况下使用HashMap。
**JDK1.7**
不是非线程安全，在多线程情况下可能会发生死循环问题。主要resize（扩容）时。`造成死循环的主要原因就是JDK1.7 在插入节点时采用了头插法。`    
`假设：`Entry数组大小为2。负载因子设置为1.2(也就是size为3开始扩容)。这时，index0出有了两个元素，a->b。这时插入C，开始扩容。`同时假设a、b、c、d，不然怎么扩容，hash后的index都是0。`
1. 首先线程1、2同时插入一个数据，并同时开始执行扩容。
2. 首先线程1获得了时间片，线程2卡在了：`Entry<K,V> next = e.next;`
3. 线程1执行了扩容代码。新Entry数组中index0: newEntry[0]=c->b->a。但是此时并没有替换oldEntry数组。释放了时间片。
4. 线程2获得了时间片。此时的e=a，next=b. 然后拷贝过程，先拷贝a：newEntry[0]=a，e=b,此时注意：执行完后的b.next已经变成了a，并不是c了。
5. 线程2开始执行完了第一个循环开始拷贝第二个节点，newEntry[0]=b->a。 执行完后的此时的e=a，next=null。
6. 开始第三个循环：newEntry[0]=a->b->a,节点a和b互相引用，形成了一个环，当在数组该位置get寻找对应的key时，就发生了死循环。
> 文字太不直观了，建议加图一起看：https://mp.weixin.qq.com/s/i_r1aLlQR8qTz7kz8vKk6w

**JDK1.8**   
解决了1.7的BUG，但是1.8还存在问题。
> https://blog.csdn.net/gs_albb/article/details/88091808


# `HashMap的内部结构？`
在JDK1.8中：内部结构为 数组(Entry[] table)+链表(链表长度大于8时会转换成红黑树，若桶中元素小于等于6时，树结构还原成链表形式)。HashMap初始容量为16，负载因子默认为0.75f。     
**`什么时候扩容:`**
Enrty数组长度*0.75f <=  size(key,value 数量)    
**`转化为什么选择6和8:`**
红黑树的平均查找长度是log(n)，长度为8，查找长度为log(8)=3，链表的平均查找长度为n/2，当长度为8时，平均查找长度为8/2=4，这才有转换成树的必要；链表长度如果是小于等于6，6/2=3，虽然速度也很快的，但是转化为树结构和生成树的时间并不会太短

# `HashMap 的扩容过程 `
首先计算扩容容量，创建新数组，同时rehash 元数组元素.    
**`JDK1.7`**
1. 先创建扩容后的`新Entry[]`.
2. 遍历`old Entry数组`，取出第一个节点e。
3. 取出`e.next`作为临时变量。
4. hash计算e在`新Entry[]`中的位置，然后设置`e.next= 新Entry[i]`
5. 替换`新Entry[i] =e`，这就是`头插法。`，`添加一个新key和4、5步骤一样，不过会先判断是否Hash冲突。`
6. new数组替换old数组。
7. 重新计算扩容阈值。

**`JDK1.8`**
和JDK1.7的类似，只是头插法，变成了尾插法。


# `为什么HashMap`扩容是2倍
2次幂能更好的使用二进制操作&，这样效率更高。同时2次幂也能是空间更均匀。减少Hash碰撞的概率。

# `什么Hash算法？Hash冲突？Hash冲突有哪些解决方法？HashMap采用的那种？`
**Hash算法**    
就是把任意长度的输入，通过散列算法，变换成固定长度的输出，该输出就是散列值。    
**hash冲突**    
所谓冲突，即两个元素通过散列函数计算得到的地址(索引))相同。    
**解决方法**    
HashMap采用的拉链法。其余的不做说明(我看不懂)，自己了解。


# `HashMap 1.7 与 1.8 的 区别，说明 1.8 做了哪些优化，如何优化的？` 
- 存储结构：JDK 7 使用的是数组 + 链表；JDK 8 使用的是数组 + 链表 + 红黑树。
- 存放数据的规则：JDK 7 `无冲突时，存放数组；冲突时，存放链表`；JDK 8 在没有冲突的情况下直接存放数组，有冲突时，当链表长度小于 8 时，存放在单链表结构中，当链表长度大于 8 时，树化并存放至红黑树的数据结构中。`转化红黑树是在扩容后，准备插入元素前。`
- 插入数据方式：JDK 7 使用的是头插法（先将原位置的数据移到后 1 位，再插入数据到该位置）；JDK 8 使用的是尾插法（直接插入到链表尾部/红黑树）。


# `final finally finalize` 
final用于声明属性，方法和类，分别表示属性不可交变，方法不可覆盖，类不可继承。    
finally是异常处理语句结构的一部分，表示总是执行。    
finalize是Object类的一个方法，在垃圾收集器执行的时候会调用被回收对象的此方法，供垃圾收集时的其他资源回收，例如关闭文件等,但是不建议使用。

# `Arrays.sort 实现原理和 Collection 实现原理 `
todo

# `LinkedHashMap的应用` 
LinkedHashMap继承于HashMap，简单说：保存了记录的插入顺序，可在遍历时保持与插入一样的顺序。
LinkedHashMap拓展了HashMap的Entry，加入了`Entry<K,V> before, after;`来保证插入顺序。同时加入了
```
    /**
     * The head (eldest) of the doubly linked list.
     */
    transient LinkedHashMap.Entry<K,V> head;

    /**
     * The tail (youngest) of the doubly linked list.
     */
    transient LinkedHashMap.Entry<K,V> tail;
```
两个变量确定首尾位置。在newNode(LinedHashMap重写了)的时候就会更新head和tail(更详细自己看源码吧)))。     
```
    private void linkNodeLast(LinkedHashMap.Entry<K,V> p) {
        LinkedHashMap.Entry<K,V> last = tail;
        tail = p;
        if (last == null)
            head = p;
        else {
            p.before = last;
            last.after = p;
        }
    }
```
<img src="https://s2.ax1x.com/2019/10/13/ujlmad.jpg" width="200" hegiht="200" />

![avatar](https://s2.ax1x.com/2019/10/13/ujl3M8.jpg)

# `cloneable接口实现原理` 
loneable其实就是一个标记接口，只有实现这个接口后，然后在类中重写Object中的clone方法，然后通过类调用clone方法才能克隆成功，如果不实现这个接口，则会抛出CloneNotSupportedException(克隆不被支持)异常。


# `数组在内存中如何分配`
![avatar](https://s2.ax1x.com/2019/10/13/uj1eyT.jpg)


# `JAVA 中的几种基本数据类型是什么，各自占用多少字节。`
boolean 、byte、char、short、int、long、float、bouble
字节详见jvm面试题



# `反射Class.forName 和 ClassLoader.loadClass 区别。`
Class.forName 默认初始化类，ClassLoader.loadClass不会。

# `有序Map：TreeMap`
TreeMap底层是使用的 红黑树。对红黑树还只是概念，以后再补充。



# `Hashset的 value为撒用Object？`
因为hashMap的remove方法会返回被移除数据，如果为null的话，无法判断是否被移除成功。


# `java8 lambda表达式中 变量为什么必须是行为final的`
为什么 Lambda 表达式(匿名类) 不能访问非 final 的局部变量呢？因为实例变量存在堆中，而局部变量是在栈上分配，Lambda 表达(匿名类) 会在另一个线程中执行。如果在线程中要直接访问一个局部变量，可能线程执行时该局部变量已经被销毁了，而 final 类型的局部变量在 Lambda 表达式(匿名类) 中其实是局部变量的一个拷贝。

# `为什么arrayList 扩容是1.5倍多？`
<img src="https://s2.ax1x.com/2019/10/13/uj3ugP.jpg" width="500" hegiht="500" />


# `动态代理与cglib实现的区别？`
todo

# `函数式编程优点？`
. 函数和变量的地位相同，可以作为参数和返回值
. 支持闭包和高阶函数，可以让对象像函数一样操作
. 支持惰性计算，可以等到需要求值的时候再进行计算
. 只使用表达式，不使用语句，所有的代码都是单纯的运算过程，所有的操作都是用于处理运算
. 没有副作用，所有的功能均返回一个新的值，不会改变外部变量的状态和值

# `Stream流`
Stream 不是集合元素，它不是数据结构并不保存数据，它是有关算法和计算的，它更像一个高级版本的 Iterator
Stream的常用中间操作
1. peek()--消费流中的每一个元素
2. map() mapToInt() mapToDouble()等等--将流中的每一个元素映射为另一个元素
3. filter()--将流中满足要求的元素过滤出来
4. sort()--将流中的元素按一定的规则排序
5. distinct()--去重
6. limit()--截断流只保留指定个数的元素


# `你实际开发遇到哪些Exception？`
NullPointerException、IOException、ArrayIndexOutOfBoundsException、IndexOutOfBoundsException、InterruptedException

# `StringBuffer和StringBuilder是否线程安全?`
StringBuffer都是同步方法，synchronized包裹。     
stringBuilder分线程安全。

# `Tomcat 优化？`
1. 8.0以前默认BIO，需要配置成NIO
2. 服务器返回给客户端的xml报文数据量太大，费时耗流量，于是要求服务端添加gzip支持。所以服务端采用gzip压缩，极大减小网络传输数据，提高响应速度和减少流量。    

**线程池优化**
name：线程池名称，用于 Connector中指定。
namePrefix：所创建的每个线程的名称前缀，一个单独的线程名称为 namePrefix+threadNumber。
maxThreads：池中最大线程数。
minSpareThreads：活跃线程数，也就是核心池线程数，这些线程不会被销毁，会一直存在。
maxIdleTime：线程空闲时间，超过该时间后，空闲线程会被销毁，默认值为6000（1分钟），单位毫秒。
maxQueueSize：在被执行前最大线程排队数目，默认为Int的最大值，也就是广义的无限。除非特殊情况，这个值不需要更改，否则会有请求不会被处理的情况发生。
prestartminSpareThreads：启动线程池时是否启动 minSpareThreads部分线程。默认值为false，即不启动。
threadPriority：线程池中线程优先级，默认值为5，值从1到10。
className：线程池实现类，未指定情况下，默认实现类为org.apache.catalina.core.StandardThreadExecutor。如果想使用自定义线程池首先需要实现 org.apache.catalina.Executor接口。

> https://blog.csdn.net/qq_28109171/article/details/84256783  tomcat 性能调优

**ARP优化**
todo

# # Java8中时间API发生了那些改变？
Java8之前的日期和时间处理比较难用，主要有几方面
1. Java的`java.util.Date`和`java.util.Calendar`类易用性查，而且不是线程安全的。
2. SimpleDateFormat格式化时间不是线程安全的。
3. 对日期的计算比较繁琐，而且容易出错，因为月份是从0开始的，从Calendar中获取的月份需要加一才能表示当前月份。

在java8中有了新增了时间类。
**LocalDate和LocalTime**     
LocalDate和LocalTime类似，LocalDate不包含具体时间，而LocalTime包含具体时间。

**LocalDateTime**     
todo
是LocalDate和LocalTime的结合体，可以直接创建LocalDate和LoaclTime。
> 参考：https://lw900925.github.io/java/java8-newtime-api.html

# # SimpleDateFormat并发隐患及其解决
非线程安全的，这个在类注释中也说明了的，主要是`format`方法(当然其他方法也有,比如`parse`方法)
![](https://s2.ax1x.com/2019/10/20/KMcIWF.md.png)     
假设线程1刚刚执行完calendar.setTime把时间设置成2018-11-11，还没等执行完，线程2又执行了calendar.setTime把时间改成了2018-12-12。这时候线程1继续往下执行，拿到的calendar.getTime得到的时间就是线程2改过之后的。所以在并发下有问题。

**解决方案**
1. ThreadLocal
2. 在调用`format`方法时加锁。
3. java8的DateTimeFormatter。他是线程安全。


# # Integer类的缓存机制

这些类都有缓存的范围，其中`Byte，Short，Integer，Long为 -128 到 127，Character范围为 0 到 127。`

除了 Integer 可以通过参数改变范围外，其它的都不行。


# # (美团)Druid 底层原理了解么？如果叫你实现一个类似Druid的你能实现么？
todo


# # (美团)如果让你实现一个线程安全的HashMap你会怎么实现？出了ConcurrentHashMap之外了？
1.  copyonwrite思想
    todo
2. 双HashMap 实现一个线程安全的HashMap？GO中的sync.map实现
    todo。

