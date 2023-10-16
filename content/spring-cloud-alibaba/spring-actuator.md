---
title: Spring Boot Actuator:健康检查、审计、统计和监控
date: 2023-10-03T14:21:26+08:00
lastmod: 2023-10-03T14:21:26+08:00
author: Lin
avatar: /me/yy.jpg
cover: /img/springcloud.jfif
images:
  - /img/springcloud.jfif
categories:
  - Spring框架
tags:
  - Spring
weight: 1
---
Spring Boot Actuator可以帮助你**监控和管理**Spring Boot应用，比如健康检查、审计、统计和HTTP追踪等。所有的这些特性可以通过JMX或者HTTP endpoints来获得。

Actuator同时还可以与外部应用监控系统整合，比如 Prometheus, Graphite, DataDog, Influx, Wavefront, New Relic等。这些系统提供了非常好的仪表盘、图标、分析和告警等功能，使得你可以通过统一的接口轻松的监控和管理你的应用。

Actuator使用Micrometer来整合上面提到的外部应用监控系统。这使得只要通过非常小的配置就可以集成任何应用监控系统。

## 1、常用端点（路径）

### 1.1 配置：

<img src="https://cdn.jsdelivr.net/gh/recordnote/cdn/img/111.png" alt="111" style="zoom: 67%;" /> 

### 1.2 端点表：

<img src="https://cdn.jsdelivr.net/gh/recordnote/cdn/img/2.png" alt="2" style="zoom: 80%;" /> 

### 1.3 自定义指定外部配置（**IDE配置优先级>配置文件优先级**）

<img src="https://cdn.jsdelivr.net/gh/recordnote/cdn/img/3.png" alt="3" style="zoom: 67%;" /> 

## 2、Profile的使用（环境切换配置）

<img src="https://cdn.jsdelivr.net/gh/recordnote/cdn/img/4.png" alt="image-20210703150924732" style="zoom:80%;" /> 

PS：此处active可用include替换，但include 包含的profile会**无条件的active**, 而active 包含的profile 有可能会被优先级高的定义覆盖

<img src="https://cdn.jsdelivr.net/gh/recordnote/cdn/img/image-2021070352215.png" alt="image-20210703151252215" style="zoom:80%;" /> 

**原则：公用的放公用的文件中，私用放私用的文件。**

<img src="https://cdn.jsdelivr.net/gh/recordnote/cdn/img/image-20703152534646.png" alt="image-20210703152534646" style="zoom: 67%;" /> 

<img src="https://cdn.jsdelivr.net/gh/recordnote/cdn/img/image-20210703157602.png" alt="image-20210703152547602" style="zoom:67%;" /> 

<img src="https://cdn.jsdelivr.net/gh/recordnote/cdn/img/image-20210752555911.png" alt="image-20210703152555911" style="zoom: 67%;" /> 
