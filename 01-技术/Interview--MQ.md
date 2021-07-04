# 如何保证不丢消息

## 生产者

- 当往 MQ 中写消息出现异常时，采用 try...catch... 捕获异常，在异常代码块中重试。

## MQ

一方面是要保证消息要持久化到磁盘，另一方面是需要保证消息有多个副本。

对于 kafka 而言，需要保证如下三点：

1. 要求每个 partition 的副本数大于 1（replication factor > 1）
2. 要求 kafka 服务端设置 broker.insync.replicas 参数的值大于 1，它的意思是要求至少有一个 flower 在和 leader 同步
3. 将 acks=all，在写数据时，要求消息写到所有的 leader 和 flower 之后，才认为消息写成功。

##  消费者

1. 关闭 Auto ACK。消费到消息后，处理完业务逻辑后再手动提交 offset。
2. 不使用异步线程池处理消息。(即使使用也要 fromId % theadPoolSize)



# 如何保证消息的顺序性

[https://mp.weixin.qq.com/s?__biz=MzIyNDU2ODA4OQ==&mid=2247483922&idx=1&sn=09214b2223b780afe441d481ae367c0b&chksm=e80db464df7a3d72b57bf1158478ad3f55c546be15af811f10dc262e0262832c9d67ff887047&scene=21#wechat_redirect](https://mp.weixin.qq.com/s?__biz=MzIyNDU2ODA4OQ==&mid=2247483922&idx=1&sn=09214b2223b780afe441d481ae367c0b&chksm=e80db464df7a3d72b57bf1158478ad3f55c546be15af811f10dc262e0262832c9d67ff887047&scene=21#wechat_redirect&fileGuid=GHTc3PcpPCR3xwvg)



