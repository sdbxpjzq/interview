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

# HashMap的实现原理

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

## 























