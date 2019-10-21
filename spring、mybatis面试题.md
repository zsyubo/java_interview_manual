#  `思维导图`
![avatar](https://s2.ax1x.com/2019/10/13/ux6tu8.md.jpg)

#  `IOC和DI区别？`
IOC是一种思想，DI则是一种实现手段。IOC有两种实现手段：DI（依赖查找），DL(依赖注入)

**IOC：控制反转 **

原来需要自己去new对象，现在有第三方来new对对象，把new好的对象送过来。

也就是创建对象由IOC容器来做，IOC容器来控制对象。

反转：以前自己new，现在IOC直接送过来。

**DI：依赖注入**

实体被动接受其依赖的其他组件被IOC容器注入

# `如果我想注册一个Mysql Driver到Spring有几种方式？`

1. xml 定义bean标签

2. FactoryBean创建，新建类继承factoryBean，里面有个getObject方法，返回Bean。然后再在Xml定义Bean标签

3. Bean注解。

4. 其他。。。。。。

# `spring依赖注入有几种方式？`

1. set

2. 构造方法

3. 自动注入auto

4. 方法注入？


# `Spring内部是怎样描述一个Bean信息的？`

BeanDefinition描述一个Bean信息，比如beanclassName、作用域、是否是懒加载等。是一个接口，其中一些重要的属性(XML为例)
![avatar](https://s2.ax1x.com/2019/10/13/ux6sg0.jpg)

要值得注意的是，这里面并没有bean name字段 ，name只是帮助我们注册bean到map中。DefaultListableBeanFactory中的registerBeanDefinition(String beanName, BeanDefinition beanDefinition))方法（其实也可以使用BeanDefinitionRegistry，只是功能没这么强大而已） 注册到：
```
// DefaultListableBeanFactory
 /** Map of bean definition objects, keyed by bean name. */
 private final Map<String, BeanDefinition> beanDefinitionMap = new ConcurrentHashMap<>(256);
// 别名，xml中name
 /** Map of singleton and non-singleton bean names, keyed by dependency type. */
 private final Map<Class<?>, String[]> allBeanNamesByType = new ConcurrentHashMap<>(64);
```
这里也可以看出，我们注册时，可以多次注册同一个bean，只需要保证每一次的beanname不同就行。

# `BeanFactory 和 FactoryBean？`
BeanFactory是Spring里面最低层的接口，提供了最简单的容器的功能，提供读取配置文件、管理加载Bean，实例化、维护Bean之间的依赖关系，同时负责Bean的生命周期。 

FactoryBean只是一个Bean，但是比较特殊，它是一个能生产对象的工厂Bean，他的实现类最终也会注册到BeanFactory。也就是我们可以自定义FactoryBean来控制一个Bean的生产过程。FactoryBean是一个接口,一个Class 对应一个FactoryBean。

# `Spring IOC 的理解，其初始化过程？ `
https://javadoop.com/post/spring-ioc
1. IOC启动的时候会从配置文件读取需要注册的Bean信息，然后封装成一个BeanDefinition。
2. 因为Spring默认是懒加载，到getBean时才会真正实例化Bean。所以会先把BeanDefinition放入到BeanDefinitionRegistry容器中。
3. BeanDefinitionRegistry接口主要负责管理所有注册过的BeanDefinition，比如增删改查功能，他的实现类需要2个工具成员变量：beanDefinitionMap、singletonObjects（都是ConcurrentHashMap结构)
4. beanDefinitionMap负责缓存所有注册过的BeanDefinition;singletonObjects为单例类的缓存器，每个通过BeanDefinitionRegistry创建出来的bean，都缓存在singletonObjects里面，除非是多例
5. 而BeanFactory接口是面向用户的，实际上他只是一个触发器，用户调用getBean方法，会先去beanDefinitionMap查找这个Bean是否已经生成了BeanDefinition，没有就报错，有就根据BeanDefinition来反射创建一个Bean，因为默认是单例的，创建完之后就缓存在singletonObjects里面

# `Spring AOP 的理解及其原理？ `
AOP是面向切面编程，是对OOP的一种补充，主要用于处理一些具有横切面性质的服务，比如日志输出、安全控制等。

Spring AOP实现的默认两种方式：JDK动态代理、CGLIB。

**AOP主要的两种实现**

- Spring AOP

  采用的是动态代理，在运行期间对业务方法进行增强，所以不会产生新类。

- AspectJ

  底层技术是静态代理。主要是在编译期时，在不改变代码的前提下织入代理。

**JDK动态代理(实现接口)、CGLIB(继承)**

- JDK动态代理

  只能为接口创建动态代理实例，需要需要获取目标类的接口信息

- CGLIB

  依赖asm包。修改器目标类的字节码生成子类。


# `BeanFactory 和 ApplicationContext？` 

BeanFactory是Spring里面最低层的接口，提供了最简单的容器的功能，提供读取配置文件、管理加载Bean，实例化、维护Bean之间的依赖关系，同时负责Bean的生命周期。

ApplicationContext：应用上下文，继承BeanFactory接口，是BeanFactory的拓展升级版。它是Spring的一各更高级的容器，提供了更多的有用的功能；

1. 国际化（MessageSource）

2. 访问资源，如URL和文件（ResourceLoader）

3. 载入多个（有继承关系）上下文 ，使得每一个上下文都专注于一个特定的层次，比如应用的web层  

4. 消息发送、响应机制（ApplicationEventPublisher）

5. AOP（拦截器）

# `Spring Bean 的生命周期，如何被管理的？`

todo

1. 实例化,也就是new
2. 设置bean的Aware
3. BeanPostProcessor.postProcessBeforeInitialization(Object bean, String beanName)
4. InitializingBean.afterPorpertiesSet
5. BeanPostProcessor.postProcessAfterInitialization(Object bean, String beanName)
6. SmartInitializingSingleton.afterSingletonsInstantiated
7. SmartLifecycle.start
8. bean已经在spring容器的管理下，可以做我们想做的事
9. SmartLifecycle.stop(Runnable callback)
10. DisposableBean.destroy()



# `Spring Bean 的获取过程是怎样的？` 
![](https://s2.ax1x.com/2019/10/13/ux6h59.jpg)


# `如果要你实现Spring AOP，请问怎么实现？` 

#  `如果要你实现Spring IOC，你会注意哪些问题？ `

#  `Spring 是如何管理事务的，事务管理机制？` 
Spring的事务机制包括声明式事务和编程式事务。Spring 采用AOP来实现生命式事务。因为这一点，所以才有了事务失效的问题

编程是事务：编程式**事务管理**使用TransactionTemplate或者直接使用底层的PlatformTransactionManager。对于编程式**事务管理**，**spring**推荐使用TransactionTemplate。

**原理**

todo

> [Spring事务原理一探](https://zhuanlan.zhihu.com/p/54067384)
>
> []()

#  `Spring 的不同事务传播行为有哪些，干什么用的？` 
![](https://s2.ax1x.com/2019/10/13/ux6XUH.jpg)

#  ` Spring 中用到了那些设计模式？` 

#  `Spring MVC 的工作原理？` 

a. 用户向服务器发送请求，请求被 springMVC 前端控制器 DispatchServlet 捕获；

b. DispatcherServle 对请求 URL 进行解析，得到请求资源标识符（ URL），然后根据该 URL 调用 HandlerMapping

将请求映射到处理器 HandlerExcutionChain；

c. DispatchServlet 根据获得 Handler 选择一个合适的 HandlerAdapter 适配器处理；

d. Handler 对数据处理完成以后将返回一个 ModelAndView（）对象给 DisPatchServlet;

e. Handler 返回的 ModelAndView()只是一个逻辑视图并不是一个正式的视图， DispatcherSevlet 通过

ViewResolver 试图解析器将逻辑视图转化为真正的视图 View;

h. DispatcherServle 通过 model 解析出 ModelAndView(视图解析器)中的参数进行解析最终展现出完整的 view 并返回给



#  `Spring 循环注入的原理？` 

Spring为了解决单例的循环依赖问题，使用了三级缓存。

singletonFactories ： 单例对象工厂的cache

earlySingletonObjects ：提前暴光的单例对象的Cache

singletonObjects：单例对象的cache



我们在创建bean的时候，首先是从cache中获取这个单例的bean，这个缓存就是singletonObjects。

如果容器允许提前曝光和引用，那么在singletonFactories 中放入该bean的singletonFactory，这样在依赖注入的时候，就能在第三级缓存中找到singletonFactory 了。



经过这样的操作后，面对 "A 依赖了B，同时B依赖了A"这种循环依赖的情况。A首先完成了初始化的第一步，并且将自己提前曝光到singletonFactories中，此时进行初始化的第二步（依赖注入），发现自己依赖对象B，此时就尝试去get(B)，发现B还没有被create，所以走create流程，B在初始化第一步的时候发现自己依赖了对象A，于是尝试get(A)，尝试一级缓存singletonObjects(肯定没有，因为A还没初始化完全)，尝试二级缓存earlySingletonObjects（也没有），尝试三级缓存singletonFactories，由于A通过ObjectFactory将自己提前曝光了，因此B能够通过ObjectFactory.getObject拿到A对象(虽然A还没有初始化完全，但是总比没有好呀)，B拿到A对象后顺利完成了初始化阶段1、2、3，完全初始化之后将自己放入到一级缓存singletonObjects中。此时返回A中，A此时能拿到B的对象顺利完成自己的初始化阶段2、3，最终A也完成了初始化，进去了一级缓存singletonObjects中，而且更加幸运的是，由于B拿到了A的对象引用，所以B现在hold住的A对象完成了初始化，最后皆大欢喜，A,B都成功的初始化了。

原文链接：https://blog.csdn.net/u014634338/article/details/83866305

#  `Spring AOP的理解，各个术语，他们是怎么相互工作的？ `

Aspect（切面）： Aspect 声明类似于 Java 中的类声明，在 Aspect 中会包含着一些 Pointcut 以及相应的 Advice。

Joint point（连接点）：表示在程序中明确定义的点，典型的包括方法调用，对类成员的访问以及异常处理程序块的执行等等，它自身还可以嵌套其它 joint point。

Pointcut（切点）：表示一组 joint point，这些 joint point 或是通过逻辑关系组合起来，或是通过通配、正则表达式等方式集中起来，它定义了相应的 Advice 将要发生的地方。

Advice（增强）：Advice 定义了在 Pointcut 里面定义的程序点具体要做的操作，它通过 before、after 和 around 来区别是在每个 joint point 之前、之后还是代替执行的代码。

Target（目标对象）：织入 Advice 的目标对象.。

Weaving（织入）：将 Aspect 和其他对象连接起来, 并创建 Adviced object 的过程

![](https://s2.ax1x.com/2019/10/13/uxcQqU.md.png)

![](https://s2.ax1x.com/2019/10/13/uxctR1.png)

原文链接：https://blog.csdn.net/q982151756/article/details/80513340

#  `Spring 如何保证 Controller 并发的安全？`

# `Spring boot 加载过程`




# `mybatis一级缓存和二级缓存区别` 
https://blog.51cto.com/zero01/2103911
一级缓存是 session中的。默认开启的。
二级缓存是跨session。需要自己实现。
# `mybatis #和$区别` 
\#号是预编译的，防止sql注入
\$是直接替换，字符串拼接，不能防止sql注入
使用#在初始化阶段，会被替换成？号，同时生成参数映射，而使用\$在初始化阶段，没有什么特别的地方，仅仅做了一个是否动态语句的判断
具体底层参见：https://blog.csdn.net/surpass0728/article/details/80697442

# `mybatis 用了哪些设计模式能说说么？` 
最显而易见的就是SqlSessionFactoryBuilder、XMLConfigBuilder、XMLMapperBuilder等，使用的是Builder。为什么使用了？因为更自由。
工厂模式：SqlSessionFactory(获取Sqlsession)、MapperProxyFactory(创建动态代理的)。
代理模式：使用JDK动态代理，通过对每一个Mapper对象生成动态代理对象。
模板方法：Executor：BaseExecutor和SimpleExecutor。


# `mybatis 初始化流程？` 
**初始化流程**
首先肯定读取配置文件并解析成一个Configuration，最终更具Configurations生成DefaultSqlSessionFactory。当然Mapper标签的解析也是在这一步。每个Mapper类会解析为MapperProxyFactory，放在MapperRegistry中。而每个Sql标签文件则会解析成一个MappedStatement对象，放在Configuration中的mappedStatements 的StrictMap中(继承自HashMap)，存放的id为类命名空间+标签id。
# `mybatis 怎么执行一次查询？` 
**获取Sqlsession**
从初始化的DefaultSqlSessionFactory中创建，先会设置一个事务管理器，默认为自动提交。之后创建一个SimpleExecutor，他是执行最终执行SQl的。
之后从Sqlsession中的MapperRegistry获取Mapper 的代理对象。默认是使用的Jdk的动态代理。最终由MapperMethod来做整个sql方法执行的入口，
之后会从一级缓存中查询数据，没有则继续执行查询 。根据既有的参数(MappedStatement)，创建StatementHandler对象来执行查询操作，默认是执行sql的是PreparedStatement
将创建Statement传递给StatementHandler对象,调用parameterize()方法赋值。调用StatementHandler.query()方法，返回List结果集
这里面值得注意的是一级缓存是默认开启。一级缓存是存放在BaseExecutor的PerpetualCache中，底层其实就是一个HashMap。Key为CacheKey，由MappedStatement的id、分页参数、sql、查询参数组装而成。
# `mybatis批量插入？` 
第一种就是for循环了，好处了代码控制，比较方便。
第二种是在打开Sqlsession指定 
```
SqlSession sqlSession = sqlSessionTemplate.getSqlSessionFactory().openSession(ExecutorType.BATCH, false);
```
第三种是通过xml,传入一个参数的list
```
<insert id="insertBatch">
    INSERT INTO t_user
            (id, name, del_flag)
    VALUES
    <foreach collection ="list" item="user" separator =",">
         (#{user.id}, #{user.name}, #{user.delFlag})
    </foreach >
</insert>
```
当然，肯定是第三种最快啦，毕竟最终是拼接成一个Sql插入，实际开发大批量插入也是第三种居多。但是第三点需要注意的是，最后的Sql如果超过1m，Mysql会抛出异常。
```
nested exception is com.mysql.jdbc.PacketTooBigException: Packet for query is too large (5677854 > 1048576).
You can change this value on the server by setting the max_allowed_packet' variable.
```
（可通过调整MySQL安装目录下的my.ini文件中[mysqld]段的＂max_allowed_packet = 1M＂）

网上也有说使用第二种的，一般说如果字段太多建议第二种，`实际使用时两种都测试下，择其优。`





# # `Spring boot自动装配原理？`
开启Spring自动装配的注解是`@EnableAutoConfiguration`,其注解有一行很重要：`@Import(AutoConfigurationImportSelector.class);`
@Import 注解可以普通类导入到 IoC容器中,AutoConfigurationImportSelector是自动装配非常核心的类，其中一段代码：
```
List<String> configurations = SpringFactoriesLoader.loadFactoryNames(
				getSpringFactoriesLoaderFactoryClass(), getBeanClassLoader());
```
SpringFactoriesLoader是Spring的，说明自动装配并不是只属于Boot的，而且Spring本来就有的，只不过Spring boot在这基础上做了整合和拓展。
此方法核心就是去加载Jar包中META-INF/spring.factories路径中的配置文件key为`EnableAutoConfiguration`的内容。
```
	public static final String FACTORIES_RESOURCE_LOCATION = "META-INF/spring.factories";

public static List<String> loadFactoryNames(Class<?> factoryClass, @Nullable ClassLoader classLoader) {
		String factoryClassName = factoryClass.getName();
		return loadSpringFactories(classLoader).getOrDefault(factoryClassName, Collections.emptyList());
	}
```
spring.factories
```
# Auto Configure
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
org.springframework.boot.autoconfigure.admin.SpringApplicationAdminJmxAutoConfiguration,\
org.springframework.boot.autoconfigure.aop.AopAutoConfiguration,\
org.springframework.boot.autoconfigure.amqp.RabbitAutoConfiguration.........
```
最终读取出来了所有自动装配的配置类。      
之后
```
@Import(AutoConfigurationPackages.Registrar.class)
public @interface AutoConfigurationPackage {

}
```
`AutoConfigurationPackages.Registrar`主要是加载注册配置类的逻辑。

**让我们来看一个自动装配类**
```
 *
 * @author Dave Syer
 * @author Andy Wilkinson
 * @author Stephane Nicoll
 * @author Brian Clozel
 */
@Configuration
@ConditionalOnWebApplication(type = Type.SERVLET)
@ConditionalOnClass({ Servlet.class, DispatcherServlet.class })
// Load before the main WebMvcAutoConfiguration so that the error View is available
@AutoConfigureBefore(WebMvcAutoConfiguration.class)
@EnableConfigurationProperties({ ServerProperties.class, ResourceProperties.class,
		WebMvcProperties.class })
public class ErrorMvcAutoConfiguration {}
```

**`总结`**    
自动装配不是什么新鲜事物。Spring boot只是做了自己的整合，依靠`@ConditionalOnClass`、`@Import()`......等注解来组合实现的。

# # Spring 事务失效问题以及原因？
代码如下图：
```
class T {
    public int createFirst(){
       //dosometing....
       try {
           this.createSecond();
       }catch (Exception e){
          throw e;
       }
       //dosometing....
       return 0;
    }
    @Transactional
    public int createSecond(){
       //dosometing with db....

    }
}
```
在实际中会出现`createSecond()方法`事务不起作用的情况。这里主要问题就是在于这个额this，this就是指当前对象，问题就出在这，想要事务执行`createSecond()方法`，必须使用代理对象执行，因为代理对象会拦截到`@Transactional`来执行相关的增强。但是此时却是直接调用，绕过了代理对象增强的部分，也就是代理增强部分失效。     
**解决方案**是，手动获取代理对象
```
T t = (T) AopContext.currentProxy(); 
//获取代理对象
t.createSecond(); 
//通过代理对象调用createSecond

//需要在@EnableAspectJAutoProxy添加属性值。
//@EnableAspectJAutoProxy(exposeProxy = true)
```
> https://blog.csdn.net/canot/article/details/80855439  从Spring AOP的原理理解@Transactional失效问题


# # @Bean注解嵌套情况
```
 @Bean
    public UserEntity userEntity() {
        System.out.println("userEntity-1");
        getTestBean();
        System.out.println("userEntity-2");
        return new UserEntity("mayikt", 21);
    }

    @Bean
    public TestBean getTestBean(){
        System.out.println("getTestBean执行");
        return new TestBean(99,"Hello");
    }
```
执行结果，不会重复执行。
```
userEntity-1
getTestBean执行
userEntity-2
```



# # Spring boot启动流程？

硬骨头，这只是讲个大概。

1. 日常编写的Spring boot启动类上有`@SpringBootApplication`注解和`SpringApplication.run()`

   注解没什么好说的，主要就是`@SpringBootConfiguration(相当于一个Configuration)、@EnableAutoConfiguration(自动装配)`两个注解。主要的启动方式`run()`方法

2.  使用`SpringFactoriesLoader`查找并加载classpath下的`MATE/spring.factories`文件的`ApplicationContextInitializer`并实例化子类。主要是对ConfiurableApplicationContext的实例做进一步的设置和处理。

3. `run`方法进入一开始就是通过依赖所包含的jar来`推断`应用的类型，是`Reactive应用`、`servlet应用`、`NONE`三种的那一种。

4. 找出所有的`SpringApplicationRunListener`并封装到`SpringApplicationRunListeners`中，用于监听run方法的执行。监听的过程中会封装成事件并广播出去让初始化过程中产生的应用程序监听器进行监听 。

5. 打印Banner。

6. 使用`SpringFactoriesLoader`查找并加载classpath下的`MATE/spring.factories`文件的`SpringBootExceptionReporter`。这个是分析异常的？todo

7. 执行Spring的初始化流程。