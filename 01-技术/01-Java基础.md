[TOC]

https://www.bilibili.com/video/BV1wf4y1W7Hj

https://www.bilibili.com/video/BV17K4y1K7CX/

https://blog.csdn.net/weixin_43314519/article/details/112603595?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522162233937916780255224659%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=162233937916780255224659&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~blog~top_positive~default-1-112603595.nonecase&utm_term=%E9%9D%A2%E8%AF%95&spm=1018.2226.3001.4450





# JVM









## 对象创建的6个步骤

1. 类是否加载
2. 为对象分配内存
3. 初始化默认值
4. 设置对象头
5. 执行init方法,进行初始化
6. 建立变量的引用关系
7. 



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



# 锁

## 可重入锁



## 锁的公平与非公平

锁的公平与非公平，是指线程请求获取锁的过程中，是否允许插队。

在公平锁上，线程将按他们发出请求的顺序来获得锁；

非公平锁则允许在线程发出请求后立即尝试获取锁，如果可用则可直接获取锁，尝试失败才进行排队等待。



# synchronized

https://juejin.cn/post/6968999143964573703?utm_source=gold_browser_extension



## Lock和synchronized区别



# AQS

参考:

https://segmentfault.com/a/1190000015739343

https://segmentfault.com/a/1190000015752512

## 独占锁

==独占锁的节点释放锁时，才会唤醒后继节点==

>  获取独占锁，对中断不敏感。 
>
>  首先尝试获取一次锁，如果成功，则返回；
>
> 否则会把当前线程包装成Node插入到队列中，在队列中会检测是否为head的直接后继，并尝试获取锁,  如果获取失败，则会通过LockSupport阻塞当前线程，直至被释放锁的线程唤醒或者被中断，随后再次尝试获取锁，如此反复。 

### 状态

```java
// state为0表示锁没有被占用，
// state大于0表示当前已经有线程持有该锁
private volatile int state;
// 当前持有锁的线程
private transient Thread exclusiveOwnerThread;
```



### 双向链表队列

#### 节点定义

```java
// 节点所代表的线程
volatile Thread thread;

// 双向链表，每个节点需要保存自己的前驱节点和后继节点的引用
volatile Node prev;
volatile Node next;

// 线程所处的等待锁的状态，初始化时，该值为0
volatile int waitStatus;

/** 因为超时或者中断，节点会被设置成取消状态，被取消的节点不会参与到竞争中，会一直是取消
            状态不会改变 */
static final int CANCELLED =  1;
/** 后继节点处于等待状态，如果当前节点释放了同步状态或者被取消，会通知后继节点，使其得以
            运行 */
static final int SIGNAL    = -1;
/** 节点在等待条件队列中，节点线程等待在condition上，当其他线程对condition调用了signal
            后，该节点将会从等待队列中进入同步队列中，获取同步状态 */
static final int CONDITION = -2;
/***
下一次共享式同步状态获取会无条件的传播下去
         */
static final int PROPAGATE = -3;

// 该属性用于条件队列或者共享锁
Node nextWaiter;
```

#### waitStatus

`waitStatus`在独占锁模式下，我们只需要关注`CANCELLED` ,`SIGNAL`两种状态即可,默认值是0

CANCELLED(1):

表示Node所代表的当前线程已经取消了排队，即放弃获取锁了。

SIGNAL(-1):

它不是表征当前节点的状态，而是当前节点的下一个节点的状态。

后继结点入队时，会将前继结点的状态更新为SIGNAL, 

当一个节点的waitStatus被置为SIGNAL，就说明它的下一个节点（即它的后继节点）已经被挂起了（或者马上就要被挂起了），因此在当前节点释放了锁或者放弃获取锁时，如果它的waitStatus属性为SIGNAL，它还要完成一个额外的操作——唤醒它的后继节点。

#### 头节点和尾节点

```java
// 头结点，不代表任何线程，是一个哑结点
private transient volatile Node head;

// 尾节点，每一个请求锁的线程会加到队尾
private transient volatile Node tail;
```

head节点不代表任何线程，它就是一个空节点！

![](https://youpaiyun.zongqilive.cn/image/20210530180228.png)

![](https://youpaiyun.zongqilive.cn/image/20210530180252.png)

#### 尾分叉

将一个节点node添加到队列的末尾需要三步:

1. 设置node的前驱节点为当前的尾节点：`node.prev = t`
2. 修改`tail`属性，使它指向当前节点
3. 修改原来的尾节点，使它的next指向当前节点

```java
// 到这里说明队列已经不是空的了, 这个时候再继续尝试将节点加到队尾
node.prev = t;
if (compareAndSetTail(t, node)) {
  t.next = node;
  return t;
}
```

![](https://youpaiyun.zongqilive.cn/image/20210530201347.png)

这里的三步并不是一个原子操作，第一步很容易成功；

而第二步由于是一个CAS操作，在并发条件下有可能失败，第三步只有在第二步成功的条件下才执行。

这里的CAS保证了同一时刻只有一个节点能成为尾节点，其他节点将失败，失败后将回到for循环中继续重试。

所以，当有大量的线程在同时入队的时候，同一时刻，只有一个线程能完整地完成这三步，**而其他线程只能完成第一步**，于是就出现了尾分叉：

![](https://youpaiyun.zongqilive.cn/image/20210530201506.png)

注意，这里第三步是在第二步执行成功后才执行的，这就意味着，有可能即使我们已经完成了第二步，将新的节点设置成了尾节点，**此时原来旧的尾节点的next值可能还是`null`**(因为还没有来的及执行第三步)，所以如果此时有线程恰巧从头节点开始向后遍历整个链表，则它是遍历不到新加进来的尾节点的，但是这显然是不合理的，因为现在的tail已经指向了新的尾节点。

所以如果我们从尾节点开始向前遍历，已经可以遍历到所有的节点。

这也就是为什么我们在AQS相关的源码中，有时候常常会出现==从尾节点开始逆向遍历链表==——因为一个节点要能入队，则它的prev属性一定是有值的，但是它的next属性可能暂时还没有值。

至于那些“分叉”的入队失败的其他节点，在下一轮的循环中，它们的prev属性会重新指向新的尾节点，继续尝试新的CAS操作，最终，所有节点都会通过自旋不断的尝试入队，直到成功为止

### CAS操作

- AQS的3个属性state,head和tail
- Node对象的2个属性waitStatus,next



### 加锁流程

加锁:

```java
public final void acquire(int arg) {
  // 1. 获取锁(也就是改变state的值)
  // 2. 做队列的一些事情
  if (!tryAcquire(arg) &&
      acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
     //并不知道线程被唤醒的原因, 如果发现当前线程曾经被中断过，那我们就把当前线程再中断一次
    // 线程在等待资源的过程中被中断唤醒，它还是会不依不饶的再抢锁，直到它抢到锁为止。也就是说，它是不响应这个中断的，仅仅是记录下自己被人中断过
    selfInterrupt();
}
```

```java
// 1. 修改state的值
protected final boolean tryAcquire(int acquires) {
  final Thread current = Thread.currentThread();
   // 首先获取当前锁的状态
  int c = getState();
  if (c == 0) {
    // 当前非公平锁, acquires =1
    // 利用CAS 改变状态
    if (compareAndSetState(0, acquires)) {
      // 获取锁, 设置当前线程
      setExclusiveOwnerThread(current);
      return true;
    }
  }
  else if (current == getExclusiveOwnerThread()) {
    // 重入锁
    // state+1
    int nextc = c + acquires;
    if (nextc < 0) // overflow
      throw new Error("Maximum lock count exceeded");
    setState(nextc);
    return true;
  }
  return false;
}
```

```java
// 2.队列的一些操作
private Node addWaiter(Node mode) {
  // mode值为Node.EXCLUSIVE，所以节点的nextWaiter属性被设为null
  // 每一个处于独占锁模式下的节点，它的nextWaiter一定是null。
  Node node = new Node(Thread.currentThread(), mode);
  // Try the fast path of enq; backup to full enq on failure
  Node pred = tail;
  // 如果队列不为空
  if (pred != null) {
    // 节点入队3步走, 就会出现尾分叉现象, (唤醒线程时需要逆序遍历队列)
    node.prev = pred;
    // 放在下面if的里面，会导致一个瞬间tail.prev = null，这样会使得队列不完整。
    // 用CAS方式将当前节点设为尾节点
    if (compareAndSetTail(pred, node)) {
      pred.next = node;
      return node;
    }
  }
  // 代码会执行到这里, 只有两种情况:
  //    1. 队列为空
  //    2. CAS失败
  // 注意, 这里是并发条件下, 所以什么都有可能发生, 尤其注意CAS失败后也会来到这里
  enq(node); //将节点插入队列
  return node;
}


private Node enq(final Node node) {
  // 通过自旋+CAS的方式，确保当前节点入队。
  for (;;) {
    Node t = tail;
    if (t == null) {
      // 队列为空, 进行初始化
      // 队列不是在构造的时候初始化的, 而是延迟到需要用的时候再初始化, 以提升性能
      // 注意，初始化时使用new Node()方法新建了一个dummy节点
      if (compareAndSetHead(new Node()))
        tail = head;
    } else {
      // 队列不为空, 这个时候利用CAS将节点加到队尾
      // 节点入队3步走, 就会出现尾分叉现象, (唤醒线程时需要逆序遍历队列)
      node.prev = t;
      // 放在下面if的里面，会导致一个瞬间tail.prev = null，这样会使得队列不完整。
      // 用CAS方式将当前节点设为尾节点
      if (compareAndSetTail(t, node)) {
        t.next = node;
        return t;
      }
    }
  }
}
```



```java
//(1) 能执行到该方法, 说明addWaiter 方法已经成功将包装了当前Thread的节点添加到了等待队列的队尾
//(2) 该方法中将再次尝试去获取锁
//(3) 在再次尝试获取锁失败后, 判断是否需要把当前线程挂起

// 在队列中的节点通过此方法获取锁，对中断不敏感。
final boolean acquireQueued(final Node node, int arg) {
  boolean failed = true;
  try {
    boolean interrupted = false;
    for (;;) {
      final Node p = node.predecessor();
      // 检测当前节点前驱是否head，这是试获取锁的资格。
      // 如果是的话，则调用tryAcquire尝试获取锁,
      // 成功，则将head置为当前节点。
      if (p == head && tryAcquire(arg)) {
        // 将当前节点, 变成了新的head节点
        // 某种程度上就是将当前线程从等待队列里面拿出来了，是一个变相的出队操作。
        setHead(node);
        /**
        private void setHead(Node node) {
    						head = node;
    						node.thread = null;
   							node.prev = null;
				}
        **/
        p.next = null; // help GC
        failed = false;
        return interrupted;
      }
      // 获取锁失败, 则根据前驱节点判断是否要将当前线程挂起
      if (shouldParkAfterFailedAcquire(p, node) &&
          parkAndCheckInterrupt())
        // 唤醒之后若是中断状态, 仍然需要去获取锁
        interrupted = true;
    }
  } finally {
    if (failed)
      cancelAcquire(node);
  }
}

static final int CANCELLED =  1;
static final int SIGNAL    = -1;
static final int CONDITION = -2;
static final int PROPAGATE = -3;
// 根据前驱节点中的waitStatus值来判断是否需要阻塞当前线程。
private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
  int ws = pred.waitStatus;
  if (ws == Node.SIGNAL)
    // 前驱节点 是SIGNAL状态，在释放锁的时候会唤醒后继节点，
    // 所以后继节点（也就是当前节点）现在可以阻塞自己。
    return true;
  if (ws > 0) {
    // 前驱节点状态为取消,向前遍历
    // 当前线程会之后会再次回到循环并尝试获取锁
    do {
      node.prev = pred = pred.prev;
    } while (pred.waitStatus > 0);
    pred.next = node;
  } else {
    // 前驱节点的状态既不是SIGNAL，也不是CANCELLED
    // 用CAS设置前驱节点的ws为 Node.SIGNAL，给自己定一个闹钟
    // 并且之后会回到循环再次重试获取锁。
    compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
  }
  return false;
}

// 挂起线程
private final boolean parkAndCheckInterrupt() {
  LockSupport.park(this);// 线程被挂起，停在这里不再往下执行了
  return Thread.interrupted(); // 返回线程的中断状态
}
```







![](https://youpaiyun.zongqilive.cn/image/AQS1.png)

![](https://youpaiyun.zongqilive.cn/image/20210606162933.png)





### 锁释放流程

```java
// 在成功释放锁之后,唤醒后继节点只是一个"附加操作",无论该操作结果怎样,最后release操作都会返回true
public final boolean release(int arg) {
  if (tryRelease(arg)) {
    /*
         * 此时的head节点可能有3种情况:
         * 1. null (AQS的head延迟初始化+无竞争的情况)
         * 2. 当前线程在获取锁时new出来的节点通过setHead设置的
         * 3. 由于通过tryRelease已经完全释放掉了独占锁，有新的节点在acquireQueued中获取到了独占锁，并设置了head

         * 第三种情况可以再分为两种情况：
         * （一）时刻1:线程A通过acquireQueued，持锁成功，set了head
         *          时刻2:线程B通过tryAcquire试图获取独占锁失败失败，进入acquiredQueued
         *          时刻3:线程A通过tryRelease释放了独占锁
         *          时刻4:线程B通过acquireQueued中的tryAcquire获取到了独占锁并调用setHead
         *          时刻5:线程A读到了此时的head实际上是线程B对应的node
         * （二）时刻1:线程A通过tryAcquire直接持锁成功，head为null
         *          时刻2:线程B通过tryAcquire试图获取独占锁失败失败，入队过程中初始化了head，进入acquiredQueued
         *          时刻3:线程A通过tryRelease释放了独占锁，此时线程B还未开始tryAcquire
         *          时刻4:线程A读到了此时的head实际上是线程B初始化出来的傀儡head
         */
    Node h = head;
    // head节点状态不会是CANCELLED，所以这里h.waitStatus != 0相当于h.waitStatus < 0
    if (h != null && h.waitStatus != 0)
      // 唤醒后继线程
      unparkSuccessor(h);
    return true;
  }
  return false;
}
```



```java
// 释放锁
protected final boolean tryRelease(int releases) {
  // 首先将当前持有锁的线程个数减1(回溯到调用源头sync.release(1)可知, releases的值为1)
  // 这里的操作主要是针对可重入锁的情况下, c可能大于1
  int c = getState() - releases; 

  // 释放锁的线程当前必须是持有锁的线程
  if (Thread.currentThread() != getExclusiveOwnerThread())
    throw new IllegalMonitorStateException();

  // 如果c为0了, 说明锁已经完全释放了
  boolean free = false;
  if (c == 0) {
    free = true;
    setExclusiveOwnerThread(null);
  }
  setState(c);
  return free;
}

// 唤醒后继节点
private void unparkSuccessor(Node node) {
  int ws = node.waitStatus;
  // 如果head节点的ws比0小, 则直接将它设为0
  if (ws < 0)
    compareAndSetWaitStatus(node, ws, 0);

  // 要唤醒的节点就是自己的后继节点
  // 如果后继节点存在且也在等待锁, 那就直接唤醒它
  Node s = node.next;
  if (s == null || s.waitStatus > 0) {
    // 1.后继节点不存在
    // 2.后继节点取消等待锁
    s = null;
    // 此时从尾节点开始向前找起, 直到找到距离head节点最近的ws<=0的节点
    for (Node t = tail; t != null && t != node; t = t.prev)
      /**
      为什么从tail向前遍历??? 尾分叉现场
      
      node.prev = pred; //step 1, 设置前驱节点
        if (compareAndSetTail(pred, node)) { // step2, 将当前节点设置成新的尾节点
            pred.next = node; // step 3, 将前驱节点的next属性指向自己
            return node;
        }

      如果读到s == null，不代表node就为tail。
      考虑如下场景：
      1.node某时刻为tail
      2.有新线程通过addWaiter中的if分支或者enq方法添加自己
      3.compareAndSetTail成功
      4.此时这里的Node s = node.next读出来s == null，但事实上node已经不是tail，它有后继了!
      **/
      if (t.waitStatus <= 0)
        s = t; // 注意! 这里找到了之并没有停止, 而是继续向前找
  }
  // 如果找到了还在等待锁的节点,则唤醒它
  if (s != null)
    LockSupport.unpark(s.thread);
}
```





















## 共享锁

共享锁，则允许多个线程同时获取锁，并发访问 共享资源，如：ReadWriteLock

共享锁则是一种乐观锁，它放宽了加锁策略，允许多个执行读操作的线程同时访问共享资源。 java的并发包中提供了ReadWriteLock，读-写锁。它允许一个资源可以被多个读操作访问，或者被一个 写操作访问，但两者不能同时进行。

==在共享锁模式下，在获取锁和释放锁结束时，都会唤醒后继节点。==











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























