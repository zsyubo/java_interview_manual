# `List 和 Set 的区别` 

# `HashSet 是如何保证不重复的` 

Hash底层其实就是一个HashMap来承装元素，同时HashSet重写了equals和hashCode方法老保证set对象的唯一性，hashset：元素作为key，Object最为value。

# `HashMap 是线程安全的吗，为什么不是线程安全的（最好画图说明多线程环境下不安全）? `

不是非线程安全，在多线程情况下可能会出现问题。在并发环境下会发生死循环问题。主要扩容rehash时，

# `HashMap 的扩容过程 `

首先计算扩容容量，创建新数组，同时rehash 元数组元素

这里写图片描述

# `为什么HashMap`扩容是2倍

2次幂能更好的使用二进制操作&，这样效率更高。同时2次幂也能是空间更均匀。

# `HashMap 1.7 与 1.8 的 区别，说明 1.8 做了哪些优化，如何优化的？` 

一个是 hash下标是

# `final finally finalize` 

# `强引用 、软引用、 弱引用、虚引用、Java反射 `

# `Arrays.sort 实现原理和 Collection 实现原理 `

# `LinkedHashMap的应用` 

# `cloneable接口实现原理` 

# `异常分类以及处理机制` 

# `wait和sleep的区别` 

# `数组在内存中如何分配`

\

# `JAVA 中的几种基本数据类型是什么，各自占用多少字节。`

boolean 、byte、char、short、int、long、float、bouble


# `讲讲类的实例化顺序，比如父类静态数据，构造函数，字段，子类静态数据，构造函数，字段，当 new 的时候， 他们的执行顺序。`

# `反射Class.forName 和 ClassLoader.loadClass 区别。`

Class.forName 默认初始化类，ClassLoader.loadClass不会。

# `有序Map：treeMap`
treeMap底层是使用的 红黑树



# `Hashset的 value为撒用Object？`

因为hashMap的remove方法会返回被移除数据，如果为null的话，无法判断是否被移除成功。


# `java8 lambda表达式中 变量为什么必须是行为final的`
为什么 Lambda 表达式(匿名类) 不能访问非 final 的局部变量呢？因为实例变量存在堆中，而局部变量是在栈上分配，Lambda 表达(匿名类) 会在另一个线程中执行。如果在线程中要直接访问一个局部变量，可能线程执行时该局部变量已经被销毁了，而 final 类型的局部变量在 Lambda 表达式(匿名类) 中其实是局部变量的一个拷贝。

# `为什么arrayList 扩容是1.5倍多？`

# `动态代理与cglib实现的区别？`

# `函数式编程优点？`
. 函数和变量的地位相同，可以作为参数和返回值
. 支持闭包和高阶函数，可以让对象像函数一样操作
. 支持惰性计算，可以等到需要求值的时候再进行计算
. 只使用表达式，不使用语句，所有的代码都是单纯的运算过程，所有的操作都是用于处理运算
. 没有副作用，所有的功能均返回一个新的值，不会改变外部变量的状态和值