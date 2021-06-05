# 总结网址



[https://mp.weixin.qq.com/s/MrWFt4jDKRQC3g4TsMde7Q](https://mp.weixin.qq.com/s/MrWFt4jDKRQC3g4TsMde7Q?fileGuid=yrQ3rVCrKppywQ9Q)

2021年05月12日（UTC+8）[https://mp.weixin.qq.com/s/UzYQRhwA4ubDry_Ve59Rpg](https://mp.weixin.qq.com/s/UzYQRhwA4ubDry_Ve59Rpg?fileGuid=yrQ3rVCrKppywQ9Q)



# 线程池

* 工作流程
* 反射
* 7大参数, 额外的参数, allowCoreThreadTimeOut
* 线程数量如何配置
* 线程如何复用
* 线程如何销毁 compareAndDecrementWorkerCount
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
* ThreadLocal原理, 应用场景, 注意的问题(内存泄露, 父子线程)
* CompLateFuture 和Future 区别
* HashMap hash值 如何计算, 为什么是2的n次幂, 扩容 (并发如何出现死循环的), JDK7和JDK8的区别
* ConcurrentHashMap, put 原理
* volatile解决什么问题, 实现原理
* happen-before原则有哪些
* ssynchronized 和 Lock的区别
* Condition 原理
* AQS队列如何唤醒
* 读写锁实现原理, 自己手写实现读写锁
* 单例模式 双重检查为什么要判断2次null
* 泛型擦除
* Airsarth的使用(CPU 100%)
* CPU100% 如何解决
* JUC下有哪些类
* 重入锁如何实现
* 锁对象, 不能是String, 不能是包装类
* 匿名内部类原理,
* 1
* [https://leetcode-cn.com/circle/discuss/Ub0DFz/](https://leetcode-cn.com/circle/discuss/Ub0DFz/?fileGuid=yrQ3rVCrKppywQ9Q)
* [https://leetcode-cn.com/circle/discuss/5r7lrA/](https://leetcode-cn.com/circle/discuss/5r7lrA/?fileGuid=yrQ3rVCrKppywQ9Q)
# GC
* cms, g1 的工作步骤 , 每个阶段做什么, stw 在哪个阶段
* GC算法有哪些
* 可达性分析算法, GCRoot 是哪些对象
* 常用参数配置有哪些
* G1和cms 的区别
* Yong GC 频繁, Full GC频繁, 如何优化
* 分析GC日志时, Yong GC, Full GC 多长时间,  回收前后内存情况

# Spring & SpringMVC & SpringBoot & Springcloude
[https://mp.weixin.qq.com/s/Rp9V-KITlNLmoJEjKWyogg](https://mp.weixin.qq.com/s/Rp9V-KITlNLmoJEjKWyogg?fileGuid=yrQ3rVCrKppywQ9Q)

* Spring 中AOP的使用有哪些
* AOP原理, 什么时候做织入
* AOP失效情况有哪些
* AOP怎么做到 http 请求之前的监控
* Spring三级缓存, 二级行不行
* Springboot启动流程
* Bean初始化流程
* new Object() 发生了什么
* 项目中如何部署启动 springboot 项目
* IOC, 注入方式有哪些
* 锁和索引的关系
* SpringBoot配置文件加载的优先级
* Spring & SpringMVC & SpringBoot & Springcloude 的区别和联系
* * 1
* 1
# 面向对象
* 六大设计原则
* # 网络
# 网络七层模型

## ![图片](https://uploader.shimo.im/f/wNfiKNrt3PkgbPMp.png!thumbnail?fileGuid=yrQ3rVCrKppywQ9Q)

* 存在大量time_wait, 如何监控机子?
* https 如何保证安全传输, 什么是中间人攻击
* 线程模型NIO BIO
* Netty 线程模型
* 

# Mysql
* 事务隔离级别有哪些, 解决什么问题
* RC和RR 锁的区别
* 脏读, 不可重复读, 幻读
* 有索引(A,B) 和索引(C) select  B from table where A=a and C=c; 如何使用索引
* 如何优化慢sql, (1. 加索引, 2左前缀查询,3 %like%查询, 4 Limit1000,10,
* 除了使用explan, 还有使用其他工具或者方式吗
* explain 关键列, id, key, type 需要背诵
* 索引下推
* 分布式事务
* in 为什么能用到索引
* 订单分表, 按UID分表,
* 分表 hash, range, group 三个思路吧, 扩表如何做数据迁移
* redolog undolog 做什么
* MVCC
* 全局唯一ID 如何生成, 各自  优缺点
* 1
* 1
* 1
## 为什么要用B+树

因为B树不管叶子节点还是非叶子节点，都会保存数据，这样导致在非叶子节点中能保存的指针数量变少（有些资料也称为扇出）

指针少的情况下要保存大量数据，只能增加树的高度，导致IO操作变多，查询性能变低；

范围查询

* **B+树如何进行动态的调整**
* 插入主键索引, B+树如何发生变化

![图片](https://uploader.shimo.im/f/P0UsWisLRGElkmc4.png!thumbnail?fileGuid=yrQ3rVCrKppywQ9Q)

# Dubbo
* 整个调用流程
* Dubbo Filter
* # Redis
* rehash 实现
* zset实现的数据结构有哪些, 跳表如何实现(手写跳表), 时间复杂度如何计算
* 分布式锁: redis实现 + ZK实现, 有啥区别
    * redis实现有哪些问题(3点, 1过期时间, 2删除,3 主节点挂掉)
* Redis集群, 增加节点, 槽迁移的整个过程, 数据如何访问, key如何找到曹节点
* 过期策略
* 数据一致性问题 , 如何解决
* 缓存穿透, 击穿, 雪崩, 是什么, 如何解决
* AOF和RDB
# Mybatis
* 工作流程
* #和$的区别
* 
# MQ
* 如何保证消息不重复发送?
* 消息不丢失
* 消息不重复消费

# ES
* fileter 和query 区别

# Linux
* 进程和线程区别
## 统计ip访问的 pv uv

ip path


# 设计
* 单点登录如何实现
* 秒杀设计
* 敏感词匹配
## 限流算法代码实现

[https://blog.csdn.net/qq_34162294/article/details/106233694](https://blog.csdn.net/qq_34162294/article/details/106233694?fileGuid=yrQ3rVCrKppywQ9Q)

# 设计模式
* 单例模式
* 观察者模式
* 责任链模式

![图片](https://uploader.shimo.im/f/ucTKUKrQZIs3rh50.png!thumbnail?fileGuid=yrQ3rVCrKppywQ9Q)


# 其他
必看 : 怎么说离职原因 新公司容易接受？

1.实际原因：原单位钱太少。离职原因：我认为我自己已经具备了一定的积累，希望可以迈向一个新的台阶。

2.实际原因：跟同事处不来。离职原因：我很重视平台的发展，我认为一个人才只有放在合适的平台才能够最大程度的发挥出自己的才干。

3.实际原因：有个傻X领导。离职原因：虽然我已经有相当的经验和技能，但仍然希望能够拓宽自己的知识面，进行更深入的学习和实践。

4.实际原因：原单位扯皮甩锅大战。离职原因：我认为一个良好的工作氛围能够提高工作效率，明确的分工和相对完善的制度是提高生产力的基本保障。

5.实际原因：提拔无望。离职原因：我认为人不应该满足于现状，而是应该积极进取，如果不能够适时的挑战自己，逼自己一把，怎么能够最大程度的挖掘自己的潜力呢？

6.实际原因：傻X管理制度太多。离职原因：贵公司所推崇的人性化管理非常符合我对工作环境的预期，我也相信在这样的环境中，我能够发挥出更大的主观能动性。

7.实际原因：太忙了。离职原因：相比于低效重复的工作进度，我更认同高效高质的工作模式。

8.实际原因：太闲了。离职原因：我是一个不满足于现状的人，我认为年轻人不应该满足于安乐的环境，应该随时具有努力和奋斗的觉悟。

从以上中，你学到了什么？[吃惊]。。。………………………………………………………………


为什么想离开现在公司？

原则：讲客观事实，不带负面情绪，简明扼要，不抱怨，尽量不讲公司和上级坏话。（比如寻求技术成长，看好赛道等等）

职业规划：对这份工作的主要诉求 想要提高的地方 想做的事情等

性格优缺点；

为什么选择我们公司：公司发展的情况  平台  业务等方面(使劲夸，猿辅导巴拉巴拉，部门业务做的挺好的，赋能，感兴趣)

期望薪资：

如果涉及就说我期望30%-40%的涨幅，具体还是希望看公司对自己的评价

1、不说具体数字：因为我对这边挺有意向的，所以也会看看咱们这边对我面试情况的评估，给一个合理的薪资方案就可以

2、说具体数字：说出自己的期望数字的同时强调对公司的意向， 可以比自己的底线稍微高一点，但是不要说死  比如：我理想期望35K 但是因为对这边很有意向，所以也会考虑对我面试情况的评估，这个可以再谈~

# sql50题
[https://www.jianshu.com/p/476b52ee4f1b](https://www.jianshu.com/p/476b52ee4f1b?fileGuid=yrQ3rVCrKppywQ9Q)

[https://sql50.readthedocs.io/zh_CN/latest/sql50.html](https://sql50.readthedocs.io/zh_CN/latest/sql50.html?fileGuid=yrQ3rVCrKppywQ9Q)


# 工具类
java技术栈小程序

