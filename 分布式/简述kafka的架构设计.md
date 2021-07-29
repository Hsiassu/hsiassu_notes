# 简述kafka的架构设计

cluster



MQ![image-20210725152140076](C:/Users/zhuyexin/AppData/Roaming/Typora/typora-user-images/image-20210725152140076.png)

kafka

topIC

可以存在不同的分区

broker



kafka的follower就是个备份

只有出事了才会来搞

通过zk来获取leader和follower的区别

