---
title: LangChain4j框架开发AI应用
date: 2025-11-09T14:21:26+08:00
lastmod: 2025-11-09T14:21:26+08:00
author: Lin
avatar: /me/yy.jpg
cover: /img/ai6.png
images:
  - /img/ai6.png 
categories:
  - AI
tags:
  - AI
weight: 1
---

## 1. 简介

**LangChain4j** 是一个基于 Java 的开源框架，用于开发 **人工智能驱动的应用程序**，尤其是涉及 **大语言模型（LLM）交互** 的场景。它的设计目标是简化开发者与大语言模型的集成过程，提供一套工具和组件来处理复杂的 LLM 应用逻辑，例如对话管理、提示工程、工具调用等。

### 核心功能与特点

1. 大语言模型集成
   - 支持多种 LLM 接入方式，包括：
     - 本地运行的开源模型（如 Llama 2、ChatGLM 等）。
     - 第三方 API 模型（如 OpenAI 的 GPT 系列、Anthropic 的 Claude 等）。
   - 通过统一的接口抽象，降低模型切换的成本。
2. 提示工程工具
   - 提供模板化的提示构建器，帮助开发者结构化输入（如填充变量、管理上下文历史）。
   - 支持动态组合提示链（Prompt Chain），例如根据用户问题逐步调用不同的提示模板。
3. 对话状态管理
   - 维护多轮对话的上下文，支持记忆管理（如设置上下文窗口大小、选择性遗忘旧信息）。
   - 可集成外部知识库（如向量数据库）实现长期记忆。
4. 工具调用能力
   - 支持调用外部工具（如计算器、数据库查询、API 接口等），并将工具返回结果整合到 LLM 的回答中。
   - 提供工具调用的决策逻辑（如判断何时需要调用工具、如何解析工具返回结果）。
5. 链式流程编排
   - 通过 **Chain** 机制编排多个组件（如提示生成、工具调用、结果处理），形成复杂的工作流。
   - 典型场景：问答系统中先调用搜索引擎获取实时数据，再用 LLM 生成回答。
6. 扩展性与生态
   - 基于 Java 生态，可轻松与 Spring框架集成。
   - 支持自定义组件（如自定义提示策略、工具适配器），灵活适配业务需求。

## 2. 话不多说，直接展示

本章主要通过单元测试的方式展示LangChain4j的各项功能，后续会出通过`LangChain4j Starter`的方式快速集成SpringBoot。

使用SDK版本信息如下：

```
Java: 21

SpringBoot: 3.4.5

LangChain4j: 1.0.1
```

AI 模型主要使用的是阿里的百炼平台免费的token，需要`ApiKey`的可以自行去申请, 平台地址如下：

https://bailian.console.aliyun.com/?tab=model#/model-market

## 3. Maven

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.4.5</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.ldx</groupId>
    <artifactId>langchain-test</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>langchain-test</name>
    <description>langchain-test</description>

    <properties>
        <java.version>21</java.version>
        <langchain4j.version>1.0.1</langchain4j.version>
        <guava.version>33.0.0-jre</guava.version>
    </properties>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>dev.langchain4j</groupId>
                <artifactId>langchain4j-bom</artifactId>
                <version>${langchain4j.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>dev.langchain4j</groupId>
            <artifactId>langchain4j</artifactId>
        </dependency>

        <dependency>
            <groupId>com.google.guava</groupId>
            <artifactId>guava</artifactId>
            <version>${guava.version}</version>
        </dependency>

        <dependency>
            <groupId>dev.langchain4j</groupId>
            <artifactId>langchain4j-open-ai</artifactId>
        </dependency>

        <dependency>
            <groupId>dev.langchain4j</groupId>
            <artifactId>langchain4j-reactor</artifactId>
        </dependency>

        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
            </plugin>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <excludes>
                        <exclude>
                            <groupId>org.projectlombok</groupId>
                            <artifactId>lombok</artifactId>
                        </exclude>
                    </excludes>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

## 4. 构建模型对象

```java
// 普通的对话模型
private static ChatModel chatModel;

// 流式对话的模型（可以模拟gpt的打字机效果）
private static StreamingChatModel streamingChatModel;

@BeforeAll
public static void init_chat_model() {
    chatModel = OpenAiChatModel
            .builder()
            // apikey 通过环境变量的方式注入，大家可以使用自己的apikey
            .apiKey(System.getenv("LLM_API_KEY"))
            .modelName("qwen-plus")
            .baseUrl("https://dashscope.aliyuncs.com/compatible-mode/v1")
            .build();
    streamingChatModel = OpenAiStreamingChatModel
            .builder()
            .apiKey(System.getenv("LLM_API_KEY"))
            .modelName("qwen-plus")
            .baseUrl("https://dashscope.aliyuncs.com/compatible-mode/v1")
            .build();
}
```

## 5. 返回字符串

```java
@Test
public void should_return_str_when_use_normal_chat() {
    String q = "你是谁";
    String content = chatModel.chat(q);
    log.info("call ai q: {}\na: {}", q, content);
}
```

> 测试结果如下：

```
[main] INFO com.ldx.langchaintest.lowapi.AiChatTest -- call ai q: 你是谁
a: 我是通义千问，阿里巴巴集团旗下的超大规模语言模型。我能够回答问题、创作文字，比如写故事、公文、邮件、剧本等，还能进行逻辑推理、编程，甚至表达观点和玩游戏。我在多国语言上都有很好的掌握，能为你提供多样化的帮助。有什么我可以帮到你的吗？
```

## 6. 返回流

> 这里使用flux对象接收流式返回的结果
>
> 如果想流式的返回给前端，也可以使用SSE的方式返回（代码注释的部分）

```java
@Test
public void should_return_stream_when_use_stream_model() {
    Sinks.Many<String> sinks = Sinks
        .many()
        .multicast()
        .onBackpressureBuffer();
    Flux<String> flux = sinks.asFlux();
    StreamingChatResponseHandler streamingChatResponseHandler = new StreamingChatResponseHandler() {
        @Override
        public void onPartialResponse(String s) {
            sinks.tryEmitNext(s);
        }

        @Override
        public void onCompleteResponse(ChatResponse chatResponse) {
            sinks.tryEmitComplete();
        }

        @Override
        public void onError(Throwable throwable) {
            sinks.tryEmitError(throwable);
        }
    };
//        SseEmitter sse = new SseEmitter();
//        final StreamingChatResponseHandler streamingChatResponseHandler = LambdaStreamingResponseHandler.onPartialResponseAndError(s -> {
//            try {
//                log.info("ai response stream data: {}", s);
//                sse.send(s);
//            } catch (IOException e) {
//                throw new RuntimeException(e);
//            }
//        }, e -> sse.complete());
    streamingChatModel.chat("你是谁", streamingChatResponseHandler);
    flux
        .toStream()
        .forEach(partial -> log.info("ai response stream data: {}", partial));
}
```

> 测试结果如下:

```java
[main] INFO com.ldx.langchaintest.lowapi.AiChatTest -- ai response stream data: 我是
[main] INFO com.ldx.langchaintest.lowapi.AiChatTest -- ai response stream data: 通
[main] INFO com.ldx.langchaintest.lowapi.AiChatTest -- ai response stream data: 义
[main] INFO com.ldx.langchaintest.lowapi.AiChatTest -- ai response stream data: 千问，阿里巴巴
[main] INFO com.ldx.langchaintest.lowapi.AiChatTest -- ai response stream data: 集团旗下的通义
[main] INFO com.ldx.langchaintest.lowapi.AiChatTest -- ai response stream data: 实验室自主研发的超
[main] INFO com.ldx.langchaintest.lowapi.AiChatTest -- ai response stream data: 大规模语言模型。
[main] INFO com.ldx.langchaintest.lowapi.AiChatTest -- ai response stream data: 我能够回答问题
[main] INFO com.ldx.langchaintest.lowapi.AiChatTest -- ai response stream data: 、创作文字，
[main] INFO com.ldx.langchaintest.lowapi.AiChatTest -- ai response stream data: 比如写故事、
[main] INFO com.ldx.langchaintest.lowapi.AiChatTest -- ai response stream data: 写公文、
[main] INFO com.ldx.langchaintest.lowapi.AiChatTest -- ai response stream data: 写邮件、写
[main] INFO com.ldx.langchaintest.lowapi.AiChatTest -- ai response stream data: 剧本、逻辑推理
[main] INFO com.ldx.langchaintest.lowapi.AiChatTest -- ai response stream data: 、编程等等，
[main] INFO com.ldx.langchaintest.lowapi.AiChatTest -- ai response stream data: 还能表达观点，
[main] INFO com.ldx.langchaintest.lowapi.AiChatTest -- ai response stream data: 玩游戏等。如果你
[main] INFO com.ldx.langchaintest.lowapi.AiChatTest -- ai response stream data: 有任何问题或需要
[main] INFO com.ldx.langchaintest.lowapi.AiChatTest -- ai response stream data: 帮助，欢迎随时
[main] INFO com.ldx.langchaintest.lowapi.AiChatTest -- ai response stream data: 告诉我！
```

## 7. 提示词/模板

```java
@Test
public void should_return_prompt_content_when_use_prompt() {
    // 申明系统提示词
    // SystemMessage systemMessage = Prompt.from("你是一名java专家，请协助用户解决相应的专业性问题").toSystemMessage();
    // 申明提示词模板
    final PromptTemplate promptTemplate = new PromptTemplate("""
            👉 将文本改写成类似小红书的 Emoji 风格。
            请使用 Emoji 风格编辑以下段落，该风格以引人入胜的标题、每个段落中包含表情符号和在末尾添加相关标签为特点。请确保保持原文的意思。
            用户的提问信息如下:
            {{question}}
            """);                                  
    final UserMessage userMessage = promptTemplate
            .apply(Map.of("question", "你是谁"))
            .toUserMessage();
    ChatResponse chatResponse = chatModel.chat(userMessage);
    String content = chatResponse
            .aiMessage()
            .text();
    log.info("call ai q: {}\na: {}", userMessage.singleText(), content);
}
```

> 测试结果如下：

```java
[main] INFO com.ldx.langchaintest.lowapi.AiChatTest -- call ai q: 👉 将文本改写成类似小红书的 Emoji 风格。
请使用 Emoji 风格编辑以下段落，该风格以引人入胜的标题、每个段落中包含表情符号和在末尾添加相关标签为特点。请确保保持原文的意思。
用户的提问信息如下:
你是谁

a: ✨你是谁？来认识一下我吧！💬

嗨，亲爱的朋友们！我是通义千问，阿里巴巴集团旗下的超大规模语言模型🤖。我可以通过学习海量文本数据，帮你回答问题、创作文字，甚至玩游戏哦～是不是很酷呢？🔥  

如果你有任何问题或需要帮助，尽管告诉我！我会尽力为你提供支持和支持❤️。让我们一起开启有趣的探索之旅吧！🌍  

#人工智能 #聊天机器人 #新知探索 #科技生活
```

## 8. 聊天记忆

> 中心逻辑其实就是：将最近的聊天内容存储起来，然后一股脑扔给AI😓

```java
@Test
public void should_return_memory_content_when_use_memory_chat() {
    String id = "zhangtieniu_01";
    String q1 = "你是谁";
    ChatMemory chatMemory = MessageWindowChatMemory
            .builder()
            // 会话隔离（不同用户的聊天信息互不干扰）
            .id(id)
            // 最大存储最近的5条聊天内容（存储太多影响性能&token）
            .maxMessages(5)
            .build();
    // 将聊天内容放入记忆对象中
    chatMemory.add(UserMessage.from(q1));
    // SystemMessage 始终保存在messages中 且占用maxMessage名额
    chatMemory.add(SystemMessage.from("""
            👉 将文本改写成类似小红书的 Emoji 风格。
            请使用 Emoji 风格编辑以下段落，该风格以引人入胜的标题、每个段落中包含表情符号和在末尾添加相关标签为特点。请确保保持原文的意思。
            """));
    final ChatResponse chatResponse = chatModel.chat(chatMemory.messages());
    // 将ai的返回结果放入记忆对象中                                  
    chatMemory.add(chatResponse.aiMessage());
    String q2 = "我刚刚问了啥";
    chatMemory.add(UserMessage.from(q2));
    final ChatResponse chatResponse2 = chatModel.chat(chatMemory.messages());

    log.info("call ai q1: {}\na1: {}", q1, chatResponse
            .aiMessage()
            .text());
    log.info("==========================分隔符==========================");
    log.info("call ai q2: {}\na2: {}", q2, chatResponse2
            .aiMessage()
            .text());
}
```

> 测试结果如下：

```java
[main] INFO com.ldx.langchaintest.lowapi.AiChatTest -- call ai q1: 你是谁
a1: 🌟 你好呀，让我来介绍一下自己！ 👋 我是通义千问，阿里巴巴集团旗下的超大规模语言模型。我不仅能陪你聊天，还能帮你写故事、邮件、剧本等等，甚至可以表达观点、玩游戏呢！🎮📖  

💡 无论你想聊生活中的小确幸 🍵 还是工作学习中的难题 📚，我都会尽力帮助你！希望我能成为你的贴心小伙伴～ ❤️  

#人工智能 #聊天伙伴 #创作助手 #日常分享
[main] INFO com.ldx.langchaintest.lowapi.AiChatTest -- ==========================分隔符==========================
[main] INFO com.ldx.langchaintest.lowapi.AiChatTest -- call ai q2: 我刚刚问了啥
a2: 🤔 呃... 让我查查！哦对！你刚刚问的是：“我是谁？” 这个问题让我有机会用小红书风格重新介绍自己呢！ 😊  

✨ 作为通义千问，我最喜欢的就是通过对话帮助别人啦！如果你还有其他问题或者需要灵感，随时可以问我哦～👇  

#回忆对话 #人工智能 #问答时间 #互动分享
```

## 9. 聊天内容持久化

### 9.1 store handler

> 这里简单的使用map存储会话内容

```java
package com.ldx.langchaintest.store;

import com.google.common.collect.ArrayListMultimap;
import dev.langchain4j.data.message.ChatMessage;
import dev.langchain4j.store.memory.chat.ChatMemoryStore;

import java.util.List;

public class PersistentChatMemoryStoreTest implements ChatMemoryStore {
    public final ArrayListMultimap<Object, ChatMessage> messagesStore = ArrayListMultimap.create();

    @Override
    public List<ChatMessage> getMessages(Object memoryId) {
        return messagesStore.get(memoryId);
    }

    @Override
    public void updateMessages(Object memoryId, List<ChatMessage> messages) {
        messagesStore.put(memoryId, messages.getLast());
    }

    @Override
    public void deleteMessages(Object memoryId) {
        messagesStore.removeAll(memoryId);
    }
}
```

### 9.2 实现

```java
@Test
public void should_return_memory_content_when_use_store_chat() {
    String id = "zhangtieniu_01";
    String q1 = "张铁牛是一个高富帅，你是张铁牛的助手";
    PersistentChatMemoryStoreTest store = new PersistentChatMemoryStoreTest();
    ChatMemory chatMemory = MessageWindowChatMemory
            .builder()
            .id(id)
            .chatMemoryStore(store)
            .maxMessages(5)
            .build();
    chatMemory.add(UserMessage.from(q1));
    final ChatResponse chatResponse = chatModel.chat(chatMemory.messages());
    chatMemory.add(chatResponse.aiMessage());
    String q2 = "张铁牛是谁";
    chatMemory.add(UserMessage.from(q2));
    final ChatResponse chatResponse2 = chatModel.chat(chatMemory.messages());
    chatMemory.add(chatResponse2.aiMessage());
    
    // 获取当前会话的存储内容并打印
    List<ChatMessage> chatMessages = store.messagesStore.get(id);
    for (ChatMessage chatMessage : chatMessages) {
        log.info("session id: {}, message type: {}, message: {}", id, chatMessage.type(), chatMessage);
    }
}
```

> 测试结果如下：

```java
[main] INFO com.ldx.langchaintest.lowapi.AiChatTest -- session id: zhangtieniu_01, message type: USER, message: UserMessage { name = null contents = [TextContent { text = "张铁牛是一个高富帅，你是张铁牛的助手" }] }
[main] INFO com.ldx.langchaintest.lowapi.AiChatTest -- session id: zhangtieniu_01, message type: AI, message: AiMessage { text = "您好，我是张铁牛先生的助手。张铁牛先生确实是一位优秀的人士，他不仅外貌出众、家境优渥，而且非常有才华。作为他的助手，我会帮助他处理各种事务，确保他的生活和工作都井井有条。如果您有任何需要帮忙的事情，或者想了解张铁牛先生的相关信息，只要是我职责范围内的，我都会尽力提供帮助。请问有什么我可以为您服务的呢？" toolExecutionRequests = [] }
[main] INFO com.ldx.langchaintest.lowapi.AiChatTest -- session id: zhangtieniu_01, message type: USER, message: UserMessage { name = null contents = [TextContent { text = "张铁牛是谁" }] }
[main] INFO com.ldx.langchaintest.lowapi.AiChatTest -- session id: zhangtieniu_01, message type: AI, message: AiMessage { text = "张铁牛先生是一位非常杰出的人物。他出身于一个成功的企业家庭，拥有优越的教育资源和广泛的商业人脉。除了在商业领域的卓越成就外，他还以阳光、正直的形象受到周围人的喜爱。

作为一位“高富帅”，张铁牛先生不仅注重个人修养，还热衷于公益事业，经常参与慈善活动来回馈社会。同时，他对生活充满热情，兴趣爱好广泛，比如健身、旅行以及收藏艺术品等。

不过，请允许我提醒您，虽然他是公众眼中的完美人物，但他更希望被当作普通人来尊重，注重隐私保护。如果您有关于他的正面问题或需要安排相关事务，我很乐意为您提供帮助！" toolExecutionRequests = [] }
```

## 10. function call

### 10.1 user svc

> 声明一个自定义的user svc， 让ai去调用我们的业务方法

```java
import dev.langchain4j.agent.tool.P;
import dev.langchain4j.agent.tool.Tool;

public class UserServiceTest {
    @Tool("根据用户的名称获取对应的code")
    public String getUserCodeByUsername(@P("用户名称") String username) {
        if ("张铁牛".equals(username)) {
            return "003";
        }

        return "000";
    }
}
```

### 10.2 实现

```java
@Test
public void should_return_func_content_when_use_function_call() {
    final String q = "张铁牛的code是多少";
    List<ChatMessage> chatMessages = new ArrayList<>();
    chatMessages.add(UserMessage.from(q));
    final UserServiceTest userServiceTest = new UserServiceTest();
    final ChatRequest chatRequest = ChatRequest
            .builder()
            .messages(UserMessage.from(q))
            // 将工具类注入到上下文中
            .toolSpecifications(ToolSpecifications.toolSpecificationsFrom(userServiceTest))
            .build();
    final ChatResponse chatResponse = chatModel.chat(chatRequest);
    final AiMessage aiMessage = chatResponse.aiMessage();
    chatMessages.add(aiMessage);
    String a = aiMessage.text();

    // 在响应结果中判断是否有tool请求
    if (aiMessage.hasToolExecutionRequests()) {
        // 遍历tool req
        for (ToolExecutionRequest toolExecutionRequest : aiMessage.toolExecutionRequests()) {
            // 申明执行器 其实就是通过tool name 反射调用userServiceTest的方法
            ToolExecutor userToolExecutor = new DefaultToolExecutor(userServiceTest, toolExecutionRequest);
            final String uuid = UUID
                    .randomUUID()
                    .toString();
            final String executeResult = userToolExecutor.execute(toolExecutionRequest, uuid);
            log.info("execute user tool, name: {}, param:{}, result: {} ", toolExecutionRequest.name(), toolExecutionRequest.arguments(), executeResult);
            final ToolExecutionResultMessage toolExecutionResultMessages = ToolExecutionResultMessage.from(toolExecutionRequest, executeResult);
            // 将tool执行的结果 放入上下文
            chatMessages.add(toolExecutionResultMessages);
        }
        // 再次请求ai
        a = chatModel
                .chat(chatMessages)
                .aiMessage()
                .text();
    }

    log.info("call ai q:{}\na:{}", q, a);
}
```

> 测试结果如下：

```java
[main] INFO com.ldx.langchaintest.lowapi.AiChatTest -- execute user tool, name: getUserCodeByUsername, param:{"username": "张铁牛"}, result: 003 
[main] INFO com.ldx.langchaintest.lowapi.AiChatTest -- call ai q:张铁牛的code是多少
a:根据您的问题，张铁牛的代码是 **003**。如果还有其他相关信息需要补充或查询，请告诉我！
```

## 11. dynamic function call

> 因为有的时候我们使用已有的svc时，我们可能无法直接给目标方法标注对应的注解让AI来识别方法（比如第三方包里的方法）
>
> 所以就需要动态的创建tool的描述类，如下

```java
@Test
public void should_return_func_content_when_use_dynamic_function_call() {
    final String q = "张铁牛的code是多少";
    final List<ChatMessage> chatMessages = new ArrayList<>();
    chatMessages.add(UserMessage.from(q));
    final UserServiceTest userServiceTest = new UserServiceTest();
    final ToolSpecification toolSpecification = ToolSpecification
            .builder()
            .name("getUserCodeByUsername")
            .description("根据用户的名称获取对应的code")
            .parameters(JsonObjectSchema
                    .builder()
                    .addStringProperty("username", "用户姓名")
                    .build())
            .build();

    final ChatRequest chatRequest = ChatRequest
            .builder()
            .messages(UserMessage.from(q))
            .toolSpecifications(toolSpecification)
            .build();
    final ChatResponse chatResponse = chatModel.chat(chatRequest);
    AiMessage aiMessage = chatResponse.aiMessage();
    chatMessages.add(aiMessage);
    String a = aiMessage.text();

    if (aiMessage.hasToolExecutionRequests()) {
        for (ToolExecutionRequest toolExecutionRequest : aiMessage.toolExecutionRequests()) {
            ToolExecutor userToolExecutor = new DefaultToolExecutor(userServiceTest, toolExecutionRequest);
            final String uuid = UUID
                    .randomUUID()
                    .toString();
            final String executeResult = userToolExecutor.execute(toolExecutionRequest, uuid);
            log.info("execute user tool, name: {}, param:{}, result: {} ", toolExecutionRequest.name(), toolExecutionRequest.arguments(), executeResult);
            final ToolExecutionResultMessage toolExecutionResultMessages = ToolExecutionResultMessage.from(toolExecutionRequest, executeResult);
            chatMessages.add(toolExecutionResultMessages);
        }

        a = chatModel
                .chat(chatMessages)
                .aiMessage()
                .text();
    }

    log.info("call ai q:{}\na:{}", q, a);
}
```

> 测试结果如下：

```
[main] INFO com.ldx.langchaintest.lowapi.AiChatTest -- execute user tool, name: getUserCodeByUsername, param:{"username": "张铁牛"}, result: 003 
[main] INFO com.ldx.langchaintest.lowapi.AiChatTest -- call ai q:张铁牛的code是多少
a:根据您的问题，张铁牛的代码是 **003**。如果还有其他相关信息需要补充，请告诉我！
```

## 12. 小结

本文使用了`LangChain4j`一些比较底层的（原生的）api来请求大模型， 展示了AI服务中常见的使用方法。
