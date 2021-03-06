

https://www.bilibili.com/video/BV1wf4y1W7Hj

https://www.bilibili.com/video/BV17K4y1K7CX/

https://blog.csdn.net/weixin_43314519/article/details/112603595?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522162233937916780255224659%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=162233937916780255224659&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~blog~top_positive~default-1-112603595.nonecase&utm_term=%E9%9D%A2%E8%AF%95&spm=1018.2226.3001.4450



# 对象创建的6个步骤

1. 类是否加载
2. 为对象分配内存
3. 初始化默认值
4. 设置对象头
5. 执行init方法,进行初始化
6. 建立变量的引用关系
7. 



# JVM

## JVM组成

> 注意方法区的变化, 为什么方法区要做出改变?
>
> 移到了本地内存中, 方法区就不受 JVM 的控制了，这个区域也就不会进行 GC，也因此提升了性能，正因为放到了本地内存，也就不存在由于永久代限制大小而导致的 OOM 异常了。

![](https://youpaiyun.zongqilive.cn/image/20210115161657.png)

![](https://youpaiyun.zongqilive.cn/image/20210115161718.png)



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

## 并发和并行

### 并发

> 一个处理器同时处理多个任务(某一时刻只有一个任务在处理)

![](https://youpaiyun.zongqilive.cn/image/20200605150841.png)



### 并行

![](https://youpaiyun.zongqilive.cn/image/20200605150848.png)

![](https://youpaiyun.zongqilive.cn/image/20200605150855.png)



## 进程与线程

区别

1. 进程是资源分配最小单位，线程是程序执行的最小单位；
2. 进程有自己独立的地址空间，每启动一个进程，系统都会为其分配地址空间，建立数据表来维护代码段、堆栈段和数据段，线程没有独立的地址空间，它使用相同的地址空间共享数据；
3. CPU切换一个线程比切换进程花费小；
6. 线程之间通信更方便，同一个进程下，线程共享全局变量，静态变量等数据，进程之间的通信需要以通信的方式（IPC）进行；（但多线程程序处理好同步与互斥是个难点）
7. 多进程程序更安全，生命力更强，一个进程死掉不会对另一个进程造成影响（源于有独立的地址空间），多线程程序更不易维护，一个线程死掉，整个进程就死掉了（因为共享地址空间）

## Java默认有几个线程

2 个,   mian和GC

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

![](https://youpaiyun.zongqilive.cn/image/20210127094440.png)



## 可重入锁



## 锁的公平与非公平

锁的公平与非公平，是指线程请求获取锁的过程中，是否允许插队。

在公平锁上，线程将按他们发出请求的顺序来获得锁；

非公平锁则允许在线程发出请求后立即尝试获取锁，如果可用则可直接获取锁，尝试失败才进行排队等待。





# JUC组成

![](https://youpaiyun.zongqilive.cn/image/20210306174450.png)



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



### 加锁流程源码分析

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
        // 某种意义上, 头节点就是持有独占锁的节点，
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

### 尾分叉

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

### 独占锁加锁流程图

![](https://youpaiyun.zongqilive.cn/image/AQS1.png)

![](https://youpaiyun.zongqilive.cn/image/20210606162933.png)





### 锁释放流程源码分析

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
      为什么从tail向前遍历??? 尾分叉现象
      
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

在共享锁模式下，当一个节点获取到了共享锁，我们在获取成功后就可以唤醒后继节点了，而不需要等到该节点释放锁的时候，这是因为共享锁可以被多个线程同时持有，一个锁获取到了，则后继的节点都可以直接来获取。

因此，**在共享锁模式下，在获取锁和释放锁结束时，都会唤醒后继节点**

### 加锁源码分析

```java
public final void acquireShared(long arg) {
  /*
  如果该值 < 0，则代表当前线程获取共享锁失败
如果该值 > 0，则代表当前线程获取共享锁成功，并且接下来其他线程尝试获取共享锁的行为很可能成功
如果该值 = 0，则代表当前线程获取共享锁成功，但是接下来其他线程尝试获取共享锁的行为会失败
  */
  if (tryAcquireShared(arg) < 0)
    doAcquireShared(arg);
}


private void doAcquireShared(long arg) {
  // 共享的节点
  final Node node = addWaiter(Node.SHARED);
  boolean failed = true;
  try {
    boolean interrupted = false;
    for (;;) {
      final Node p = node.predecessor();
      if (p == head) {
        long r = tryAcquireShared(arg);
        if (r >= 0) { // 获取到锁
          // 设置head, 里边还调用doReleaseShared,唤醒后继的节点(不必等待锁释放)
          setHeadAndPropagate(node, r);
          p.next = null; // help GC
          if (interrupted)
            selfInterrupt();
          failed = false;
          return;
        }
      }
      if (shouldParkAfterFailedAcquire(p, node) &&
          parkAndCheckInterrupt())
        interrupted = true;
    }
  } finally {
    if (failed)
      cancelAcquire(node);
  }
}

private void setHeadAndPropagate(Node node, long propagate) {
  Node h = head;
  // 设置head, 变相出队
  setHead(node);
  if (propagate > 0 || h == null || h.waitStatus < 0 ||
      (h = head) == null || h.waitStatus < 0) {
    Node s = node.next;
    if (s == null || s.isShared())
      doReleaseShared();
  }
}

```



### 释放锁源码分析

```java
public final boolean releaseShared(int arg) {
  if (tryReleaseShared(arg)) {
    doReleaseShared();
    return true;
  }
  return false;
}
```

### doReleaseShared

方法有两处调用，

一处在`acquireShared`方法的末尾，当线程成功获取到共享锁后，在一定条件下调用该方法；

一处在`releaseShared`方法中，当线程释放共享锁的时候调用。(线程想要获得共享锁，则它们必然**曾经成为过头节点，或者就是现在的头节点**。`releaseShared`方法中调用的`doReleaseShared`，可能此时调用方法的线程已经不是头节点所代表的线程了，头节点可能已经被易主好几次了。)

```java
private void doReleaseShared() {
  for (;;) {
    Node h = head;
    if (h != null && h != tail) {
      int ws = h.waitStatus;
      if (ws == Node.SIGNAL) {
        if (!compareAndSetWaitStatus(h, Node.SIGNAL, 0)) {
         continue; 
        }
        // 唤醒后继节点 
        unparkSuccessor(h);
      }
      else if (ws == 0 &&
               !compareAndSetWaitStatus(h, 0, Node.PROPAGATE))
        continue;                // loop on failed CAS
    }
    // 头节点可能已经发生变化, 发生 调用风暴, 
    // 大量的线程在同时执行doReleaseShared，这极大地加速了唤醒后继节点的速度，提升了效率，
    // 同时doReleaseShared方法内部的CAS操作又保证了多个线程同时唤醒一个节点时，只有一个线程能操作成功。
    if (h == head) 
      break;
  }
}

调用风暴是怎么结束的呢？
1. 自己是头节点可以结束
2. 发现头节点后面的节点是写锁 ， 这个时候如果直接退出 有可能这个时候读锁已经计数为0了；
  先判断下这个时候读锁计数是否为0. 如果为0 就唤醒后面的写锁
  如果这个时候读锁计数不为0 ， 还会不会唤醒写锁？
  解决这个问题方式：
  因为写锁不是自己的下一个锁， 不唤醒。 写锁自有他的前一个节点唤醒, 甚至可以不用判断写锁计数， 直接结束。
```



# 线程池

## 构造参数

ThreadPoolExecutor参数最全的构造方法：

![图片](https://youpaiyun.zongqilive.cn/image/640.png)

- **corePoolSize：**线程池的核心线程数，说白了就是，即便是线程池里没有任何任务，也会有corePoolSize个线程在候着等任务。
- **maximumPoolSize：**最大线程数，不管你提交多少任务，线程池里最多工作线程数就是maximumPoolSize。
- **keepAliveTime：**非核心线程的存活时间。当线程池里的线程数大于corePoolSize时，如果等了keepAliveTime时长还没有任务可执行，则线程退出。
- **unit：**这个用来指定keepAliveTime的单位，比如秒:TimeUnit.SECONDS。
- **workQueue：**一个阻塞队列，提交的任务将会被放到这个队列里。
- **threadFactory：**线程工厂，用来创建线程，主要是为了给线程起名字，默认工厂的线程名字：pool-1-thread-3。
- **handler：**拒绝策略，当线程池里线程被耗尽，且队列也满了的时候会调用。

![](https://youpaiyun.zongqilive.cn/image/20210124145917.png)

## 线程回收利用CAS方式

```java
// Are workers subject to culling?
boolean timed = allowCoreThreadTimeOut || wc > corePoolSize;
if ((wc > maximumPoolSize || (timed && timedOut))
    && (wc > 1 || workQueue.isEmpty())) {
    if (compareAndDecrementWorkerCount(c))
        return null;
    continue;
}

// cas的方式 worker数量减一，返回null，之后会进行Worker回收工作
private boolean compareAndDecrementWorkerCount(int expect) {
    return ctl.compareAndSet(expect, expect - 1);
}
```



## 线程数的设定



# 对象的内存布局

## 对象头

![](https://youpaiyun.zongqilive.cn/image/20200708170053.png)

- ==**对象头**：其主要包括两部分数据：`Mark Word、Class对象指针。==
  - `Class Point`(类型指针)：是对象指向它的类元数据的指针，虚拟机通过这个指针来确定这个对象是哪个类的实例。
  - `Mark Word`(标记字段)：这一部分用于储存对象自身的运行时数据，如`哈希码`，`GC`分代年龄，`锁状态标志`，`锁指针`等
- ==特别地对于数组对象而言，其还包括了数组长度数据。==

### Mark Word

![64位图](https://youpaiyun.zongqilive.cn/image/20200708170208.png)



## 实例数据



## 对齐填充



# synchronized

锁信息存储在对象头的`Mark Word`中, 锁信息是一个指针，它指向一个monitor对象的起始地址.

![](https://youpaiyun.zongqilive.cn/image/20210620141850.png)

上图所示: 图片的最左边是线程的调用栈，它引用了堆中的一个对象，该对象的对象头部分记录了该对象所使用的监视器锁，该监视器锁指向了一个monitor对象

## monitor对象是什么呢

monitor是由ObjectMonitor实现的，其主要数据结构如下:

```java
ObjectMonitor() {
  // 重点关注下边几个字段:
  _owner        = NULL; // 当前拥有该 ObjectMonitor 的线程
  _WaitSet      = NULL; // 调用了Object.wait()方法而进入等待状态的线程的集合
  _EntryList    = NULL ; // 当前等待锁的集合

  _recursions   = 0; // 锁的重入次数
  _count        = 0; // 用来记录该线程获取锁的次数


  _header       = NULL;
  _waiters      = 0,
  _object       = NULL;
  _WaitSetLock  = 0 ;
  _Responsible  = NULL ;
  _succ         = NULL ;
  _cxq          = NULL ;
  FreeNext      = NULL ;
  _SpinFreq     = 0 ;
  _SpinClock    = 0 ;
  OwnerIsThread = 0 ;
  _previous_owner_tid = 0;
}
```

每一个等待锁的线程都会被封装成ObjectWaiter对象，当多个线程同时访问一段同步代码时，

1. 首先会被扔进 _EntryList 集合中，
2. 如果其中的某个线程获得了monitor对象，他将成为` _owner`，同时计数器`_count`加1;
3. 如果在它成为` _owner`之后又调用了`wait`方法，则他将释放获得的monitor对象，`_owner`变量恢复为`null`，`_count`自减1，进入 _WaitSet集合中等待被唤醒。

![](https://youpaiyun.zongqilive.cn/image/20210620142256.png)

<img src="https://youpaiyun.zongqilive.cn/image/20210205154700.png"/>

## 获得锁源码分析

![](https://youpaiyun.zongqilive.cn/image/20210620144226.png)

```java
void ATTR ObjectMonitor::enter(TRAPS) {
  Thread * const Self = THREAD ;
  void * cur ;
  //通过CAS尝试把monitor的`_owner`字段设置为当前线程
  cur = Atomic::cmpxchg_ptr (Self, &_owner, NULL) ;
  //获取锁失败
  if (cur == NULL) {         assert (_recursions == 0   , "invariant") ;
                    assert (_owner      == Self, "invariant") ;
                    // CONSIDER: set or assert OwnerIsThread == 1
                    return ;
                   }
  // 如果旧值和当前线程一样，说明当前线程已经持有锁，此次为重入，_recursions自增，并获得锁。
  if (cur == Self) { 
    // TODO-FIXME: check for integer overflow!  BUGID 6557169.
    _recursions ++ ;
    return ;
  }

  // 如果当前线程是第一次进入该monitor，设置_recursions为1，_owner为当前线程
  if (Self->is_lock_owned ((address)cur)) { 
    assert (_recursions == 0, "internal state error");
    _recursions = 1 ;
    // Commute owner from a thread-specific on-stack BasicLockObject address to
    // a full-fledged "Thread *".
    _owner = Self ;
    OwnerIsThread = 1 ;
    return ;
  }

  // 省略部分代码。
  // 通过自旋执行ObjectMonitor::EnterI方法等待锁的释放
  for (;;) {
    jt->set_suspend_equivalent();
    // cleared by handle_special_suspend_equivalent_condition()
    // or java_suspend_self()

    EnterI (THREAD) ;

    if (!ExitSuspendEquivalent(jt)) break ;

    _recursions = 0 ;
    _succ = NULL ;
    exit (Self) ;

    jt->java_suspend_self();
  }
}
```



## 释放锁源码

![](https://youpaiyun.zongqilive.cn/image/20210620144626.png)

```java
void ATTR ObjectMonitor::exit(TRAPS) {
  Thread * Self = THREAD ;
  //如果当前线程不是Monitor的所有者
  if (THREAD != _owner) { 
    if (THREAD->is_lock_owned((address) _owner)) { // 
      // Transmute _owner from a BasicLock pointer to a Thread address.
      // We don't need to hold _mutex for this transition.
      // Non-null to Non-null is safe as long as all readers can
      // tolerate either flavor.
      assert (_recursions == 0, "invariant") ;
      _owner = THREAD ;
      _recursions = 0 ;
      OwnerIsThread = 1 ;
    } else {
      // NOTE: we need to handle unbalanced monitor enter/exit
      // in native code by throwing an exception.
      // TODO: Throw an IllegalMonitorStateException ?
      TEVENT (Exit - Throw IMSX) ;
      assert(false, "Non-balanced monitor enter/exit!");
      if (false) {
        THROW(vmSymbols::java_lang_IllegalMonitorStateException());
      }
      return;
    }
  }
  // 如果_recursions次数不为0.自减
  if (_recursions != 0) {
    _recursions--;        // this is simple recursive enter
    TEVENT (Inflated exit - recursive) ;
    return ;
  }

  //省略部分代码，根据不同的策略（由QMode指定），从cxq或EntryList中获取头节点，通过ObjectMonitor::ExitEpilog方法唤醒该节点封装的线程，唤醒操作最终由unpark完成。
```

## 小总结

`sychronized`加锁的时候，会调用objectMonitor的`enter`方法，解锁的时候会调用`exit`方法

`sychronized`是可重入锁

# sychronized优化

## 锁消除

```java
// 就是把不必要的同步在编译阶段进行移除
// 原理- 逃逸分析
// 逃逸分析: 就是变量不会外泄
public Demo {
  int x;
  public void locked() {
  synchronized(new Object) {
    	x++;
  	}
	}
}
```

## 适应性自旋锁

所谓自适应就意味着==自旋的次数不再是固定的==，它是由前一次在同一个锁上的自旋时间及锁的拥有者的状态来决定。

## 锁粗化

在一段代码中连续的用同一个监视器锁反复的加锁解锁，甚至加锁操作出现在循环体中的时候，就会导致不必要的性能损耗，这种情况就需要锁粗化。

把同步的区域扩大，尽量避免不必要的加解锁操作。

```java
```java
for(int i=0;i<100000;i++){
    synchronized(this){
        do();
}
​```

会被粗化成：

​```java
synchronized(this){
    for(int i=0;i<100000;i++){
        do();
}
​```
```

## 锁升级

![](https://youpaiyun.zongqilive.cn/image/20200709192414.png)

![](https://youpaiyun.zongqilive.cn/image/20200710165501.png)



### 偏向锁

>  只有一个线程执行同步代码块

获取偏向锁流程:

1. 首先线程访问同步代码块，会通过检查对象头 Mark Word 的`锁标志位`判断目前锁的状态，如果是 01，说明就是无锁或者偏向锁，然后再根据`是否偏向锁` 的标示判断是无锁还是偏向锁，如果是无锁情况下，执行下一步

2. 线程使用 CAS 操作来尝试对对象加锁，如果使用 CAS 替换 ThreadID 成功，就说明是第一次上锁，那么当前线程就会获得对象的偏向锁，此时会在对象头的 Mark Word 中记录当前线程 ID 和获取锁的时间 epoch 等信息，然后执行同步代码块。

等到下一次线程在进入和退出同步代码块时就不需要进行 `CAS` 操作进行加锁和解锁，只需要简单判断一下对象头的 Mark Word 中是否存储着指向当前线程的线程ID

![](https://youpaiyun.zongqilive.cn/image/20210622110437.png)

### 轻量级锁

`轻量级锁`是指当前锁是偏向锁的时候，资源被另外的线程所访问，那么偏向锁就会升级为`轻量级锁`，其他线程会通过`自旋`的形式尝试获取锁，不会阻塞，从而提高性能，

适应的场景是线程交替执行同步块的场合，如果存在同一时间访问同一锁的场合，就会导致轻量级锁就会失效，进而膨胀为重量级锁。

JVM会利用CAS尝试把对象原本的Mark Word 更新为Lock Record的指针，成功就说明加锁成功，改变锁标志位为00，然后执行相关同步操作。

轻量级锁加锁流程:

1. 紧接着上一步，如果 CAS 操作替换 ThreadID 没有获取成功，执行下一步
2. 如果使用 CAS 操作替换 ThreadID 失败（这时候就切换到另外一个线程的角度）说明该资源已被同步访问过，这时候就会执行锁的撤销操作，撤销偏向锁，然后等原持有偏向锁的线程到达`全局安全点（SafePoint）`时，会暂停原持有偏向锁的线程，然后会检查原持有偏向锁的状态，如果已经退出同步，就会唤醒持有偏向锁的线程，执行下一步
3. 检查对象头中的 Mark Word 记录的是否是当前线程 ID，如果是，执行同步代码，如果不是，执行**偏向锁获取流程** 的第2步。

![](https://youpaiyun.zongqilive.cn/image/20210622103454.png)



### 重量级锁

要阻塞或唤醒一个线程, 需要从用户态转换到核心态,切换成本非常高



![](https://youpaiyun.zongqilive.cn/image/20210622113018.png)

### 小总结

![](https://youpaiyun.zongqilive.cn/image/20200712102254.png)



### 锁的整个升级过程

(1）当没有被当成锁时，这就是一个普通的对象，Mark Word记录对象的HashCode，锁标志位是01，是否偏向锁那一位是0;

(2）当对象被当做同步锁并有一个线程A抢到了锁时，锁标志位还是01，但是否偏向锁那一位改成1，前23bit记录抢到锁的线程id，表示进入偏向锁状态;

(3) 当线程A再次试图来获得锁时，JVM发现同步锁对象的标志位是01，是否偏向锁是1，也就是偏向状态，Mark Word中记录的线程id就是线程A自己的id，表示线程A已经获得了这个偏向锁，可以执行同步中的代码;

(4) 当线程B试图获得这个锁时，JVM发现同步锁处于偏向状态，但是Mark Word中的线程id记录的不是B，那么线程B会先用CAS操作试图获得锁，这里的获得锁操作是有可能成功的，因为线程A一般不会自动释放偏向锁。如果抢锁成功，就把Mark Word里的线程id改为线程B的id，代表线程B获得了这个偏向锁，可以执行同步代码。如果抢锁失败，则继续执行步骤5;

(5) 偏向锁状态抢锁失败，代表当前锁有一定的竞争，偏向锁将升级为轻量级锁。JVM会在当前线程的线程栈中开辟一块单独的空间，里面保存指向对象锁Mark Word的指针，同时在对象锁Mark Word中保存指向这片空间的指针。上述两个保存操作都是CAS操作，如果保存成功，代表线程抢到了同步锁，就把Mark Word中的锁标志位改成00，可以执行同步代码。如果保存失败，表示抢锁失败，竞争太激烈，继续执行步骤6;

(6) 轻量级锁抢锁失败，JVM会使用自旋锁，自旋锁不是一个锁状态，只是代表不断的重试，尝试抢锁。从JDK1.7开始，自旋锁默认启用，自旋次数由JVM决定。如果抢锁成功则执行同步代码，如果失败则继续执行步骤7;

(7) 自旋锁重试之后如果抢锁依然失败，同步锁会升级至重量级锁，锁标志位改为10。在这个状态下，未抢到锁的线程都会被阻塞。

# Lock和synchronized区别



# Condition

一个Lock对象可以创建多个Condition对象，它们是一个对多的关系。

## Condition原理







# Java内存模型

![](https://youpaiyun.zongqilive.cn/image/20200421161724.png)

## as-if-serial语义

不管怎么重排序，单线程的执行结果不会改变。



## happens-before原则

happens-before关系给编写正确同步的多线程程序的程序员创造了一个幻境：正确同步的多线程程序是按happens-before指定的顺序来执行的。

两个操作之间具有happens-before关系，并不意味着前一个操作必须要在后一个操作之前执行！
happens-before仅仅要求前一个操作（执行的结果）对后一个操作可见

### 1-程序的顺序性规则

```java
这条规则指在一个线程中，按照程序顺序，程序前面对某一个变量的修改一定对后续操作可见的。
double pi = 3.14; // A
double r = 1.0; // B
double area = pi * r * r; // C

比如上面那三行代码，第一行的 "double pi = 3.14; " happens-before 于 “double r = 1.0;”，这就是规则 1 的内容，比较符合单线程里面的逻辑思维，很好理解。
```

### 2-锁规则

一个锁的`解锁` Happens-Before 于后续对这个锁的`加锁`

这个规则中说的锁其实就是 Java 里的 `synchronized`

### 3-volatile 变量规则

```
对一个 volatile 域的写，happens-before 于任意后续对这个 volatile 域的读

简单的理解：一个线程修改了volatile变量，则对另外一个线程读取volatile的变量是可见的。
```

### 4-传递规则

如果 A happens-before B，且 B happens-before C，那么 A happens-before C

```java
class VolatileExample {
  int x = 0;
  volatile boolean v = false;
  public void writer() {
    x = 42;
    v = true;
  }
  public void reader() {
    if (v == true) {
      // 这里x会是多少呢？
    }
  }
}
```

两个线程分别执行`writer()`和`reader()`方法，如下图：

![](https://youpaiyun.zongqilive.cn/image/20210306161035.png)

从上图可以知道以下内容：

1. `x=42`对于写变量`v=true`是可见的，符合`程序的顺序性规则`。
2. 写变量`v=true`对于读变量`v==true`是可见的，符合`volatile变量规则`

结合这个传递性，则`x=42`对于读变量`v==true`是可见的。则如果线程B读到了`v==true`，那么线程A设置的`x=42`对于线程B来说是可见的。

### 5-线程 start() 规则

它是指主线程 A 启动子线程 B 后，子线程 B 能够看到主线程在启动子线程 B 前的操作。

### 6-线程 join() 规则

如果线程 A 执行操作 ThreadB.join()并成功返回，那么线程 B 中的任意操作 happens-before 于线程 A 从 ThreadB.join()操作成功返回。



# volatitle

https://juejin.cn/post/6973930636448890893

>  要注意volatile关键字并不能保证原子性。

## 保证可见性

其实就是禁用 CPU 缓存。

对volatile变量的写操作与普通变量的主要区别有两点：

- 修改volatile变量时会强制将修改后的值刷新的主内存中。
- 修改volatile变量后会导致其他线程工作内存中对应的变量值失效。因此，再读取该变量值的时候就需要重新从读取主内存中的值。

通过这两个操作，就可以解决volatile变量的可见性问题。



## 保证有序性

禁止指令重排

volatile 关键字禁止指令重排序有两层意思：

1.  执行volatile读或写操作时，在其前面的操作肯定已经全部执行，且结果已经对后面的操作可见，在volatile后面的操作肯定还没有进行

2. 在进行指令优化时，不能将 volatile 之前的语句放在对 volatile 变量的读写操作之后，也不能把 volatile 变量后面的语句放到其前面执行

happen-before规则

volatile变量规则：对一个 volatile 域的写，happens-before 于任意后续对这个 volatile 域的读

上面是 volatile变量的保证有序性的规则。为了实现volatile内存语义，JMM会对volatile变量限制重排序。



> 双重锁校验的单利模式

![](https://youpaiyun.zongqilive.cn/image/20210127154837.png)



# ThreadLocal

https://juejin.cn/post/6975377341782589453?utm_source=gold_browser_extension



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























