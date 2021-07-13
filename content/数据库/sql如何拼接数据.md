---
title: sql如何拼接数据
date: 2021-02-23T14:21:26+08:00
lastmod: 2021-02-23T14:21:26+08:00
author: Aaron
avatar: /me/yy.jpg
cover: /img/shujuku.jpg
images:
  - /img/shujuku.jpg
categories:
  - 数据库
tags:
  - 数据库
weight: 1
---

数据库中的例子表格：

<img src="https://gitee.com/aaronlynn/picture/raw/master/img/image-20210713153514969.png" alt="image-20210713153514969" style="zoom: 80%;" /> 

##  1.符号 +

   在MySQL中支持使用加号拼接结果。在两个字段都是整型时，都会返回两个整型值得和，但是在两个字段中有一个为字符串时，MySQL的返回结果不是拼接两字符串，而是默认字符串为零，再返回两个字段之和。

~~~ java
//返回157
select Score2 + Score1 FROM mytable where ID = 1 ;
//返回90
select Name + Score1 from mytable where ID = 1 ;
如上所示，因为Name字段不是整型，所以相当于Score1 + 0，所以返回值就是Score1的值，即90；另外，如果相加的两个字段全都是字符串，则返回0。
~~~

##    2.CONCAT

   CONTACT的功能是直接将数据按照字符串格式拼接，类似于Sql Server中加号拼接字符串的功能。

~~~ java
select CONCAT(ID,Name,Score1) from MyTable where ID = 1;
//返回值为 1A90;
ps：CONCAT后面括号中的参数只要有一个值为null，整个函数的返回值就会为null。
~~~

![image-20210713154351346](https://gitee.com/aaronlynn/picture/raw/master/img/image-20210713154351346.png)   

## 3.CONCAT_WS

   CONOCAT_WS的用法和CONCAT类似，但是它的第一个参数为分隔符，在返回的值中，每一个参数之间都会有第一个参数作为分隔。

~~~ java
select CONCAT_WS('-',ID,Name,Score1) from MyTable where ID = 1;
//返回值为 1-A-90;
   如果CONCAT_WS的第一个参数为null，则返回值为null，如果后面的参数中有null，则这些参数会被忽略，只返回其他参数和分隔符组成的字符串。
~~~

![image-20210713154436517](https://gitee.com/aaronlynn/picture/raw/master/img/image-20210713154436517.png)   

##  4.根据字段拼接 GROUP_CONTACT


   在我们使用数据库时，会碰到这样一种情况：同一个Name的人有两条数据，但是他们的其他数据并不相同，而我们所需要的是同一个Name的人的Score1集合，即需要获得Name为A的所有的Score1并且希望将其拼接为一个字段，这里我们就需要用GROUP_CONCAT。

函数语法如下：group_concat( 要连接的字段 )  [Order BY 排序字段 ASC/DESC]   [Separator '分隔符'] 

~~~ java
select Name,GROUP_CONCAT(Score1 Separator '-') FROM MyTable GROUP BY Name;
-- 其中GROUP BY 后面的字段是Name，所有同一排序字段的数据会被拼接后存入同一字段中，并以相应的分隔符分分隔。
  
~~~

如上的sql语句，返回值为：

<img src="https://gitee.com/aaronlynn/picture/raw/master/img/image-20210713154527401.png" alt="image-20210713154527401" style="zoom: 80%;" /> 

   因为上面的数据中只有Name为A和D的两项数据有重复，所有他们的Score1是拼接出来的。

   ps：GROUP_CONCAT是有长度限制的，MySQL对其的默认长度限制为1024，可以使用如下语句来进行修改长度：

~~~ java
SHOW VARIABLES LIKE "%group_concat_max_len%"//查看长度

SET SESSION group_concat_max_len = 102400; //修改长度
~~~

