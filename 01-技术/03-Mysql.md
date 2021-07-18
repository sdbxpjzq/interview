```
https://mp.weixin.qq.com/mp/homepage?__biz=Mzg3MzU2Njk3MA==&hid=1&sn=5e482808164cff0f8e9ae8de486d3dd6&scene=1&devicetype=iOS14.2&version=1800062e&lang=zh_CN&nettype=WIFI&ascene=7&session_us=gh_d355cc0acb3e&fontScale=109&wx_header=1

https://juejin.cn/post/6973617121347682334?utm_source=gold_browser_extension



1. Innodb存储基本单位页结构详解
2. 索引底层原理与执行流程精讲
3. Mysql是如何选择最优索引的
4. 覆盖索引底层原理与执行流程精讲
5. 索引下推底层原理与执行流程精讲
6. Mysql为什么会出现索引失效 
7. 亿级流量下Mysql索引优化策略
8. 一线大厂为什么要基于Mysql开发自研数据库
[表情]8点直播链接：https://ke.qq.com/webcourse/index.html#cid=230866&term_id=100272363&taid=4409398109898194&from=41
你不需要很厉害才开始，但是需要开始才能很厉害，今晚8点，让周瑜老师陪你一起开始变得很厉害
```



# Mysql架构

![](https://youpaiyun.zongqilive.cn/image/20210701191759.png)

> **MySQL 8.0 版本直接将查询缓存的整块功能删掉了，**





# 索引结构

https://zhuanlan.zhihu.com/p/281933182

## B树

MongoDB使用B树。

MongoDB读写单个记录的性能（即MongoDB是为了迅速获取对应的键值对信息，所以使用B树可以更好满足需求，如果使用B+树就可能出现回表的情况，性能反而比较差

## 为什么使用B+树

1. 非叶子节点存储key, 叶子节点存储key和数据

2. 一个结点存储更多的数据 , 能形成的叉数越多,可以减少树高度

   **数据预读**的思路是：磁盘读写并不是按需读取，而是按页预读，一次会读一页的数据，每次加载更多的数据，以便未来减少磁盘IO

3. 叶子节点有序且增加了双向链表, 有利于范围查询





# 非聚集索引

MyISAM的索引与行记录是分开存储的，叫做**非聚集索引**

其主键索引与普通索引没有本质差异：

- 有连续聚集的区域单独存储行记录
- 主键索引的叶子节点，存储主键，与对应行记录的指针
- 普通索引的叶子结点，存储索引列，与对应行记录的指针

![](https://youpaiyun.zongqilive.cn/image/20210622200928.png)'

# 聚集索引

InnoDB的`主键索引`与行记录是存储在一起的，叫做**聚集索引**

- 主键索引的叶子节点，存储主键，与对应行记录（而不是指针）
- 普通索引的叶子节点，存储主键（也不是指针）-- 非聚集索引

![](https://youpaiyun.zongqilive.cn/image/20210622202251.png)

聚簇索引一定是主键索引吗?

不一定, 聚集索引不一定是主键，但是主键一定是聚集索引：

原因是如果没有定义主键，聚集索引可能是第一个不允许为 null 的唯一索引，如果也没有这样的唯一索引，InnoDB 会选择内置 6 字节长的 ROWID 作为隐含的聚集索引。



# 索引设计原则

### 字段适不适合建索引

`不同值的数目 /  总记录数 `,  越接近 `1`, 这个索引的效率就越高

假设一个表有2000条记录,  索引列 有1980个不同的值, 那么索引的选择性  =  1980 / 2000 = 0.99.

例如有一个国籍字段,  14亿中国人的国籍都是中国, 不需要创建索引.

### 其他原则

- 使用短索引，如果对长字符串列进行索引，应该指定一个前缀长度，这样能够节省大量索引空间
- 更新频繁字段不适合创建索引
- 数据区分度不高的列不适合做索引列(如性别), 区分度的公式是count(distinct col)/count(*)
- 尽量的扩展索引，不要新建索引。比如表中已经有a的索引，现在要加(a,b)的索引，那么只需要修改原来的索引即可。
- 对于定义为text、image和bit的数据类型的列不要建立索引。





# 联合索引的认识

全值匹配我最爱, 最左前缀要遵守;

带头大哥不能死(不能使用范围查询), 中间兄弟不能断;

索引列上少计算, 范围之后(in除外)全失效;

`like`百分写右边, 覆盖索引不写星;

不等空值还有`or`, 索引失效要少用;

变量引号不可丢, `SQL`高级也不难!



对(a,b)字段建立索引
![](https://youpaiyun.zongqilive.cn/image/20210530133613.png)
如图所示他们是按照a来进行排序，在a相等的情况下，才按b来排序。
因此，我们可以看到a是有序的1，1，2，2，3，3。而b是一种全局无序，局部相对有序状态!
什么意思呢？
从全局来看，b的值为1，2，1，4，1，2，是无序的，因此直接执行b = 2这种查询条件没有办法利用索引。
从局部来看，当a的值确定的时候，b是有序的。例如a = 1时，b值为1，2是有序的状态。当a=2时候，b的值为1,4也是有序状态。
因此，你执行a = 1 and b = 2是a,b字段能用到索引的。而你执行a > 1 and b = 2时，a字段能用到索引，b字段用不到索引。因为a的值此时是一个范围，不是固定的，在这个范围内b值不是有序的，因此b字段用不上索引。

#### 典型题目
##### 题目1
SELECT * FROMtableWHERE a = 1and b = 2and c = 3;
如何建立索引?
如果此题回答为对(a,b,c)建立索引，那都可以回去等通知了。
此题正确答法是，(a,b,c)或者(c,b,a)或者(b,a,c)都可以，重点要的是将区分度高的字段放在前面，区分度低的字段放后面。像性别、状态这种字段区分度就很低，我们一般放后面。
​

例如假设区分度由大到小为b,a,c。那么我们就对(b,a,c)建立索引。在执行sql的时候，优化器会 帮我们调整where后a,b,c的顺序，让我们用上索引
##### 题目2
SELECT * FROMtableWHERE a > 1and b = 2; 
如何建立索引?
如果此题回答为对(a,b)建立索引，那都可以回去等通知了。
此题正确答法是，对(b,a)建立索引。如果你建立的是(a,b)索引，那么只有a字段能用得上索引，毕竟最左匹配原则遇到范围查询就停止匹配。
如果对(b,a)建立索引那么两个字段都能用上，优化器会帮我们调整where后a,b的顺序，让我们用上索引。
##### 题目3
SELECT * FROM`table`WHERE a > 1and b = 2and c > 3; 
如何建立索引?
此题回答也是不一定，(b,a)或者(b,c)都可以，要结合具体情况具体分析。
##### 题目4
SELECT * FROM `table` WHERE a = 1 ORDER BY b; 
如何建立索引？
这还需要想？一看就是对(a,b)建索引，当a = 1的时候，b相对有序，可以避免再次排序！
那么
SELECT * FROM `table` WHERE a > 1 ORDER BY b;  
如何建立索引？
对(a)建立索引，因为a的值是一个范围，这个范围内b值是无序的，没有必要对(a,b)建立索引。
拓展一下
SELECT * FROM `table` WHERE a = 1 AND b = 2 AND c > 3 ORDER BY c; 
怎么建索引?

##### 题型5
SELECT * FROM `table` WHERE a IN (1,2,3) and b > 1;  

如何建立索引？
还是对(a，b)建立索引，因为IN在这里可以视为等值引用，不会中止索引匹配，所以还是(a,b)!
拓展一下
SELECT * FROM `table` WHERE a = 1 AND b IN (1,2,3) AND c > 3 ORDER BY c; 
如何建立索引？此时c排序是用不到索引的。
​

## 遇到慢查询, 如何解决的?
```sql
select
   count(*) 
from
   task 
where
   status=2 
   and operator_id=20839 
   and operate_time>1371169729 
   and operate_time<1371174603 
   and type=2;
```
那么索引建立成(status,type,operator_id,operate_time)就是非常正确的，因为可以覆盖到所有情况。这个就是利用了索引的最左匹配的原则

# 回表优化

## 索引下推

`idx_name_title_age (name,title,age)`联合索引,

```sql
select id, name, sex from index_opt_test where name='cc' and title like '%7' and sex='male';
```

Server层会将where条件中`在组合索引中的字段全部推送`到引擎层，引擎层根据断桥原则匹配出索引数据，然后将其他索引字段带入再进行一次筛选，然后拿最终匹配的主键关键字回表查询出数据后返回给Server层，Server层再根据剩余的where条件做一次筛选，然后返回给Client

- 执行过程：

- 1. Server把name和title都推到引擎层
  2. 引擎层根据name去idx_cb中查询出主键关键字和title、age  , ( ==这里加强了过滤, 数据量少了, 回表的也就少了==)
  3. 再由title筛选出匹配的主键关键字
  4. 回表去捞数据返回给Server层
  5. Server层再根据sex筛选出最终的数据
  6. 再返回给客户端







## MRR 优化

`MRR`的思想就很简单，开启`MRR`之后，`MySQL`会将所有返回的主键先进行排序，然后再进行回表。这样就避免了离散读取的问题。

![](https://youpaiyun.zongqilive.cn/image/20200916135147.png)











# SQL语句优化

1. 应尽量避免全表扫描，首先应考虑在 where 及 order by 涉及的列上建立索引。
2. 应尽量避免在 where 子句中对字段进行 null 值判断，否则将导致引擎放弃使用索引而进行全表扫描
3. 索引列不能参与计算
4. 利用索引覆盖
5. 查询满足最左前缀匹配原则
6. 分页优化



## explain分析

### id

- id相同执行顺序由上至下。
- id不同，id值越大优先级越高，越先被执行。
- id为null时表示一个结果集，不需要使用它查询，常出现在包含union等查询语句中。

### type

consts 表中最多只有一个匹配行（主键或者唯一索引），在优化阶段即可读取到数据

ref 指的是使用普通的索引（normal index）

![](https://youpaiyun.zongqilive.cn/image/20210703161532.png)

### key

在查询中实际使用的索引，若没有使用索引，显示为NULL

### Extra

- Using Index

  使用了索引覆盖，不需要回表查询

- Using Where

  (没有用用到索引看type)回表拿数据，使用where条件再Sever层过滤数据, 

  using where和是否走索引进行关联，是不正确的

- Using Index Condition

  使用了索引下推(5.6+版本)

- Using Filesort

  得到所需结果集，需要对所有记录进行文件排序, 需要优化

- Using Temporary

  使用了临时表保存了中间结果集, 需要优化

- Using Join Buffer

  连表查询时使用了循环嵌套扫描, 需要优化
  
- null

  没有索引覆盖，回表拿数据



https://juejin.cn/post/6969120307814596645?utm_source=gold_browser_extension


## optimizer trace

# InnoDB行格式

![](https://youpaiyun.zongqilive.cn/image/20200826100856.png)

### 3个隐藏列

![](https://youpaiyun.zongqilive.cn/image/20200826110844.png)







# InnoDB内存结构

InnoDB内存结构包含四大核心组件，分别是：

（1）**缓冲池**(Buffer Pool)；

（2）**写缓冲**(Change Buffer)；

（3）**自适应哈希索引**(Adaptive Hash Index)；

（4）**日志缓冲**(Log Buffer)；

## 缓冲池(Buffer Pool)

以页为单位，缓存最热的数据页(data page)与索引页(index page)

读请求，缓冲池能够减少磁盘IO，提升性能。

![](https://youpaiyun.zongqilive.cn/image/20210704152941.png)



### free链表

记录bufferPool哪些页是空白的

当把磁盘中的页要加载到buffer pool时, 先来free链表获取一个空白页对应的控制块,数据页放到指定位置后, 则从free链表删除该控制块

![](https://youpaiyun.zongqilive.cn/image/20210704153430.png)



### flush链表

记录Buffer Pool哪些写页是被修改过的, 由后台线程定时刷到磁盘



### LRU链表(并非简单的)

将哪些页淘汰, 每次buffer pool添加新的页数据, 将会有新的控制块加入链表

`热数据区域` 和 `冷数据区`(比例: 5:3)

新数据最先加入到冷数据区域, 

什么情况下冷数据转移到热数据呢?

 同一页 当两次访问的时间间隔 大于1秒, t2 -t1 > 1s, 转移到热数据区域





## Double write Buffer

Doubel write保证了页的可靠性

![](https://youpaiyun.zongqilive.cn/image/20210704154823.png)

1. 当一系列机制触发数据缓冲池中的脏页刷新时，并不直接写入磁盘数据文件中，而是先拷贝至内存中的doublewrite buffer中；

2、接着从两次写缓冲区分两次写入磁盘共享表空间中(连续存储，顺序写，性能很高)，每次写1MB；

3、待第二步完成后，再将doublewrite buffer中的脏页数据写入实际的各个表空间文件(离散写)；(脏页数据固化后，即进行标记对应doublewrite数据可覆盖)



写入共享表空间失败:

MySQL会直接使用redo log的条数据构建一个新页写入到【磁盘数据文件.idb】中，此时的故障恢复不需要doublewrite buffer的参与。



当写入磁盘（共享表空间）的doublewirte buffer完成，写入【磁盘数据文件.idb】时发生故障:

MySQL进行故障恢复时会使用磁盘（共享表空间）的doublewirte buffer进行恢复，因为此时磁盘（共享表空间）的doublewirte buffer已经写入完成，保存的是完整页，故也可保证的【磁盘数据文件.idb】数据的完整性，



## 写缓冲(Change Buffer)

主要目的是将对二级索引的数据操作缓存下来，以此减少二级索引的随机IO，并达到操作合并的效果。

种应用在非唯一普通索引页不在缓冲池中，对页进行了写操作，并不会立刻将磁盘页加载到缓冲池，而仅仅记录缓冲变更(buffer changes)，等未来数据被读取时，再将数据合并(merge)恢复到缓冲池中的技术.

![](https://youpaiyun.zongqilive.cn/image/20210704155343.png)

如果在此时，机器断电重启了，会不会导致 change buffer 丢失呢？change buffer 丢失可不是小事情，再从磁盘读出来的话就没有办法进行merge了，相当于数据丢失了？

是不会丢失的。虽然只是更新内存，但是在事务提交的时候，change buffer 的操作也会记录到 redo log 里面，所以崩溃回复的时候，change buffer 也是可以找回来的。



merge过程是否会把数据直接写回磁盘？

先看下merge流程是什么样子的吧：

 1、从磁盘读入数据页到内存中（老版本的数据页）

 2、从change buffer 里找出这个数据页的 change buffer 记录，可能是多个，以此按照顺序进行应用，得到最新的版本数据页。

 3、写redo log。这个redo log 包含了 数据的变更 和 change buffer 的 变更。

在此时 merge 的操作就结束了。这时候数据页 和 内存中的 change buffer 对应的磁盘位置都还没有进行修改， 属于脏页。之后各自在刷回自己的物理数据，就是另一个过程。



**在普通索引和唯一索引上该如何进行选择呢？**

尽可能的使用普通索引。两类索引在查询上没有什么差别，只要考虑更新性能的影响。因为普通索引可以很好的使用到 change buffer。



## 日志缓冲(Log Buffer)

redo log buffer



# 日志文件

## redo log

进行数据的新增、删除、修改操作时，会写 **redo log**, 解决数据库宕机重启丢失数据的问题

### 两阶段事务提交

保证数据不丢失

1.  记录一条 redo log 到 redo log buffer 中, redo log 的状态标记为 prepare 状态；
2. 接着存储引擎告诉执行器，可以提交事务了。执行器接到通知后，会写 binlog 日志，然后提交事务；
3. 存储引擎接到提交事务的通知后，将 redo log 的日志状态标记为 commit 状态；
4. 接着根据 innodb_flush_log_at_commit 参数的配置，(默认是1, 事务提交时,立即持久化)决定是否将 redo log buffer 中的日志刷入到磁盘。

redo log 在进行数据重做时，只有读到了 commit 标识，才会认为这条 redo log 日志是完整的，才会进行数据重做，否则会认为这个 redo log 日志不完整，不会进行数据重做。



为什么引入redo log ?直接更新磁盘数据不行吗?

写 redo log 时，我们将 redo log 日志追加到文件末尾，虽然也是一次磁盘 IO，但是这是顺序写操作, 而对于直接将数据更新到磁盘，涉及到的操作是将 buffer pool 中缓存页写入到磁盘上的数据页上，由于涉及到寻找数据页在磁盘的哪个地方，这个操作发生的是随机写操作

从另一方面来讲，通常一次更新操作，我们往往只会涉及到修改几个字节的数据，而如果因为仅仅修改几个字节的数据，就将整个数据页写入到磁盘（无论是磁盘还是 buffer pool，他们管理数据的单位都是以页为单位），这个代价未免也太了（每个数据页默认是 16KB），而一条 redo log 日志的大小可能就只有几个字节，因此每次磁盘 IO 写入的数据量更小，那么耗时也会更短。



正是由于 Redo Log 的存在，可以让内存中出现大量的脏页。



## undo log 

undo log，它是为了实现事务的回滚 和 MVCC

![](https://youpaiyun.zongqilive.cn/image/20210703152616.png)



## binlog

主从复制 延迟

mult thread slave



### 主从复制

![](https://youpaiyun.zongqilive.cn/image/20200916155742.png)

master

（1）**binlog dump线程**：log dump线程通知slave有数据更新，当I/O线程请求日志内容时，会将此时的binlog名称和当前更新的位置同时传给slave的I/O线程。

slave

（2）**I/O线程**：该线程会连接到master，向log dump线程请求一份指定binlog文件位置的副本，并将请求回来的binlog存到本地的relay log中，relay log和binlog日志一样也是记录了数据更新的事件，它也是按照递增后缀名的方式，产生多个relay log（ host_name-relay-bin.000001）文件，slave会使用一个index文件（ host_name-relay-bin.index）来追踪当前正在使用的relay log文件。

（3）**SQL线程**：该线程检测到relay log有更新后，会读取并在本地做redo操作，将发生在主库的事件在本地重新执行一遍，来保证主从数据同步。此外，如果一个relay log文件中的全部事件都执行完毕，那么SQL线程会自动将该relay log 文件删除掉。



为什么需要relay log?

同步过来的bin log 数据, SQL thread 不一定能及时处理



### 主从延迟

mult thread slave



# SQL更新执行流程

![](https://youpaiyun.zongqilive.cn/image/20210123134446.png)

![](https://youpaiyun.zongqilive.cn/image/20200914192953.png)



# 事务

![](https://youpaiyun.zongqilive.cn/image/20210704154525.png)

## 隔离级别
1. 读未提交 read uncommitted
2. 读已提交 read committed
3. 可重复读 repeatable read  -- mysql默认
   1. 该隔离级别消除了不可重复读(MVCC)和幻象读(mysql解决 GAP锁解决)
4. 可串行化  serializable -- 对于同一行记录，“写”会加“写锁”，“读”会加“读锁”，当出现读写锁冲突的时候，后访问的事务必须等前一个事务执行完成，才能继续执行。

隔离程度 从上到下 , 越来越强

## 问题


1. `脏读` -- 当一个事务读取到另一个事务`尚未提交`的修改时, 产生脏读.
1. `不可重复读` -- 同一查询在同一事务中执行多次,由于其他提交事务所提交的`修改`或者`删除`, 每次返回不同的结果集, 此时发生非重复读.
1. `幻读`  -- 同一查询在同一事务中执行多次, 由于其他提交事务所做的`插入操作`,  每次返回不同的结果集, 此时发生 幻读.



## RC与 RR在锁方面的区别
1. RR 支持 gap lock(next-key lock)，而RC则没有gap lock。   
因为MySQL的RR需要gap lock来解决幻读问题。而RC隔离级别则是允许存在不可重复读和幻读的。所以RC的并发一般要好于RR
​

2. RC 隔离级别，通过 where 条件过滤之后，不符合条件的记录上的行锁，会释放掉(虽然这里破坏了“两阶段加锁原则”)；但是RR隔离级别，即使不符合where条件的记录，也不会释放行锁和gap lock；所以从锁方面来看，RC的并发应该要好于RR







# MVCC

## 三个概念

`undolog`, ` 版本链`,` readview(在版本链 里选择哪一条记录)`

在RR级别中，通过MVCC机制，虽然让数据变得可重复读

MVCC解决的是快照读的幻读问题，并不能解决当前读的幻读问题,当前读的幻读问题是通过间隙锁解决的

- 快照读：就是select
   - select * from table ….;
   
- 当前读：特殊的读操作，插入/更新/删除操作，属于当前读，处理的都是当前的数据，需要加锁。

   `强制性的读取最新版本的数据`

   - select * from table where ? lock in share mode;
   - select * from table where ? for update;
   - insert;
   - update ;
   - delete;

## ReadView机制

当事务在开始执行的时候，会给每个事务生成一个 ReadView。这个 ReadView 会记录 4 个非常重要的属性：

1. **creator_trx_id**: 当前事务的 id；
2. `m_ids`: 当前系统中所有的活跃事务的 id列表，活跃事务指的是当前系统中开启了事务，但是还没有commit的事务；
3. **min_trx_id**: 就是 m_id 数组中最小的事务 id；
4. **max_trx_id**: 就是 m_id 数组中最大的事务 id 再加 1，也就是系统中下一个要生成的事务 id。

### 判断版本可用

![](https://youpaiyun.zongqilive.cn/image/20210703154034.png)

> 读已提交和RR , 生成readview的时机是不同的, 
>
> RC: 每次执行select查询的时候生成 readview, 是以select查询为单位的
>
> RR; 是以一个事务为单位, 第一次selected 生成readview, 后边的查询都是用这个readview, 查询的是同一份readview
>
> RR下已经生成了 ReadView 数据, 后期不会随着其他事务的提交而变化 



# 锁机制

## 锁类型

按互斥程度来划分: 

- 共享锁(S锁、IS锁)，可以提高读读并发；
- 排他锁: 为了保证数据强一致，InnoDB使用强互斥锁(X锁、IX锁)，保证同一行记录修改与删除的串行性；

按锁的粒度来划分: 

- 表锁：意向锁(IS锁、IX锁)、自增锁；
- 行锁：记录锁、间隙锁(Gap Locks)、临键锁(Next-key Locks)、插入意向锁；

其中

1. 记录锁锁定索引记录；间隙锁锁定间隔，防止间隔中被其他事务插入；临键锁锁定索引记录+间隔，防止幻读；
2. InnoDB使用插入意向锁，可以提高插入并发；
3. 间隙锁(gap lock)与临键锁(next-key lock) **只在RR以上的级别生效，RC下会失效** ；

### 表锁

#### 意向锁

InnoDB为了支持多粒度锁机制(multiple granularity locking)，即允许行级锁与表级锁共存，而引入了意向锁

意向锁是指，未来的某个时刻，事务可能要加共享/排它锁了，先提前声明一个意向。

1. 意向锁是一个表级别的锁(table-level locking)；

2. 意向锁又分为：

3. - 意向共享锁，它预示着，事务有意向对表中的某些行加共享S锁；
   - 意向排它锁，它预示着，事务有意向对表中的某些行加排它X锁；

>  事务要获得某些行的S/X锁，必须先获得表对应的IS/IX锁，意向锁仅仅表明意向

意向锁之间相互兼容，兼容互斥表如下：

![](https://youpaiyun.zongqilive.cn/image/20210704174507.png)

![](https://youpaiyun.zongqilive.cn/image/20210704174530.png)

#### 自增锁

自增锁是一种特殊的表级别锁, 专门针对事务插入`AUTO_INCREMENT`类型的列

如果一个事务正在往表中插入记录，所有其他事务的插入必须等待，以便第一个事务插入的行，是连续的主键值。



### 行锁

#### 临键锁(Next-key Lock)--默认

是记录锁与间隙锁的组合，它的封锁范围，既包含索引记录，又包含索引区间

`默认情况下`，innodb使用next-key locks来锁定记录

>  当查询的索引含有唯一属性的时候，Next-Key Lock会进行优化，将其降级为Record Lock，即仅锁住索引本身，不是范围

```sql
索引情况: index_num(num) 
数据情况: 10,20,30
事务A执行如下语句，未提交：  
    select * from lock_example where num = 20 for update;  

事务B开始，执行如下语句，会阻塞：
    insert into lock_example values('zhang',15);
```

事务A执行查询语句之后，默认给id=20这条记录加上了next-keylock，所以事务B插入[10, 30)之间的记录都会阻塞

> 临键锁的主要目的，也是为了避免幻读(Phantom Read)。如果把事务的隔离级别降级为RC，临键锁则也会失效

##### 加锁场景

1. 唯一(主键/唯一)索引进行范围查询，会加 大于查询范围前开后闭最小范围的临键锁, ( ]
2. 通过非唯一键查询，会锁定对应索引记录及其之前的间隙
3. 如果没有建立索引，那么在查询过程中实际上扫描的是全表，所以最终会锁全表



![image-20210718174900433](https://youpaiyun.zongqilive.cn/image/image-20210718174900433.png)



#### 记录锁(Record Lock)

仅仅锁住索引记录的一行，在单条索引记录上加锁

唯一索引(唯一/主键)等值查询,  退化成Record Lock





#### 间隙锁(Gap Lock)

间隙锁的主要目的，就是为了防止其他事务在间隔中插入数据，以导致“不可重复读”。

如果把事务的隔离级别降级为读提交(Read Committed,RC)，间隙锁则会自动失效。 

间隙锁可用于`防止幻读`，保证索引间的不会被插入数据 



##### 加锁场景

1. 所有的范围查询
2. 唯一索引的等值查询, 且记录不存在
3. 非唯一索引的等值查询





例子:

![](https://youpaiyun.zongqilive.cn/image/20200704110957.png)

1. 找到了c2=11的记录，
2. 发现下一条记录是c2=18，不满足条件，InnoDB遇到了第一个不满足查询条件的记录18，于是InnoDB在18上设置gap lock，
3. 此gap lock锁定了区间(11, 18)。



![](https://youpaiyun.zongqilive.cn/image/20210314183429.png)







#### 插入意向锁(Insert Intention Lock)

是专门针对`insert`操作的

多个事务，在同一个索引，同一个范围区间插入记录时，如果插入的位置不冲突，不会阻塞彼此。





## 锁和索引的关系

### 主键索引

只在主键记录上加X锁

![](https://youpaiyun.zongqilive.cn/image/20210319104317.png)

### 唯一索引

![](https://youpaiyun.zongqilive.cn/image/20210319104348.png)

SQL需要加两个X锁，

1. 对应于id unique索引上的id = 10的记录，
2. 对应于聚簇索引上的[name=’d’,id=10]的记录。

为什么聚簇索引上的记录也要加锁？

试想一下，如果并发的一个SQL，是通过主键索引来更新：update t1 set id = 100 where name = ‘d’; 此时，如果delete语句没有将主键索引上的记录加锁，那么并发的update就会感知不到delete语句的存在，违背了同一记录上的更新/删除需要串行执行的约束。



### 非唯一索引

![](https://youpaiyun.zongqilive.cn/image/20210319104825.png)



### 无索引

全表扫描

![](https://youpaiyun.zongqilive.cn/image/20210319105825.png)







InnoDB存储引擎的锁算法的一些规则如下所示. 都是在Mysql的默认 RR级别下讨论.

```sql
select * from t3 where num = 4 for update;
num
1
4
7
10
```

1. num是主键索引
   1. 降级为行锁,  对当前行加X锁.

2. num是唯一键索引
   1. 对 普通索引上对应的一行记录加排他锁
   2. 对主键索引上对应的记录加排他锁

3. num是普通索引
   1. 上边的会锁住`(1,4],(4,7)`, 既Next-key Lock

4. num无索引
   1. 扫描所有的行加X锁和间隙锁, , (像是加了表锁), 所以建议where条件要命中索引.

5. num主键索引, 值不存在
   1. 会形成间隙锁, -- (x, y) 

6. num唯一索引索引, 值不存在
   1. 会形成间隙锁, -- (x, y) 

7. num普通索引, 值不存在
   1. 会形成间隙锁, -- (x, y) 

8. num无索引, 值不存在
   1. 扫描所有的行加X锁和间隙锁, , (像是加了表锁), 所以建议where条件要命中索引.



## 锁的选择

```sql
update test set name=“hello” where name=“world"
```

1. 如果更新条件没有走索引，会进行全表扫描，扫表的时候，要阻止其他任何的更新操作，所以上升为表锁。
2. 如果更新条件为唯一索引，则使用Record Lock（记录锁）
3. 如果更新条件为非唯一索引
   1. 会使用Next-Key Lock
   2. 保证在符合条件的记录上加上排他锁，会锁定当前非唯一索引和对应的主键索引的值；
   3. 保证锁定的区间不能插入新的数据



## 死锁









# 分库分表


## 全局id


### UUID


```
优点:
- 通过本地生成，没有经过网络I/O，性能较快
- 无序，无法预测他的生成顺序。(当然这个也是他的缺点之一)
缺点:
- 128位二进制一般转换成36位的16进制，太长了只能用String存储，空间占用较多。
- 不能生成递增有序的数字
适用场景:
UUID的适用场景可以为不担心过多的空间占用，以及不需要生成有递增趋势的数字。
在Log4j里面他在UuidPatternConverter中加入了UUID来标识每一条日志。
```


### Redis的自增


```
优点：
- 性能比数据库好，能满足有序递增。
缺点：
- 由于redis是内存的KV数据库，即使有AOF和RDB，但是依然会存在数据丢失，有可能会造成ID重复。
- 依赖于redis，redis要是不稳定，会影响ID生成。
适用：
由于其性能比数据库好，但是有可能会出现ID重复和不稳定，这一块如果可以接受那么就可以使用。
也适用于到了某个时间，比如每天都刷新ID，那么这个ID就需要重置，通过(Incr Today)，每天都会从0开始加。
```


### 数据库分段


```
比如说，现在有 8 个服务节点，每个服务节点使用一个 sequence 功能来产生 ID，每个 sequence 的起始 ID 不同，并且依次递增，步长都是 8。
缺点:
- 将来如果还要增加服务节点，就不好搞了。
适合的场景：在用户防止产生的 ID 重复时，这种方案实现起来比较简单，也能达到性能目标。但是服务节点固定，步长也固定
```


![](https://uploader.shimo.im/f/IHdMBu2uVwPzI7h9.png!thumbnail?fileGuid=tRP98DKjtKhRRW3W#id=NG17a&originalType=binary&status=done&style=none)


### 雪花算法


### 时钟回拨问题


```
如果时间发生回拨，有可能会生成重复的ID
用当前时间和上一次的时间进行判断，如果当前时间小于上一次的时间那么肯定是发生了回拨，普通的算法会直接抛出异常,这里我们可以对其进行优化,一般分为两个情况:
如果时间回拨时间较短，比如配置5ms以内，那么可以直接等待一定的时间，让机器的时间追上来。
如果时间的回拨时间较长，我们不能接受这么长的阻塞等待，那么又有两个策略:
直接拒绝，抛出异常，打日志，通知RD时钟回滚。
利用扩展位，上面我们讨论过不同业务场景位数可能用不到那么多，那么我们可以把扩展位数利用起来了，比如当这个时间回拨比较长的时候，我们可以不需要等待，直接在扩展位加1。2位的扩展位允许我们有3次大的时钟回拨，一般来说就够了，如果其超过三次我们还是选择抛出异常，打日志。
```



# 表情存储

emoji表情，原来一般的字符包括中文用utf8的话，mysql是用3个字节去存储的，而emoji表情要用4个字节的utf8，也就是utf8mb4格式。





