---
title: jdk8新特性-Optional类
date: 2023-07-14T14:21:26+08:00
lastmod: 2023-07-14T14:21:26+08:00
author: Lin
avatar: /me/yy.jpg
cover: /img/java.png
images:
  - /img/java.png
categories:
  - Java
tags:
  - Java
weight: 1
---



## 1、定义

​		Optional 类 (java.util.Optional) 是一个容器类，代表一个值存在或不存在，原来用 null 表示一个值不存在，现在用 Optional 可以更好的表达这个概念；并且可以避免空指针异常

## 2、常用API（带*号表示常用）

Optional.of(T t)：创建一个 Optional 实例

~~~ java
@Test
public void test01(){
// Optional.of - 如果传递的参数是 null，抛出异常 NullPointerException
    Optional<Employee> op = Optional.of(new Employee());
    Employee employee = op.get();
}
~~~

Optional.ofNullable(T t)：若 t 不为 null，创建 Optional 实例，否则空实例

~~~ java
@Test
public void test03(){
    Optional<Employee> op = Optional.ofNullable(new Employee());
    Employee employee = op.get();
}
~~~

isPresent()：判断是否包含某值

~~~ java
// Optional.isPresent - 判断值是否存在
//a是一个Optional<Integer>对象
System.out.println("第一个参数值存在: " + a.isPresent());

//输出    第一个参数值存在: false
~~~

orElse(T t)：如果调用对象包含值，返回该值，否则返回 t

~~~ java
// Optional.orElse - 如果值存在，返回它，否则返回默认值
//a是一个Optional<Integer>对象
    Integer value1 = a.orElse(0);
~~~


get()：Optional中包含这个值，返回值，否则抛出异常：NoSuchElementException

~~~ java
//Optional.get - 获取值，值需要存在，否则抛出异常：NoSuchElementException
//a是一个Optional<Integer>对象
   Integer value2 = a.get();
~~~

map(Function f)：如果有值对其处理，并返回处理后的 Optional，否则返回 Optional.empty()

~~~ java
Optional<Employee> op4 = Optional.ofNullable(new Employee(1, "张三"));
Optional<Integer> map = op4.map(e -> e.getAge());
System.err.println(map.get());
~~~

Optional.empty(T t)：创建一个空的 Optional 实例

orElseGet(Supplier s)：如果调用对象包含值，返回该值，否则返回 s 获取的值

flatmap(Function mapper)：与 map 相似，要求返回值必须是 Optional
