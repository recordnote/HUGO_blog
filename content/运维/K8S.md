---
title: K8s
date: 2022-03-31T14:21:26+08:00
lastmod: 2022-03-31T14:21:26+08:00
author: Aaron
avatar: /me/yy.jpg
cover: /img/k8s.png
images:
  - /img/k8s.png
categories:
  - 运维
tags:
  - 运维
weight: 1
---



## k8s基本概念

![image-20210828105835156](https://gitee.com/aaronlynn/picture/raw/master/img/image-20210828105835156.png)

##  k8s架构 

![x](https://gitee.com/aaronlynn/picture/raw/master/img/image-20210827220754158.png)



![image-20210827220845054](https://gitee.com/aaronlynn/picture/raw/master/img/image-20210827220845054.png)

![image-20210827091531306](https://gitee.com/aaronlynn/picture/raw/master/img/image-20210827091531306.png)

![image-20210827091507432](https://gitee.com/aaronlynn/picture/raw/master/img/image-20210827091507432.png)



![image-20210827093757348](https://gitee.com/aaronlynn/picture/raw/master/img/image-20210827093757348.png)



![image-20210827093626787](https://gitee.com/aaronlynn/picture/raw/master/img/image-20210827093626787.png) 

![image-20210827101014984](https://gitee.com/aaronlynn/picture/raw/master/img/image-20210827101014984.png)

![image-20210827101042761](https://gitee.com/aaronlynn/picture/raw/master/img/image-20210827101042761.png)

![image-20210827101655048](https://gitee.com/aaronlynn/picture/raw/master/img/image-20210827101655048.png)

![image-20210827102758422](https://gitee.com/aaronlynn/picture/raw/master/img/image-20210827102758422.png)

![image-20210827103204036](https://gitee.com/aaronlynn/picture/raw/master/img/image-20210827103204036.png)

<img src="C:/Users/Aaron/AppData/Roaming/Typora/typora-user-images/image-20210827215539933.png" alt="image-20210827215539933" style="zoom: 80%;" />

## Kubelet

- **节点代理**，**pod 管理**，处理master中**api-server**下发的节点任务，每个节点都会启动 kubelet进程

- **容器健康检查**：kubelet 创建了容器之后还要查看容器是否正常运行，如果容器运行出错，就要根据 pod 设置的重启策略进行处理。

- **容器监控**：kubelet 会监控所在节点的资源使用情况，并定时向 master 报告，资源使用数据都是通过 cAdvisor 获取的。知道整个集群所有节点的资源情况，对于 pod 的调度和正常运行至关重要。

  

## k8sPod

### Pod基本概念

- **最小部署单元**

- **一组容器的集合**

- **一个Pod中的容器共享网络命名空间与存储**

- **Pod是短暂**

### Pod存在的意义

<img src="https://gitee.com/aaronlynn/picture/raw/master/img/image-20210828165716277.png" alt="image-20210828165716277" style="zoom: 67%;" /> 

### Pod实现机制

- **共享网络**：在一个Pod中创建一个或者多个容器，前提会创建一个infrastructure Container(基础容器）维护整个Pod网络空间，其他创建容器会与该容器有联系，使用共同的网络（IP相同），相同的namespace。 
- **共享存储**：独立于Node之外的**数据卷**

### Pod容器分类与设计模式

<img src="https://gitee.com/aaronlynn/picture/raw/master/img/image-20210828172744077.png" alt="image-20210828172744077" style="zoom: 80%;" /> 

<img src="https://gitee.com/aaronlynn/picture/raw/master/img/image-20210828174835675.png" alt="image-20210828174835675" style="zoom:80%;" /> 

### Pod与controller的关系

通过应用部署controller，controller通过label-selector关联Pod

<img src="https://gitee.com/aaronlynn/picture/raw/master/img/image-20210828175441008.png" alt="image-20210828175441008" style="zoom:67%;" /> 

## k8s控制器

### Deployment控制器功能与应用场景

- 部署无状态应用
- 管理Pod与ReplicaSet
- 具有上线部署、副本设定、滚动升级、回滚等功能
- 提供声明式更新，例如只更新一个新的Image

​    **应用场景**：Web服务、微服务

### yaml字段解析

<img src="https://gitee.com/aaronlynn/picture/raw/master/img/image-20210828180541134.png" alt="image-20210828180541134" style="zoom: 80%;" /> 

## 应用升级和回滚

![image-20210828181613811](https://gitee.com/aaronlynn/picture/raw/master/img/image-20210828181613811.png)

## Service存在的意义

- **service是通过kube-proxy来实现的**

- **防止Pod失联（服务发现）**{获取可用Pod的IP列表} 
- **定义一组Pod的访问策略（负载均衡）**

## Pod与Service的关系 

![image-20210828201922756](https://gitee.com/aaronlynn/picture/raw/master/img/image-20210828201922756.png) 

![image-20210828201938583](https://gitee.com/aaronlynn/picture/raw/master/img/image-20210828201938583.png) 

## Service类型

![image-20210828212256785](https://gitee.com/aaronlynn/picture/raw/master/img/image-20210828212256785.png)

### 第一种 

<img src="https://gitee.com/aaronlynn/picture/raw/master/img/image-20210828205053823.png" alt=" " style="zoom: 80%;" /> 

### 第二种

<img src="https://gitee.com/aaronlynn/picture/raw/master/img/image-20210828205114006.png" alt="image-20210828205114006" style="zoom:80%;" /> 

### 第三种（主流 ）

  ![image-20210828205132983](https://gitee.com/aaronlynn/picture/raw/master/img/image-20210828205132983.png)

## Deployment和StatefulSet-有状态应用守护者

<img src="https://gitee.com/aaronlynn/picture/raw/master/img/image-20210831112312100.png" alt="image-20210831112312100" style="zoom:50%;" /> 

 <img src="https://gitee.com/aaronlynn/picture/raw/master/img/image-20210831112339399.png" alt="image-20210831112339399" style="zoom:50%;" />

## Ingress Controller

**意义**：在Kubernetes中，服务和Pod的IP地址仅可以在集群网络内部使用，对于集群外的应用是不可见的。为了使外部的应用能够访问集群内的服务，在Kubernetes 目前 提供了以下几种方案：NodePort、LoadBalancer、Ingress

**Ingress是基于域名，负载均衡还是用nginx**

![image-20210828225825215](https://gitee.com/aaronlynn/picture/raw/master/img/image-20210828225825215.png) 

 ![image-20210829222500523](https://gitee.com/aaronlynn/picture/raw/master/img/image-20210829222500523.png)

 <img src="https://gitee.com/aaronlynn/picture/raw/master/img/image-20210831102146878.png" alt="image-20210831102146878" style="zoom: 80%;" />  



## CICD

### **传统模式：**

![](https://gitee.com/aaronlynn/picture/raw/master/img/image-20210830113428034.png)

ps：传统方式会将打包好的jar包 

### **使用k8s集成：**

![image-20210830113356921](https://gitee.com/aaronlynn/picture/raw/master/img/image-20210830113356921.png)

ps：一次构建多环境部署，前提应用的<font color='red'>**配置不能写死**</font>，可配置在配置中心或者configmap

## Namespace命名空间

<img src="https://gitee.com/aaronlynn/picture/raw/master/img/image-20210830171243493.png" alt="image-20210830171243493" style="zoom:50%;" /> 

 <img src="https://gitee.com/aaronlynn/picture/raw/master/img/image-20210830171253906.png" alt="image-20210830171253906" style="zoom:50%;" /> 

ps：通过服务命名隔离，但是通过cluster-ip或者是Pod的ip访问是不受限制的，属于**<font color='red'>单维度</font>。**

## Resource多维度集群资源管理

##### 资源：

- CPU
- GPU
- 内存
- 持久化存储

<img src="https://gitee.com/aaronlynn/picture/raw/master/img/image-20210830171939588.png" alt="image-20210830171939588" style="zoom:50%;" /> 

##### 核心设计

- Requests
- Limits

样例：

<img src="https://gitee.com/aaronlynn/picture/raw/master/img/image-20210830172336223.png" alt="image-20210830172336223" style="zoom:67%;" /> 

ps：cpu：100m相当于0.1核cpu，1cpu = 1000m

##### **k8s**会根据requests和limits判断**服务等级**：

<img src="https://gitee.com/aaronlynn/picture/raw/master/img/image-20210830173102559.png" alt="image-20210830173102559" style="zoom:50%;" /> 

#####  Pod驱逐策略-Eviction

<img src="https://gitee.com/aaronlynn/picture/raw/master/img/image-20210830173801152.png" alt="image-20210830173801152" style="zoom:50%;" /> 

<img src="https://gitee.com/aaronlynn/picture/raw/master/img/image-20210830173843104.png" alt="image-20210830173843104" style="zoom: 50%;" />  

## Label标签

<img src="https://gitee.com/aaronlynn/picture/raw/master/img/image-20210830174256398.png" alt="image-20210830174256398" style="zoom:50%;" />  

例如：Pod打上标签，ReplicaSer（RS）可以知道哪些是自己的，Service知道哪些是自己的

## Scheduler调度策略

<img src="https://gitee.com/aaronlynn/picture/raw/master/img/image-20210830175330847.png" alt="image-20210830175330847" style="zoom:67%;" /> 

## 部署策略

<img src="https://gitee.com/aaronlynn/picture/raw/master/img/image-20210830180104543.png" alt="image-20210830180104543" style="zoom:50%;" /> 

## 实现业务负载均衡四层和七层代理

&emsp;&emsp;**k8s下的service、cluster-ip、nodeport都是四层的负载均衡。**

&emsp;&emsp;Ingress-nginx是7层的负载均衡器 ，同时负责统一管理外部对k8s cluster中Service的请求。

&emsp;&emsp;负载均衡，就是根据请求的信息不同，来决定怎么样转发流量。

**&emsp;&emsp;四层负载均衡**，就是根据请求的ip+端口，根据设定的规则，将请求转发到后端对应的IP+端口上。

&emsp;**&emsp;七层负载均衡**，则是在四层基础上，再去考虑应用层的特征。比如说一个WEB服务器的负载均衡，除了根据IP+80端口来判断之外，还可以根据七层URL，浏览器类别，来决定如何去转发流量。

## K8s常见的日志采集问题和解决方案

​    <img src="https://gitee.com/aaronlynn/picture/raw/master/img/image-20210901170636589.png" alt="image-20210901170636589" style="zoom: 33%;" />

<img src="https://gitee.com/aaronlynn/picture/raw/master/img/image-20210901170705490.png" alt="image-20210901170705490" style="zoom:50%;" /> 

### 常见方案

#### 远程日志

<img src="C:/Users/Aaron/AppData/Roaming/Typora/typora-user-images/image-20210901210053905.png" alt="image-20210901210053905" style="zoom: 67%;" /> 

#### SideCar

<img src="https://gitee.com/aaronlynn/picture/raw/master/img/image-20210901210157926.png" alt="image-20210901210157926" style="zoom:67%;" /> 

#### LogAgent

<img src="https://gitee.com/aaronlynn/picture/raw/master/img/image-20210901210324348.png" alt="image-20210901210324348" style="zoom:50%;" /> 

<img src="https://gitee.com/aaronlynn/picture/raw/master/img/image-20210901210434834.png" alt="image-20210901210434834" style="zoom:50%;" /> 

#### LogPilot

<img src="https://gitee.com/aaronlynn/picture/raw/master/img/image-20210901210544797.png" alt="image-20210901210544797" style="zoom:50%;" /> 

## K8s的监控

<img src="https://gitee.com/aaronlynn/picture/raw/master/img/image-20210901210712114.png" alt="image-20210901210712114" style="zoom:50%;" /> 



### Prometheus

- 一系列服务的组合
- 系统和服务的监控报警平台

#### 特征

##### <img src="https://gitee.com/aaronlynn/picture/raw/master/img/image-20210901210946438.png" alt="image-20210901210946438" style="zoom:33%;" />

#### 架构

![image-20210901211040147](https://gitee.com/aaronlynn/picture/raw/master/img/image-20210901211040147.png)

#### 数据类型

<img src="https://gitee.com/aaronlynn/picture/raw/master/img/image-20210901211108838.png" alt="image-20210901211108838" style="zoom:50%;" /> 

#### 数据来源

##### 服务器基础指标

<img src="https://gitee.com/aaronlynn/picture/raw/master/img/image-20210901212054932.png" alt="image-20210901212054932" style="zoom: 67%;" /> 

##### docker容器指标

<img src="https://gitee.com/aaronlynn/picture/raw/master/img/image-20210901212151458.png" alt="image-20210901212151458" style="zoom: 67%;" /> 

k8s组件

![image-20210901212351674](https://gitee.com/aaronlynn/picture/raw/master/img/image-20210901212351674.png)



## 基于K8s开发一个容器管理平台

&emsp;&emsp;未完待续.....

