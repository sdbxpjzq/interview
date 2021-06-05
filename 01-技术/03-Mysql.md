```
https://mp.weixin.qq.com/mp/homepage?__biz=Mzg3MzU2Njk3MA==&hid=1&sn=5e482808164cff0f8e9ae8de486d3dd6&scene=1&devicetype=iOS14.2&version=1800062e&lang=zh_CN&nettype=WIFI&ascene=7&session_us=gh_d355cc0acb3e&fontScale=109&wx_header=1


```





# 索引

## 为什么使用B+树


## 索引设计优化原则

1. 尽量选择区分度高的列作为索引,区分度的公式是count(distinct col)/count(*)
1. 最左前缀匹配原则
1. 索引列不能参与计算
1. 利用覆盖索引

## 聚簇索引和非聚簇索引

MyI

## 联合索引的认识
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








## 索引优化手段
### explain

https://juejin.cn/post/6969120307814596645?utm_source=gold_browser_extension


### optimizer trace








# 事务
## 隔离级别
### 未提交读(Read Uncommitted)
允许脏读，也就是可能读取到其他会话中未提交事务修改的数据
### 提交读(Read Committed)
只能读取到已经提交的数据
### 可重复读(Repeated Read)
mysql默认
在同一个事务内的查询都是事务开始时刻一致的，InnoDB默认级别。
在SQL标准中，该隔离级别消除了不可重复读，但是还存在幻象读(mysql解决 GAP锁解决)
### 串行读(Serializable)
完全串行化的读，每次读都需要获得表级共享锁，读写相互都会阻塞
​


1. `脏读` -- 当一个事务读取到另一个事务`尚未提交`的修改时, 产生脏读.
1. `不可重复读` -- 同一查询在同一事务中执行多次,由于其他提交事务所提交的`修改`或者`删除`, 每次返回不同的结果集, 此时发生非重复读.
1. `幻读`  -- 同一查询在同一事务中执行多次, 由于其他提交事务所做的`插入操作`,  每次返回不同的结果集, 此时发生 幻读.



## RC(读已提交) 与 RR(可重复读) 在锁方面的区别
1. RR 支持 gap lock(next-key lock)，而RC则没有gap lock。   
因为MySQL的RR需要gap lock来解决幻读问题。而RC隔离级别则是允许存在不可重复读和幻读的。所以RC的并发一般要好于RR
​

2. RC 隔离级别，通过 where 条件过滤之后，不符合条件的记录上的行锁，会释放掉(虽然这里破坏了“两阶段加锁原则”)；但是RR隔离级别，即使不符合where条件的记录，也不会释放行锁和gap lock；所以从锁方面来看，RC的并发应该要好于RR

## MVCC

undolog, 版本链, readview



在RR级别中，通过MVCC机制，虽然让数据变得可重复读
快照读

- 快照读：就是select
   - select * from table ….;
- 当前读：特殊的读操作，插入/更新/删除操作，属于当前读，处理的都是当前的数据，需要加锁。
   - select * from table where ? lock in share mode;
   - select * from table where ? for update;
   - insert;
   - update ;
   - delete;

当前读 
​

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
