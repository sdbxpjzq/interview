# 线程池

## 构造参数

ThreadPoolExecutor参数最全的构造方法：

![图片](https://youpaiyun.zongqilive.cn/image/640.png)

- **corePoolSize：**线程池的核心线程数，说白了就是，即便是线程池里没有任何任务，也会有corePoolSize个线程在候着等任务。
- **maximumPoolSize：**最大线程数，不管你提交多少任务，线程池里最多工作线程数就是maximumPoolSize。
- **keepAliveTime：**线程的存活时间。当线程池里的线程数大于corePoolSize时，如果等了keepAliveTime时长还没有任务可执行，则线程退出。
- **unit：**这个用来指定keepAliveTime的单位，比如秒:TimeUnit.SECONDS。
- **workQueue：**一个阻塞队列，提交的任务将会被放到这个队列里。
- **threadFactory：**线程工厂，用来创建线程，主要是为了给线程起名字，默认工厂的线程名字：pool-1-thread-3。
- **handler：**拒绝策略，当线程池里线程被耗尽，且队列也满了的时候会调用。





# GC

## 可达性分析算法

不可达的对象，也并非是“非死不可”的，这时候它们暂时处于“缓刑”阶段，要真正宣告一个对象死亡，至少要经历两次标记过程：如果对象在进行可达性分析后发现没有与GC Roots相连接的引用，那它将会被第一次标记并且进行一次筛选，筛选的条件是此对象是否是否有必要执行finalize()方法。当对象没有覆盖finalize()方法，或者finalize()方法已经被虚拟机调用过，虚拟机将这两种情况都视为“没有必要执行”。

如果这个对象被判定为有必要执行finalize()方法，那么这个对象将会放置在一个叫做F-Queue的队列之中。并在稍后由一个虚拟机自动建立的，低优先级的Finalizer线程去执行它。这里所谓“执行”是指虚拟机会触发这个方法，但并不承诺会等待它运行结束，这样做的原因是，如果有一个对象在finalize()方法中执行缓慢，或者发生死循环，将可能会导致F-Queue队列中其他对象永久处于等待，甚至导致整个内存回收系统崩溃。

**finalize()方法是对象逃脱死亡命运的最后一次机会，稍后GC将对F-Queue中的对象进行第二次小规模的标记，如果对象这个时候，未被重新引用，那它基本上就真的被回收了。**

## GC Roots

哪些可以作为GC Roots?



## 类加载机制







# 集合相关

## ArrayList和LinkedList

```
1．对ArrayList和LinkedList而言，在列表末尾增加一个元素所花的开销都是固定的。
对ArrayList而言，主要是在内部数组中增加一项，指向所添加的元素，偶尔可能会导致对数组重新进行分配；
而对LinkedList而言，这个开销是统一的，分配一个内部Entry对象。

2．在ArrayList的中间插入或删除一个元素意味着这个列表中剩余的元素都会被移动；而在LinkedList的中间插入或删除一个元素的开销是固定的。

3．LinkedList不支持高效的随机元素访问。

4．ArrayList的空间浪费主要体现在在list列表的结尾预留一定的容量空间，而LinkedList的空间花费则体现在它的每一个元素都需要消耗相当的空间
```



## HashMap

### 为何HashMap的数组长度一定是2的次幂





### 线程不安全

#### 扩容造成死循环(JDK7)

这里假设

> 1. hash算法为简单的用key mod链表的大小。
> 2. 最开始hash表size=2，key=3,7,5，则都在table[1]中。
> 3. 然后进行resize，使size变成4。

未resize前的数据结构如下：

![](https://youpaiyun.zongqilive.cn/image/20210514140742.png)

如果在单线程环境下，最后的结果如下：

![](https://youpaiyun.zongqilive.cn/image/20210514140755.png)

这里的转移过程，不再进行详述，只要理解transfer函数在做什么，其转移过程以及如何对链表进行反转应该不难。

然后在多线程环境下，假设有两个线程A和B都在进行put操作。线程A在执行到transfer函数中第11行代码处挂起，因为该函数在这里分析的地位非常重要，因此再次贴出来。

![](https://youpaiyun.zongqilive.cn/image/20210514140821.png)

此时线程A中运行结果如下：

![](https://youpaiyun.zongqilive.cn/image/20210514140839.png)

线程A挂起后，此时线程B正常执行，并完成resize操作，结果如下：

![](https://youpaiyun.zongqilive.cn/image/20210514140852.png)

这里需要特别注意的点：**由于线程B已经执行完毕，根据Java内存模型，现在newTable和table中的Entry都是主存中最新值：****7.next=3，3.next=null。**

此时切换到线程A上，在线程A挂起时内存中值如下：e=3，next=7，newTable[3]=null，代码执行过程如下：

```
newTable[3]=e ----> newTable[3]=3
e=next ----> e=7
```

此时结果如下：

![](https://youpaiyun.zongqilive.cn/image/20210514140919.png)

继续循环：

```
e=7
next=e.next ----> next=3【从主存中取值】
e.next=newTable[3] ----> e.next=3【从主存中取值】
newTable[3]=e ----> newTable[3]=7
e=next ----> e=3
```

结果如下：

![](https://youpaiyun.zongqilive.cn/image/20210514140934.png)

再次进行循环：

```
e=3
next=e.next ----> next=null
e.next=newTable[3] ----> e.next=7 即：3.next=7
newTable[3]=e ----> newTable[3]=3
e=next ----> e=null
```

注意此次循环：e.next=7，而在上次循环中7.next=3，出现环形链表，并且此时e=null循环结束。

结果如下：

![](https://youpaiyun.zongqilive.cn/image/20210514141039.png)

#### 扩容造成数据丢失(JDK7)

初始时：

![](https://youpaiyun.zongqilive.cn/image/20210514141628.png)

线程A和线程B进行put操作，同样线程A挂起：

![](https://youpaiyun.zongqilive.cn/image/20210514141649.png)

此时线程A的运行结果如下：

![](https://youpaiyun.zongqilive.cn/image/20210514141703.png)

此时线程B已获得CPU时间片，并完成resize操作：

![](https://youpaiyun.zongqilive.cn/image/20210514141717.png)

同样注意由于线程B执行完成，newTable和table都为最新值：5.next=null。

此时切换到线程A，在线程A挂起时：e=7，next=5，newTable[3]=null。

执行newtable[i]=e，就将7放在了table[3]的位置，此时next=5。接着进行下一次循环：

```
e=5
next=e.next ----> next=null，从主存中取值
e.next=newTable[1] ----> e.next=5，从主存中取值
newTable[1]=e ----> newTable[1]=5
e=next ----> e=null
```

将5放置在table[1]位置，此时e=null循环结束，3元素丢失，并形成环形链表。并在后续操作hashmap时造成死循环。

![](https://youpaiyun.zongqilive.cn/image/20210514141735.png)

#### 数据覆盖的情况(JDK8)

如果线程A和线程B同时进行put操作，刚好这两条不同的数据hash值一样，并且该位置数据为null，所以这线程A、B都会进入第6行代码中。

假设一种情况，线程A进入后还未进行数据插入时挂起，而线程B正常执行，从而正常插入数据，然后线程A获取CPU时间片，此时线程A不用再进行hash判断了，问题出现：线程A会把线程B插入的数据给覆盖，发生线程不安全。





# JUC并发

## 进程与线程的区别

区别

1. 进程是资源分配最小单位，线程是程序执行的最小单位；
2. 进程有自己独立的地址空间，每启动一个进程，系统都会为其分配地址空间，建立数据表来维护代码段、堆栈段和数据段，线程没有独立的地址空间，它使用相同的地址空间共享数据；
3. CPU切换一个线程比切换进程花费小；
4. 创建一个线程比进程开销小；
5. 线程占用的资源要⽐进程少很多。
6. 线程之间通信更方便，同一个进程下，线程共享全局变量，静态变量等数据，进程之间的通信需要以通信的方式（IPC）进行；（但多线程程序处理好同步与互斥是个难点）
7. 多进程程序更安全，生命力更强，一个进程死掉不会对另一个进程造成影响（源于有独立的地址空间），多线程程序更不易维护，一个线程死掉，整个进程就死掉了（因为共享地址空间）；
8. 进程对资源保护要求高，开销大，效率相对较低，线程资源保护要求不高，但开销小，效率高，可频繁切换；



**加强理解，做个简单的比喻：进程=火车，线程=车厢**

- 线程在进程下行进（单纯的车厢无法运行）
- 一个进程可以包含多个线程（一辆火车可以有多个车厢）
- 不同进程间数据很难共享（一辆火车上的乘客很难换到另外一辆火车，比如站点换乘）
- 同一进程下不同线程间数据很易共享（A车厢换到B车厢很容易）
- 进程要比线程消耗更多的计算机资源（采用多列火车相比多个车厢更耗资源）
- 进程间不会相互影响，一个线程挂掉将导致整个进程挂掉（一列火车不会影响到另外一列火车，但是如果一列火车上中间的一节车厢着火了，将影响到所有车厢）
- 进程可以拓展到多机，进程最多适合多核（不同火车可以开在多个轨道上，同一火车的车厢不能在行进的不同的轨道上）
- 进程使用的内存地址可以上锁，即一个线程使用某些共享内存时，其他线程必须等它结束，才能使用这一块内存。（比如火车上的洗手间）－"互斥锁"
- 进程使用的内存地址可以限定使用量（比如火车上的餐厅，最多只允许多少人进入，如果满了需要在门口等，等有人出来了才能进去）－“信号量”

## 创建线程的方式

有三种方式可以用来创建线程：

- 继承Thread类
- 实现Runnable接口
- 应用程序可以使用Executor框架来创建线程池

## 线程的状态

**新建( new )：**新创建了一个线程对象；

**可运行( runnable )：**线程对象创建后，其他线程(比如 main 线程）调用了该对象的 start ()方法。该状态的线程位于可运行线程池中，等待被线程调度选中，获 取CPU的使用权；

**运行( running )：**可运行状态( runnable )的线程获得了CPU时间片（ timeslice ） ，执行程序代码；

**阻塞( block )：**阻塞状态是指线程因为某种原因放弃了CPU 使用权，也即让出了 CPU timeslice ，暂时停止运行。直到线程进入可运行( runnable )状态，才有 机会再次获得 cpu timeslice 转到运行( running )状态。

阻塞的情况分三种：

1. 等待阻塞：运行( running )的线程执行 o . wait ()方法， JVM 会把该线程放 入等待队列( waitting queue )中。
2. 同步阻塞：运行( running )的线程在获取对象的同步锁时，若该同步锁被别的线程占用，则 JVM 会把该线程放入锁池( lock pool )中。
3. 其他阻塞: 运行( running )的线程执行 Thread . sleep ( long ms )或 t . join ()方法，或者发出了 I / O 请求时， JVM 会把该线程置为阻塞状态。当 sleep ()状态超时、 join ()等待线程终止或者超时、或者 I / O 处理完毕时，线程重新转入可运行( runnable )状态。

**死亡( dead )：**线程 run ()、 main () 方法执行结束，或者因异常退出了 run ()方法，则该线程结束生命周期。死亡的线程不可再次复生。



## Lock和synchronized区别







# wait()方法与sleep()方法的区别

# equals() 与 == 的区别是什么？

# hashCode() 和 equals() 之间有什么联系？

![图片](https://uploader.shimo.im/f/Sa5xLPGX650VFnZo.png!thumbnail?fileGuid=dCCdkkCHqCJKkpxT)

原则

1.同一个对象（没有发生过修改）无论何时调用hashCode()得到的返回值必须一样。

如果一个key对象在put的时候调用hashCode()决定了存放的位置，而在get的时候调用hashCode()得到了不一样的返回值，这个值映射到了一个和原来不一样的地方，那么肯定就找不到原来那个键值对了。

2.hashCode()的返回值相等的对象不一定相等，通过hashCode()和equals()必须能唯一确定一个对象。不相等的对象的hashCode()的结果可以相等。hashCode()在注意关注碰撞问题的时候，也要关注生成速度问题，完美hash不现实。

3.一旦重写了equals()函数（重写equals的时候还要注意要满足自反性、对称性、传递性、一致性），就必须重写hashCode()函数。而且hashCode()的生成哈希值的依据应该是equals()中用来比较是否相等的字段。

如果两个由equals()规定相等的对象生成的hashCode不等，对于hashMap来说，他们很可能分别映射到不同位置，没有调用equals()比较是否相等的机会，两个实际上相等的对象可能被插入不同位置，出现错误。其他一些基于哈希方法的集合类可能也会有这个问题


# wait()方法与sleep()方法的区别

# 为什么重写了equals就必须重写hashCode



# 谈谈类加载机制



# AOP

* 实现原理
    * JDK动态代理 (需要接口)
    * CGLIB动态代理(无需接口, 通过继承)
* 执行顺序
* 不支持static方法(原因是什么)
    * 静态方法是类级别的，调用需要知道类信息，而类信息在编译器就已经知道了，并不支持在运行期的动态绑定。

面试官一问：是编译时期进行织入，还是运行期进行织入？

- 运行期，生成字节码，再加载到虚拟机中，JDK是利用反射原理，CGLIB使用了ASM原理。

面试官再问：初始化时期织入还是获取对象时织入？

- 初始化的时候，已经将目标对象进行代理，放入到spring 容器中

面试官再再问：spring AOP 默认使用jdk动态代理还是cglib？

- 要看条件，如果实现了接口的类，是使用jdk。如果没实现接口，就使用cglib。


# Spring，SpringMVC，SpringBoot，SpringCloud有什么区别和联系

Spring是一个轻量级的控制反转(IoC)和面向切面(AOP)的容器框架

Spring MVC是Spring的一个模块，一个web框架。通过Dispatcher Servlet, ModelAndView 和 View Resolver，开发web应用变得很容易。

Spring配置复杂，繁琐，所以推出了SpringBoot, SpringBoot简化了spring的配置流程, 约定优于配置,

SpringBoot框架相对于SpringMVC框架来说，更专注于开发微服务后台接口，不开发前端视图；

Spring Cloud构建于Spring Boot之上，是一个关注全局的服务治理框架。

# Spring框架中Bean的生命周期

```plain
1、实例化一个Bean－－也就是我们常说的new；
2、按照Spring上下文对实例化的Bean进行配置－－也就是IOC注入；
3、如果这个Bean已经实现了BeanNameAware接口，会调用它实现的setBeanName(String)方法，此处传递的就是Spring配置文件中Bean的id值
4、如果这个Bean已经实现了BeanFactoryAware接口，会调用它实现的setBeanFactory(setBeanFactory(BeanFactory)传递的是Spring工厂自身（可以用这个方式来获取其它Bean，只需在Spring配置文件中配置一个普通的Bean就可以）；
5、如果这个Bean已经实现了ApplicationContextAware接口，会调用setApplicationContext(ApplicationContext)方法，传入Spring上下文（同样这个方式也可以实现步骤4的内容，但比4更好，因为ApplicationContext是BeanFactory的子接口，有更多的实现方法）；
6、如果这个Bean关联了BeanPostProcessor接口，将会调用postProcessBeforeInitialization(Object obj, String s)方法，BeanPostProcessor经常被用作是Bean内容的更改，并且由于这个是在Bean初始化结束时调用那个的方法，也可以被应用于内存或缓存技术；
7、如果Bean在Spring配置文件中配置了init-method属性会自动调用其配置的初始化方法。
8、如果这个Bean关联了BeanPostProcessor接口，将会调用postProcessAfterInitialization(Object obj, String s)方法、；
注：以上工作完成以后就可以应用这个Bean了，那这个Bean是一个Singleton的，所以一般情况下我们调用同一个id的Bean会是在内容地址相同的实例，当然在Spring配置文件中也可以配置非Singleton，这里我们不做赘述。
9、当Bean不再需要时，会经过清理阶段，如果Bean实现了DisposableBean这个接口，会调用那个其实现的destroy()方法；
10、最后，如果这个Bean的Spring配置中配置了destroy-method属性，会自动调用其配置的销毁方法。
```
## Bean的调用

有三种方式可以得到Bean并进行调用：

1. 使用BeanWrapper
```java
HelloWorld hw=new HelloWorld();
BeanWrapper bw=new BeanWrapperImpl(hw);
bw.setPropertyvalue(”msg”,”HelloWorld”);
system.out.println(bw.getPropertyCalue(”msg”));
```
使用BeanFactory

```plain
InputStream is=new FileInputStream(”config.xml”);
XmlBeanFactory factory=new XmlBeanFactory(is);
HelloWorld hw=(HelloWorld) factory.getBean(”HelloWorld”);
system.out.println(hw.getMsg());
```

使用ApplicationConttext

```plain
ApplicationContext actx=new FleSystemXmlApplicationContext(”config.xml”);
HelloWorld hw=(HelloWorld) actx.getBean(”HelloWorld”);
System.out.println(hw.getMsg());
```
## 调用流程图

![图片](https://uploader.shimo.im/f/WTxGrqqQgxWv3vqb.png!thumbnail?fileGuid=dCCdkkCHqCJKkpxT)



# Mybatis

## #{}和${}的区别是什么























