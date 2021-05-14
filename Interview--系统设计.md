后端程序设计题：日uv两千万，设计一个投票系统，要求限制每个用户任意十分钟内只能投五票。给出系统设计以及需要资源情况。

# 接口幂等性

效果：系统对某接口的多次请求，都应该返回同样的结果！（网络访问失败的场景除外）

目的：避免因为各种原因，重复请求导致的业务重复处理

幂等性的实现方式

实现方法：客户端做某一请求的时候带上识别参数标识，服务端对此标识进行识别，重复请求则重复返回第一次的结果即可。

举个栗子：比如添加请求的表单里，在打开添加表单页面的时候，就生成一个AddId标识，这个AddId跟着表单一起提交到后台接口。

后台接口根据这个AddId，服务端就可以进行缓存标记并进行过滤，缓存值可以是AddId作为缓存key，返回内容作为缓存Value，这样即使添加按钮被多次点下也可以识别出来。

这个AddId什么时候更新呢？只有在保存成功并且清空表单之后，才变更这个AddId标识，从而实现新数据的表单提交


# 单点登录实现

[https://mp.weixin.qq.com/s?__biz=MzIyNDU2ODA4OQ==&mid=2247483960&idx=1&sn=d6502108ecd21d1f923da420d8e0f3aa&chksm=e80db44edf7a3d580b2af7c167c1c2102d258657ef7964ba7947e28059a7d1faca6cb841d48a&scene=21#wechat_redirect](https://mp.weixin.qq.com/s?__biz=MzIyNDU2ODA4OQ==&mid=2247483960&idx=1&sn=d6502108ecd21d1f923da420d8e0f3aa&chksm=e80db44edf7a3d580b2af7c167c1c2102d258657ef7964ba7947e28059a7d1faca6cb841d48a&scene=21#wechat_redirect&fileGuid=xyjwRTK9vxw9wjGG)





