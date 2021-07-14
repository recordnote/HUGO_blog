---
title: hashcode总结
date: 2021-01-23T14:21:26+08:00
lastmod: 2021-01-23T14:21:26+08:00
author: Aaron
avatar: /me/yy.jpg
cover: /img/java.png
categories:
  - Java
tags:
  - Java
weight: 1
---

## 一、hashcode是什么？

### 1、hash和hash表是什么？　　　

​		想要知道这个hashcode，首先得知道hash，通过百度百科看一下

​             <img src="https://gitee.com/aaronlynn/picture/raw/master/img/image-20210714203416559.png" alt="img" style="zoom: 50%;" />	 

​	  <img src="https://gitee.com/aaronlynn/picture/raw/master/img/874710-20161116202950092-1019467368.png" alt="img" style="zoom: 80%;" />  

​		hash是一个函数，该函数中的实现就是一种算法，就是通过一系列的算法来得到一个hash值，这个时候，我们就需要知道另一个东西，hash表，通过hash算法得到的hash值就在这张hash表中，也就是说，hash表就是所有的hash值组成的，有很多种hash函数，也就代表着有很多种算法得到hash值，如上面截图的三种，等会我们就拿第一种来说。

 

### 2、hashcode　

​		有了前面的基础，这里讲解就简单了，hashcode就是通过hash函数得来的，通俗的说，就是通过某一种算法得到的，hashcode就是在hash表中有对应的位置。

​		每个对象都有hashcode，对象的hashcode怎么得来的呢？

​		首先一个对象肯定有物理地址，在别的博文中会hashcode说成是代表对象的地址，这里肯定会让读者形成误区，对象的物理地址跟这个hashcode地址不一样，hashcode代表对象的地址说的是对象在hash表中的位置，物理地址说的对象存放在内存中的地址，那么对象如何得到hashcode呢？通过对象的内部地址(也就是物理地址)转换成一个整数，然后该整数通过hash函数的算法就得到了hashcode，所以，hashcode是什么呢？就是在hash表中对应的位置。这里如果还不是很清楚的话，举个例子，hash表中有 hashcode为1、hashcode为2、(...)3、4、5、6、7、8这样八个位置，有一个对象A，A的物理地址转换为一个整数17(这是假如)，就通过直接取余算法，17%8=1，那么A的hashcode就为1，且A就在hash表中1的位置。肯定会有其他疑问，接着看下面，这里只是举个例子来让你们知道什么是hashcode的意义。

 

## 二、hashcode有什么作用呢？

​		前面说了这么多关于hash函数，和hashcode是怎么得来的，还有hashcode对应的是hash表中的位置，可能大家就有疑问，为什么hashcode不直接写物理地址呢，还要另外用一张hash表来代表对象的地址？接下来就告诉你hashcode的作用，

​		1、HashCode的存在主要是为了查找的快捷性，HashCode是用来在散列存储结构中确定对象的存储地址的(后半句说的用hashcode来代表对象就是在hash表中的位置)

​		为什么hashcode就查找的更快，比如：我们有一个能存放1000个数这样大的内存中，在其中要存放1000个不一样的数字，用最笨的方法，就是存一个数字，就遍历一遍，看有没有相同得数，当存了900个数字，开始存901个数字的时候，就需要跟900个数字进行对比，这样就很麻烦，很是消耗时间，用hashcode来记录对象的位置，来看一下。hash表中有1、2、3、4、5、6、7、8个位置，存第一个数，hashcode为1，该数就放在hash表中1的位置，存到100个数字，hash表中8个位置会有很多数字了，1中可能有20个数字，存101个数字时，他先查hashcode值对应的位置，假设为1，那么就有20个数字和他的hashcode相同，他只需要跟这20个数字相比较(equals)，如果每一个相同，那么就放在1这个位置，这样比较的次数就少了很多，实际上hash表中有很多位置，这里只是举例只有8个，所以比较的次数会让你觉得也挺多的，实际上，如果hash表很大，那么比较的次数就很少很少了。 通过对原始方法和使用hashcode方法进行对比，我们就知道了hashcode的作用，并且为什么要使用hashcode了

## 三、equals方法和hashcode的关系？

​		通过前面这个例子，大概可以知道，先通过hashcode来比较，如果hashcode相等，那么就用equals方法来比较两个对象是否相等，用个例子说明：上面说的hash表中的8个位置，就好比8个桶，每个桶里能装很多的对象，对象A通过hash函数算法得到将它放到1号桶中，当然肯定有别的对象也会放到1号桶中，如果对象B也通过算法分到了1号桶，那么它如何识别桶中其他对象是否和它一样呢，这时候就需要equals方法来进行筛选了。

​	1、如果两个对象equals相等，那么这两个对象的HashCode一定也相同

​	2、如果两个对象的HashCode相同，不代表两个对象就相同，只能说明这两个对象在散列存储结构中，存放于同一个位置

这两条你们就能够理解了。

 

## 四、为什么equals方法重写的话，建议也一起重写hashcode方法？

​		如果对象的equals方法被重写，那么对象的HashCode方法也尽量重写）

​		举个例子，其实就明白了这个道理，

​		比如：有个A类重写了equals方法，但是没有重写hashCode方法，看输出结果，对象a1和对象a2使用equals方法相等，按照上面的hashcode的用法，那么他们两个的hashcode肯定相等，但是这里由于没重写hashcode方法，他们两个hashcode并不一样，所以，我们在重写了equals方法后，尽量也重写了hashcode方法，通过一定的算法，使他们在equals相等时，也会有相同的hashcode值。

　　　　　　　　　　　　　　![img](https://gitee.com/aaronlynn/picture/raw/master/img/874710-20161116212819810-31016841.png)

 

​		实例：现在来看一下String的源码中的equals方法和hashcode方法。这个类就重写了这两个方法，现在为什么需要重写这两个方法了吧？

​		equals方法：其实跟我上面写的那个例子是一样的原理，所以通过源码又知道了String的equals方法验证的是两个字符串的值是否一样。还有Double类也重写了这些方法。很多类有比较这类的，都重写了这两个方法，因为在所有类的父类Object中。equals的功能就是 “==”号的功能。你们还可以比较String对象的equals和==的区别啦。这里不再说明。

![img](https://gitee.com/aaronlynn/picture/raw/master/img/874710-20161116213248701-1976448309.png) 

 

​			hashcode方法![img](https://gitee.com/aaronlynn/picture/raw/master/img/874710-20161116213322170-1989555882.png)

 