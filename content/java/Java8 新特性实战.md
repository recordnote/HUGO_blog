---
title: Java 8 新特性实战指南
date: 2024-05-18T14:21:26+08:00
lastmod: 2024-05-18T14:21:26+08:00
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

# Java 8 新特性实战指南

## 目录
- [接口的默认方法与静态方法](#接口的默认方法与静态方法)
- [函数式接口与Lambda表达式](#函数式接口与lambda表达式)
- [Stream API](#stream-api)
- [Optional类](#optional类)
- [新的日期时间API](#新的日期时间api)

---

## 接口的默认方法与静态方法

### 设计初衷
- 解决接口修改与现有实现不兼容的问题
- 允许接口包含具体方法实现（默认方法）
- 支持静态方法定义

### 核心特性
```java
public interface InterfaceNew {
    static void sm() { 
        System.out.println("接口静态方法");
    }
    
    default void def() {
        System.out.println("默认方法");
    }
    
    void f(); // 抽象方法
}
```

### 使用规范
1. **默认方法**：
   - 实例方法，可用`this`调用
   - 可被子类继承/重写
2. **静态方法**：
   - 只能通过接口名调用
   - 不能被实现类继承

### 冲突解决
```java
public class InterfaceNewImpl implements InterfaceNew, InterfaceNew1 {
    @Override
    public void def() {
        InterfaceNew1.super.def(); // 显式指定接口
    }
}
```

### 接口 vs 抽象类
| 特性       | 接口                | 抽象类         |
| ---------- | ------------------- | -------------- |
| 继承方式   | 多实现              | 单继承         |
| 方法修饰符 | public abstract     | 任意访问修饰符 |
| 变量修饰符 | public static final | 无限制         |
| 设计目的   | 行为扩展            | 代码复用       |

---

## 函数式接口与Lambda表达式

### 函数式接口定义
```java
@FunctionalInterface
public interface Comparator<T> {
    int compare(T o1, T o2);
}
```

### Lambda语法
```java
// 传统方式
new Thread(new Runnable() {
    @Override 
    public void run() {
        System.out.println("传统实现");
    }
}).start();

// Lambda实现
new Thread(() -> System.out.println("Lambda表达式")).start();
```

### 方法引用
```java
List<String> list = Arrays.asList("a", "b", "c");
list.forEach(System.out::println);  // 静态方法引用
list.forEach(String::toUpperCase);  // 实例方法引用
```

### 变量捕获
- 只能引用final或等效final的局部变量
- 禁止修改捕获变量

---

## Stream API

### 核心特性
- **无存储**：不存储数据
- **函数式编程**：支持lambda表达式
- **延迟执行**：终端操作触发执行
- **并行处理**：parallelStream()

### 常用操作
```java
List<String> strings = Arrays.asList("abc", "", "bc", "efg");

// 过滤空字符串
long count = strings.stream()
                   .filter(string -> !string.isEmpty())
                   .count();

// 映射转换
List<Integer> numbers = Arrays.asList(3, 2, 2, 3);
List<Integer> squares = numbers.stream()
                              .map(i -> i*i)
                              .distinct()
                              .collect(Collectors.toList());

// 并行处理
strings.parallelStream()
       .filter(s -> s.startsWith("a"))
       .forEach(System.out::println);
```

### 操作类型
| 操作类型 | 方法示例                      |
| -------- | ----------------------------- |
| 中间操作 | filter(), map(), sorted()     |
| 终端操作 | forEach(), collect(), count() |

---

## Optional类

### 空值处理
```java
public String getCity(User user) {
    return Optional.ofNullable(user)
                  .flatMap(User::getAddress)
                  .map(Address::getCity)
                  .orElse("Unknown");
}
```

### 核心方法
| 方法         | 说明                     |
| ------------ | ------------------------ |
| ofNullable() | 创建可能为null的Optional |
| map()        | 值转换                   |
| orElse()     | 默认值返回               |
| ifPresent()  | 值存在时执行操作         |

---

## 新的日期时间API

### 核心类
| 类名          | 说明               |
| ------------- | ------------------ |
| LocalDate     | 日期（yyyy-MM-dd） |
| LocalTime     | 时间（HH:mm:ss）   |
| LocalDateTime | 日期+时间          |
| ZonedDateTime | 带时区时间         |

### 使用示例
```java
// 获取当前日期
LocalDate today = LocalDate.now();

// 日期计算
LocalDate nextWeek = today.plus(1, ChronoUnit.WEEKS);

// 时区处理
ZonedDateTime tokyoTime = ZonedDateTime.now(ZoneId.of("Asia/Tokyo"));

// 日期格式化
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
String formatted = LocalDateTime.now().format(formatter);
```

### 优势对比
1. **线程安全**：所有类都是不可变的
2. **清晰设计**：分离日期/时间概念
3. **时区支持**：内置完善的时区处理
4. **链式操作**：支持流畅的API调用

---

> 完整官方文档参考：[Oracle Java 8 新特性](https://docs.oracle.com/javase/8/docs/api/)

