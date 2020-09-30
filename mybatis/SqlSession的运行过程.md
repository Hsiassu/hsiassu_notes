### SqlSession的运行过程

sqlSession是个接口

#### 映射器（Mapper）的动态代理

![image-20200930161603938](assets/image-20200930161603938.png)



![image-20200930161928621](assets/image-20200930161928621.png)

![image-20200930162255469](assets/image-20200930162255469.png)

![image-20200930162311955](assets/image-20200930162311955.png)![image-20200930162333636](assets/image-20200930162333636.png)

#### MyBatis使用Mapper接口能运行SQL的原因

Mapper的命名空间对应的就是接口的全路径 根据全路径和接口名就能够通过动态代理，让接口实现。采用命令模式 ，最后还是使用SqlSession的接口方法执行查询，说白了就是套了一层 ，实际上还是用sqlSession实现 只是多了层封装。

