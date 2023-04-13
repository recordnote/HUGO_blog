---
title: MyBatis相关
date: 2022-06-31T14:21:26+08:00
lastmod: 2022-06-31T14:21:26+08:00
author: Aaron
avatar: /me/yy.jpg
cover: /img/MySql.png
images:
  - 
categories:
  - 数据库
tags:
  - 数据库
weight: 1
---

## 1、MyBatis中嵌套查询和嵌套结果的区别？



最后一个问题是问Thread.sleep()和Object.wait()调用时线程处于什么状态

项目里springboot和spring的版本



HTTP和HTTPS的区别



tcp和udp区别， 三次握手四次挥手

HTTP状态码



分布式怎么理解的？

负载均衡策略？

eureka底层，挂了怎么办？

spring 扯到 ioc，aop    AspectJ类？AOP具体能做什么场景（权限，日志等）

再到  代理模式   jdk代理和cglib代理

spring管理生成的对象，对象生命周期，spring的作用域有哪些？

Spring创建的是单例，线程安全是怎么解决？



MyBatis

dollar和#的区别？

dao里面的接口，怎么和xml中的查询语句进行绑定的？



mysql隔离级别？

mysql死锁？

mysql优化？具体做过什么？

mysql哪些索引？底层（B+Tree）？B+Tree和红黑树有什么区别？

聚集索引和非聚集索引？组合索引？联合索引？



redis

怎么保证缓存与数据库一致性

哪些数据类型？RDB和AOF？



RESTful风格？



Linux 命令？查看日志？





对分布式如何理解
项目分了多少种服务
业务服务
SpringCloud用了哪些组件
负载均衡策略
辅助中心的底层
服务挂了怎么办
spring的理解（ioc和aop）
jdk动态代理和cglib动态代理区别
性能哪个更好
AspectJ
Bean的作用域
spring默认创建的是单例的，单例存在了线程安全问题，怎么解决单例（ThreadLocal解决）
ORM框架
\#{}和${}区别
接口种的方法如何和xml绑定
mybatis的xml中国常用的标签有哪些
嵌套查询和嵌套结果区别有哪些
插入一条数据获取这条数据的组件的返回值
hashmap底层是怎么样的
hashmap扩容
初始容量
为什么总是扩容为2的次幂
cookie和session的区别
浏览器把cookie禁用了怎么办
数据库的隔离级别有哪些
脏读？幻读？不可重复读？
数据库的死锁
数据库的优化
水平分表后如何确定数据在哪个表里
具体做过哪些数据库查询优化
mysql支持的索引类型有哪些
索引的底层结构
B+Tree的底层结构
和红黑树的区别
聚集索引和非聚集索引
怎么使用redis
怎么保证对数据库的双写一致性
redis还了解什么
rda和aof的持久化策略有什么没区别
哪种效率高
项目里用了什么持久化策略
什么是Restful风格
接口幂等性
linux如何动态查看日志（tail -f)

spring单例缓存查找，aop没答到点，可能是代理模式不清楚吧，InvocationHandler或者MethodInterceptor，spring cloud 心跳检查，建议以后问问spring4或者5的新特性，mybatis接口绑定没答出来，hashMap底层不懂，起码说一下头插尾插，1.8之后的优化put get什么的，还有就是扩容好像就不懂一样，数据事务处理就太混乱了，不忍直视，数据库要是问问底层索引，innerdb，myism索引优化就可能直接出门右转，知道的问题研究太浅了，数据库优化真的是了解太浅了。redis了解的太少，aof和rdb答得不好，up下次问问redlock；restful没有达到点，接口幂等性完全不懂，linux完全不懂，tail-  f，但是说知道docker以后可以考考docker基础知识







说一下你对分布式理解是什么样的？
springcloud里面的几个组件？
权限是怎么设计的？
项目用的权限框架是什么？
jwt？
分布式事务是怎么处理的？
tcc 二阶段提交 有没有基于tcc的一些其他解决方案 比如说最大努力通知(基于消息队列来说)？
rocket mq用过吗？
更新数据库 redis缓存数据怎么去更新(双写一致性)？
redis除了做缓存你们还做其他东西吗？
redis有没有用它做分布式锁？
zookeeper是怎么去实现分布式锁？
redis缓存和我们框架项目去做缓存，有什么其他区别(比如用map做缓存)？
数据库优化？
有没有用过相关命令去查看sql执行？
分表！分好以后怎么去确认数据在那张表？
分表策略？
分表以后查询一个范围但跨表了？怎么办？
union和unionall区别？
索引有去了解他的底层吗
B+tree能做到什么优化？ 它和二叉树有什么不同？
myisam和Innodb区别引擎？
有没有遇到锁表的情况？



### 自己整理的题目？

Mysql数据库的连接？？左连接和右连接？

TCP和UDP？

进程和线程间的通信？进程和线程之间的关系？



