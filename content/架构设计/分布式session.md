---
title: 分布式session
date: 2021-06-31T14:21:26+08:00
lastmod: 2021-06-31T14:21:26+08:00
author: Aaron
avatar: /me/yy.jpg
cover: /img/java.png
images:
  - /img/java.png
categories:
  - 架构设计
tags:
  - 架构设计
weight: 1
---

## 1.传统session

<img src="https://gitee.com/aaronlynn/picture/raw/master/img/image-20210819104404146.png" alt="image-20210819104404146" style="zoom:50%;" />  

session存在服务端tomcat里面，coockie存在客户端，请求过程如下：

<img src="https://gitee.com/aaronlynn/picture/raw/master/img/image-20210819113513752.png" alt="image-20210819113513752" style="zoom: 67%;" /> 

第一次请求的时候，服务端设置session操作，<font color='red'>**同时服务端会设置浏览器cookie（设置在响应头上，给浏览器）**</font>

<img src="C:/Users/Aaron/AppData/Roaming/Typora/typora-user-images/image-20210819172639554.png" alt="image-20210819172639554" style="zoom: 67%;" /> 

第二次请求的时候，请求头会带上cookie

<img src="https://gitee.com/aaronlynn/picture/raw/master/img/image-20210819172935173.png" alt="image-20210819172935173" style="zoom:67%;" /> 

ps：由于cookie存储在客户端，可以被修改，相对于服务端session不安全

## 2.Spring-Session

<img src="https://gitee.com/aaronlynn/picture/raw/master/img/image-20210819104132378.png" alt="image-20210819104132378" style="zoom: 50%;" /> 

原理：对HttpServletRequest进行拦截，源码org.springframework.session.web.http.SessionRepositoryFilter.SessionRepositoryRequestWrapper

引入依赖：

<img src="https://gitee.com/aaronlynn/picture/raw/master/img/image-20210819181627300.png" alt="image-20210819181627300" style="zoom: 67%;" /> 

加配置

<img src="https://gitee.com/aaronlynn/picture/raw/master/img/image-20210819181839747.png" alt="image-20210819181839747" style="zoom:67%;" /> 

## 3.Token + Redis

使用token自由度会比较高

代码示例：

<img src="https://gitee.com/aaronlynn/picture/raw/master/img/image-20210819183927132.png" alt="image-20210819183927132" style="zoom: 67%;" /> 

## 4.JWT

<img src="https://gitee.com/aaronlynn/picture/raw/master/img/image-20210819184416567.png" alt="image-20210819184416567" style="zoom: 67%;" /> 

JWT（Json web token），一种协议，其他语言也可用。

通过算法和自定义key会将用户信息加密生成一个token，通过base64编码将token发送给客户端，服务器不需要记录任何东西，每一个是无状态请求，通过解密来验证是否合法。

JWT：token里的内容可以被解析，但不能被篡改，敏感信息不能存在里面。

Spring-Session/Token+Redis 信息存储在服务端，泄露也解析不了

示列代码：

<img src="https://gitee.com/aaronlynn/picture/raw/master/img/image-20210819210010606.png" alt="image-20210819210010606" style="zoom:67%;" /> 

<img src="https://gitee.com/aaronlynn/picture/raw/master/img/image-20210819210056048.png" alt="image-20210819210056048" style="zoom:67%;" /> 

## 5.OAuth2

一种第三方授权机制，协议标准。

例如作为第三方去获取QQ，微信等账户名称、性别等信息。

<img src="https://gitee.com/aaronlynn/picture/raw/master/img/image-20210819211616383.png" alt="image-20210819211616383"  /> 

spring-security-oauth2：这个是授权数据给第三方、还可作权限管理。

​             
