---
title: Sentinel（轻量级的流量控制、熔断降级Java库）
date: 2023-05-02T14:21:26+08:00
lastmod: 2023-05-02T14:21:26+08:00
author: Aaron
avatar: /me/yy.jpg
cover: /img/2017.jpg
images:
  - /img/2017.jpg
categories:
  - 微服务
tags:
  - 微服务
weight: 1
---

​	

## 1、雪崩效应（级联失效、级联故障）

<img src="https://gitee.com/aaronlynn/picture/raw/master/img/image-20210717141215463.png" alt="image-20210717141215463" style="zoom:50%;" />

## 2、服务容错的思想

- 超时
- 限流
- 仓壁模式（类似于为每个被调用的服务端api接口设置相应数量的线程池，当某个接口达到线程池阈值，不影响其他api服务的调用）
- 断路器模式

 

<img src="https://gitee.com/aaronlynn/picture/raw/master/img/image-20210717141907072.png" alt="image-20210717141907072" style="zoom:50%;" /> 

## sentinel控制台流控的三种模式：

 <img src="https://gitee.com/aaronlynn/picture/raw/master/img/image-20210717150420349.png" alt="image-20210717150420349" style="zoom:50%;" />

<img src="C:/Users/Aaron/AppData/Roaming/Typora/typora-user-images/image-20210717150503417.png" alt="image-20210717150503417" style="zoom:50%;" /> 

## 三种流控效果：快速失败、Warm Up、 排队等待

<img src="https://gitee.com/aaronlynn/picture/raw/master/img/image-20210717151512328.png" alt="image-20210717151512328" style="zoom: 33%;" /> 

<img src="https://gitee.com/aaronlynn/picture/raw/master/img/image-20210717151725306.png" alt="image-20210717151725306" style="zoom:33%;" /> 

<img src="https://gitee.com/aaronlynn/picture/raw/master/img/image-20210717151810731.png" alt="image-20210717151810731" style="zoom:33%;" /> 

## 系统规则



<img src="https://gitee.com/aaronlynn/picture/raw/master/img/image-20210717155641451.png" alt="image-20210717155641451" style="zoom:50%;" /> 



使用uptime命令可查看1分钟，5分钟和15分钟的系统负载：

<img src="https://gitee.com/aaronlynn/picture/raw/master/img/image-20210717155715368.png" alt="image-20210717155715368" style="zoom: 80%;" /> 

<img src="https://gitee.com/aaronlynn/picture/raw/master/img/image-20210717155342331.png" alt="image-20210717155342331" style="zoom: 50%;" /> 

## 授权规则

<img src="https://gitee.com/aaronlynn/picture/raw/master/img/image-20210717160021812.png" alt="image-20210717160021812" style="zoom:50%;" /> 



# **Alibaba Sentinel 规则参数总结**

## 一、流控规则

1.1 配置

![流控规则](http://www.itmuch.com/images/spring-cloud-alibaba/sentinel/flow.png)

1.2 参数

| Field           | 说明                                                         | 默认值                        |
| :-------------- | :----------------------------------------------------------- | :---------------------------- |
| resource        | 资源名，资源名是限流规则的作用对象                           |                               |
| count           | 限流阈值                                                     |                               |
| grade           | 限流阈值类型，QPS 或线程数模式                               | QPS 模式                      |
| limitApp        | 流控针对的调用来源                                           | `default`，代表不区分调用来源 |
| strategy        | 判断的根据是资源自身，还是根据其它关联资源 (`refResource`)，还是根据链路入口 | 根据资源本身                  |
| controlBehavior | 流控效果（直接拒绝 / 排队等待 / 慢启动模式）                 | 直接拒绝                      |

1.3 代码配置示例

```
private void initFlowQpsRule() {
    List<FlowRule> rules = new ArrayList<>();
    FlowRule rule = new FlowRule(resourceName);
    // set limit qps to 20
    rule.setCount(20);
    rule.setGrade(RuleConstant.FLOW_GRADE_QPS);
    rule.setLimitApp("default");
    rules.add(rule);
    FlowRuleManager.loadRules(rules);
}
```

1.4 参考：`https://github.com/alibaba/Sentinel/wiki/如何使用#流量控制规则-flowrule`

1.5 参考：[流量控制](https://github.com/alibaba/Sentinel/wiki/流量控制)

## 二、降级规则

2.1 配置

![降级规则](https://gitee.com/aaronlynn/picture/raw/master/img/degrade.png)

2.2 参数

| Field      | 说明                                       | 默认值 |
| :--------- | :----------------------------------------- | :----- |
| resource   | 资源名，即限流规则的作用对象               |        |
| count      | 阈值                                       |        |
| grade      | 降级模式，根据 RT 降级还是根据异常比例降级 | RT     |
| timeWindow | 降级的时间，单位为 s                       |        |

2.3 代码配置示例

```
private void initDegradeRule() {
    List<DegradeRule> rules = new ArrayList<>();
    DegradeRule rule = new DegradeRule();
    rule.setResource(KEY);
    // set threshold RT, 10 ms
    rule.setCount(10);
    rule.setGrade(RuleConstant.DEGRADE_GRADE_RT);
    rule.setTimeWindow(10);
    rules.add(rule);
    DegradeRuleManager.loadRules(rules);
}
```

2.4 参考：`https://github.com/alibaba/Sentinel/wiki/如何使用#熔断降级规则-degraderule`

## 三、热点规则

3.1 配置

![热点规则](https://gitee.com/aaronlynn/picture/raw/master/img/hot.png)

3.2 参数

| 属性              | 说明                                                         | 默认值   |
| :---------------- | :----------------------------------------------------------- | :------- |
| resource          | 资源名，必填                                                 |          |
| count             | 限流阈值，必填                                               |          |
| grade             | 限流模式                                                     | QPS 模式 |
| durationInSec     | 统计窗口时间长度（单位为秒），1.6.0 版本开始支持             | 1s       |
| controlBehavior   | 流控效果（支持快速失败和匀速排队模式），1.6.0 版本开始支持   | 快速失败 |
| maxQueueingTimeMs | 最大排队等待时长（仅在匀速排队模式生效），1.6.0 版本开始支持 | 0ms      |
| paramIdx          | 热点参数的索引，必填，对应 `SphU.entry(xxx, args)` 中的参数索引位置 |          |
| paramFlowItemList | 参数例外项，可以针对指定的参数值单独设置限流阈值，不受前面 `count` 阈值的限制。**仅支持基本类型** |          |
| clusterMode       | 是否是集群参数流控规则                                       | `false`  |
| clusterConfig     | 集群流控相关配置                                             |          |

3.3 代码配置示例

```
ParamFlowRule rule = new ParamFlowRule(resourceName)
    .setParamIdx(0)
    .setCount(5);
// 针对 int 类型的参数 PARAM_B，单独设置限流 QPS 阈值为 10，而不是全局的阈值 5.
ParamFlowItem item = new ParamFlowItem().setObject(String.valueOf(PARAM_B))
    .setClassType(int.class.getName())
    .setCount(10);
rule.setParamFlowItemList(Collections.singletonList(item));

ParamFlowRuleManager.loadRules(Collections.singletonList(rule));
```

3.4 参考：`https://github.com/alibaba/Sentinel/wiki/热点参数限流`

## 四、系统规则

4.1 配置

![系统规则](https://gitee.com/aaronlynn/picture/raw/master/img/system.png)

4.2 参数

| Field             | 说明                       | 默认值      |
| :---------------- | :------------------------- | :---------- |
| highestSystemLoad | 最大的 `load1`             | -1 (不生效) |
| avgRt             | 所有入口流量的平均响应时间 | -1 (不生效) |
| maxThread         | 入口流量的最大并发数       | -1 (不生效) |
| qps               | 所有入口资源的 QPS         | -1 (不生效) |

4.3 代码配置示例

```
private void initSystemRule() {
    List<SystemRule> rules = new ArrayList<>();
    SystemRule rule = new SystemRule();
    rule.setHighestSystemLoad(10);
    rules.add(rule);
    SystemRuleManager.loadRules(rules);
}
```

4.4 参考：`https://github.com/alibaba/Sentinel/wiki/如何使用#系统保护规则-systemrule`

## 五、授权规则

5.1 配置

![授权规则](https://gitee.com/aaronlynn/picture/raw/master/img/auth.png)

5.2 参数

| Field    | 说明                                                         | 默认值                      |
| :------- | :----------------------------------------------------------- | :-------------------------- |
| resource | 资源名，即限流规则的作用对象                                 | -                           |
| limitApp | 对应的黑名单/白名单，不同 origin 用 `,` 分隔，如 `appA,appB` | default，代表不区分调用来源 |
| strategy | 限制模式，`AUTHORITY_WHITE` 为白名单模式，`AUTHORITY_BLACK` 为黑名单模式，默认为白名单模式 | AUTHORITY_WHITE             |

5.3 代码配置示例

```
AuthorityRule rule = new AuthorityRule();
rule.setResource("test");
rule.setStrategy(RuleConstant.AUTHORITY_WHITE);
rule.setLimitApp("appA,appB");
AuthorityRuleManager.loadRules(Collections.singletonList(rule));
```

5.4 参考：`https://github.com/alibaba/Sentinel/wiki/如何使用#访问控制规则-authorityrule





## sentinel与控制台通信原理

<img src="https://gitee.com/aaronlynn/picture/raw/master/img/image-20210717161108537.png" alt="image-20210717161108537" style="zoom:50%;" /> 



<img src="https://gitee.com/aaronlynn/picture/raw/master/img/image-20210717161610202.png" alt="image-20210717161610202" style="zoom:33%;" /> 



## 控制台相关配置

<img src="https://gitee.com/aaronlynn/picture/raw/master/img/image-20210717161731701.png" alt="image-20210717161731701" style="zoom: 50%;" /> 

<img src="https://gitee.com/aaronlynn/picture/raw/master/img/image-20210717161914554.png" alt="image-20210717161914554" style="zoom:50%;" /> 

使用例子：<img src="https://gitee.com/aaronlynn/picture/raw/master/img/image-20210717162004595.png" alt="image-20210717162004595" style="zoom: 50%;" />



## sentinel-api

<img src="https://gitee.com/aaronlynn/picture/raw/master/img/image-20210717163500422.png" alt="image-20210717163500422" style="zoom:33%;" /> 

<img src="C:/Users/Aaron/AppData/Roaming/Typora/typora-user-images/image-20210718160725207.png" alt="image-20210718160725207" style="zoom:67%;" /> 

## 规则持久化

<img src="https://gitee.com/aaronlynn/picture/raw/master/img/image-20210718163113333.png" alt="image-20210718163113333" style="zoom:50%;" /> 













