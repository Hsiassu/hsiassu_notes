首先这是一个强大的东西

实现了BeanFactoryPostProcessor 和他的优先级子类 BeanDefinitionRegistry emmmm一个在beanDefinition 加载后 一个在加载前 执行回调emmmmm 

把其他自动装配的beanFactoryPostProcessor加入进IOC容器中

一切是从他开始的



#### 加载时机

从源码中可以看到 这个东东本质上是从DefaultListableBeanFactory中加载的 

![image-20210727170940624](assets/image-20210727170940624.png)

从他的beanDefinitionNames里

那这个是什么时候加进去的呢

![image-20210727171100709](assets/image-20210727171100709.png)

![image-20210727171020653](assets/image-20210727171020653.png)

在这个注解工具类中 如果没有名为这个的beanDefinition 那我就送你一个 

那个这个AnnotationConfigUtil又是何方神圣呢

我们可以看到

![image-20210727171310124](assets/image-20210727171310124.png)

![image-20210727171333493](assets/image-20210727171333493.png)

![image-20210727171344078](assets/image-20210727171344078.png)

![image-20210727171359448](assets/image-20210727171359448.png)

![image-20210727171417487](assets/image-20210727171417487.png)

![image-20210727171446876](assets/image-20210727171446876.png)

emmmm结论就是 在web容器创建 并做准备工作的时候 spring开始为他的beanDefinition配置了加载器 这个加载器默认会加入这个类（在没有的情况下）

#### 作用

解析Configuration类 识别注解 引入这些注解Import 或是实现ImportBeanDefinitionRegister的方法   



自动装配和这个息息相关



因为我们进入@Sp

#### ConfigurationClassParse

![image-20210728101102018](assets/image-20210728101102018.png)

大白话 这个方法就是 构建完整的配置的文件  通过读取注解 成员 方法 从源类中   

@PropertyResource

通过这个ComponentScan调用 过滤啥啥的  最终通过ClassPathBeanDefinitionScanner来获取这些beanDefinition信息（还不是bean)

当然还有@Import @ImportResource 处理来源的文件   处理单独的@Bean

![image-20210728101642761](assets/image-20210728101642761.png)

具体来说就是这个ClassPathScanningCandidateProvider 干的事情了 找到对应目录下的包 然后对应的class尾椎

![image-20210728102158261](assets/image-20210728102158261.png)

![image-20210728102331384](assets/image-20210728102331384.png)

获取了一堆包 然后根据ComponentScanner的TypeFilter搞个过滤



![image-20210728100939942](assets/image-20210728100939942.png)

#### 关于主类的加载是在 初始化 beanDefinitionLoader 开始的 这里传入了args[0] 

![image-20210728095122044](assets/image-20210728095122044.png)

![image-20210728095138106](assets/image-20210728095138106.png)

![image-20210728095147553](assets/image-20210728095147553.png)

用一个BeanDefinitionHolder装载进去

#### @Mapper的加载

这个东西不是Component 所以按照概率大概率也是 要独立的BeanDefinitonPostProcessor 来处理了



```
org.mybatis.spring.mapper;
MapperScannerConfigurer
```

这个实现的

这个本质上和上面![image-20210728105837475](assets/image-20210728105837475.png)



一样 都是整体扫描进来 然后根据TypeFilter来判断

这里就是MyBatis 自己实现了一个Filter 规定注解有Mapper的就可以过 所以被加入进来了 

![image-20210728111234491](assets/image-20210728111234491.png)

那么 现在问题来了 这个Mapper   BeanDefinitionRegisterPostprocessor是什么时候被加到ioc容器中的呢

按照惯例 肯定是自动装配进去的 那我们来看看

![image-20210728111400956](assets/image-20210728111400956.png)

emmmm果然自动转配进了一个类  会把这个类的信息加入beanDefinition中作为Configuration类解析 

![image-20210728111553935](assets/image-20210728111553935.png)

老花样了  先实现一个ImportBeanDefinitionRegister接口 这样就会被执行装配进beanDefinition中，然后被调用registerBeanDefinition这个方法...结果就是在这里面为registy（beanFactory)

![image-20210728111726130](assets/image-20210728111726130.png)

![image-20210728111731031](assets/image-20210728111731031.png)

加入一个BeanDefinitionRegistryPostProcessor

![image-20210728112730806](assets/image-20210728112730806.png)

emmmmm老ImportBeanDefinitionRegistry了 Configuration后会调用这个方法

#### 关于FactoryBean调用的一点说明

所有beanDefinition会经过DataInit![image-20210728114546652](assets/image-20210728114546652.png)

然后如果实现了factoryBean接口 会被提前实例化这样 

![image-20210728114637108](assets/image-20210728114637108.png)

这个东西是在autoconfigure里的被Import的 懒得找了

![image-20210728114733986](assets/image-20210728114733986.png)

![image-20210728114756246](assets/image-20210728114756246.png)

