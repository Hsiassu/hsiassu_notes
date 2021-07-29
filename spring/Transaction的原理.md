# Transaction的原理



怎么来的

首先是TranscationInterceptor 为啥是这个呢 因为代理之后看到的实际对象是这个

这个东西继承了 TransactionAspectSupport 

观察 TransactionInterceptor 的构造方法 被调用的时间 发现org.springframework.transaction.config.internalTransactionAdvisor

这个beanDefinition 会被bean后置处理器 InfrustartAutoProxyCreator 调用  通过选择

![image-20210728161653231](assets/image-20210728161653231.png)从一堆beanDefinition中抓到   

![image-20210728161738144](assets/image-20210728161738144.png)





![image-20210728164242036](assets/image-20210728164242036.png)

有点儿尴尬 看了半天发现就是一开始那个地方引入的 悲

![image-20210728172837196](assets/image-20210728172837196.png)



就很骚

就是那个internalTransactionAdvisor  可喜可贺的是TranscationINterceptor   TransactionAspectSupport好像也在这里

![image-20210729092041083](assets/image-20210729092041083.png)

为了回忆下 advice = 增强   pointcut 切入点      advice+pointcut = advisor 

![image-20210729085158825](assets/image-20210729085158825.png)

所以在bean生成的时候 会经过一个InfrustrAutoProxyCreator（beanPostProcessor) 就是之前一起注入的那个  这个会判断是否需要包装   

![image-20210729085626152](assets/image-20210729085626152.png)

然后就是从beanFactory中寻找可能Advisor 一般情况下就是一起注入的那个BeanFactoryTransactionAttributeSourceAdvisor(org.springframework.transaction.config.internalTransactionAdvisor)



![image-20210729085825884](assets/image-20210729085825884.png)

没其他人会注入了  悲

![image-20210729090050924](assets/image-20210729090050924.png)

然后实例化这个internalTransactionAttributeSourceAdvisor 的时候就会去解析构造函数 实例化这个TransactionInterceptor

说白了就是执行那个InfrustrAutoProxy的beanPostProcessor会去找可用的Advisor  也就是上面的transactionAttributeSourceAdvisor 然后去构建这个bean 构造器解析器会去构造参数 也就是 tranSactionAttributeSource和TransactionInterceptor



![image-20210729100439039](assets/image-20210729100439039.png)



然后就是一个TransactionResource TransactionInterceptor TransactionManager 

TransactionStatus 初始状态 

然后进行实务操作





然后还是老虎案例 DataSourceTransacti自动注入了一个TransactionManager 懒加载



说白了还是什么时候注入的

按照猜想应该是通过BeanPostProcessor那个时候注入的

也就是这个东西实际生成的时候 通过某个后置处理器写进去的



首先明确这个东西还是规规矩的在 bean 生成的时候进行

```
TransactionAutoConfiguration
```

​	![image-20210728124519145](assets/image-20210728124519145.png)

首先老黄历了

ConfigurationClassPostProcessor获取了注解@AutowiredOn然后开始自动装配这些类到candidate

这些candidate同时被注入进去 然后也是一样的解析成COnfiguration  

![image-20210728135040093](assets/image-20210728135040093.png)

![image-20210728140136046](assets/image-20210728140136046.png)是吧还是有这个EnableTransactionManageMent的

![image-20210728124809011](assets/image-20210728124809011.png)

![image-20210728135055075](assets/image-20210728135055075.png)

![image-20210728135018224](assets/image-20210728135018224.png)

### AutoRegister

```
AutoProxyRegistrar
```

![image-20210728135924996](assets/image-20210728135924996.png)

一大堆没用的说白了就是为了获取代理类型

实际上就是给ioc容器beanDefinitionMap加了一个beanDefinition

![image-20210728140324620](assets/image-20210728140324620.png)

![image-20210728140341671](assets/image-20210728140341671.png)

这个叫什么基础通知器自动代理创造者

然后Aop的自动装配类

![image-20210728141703921](assets/image-20210728141703921.png)





emmmm扯远了

好像这样没有看到这个类是如何被代理的

拿到过来思维一下 我们首先通过rollback找到实际代理的对象

![image-20210728150224579](assets/image-20210728150224579.png)

跟踪他的构造方法就知道他在哪里被写入了 挺好的 这方法就很好

![image-20210728150334300](assets/image-20210728150334300.png)

看调用链就知道是在 倒数第二部 实例化 初始化 非懒加载的 单例bean的时候 执行了BeanPostProcessor的后置处理器 的 After方法 赋值的  那再看看  是哪个BeanPostProcessor

![image-20210728150512273](assets/image-20210728150512273.png)

j就很巧 就是我们上面AutoProxyUtil 加入的那个BeanPostProcessor 

![image-20210728150721350](assets/image-20210728150721350.png)

找到org.springframework.transaction.config.internalTransactionAdvisor 

![image-20210728152014448](assets/image-20210728152014448.png)

getAdvicesAndAdvisorsForBean方法会遍历容器中所有的切面，查找与当前实例化bean匹配的切面，这里就是获取事务属性切面，查找@Transactional注解及其属性值，具体实现比较复杂，这里暂不深入分析，最终会得到BeanFactoryTransactionAttributeSourceAdvisor实例，然后根据得到的切面进入createProxy方法，创建一个AOP代理。



这里只关注JdkDynamicAopProxy的invoke方法的重点代码。

this.advised.getInterceptorsAndDynamicInterceptionAdvice获取的是当前目标方法对应的拦截器，里面是根据之前获取到的切面来获取相对应拦截器，这时候会得到TransactionInterceptor实例。如果获取不到拦截器，则不会创建MethodInvocation，直接调用目标方法。这里使用TransactionInterceptor创建一个ReflectiveMethodInvocation实例，调用的时候进入ReflectiveMethodInvocation的proceed方法。

代码中的interceptorOrInterceptionAdvice就是TransactionInterceptor的实例，执行invoke方法进入TransactionInterceptor的invoke方法。

![image-20210728160836086](assets/image-20210728160836086.png)

![img](assets/38e5d8b2f765124fce19dc49f493473d)

