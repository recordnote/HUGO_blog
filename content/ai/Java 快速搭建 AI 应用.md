---
title: Java 也能快速搭建 AI 应用
date: 2025-06-16T14:21:26+08:00
lastmod: 2025-06-16T14:21:26+08:00
author: Lin
avatar: /me/yy.jpg
cover: /img/ai.png
images:
  - /img/ai.png 
categories:
  - AI
tags:
  - AI
weight: 1
---

## 概述

随着 LLM（大语言模型）基础技术的不断成熟和应用领域的广泛挖掘，越来越多的企业和开发者开始将 LLM 技术集成到自己的互联网服务架构中，市场上涌现出了一大批基于 LLM 技术搭建的爆款应用。Python 语言受益于其丰富的框架和社区生态，成为了众多开发者搭建这些 AI 应用时的第一选择。但随着 AI 应用架构日益成熟，吞吐量、访问性能、可扩展性、微服务生态等重要指标也成为众多开发者和运维人员关注的焦点。恰好，经历了互联网时代考验的 Java 语言在这些方面已经有了很成熟的解决方案和生态。那么，使用 Java 语言能否也像 Python 一样快速搭建出来 AI 应用呢？

作为炙手可热的 Java 应用开发框架，Spring 给出了解决方案——Spring AI [1]。Spring AI 旨在简化 Java AI 应用程序开发，让 Java 开发者像使用 Spring 开发普通应用一样开发 AI 应用。以 Spring AI 为底座，Spring AI Alibaba 项目 [2] 引入了阿里云通义系列大模型的全面适配，带来了丰富的工具集和深度的云服务集成，让开发者数分钟就能搭建一个 AI 应用。

在生成式 AI 应用中，可观测性也是一个非常重要的能力，它不仅可以应对应用本身的性能调优、错误追踪等常规问题，还能成为解决 AI 应用中成本控制、模型偏见、模型幻觉等问题的利器。Spring AI Alibaba 在 Spring AI 可观测性基础上进行了扩展，对通义系列大模型及阿里云工具集的可观测性进一步扩展，提供了更加细节的可观测能力。此外，阿里云应用实时监控服务（ARMS）原生集成了对 Spring AI 可观测性数据的支持，使用者无需修改业务代码，只需适当调整启动配置，就能获得全套的企业级可观测服务。

本文将基于 Spring AI Alibaba，借由通义千问提供的模型服务搭建一个简单的在线聊天 AI 应用，并借助 ARMS 完成对 AI 应用中调用过程的追踪和用量观测。

## 快速搭建 Spring AI 应用

本节演示如何基于 Spring AI Alibaba 开发一个在线聊天 Agent 应用，并支持大模型调用本地函数来查询某城市某天的天气，可以在此处查看示例源码 [3]。

1. 新建一个项目，在项目的 pom.xml 中引入 spring-ai-alibaba-starter 依赖：

```java
<dependency>

  <groupId>com.alibaba.cloud.ai</groupId>

  <artifactId>spring-ai-alibaba-starter</artifactId>

  <version>1.0.0-M3.2</version>

</dependency>
```

1. 修改 application.yml，添加 dashscope 的 api key，请将`${AI_DASHSCOPE_API_KEY}`替换为您通义大模型的 API Key，获取方式参见 [4]：

```yaml
spring:
  application:
    name: chatmodel-example

  ai:
    dashscope:
      api-key: ${AI_DASHSCOPE_API_KEY}
```

1. 编写聊天服务 Controller 类，`/weather-service`基于客户的提示词查询天气：

```java
@RestController
@RequestMapping("/ai/func")
public class FunctionCallingController {

  private final ChatClient chatClient;

  public FunctionCallingController(ChatClient.Builder chatClientBuilder) {
    this.chatClient = chatClientBuilder.build();
  }

  @GetMapping("/weather-service")
  public String weatherService(String subject) {
    return chatClient.prompt()
      .function("getWeather", "根据城市查询天气", new MockWeatherService())
      .user(subject)
      .call()
      .content();
  }
}
```

1. 编写 function 供大模型调用：

```java
public class MockWeatherService implements Function<MockWeatherService.Request, Response> {
  @Override
  public Response apply(Request request) {
    if (request.city().contains("杭州")) {
      return new Response(String.format("%s%s晴转多云, 气温32摄氏度。", request.date(), request.city()));
    }
    else if (request.city().contains("上海")) {
      return new Response(String.format("%s%s多云转阴, 气温31摄氏度。", request.date(), request.city()));
    }
    else {
      return new Response(String.format("暂时无法查询%s的天气状况。", request.city()));
    }
  }

  @JsonInclude(JsonInclude.Include.NON_NULL)
  @JsonClassDescription("根据日期和城市查询天气")
  public record Request(
      @JsonProperty(required = true, value = "city") @JsonPropertyDescription("城市, 比如杭州") String city,
      @JsonProperty(required = true, value = "date") @JsonPropertyDescription("日期, 比如2024-08-22") String date) {
  }
}
```

1. 编写 Spring Boot 启动类：

```java
@SpringBootApplication
public class FunctionCallingExampleApplication {
  public static void main(String[] args) {
    SpringApplication.run(FunctionCallingExampleApplication.class, args);
  }
}
```

## 部署应用

通过以上五步，我们的 AI Agent 应用已经可以正常部署，要集成可观测性并将数据上报到 ARMS，还需要做少量改动。

1. 修改 application.yml 文件，开启可观测性相关数据的开关，实际生产中可以按需开启：

```yaml
spring:
  ai:
    chat:
      client:
        observations:
          # 记录调用者输入的内容
          include-input: true
      observations:
          # 记录大模型输出
          include-completion: true
          # 记录大模型提示词
          include-prompt: true
```

1. 修改 pom.xml 文件，引入可观测性相关依赖：

```xml
<dependency>
  <!-- spring 提供的可观测工具包，用于初始化 micrometer 组件 -->
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>

<dependency>
  <!-- micrometer-opentelemetry 桥接器，用于将 micrometer 链路追踪代理到 opentelemetry -->
  <groupId>io.micrometer</groupId>
  <artifactId>micrometer-tracing-bridge-otel</artifactId>
</dependency>
```

1. 在 Spring Boot 启动类中调整 OpenTelemetrySdk 获取方式，改为直接从 GlobalOpenTelemetry 中获取（这一步是为了获取到 Java Agent 中的 sdk 实例，而 micrometer 默认行为是初始化一个新的 sdk 实例）

```java
@Bean
public OpenTelemetry openTelemetry() {
    return GlobalOpenTelemetry.get();
}
```

1. 下载 Aliyun Java Agent 并在应用的启动脚本中添加以下三行，相关内容获取可以参考接入文档 [5]，请将：`${path-to-agent}`和`${your-license-key}`分别替换为 Java Agent 的解压路径和从 ARMS 控制台获取到的 licenseKey：

> 如果您正在使用 K8S 部署您的应用，则不需要修改任何的启动命令，直接在您的集群安装 ack-onepilot，并为应用添加相关 label 即可，详情可参考文档 [6]。

```java
-javaagent:/${path-to-agent}/aliyun-java-agent.jar

-Darms.licenseKey=${your-license-key}

-Darms.appName=spring-ai-alibaba-chat-demo
```

1. 启动应用并验证效果。

## 验证调用效果

1. 在浏览器地址栏输入以下链接访问：

```
http://localhost:8080/ai/func/weather-service?subject=2024年8月12日杭州天气怎么样？
```

返回如下响应：

```
2024年8月12日，杭州的天气预报为晴转多云，气温32摄氏度。请做好防晒措施，并留意实际天气变化。
```

1. 登录 ARMS 控制台，找到 spring-ai-alibaba-chat-demo 应用查看对应调用链信息![spring ai alibaba tracing](https://java2ai.com/img/blog/observability-arms/tracing.png)
2. 查看某一条特定的 trace，可以看到用量信息及其他关键信息，如大模型调用的 response id、模型名称、temperature 等：![spring ai alibaba arms tracing detail](https://java2ai.com/img/blog/observability-arms/tracing-detail.png)
3. 单击右侧的“Events”，可以查看到模型调用过程的输入输出信息：![spring ai alibaba arms tracing detail events](https://java2ai.com/img/blog/observability-arms/tracing-detail-events.png)
