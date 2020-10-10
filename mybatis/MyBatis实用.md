## MyBatis实用

1.设置执行器为批量提交

== commit之后一起执行sql

所以会出现执行sql语句后 如果没有提交事务 没法获取到变更结果的情况	

![image-20201009115352596](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20201009115352596.png)

通过flushStatements 提交缓存的sql



#### 分页RowBound

RowBound 查询出所有结果再分页 性能有很大问题