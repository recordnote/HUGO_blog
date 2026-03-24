---
title: LangChain4j-AIServices使用教程
date: 2025-11-11T14:21:26+08:00
lastmod: 2025-11-11T14:21:26+08:00
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

上一篇讲了如何使用`LangChain4J`的底层组件来进行AI的交互，如 `ChatLanguageModel`、`ChatMessage`、`ChatMemory` 等。 在这个层面上工作非常灵活/自由，但也迫使我们编写大量的样板代码。 由于 LLM 驱动的应用程序通常不仅需要单个组件，还需要多个组件协同工作 （例如，提示模板、聊天记忆、LLM、输出解析器、RAG 组件：嵌入模型和存储） 并且经常涉及多次交互，协调所有这些组件变得更加繁琐。

而我们使用框架的目的是希望专注于业务逻辑，而不是低级实现细节。 为此，`LangChain4J`衍生出一个高级概念可以帮助更快的进行AI应用的开发：`AI Services`。

## 2. 环境信息

本章的环境信息跟上一章完全一样，这里就不过多展示了，感兴趣的可以看[上一章内容](https://www.cnblogs.com/ludangxin/p/18913321)

## 3. 构建AI Service

### 3.1 申明AI Service 接口

```java
package com.ldx.langchaintest.aisvc;

import com.ldx.langchaintest.model.PersonTest;
import dev.langchain4j.service.Result;
import dev.langchain4j.service.SystemMessage;
import dev.langchain4j.service.TokenStream;
import dev.langchain4j.service.UserMessage;
import dev.langchain4j.service.V;
import reactor.core.publisher.Flux;

import java.util.List;

/**
 * ai svc
 *
 */
public interface AiAssistantServiceTest {
    String chat(String message);

    TokenStream chatWithTokenStream(String message);

    Flux<String> chatWithFlux(String message);

    @SystemMessage("""
                👉 将文本改写成类似小红书的 Emoji 风格。
                请使用 Emoji 风格编辑以下段落，该风格以引人入胜的标题、每个段落中包含表情符号和在末尾添加相关标签为特点。请确保保持原文的意思。
                """)
    String chatWithSysPrompt(String userMessage);

    @UserMessage("请帮我判断一下这段表达式「{{expression}}」的返回值 如果成立则返回true 反之 false")
    boolean chatWithPrompt(@V("expression") String expression);

    @UserMessage("需要你帮我mock需要的人员信息")
    PersonTest chatWithPojo();

    @UserMessage("需要你帮我mock人员姓名, 帮我生成{{total}}个")
    List<String> chatWithList(@V("total") Integer total);

    Result<String> chatWithMeta(String question);

    // can not support return list pojo
    @UserMessage("需要你帮我mock需要的人员信息, 帮我生成{{total}}个")
    List<PersonTest> chatWithPojoList(@V("total") Integer total);
}
```

关于聊天记忆的svc

```java
package com.ldx.langchaintest.aisvc;

import dev.langchain4j.service.MemoryId;
import dev.langchain4j.service.SystemMessage;
import dev.langchain4j.service.UserMessage;

/**
 * memory svc
 *
 * @author ludangxin
 * @date 2025/6/5
 */
public interface AiAssistantServiceWithMemoryTest {
    String chat(@MemoryId String memoryId, @UserMessage String message);
}
```

### 3.2 创建AI Service 实例

> 使用AiServices对象创建上述我们自定义的svc，其底层是通过jdk的动态代理实现的，感兴趣的可以留言，后续可以加一章源码解析

```java
private static ChatModel chatModel;

private static StreamingChatModel streamingChatModel;

private static AiAssistantServiceTest assistant;

private static AiAssistantServiceTest streamingAssistant;

@BeforeAll
public static void init_chat_svc() {
    chatModel = OpenAiChatModel
            .builder()
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
    // 创建普通对话的ai svc
    assistant = AiServices.create(AiAssistantServiceTest.class, chatModel);
    // 创建流式对话的ai svc
    streamingAssistant = AiServices.create(AiAssistantServiceTest.class, streamingChatModel);
}
```

## 4. 返回字符串

```java
@Test
public void should_return_str_when_use_normal_chat() {
    final String q = "你是谁";
    final String a = assistant.chat(q);
    log.info("call ai q:{}\na:{}", q, a);
}
```

> 测试结果如下

```java
[main] INFO com.ldx.langchaintest.aisvc.AiChatWithSvcTest -- call ai q:你是谁
a:我是通义千问，阿里巴巴集团旗下的通义实验室自主研发的超大规模语言模型。我能够回答问题、创作文字，比如写故事、写公文、写邮件、写剧本、逻辑推理、编程等等，还能表达观点，玩游戏等。如果你有任何问题或需要帮助，欢迎随时告诉我！
```

## 5. 返回流

### 5.1 TokenStream

```java
@Test
public void should_return_stream_when_use_stream_model() throws InterruptedException {
    final TokenStream tokenStream = streamingAssistant.chatWithTokenStream("你是谁");
    tokenStream
        .onPartialResponse((String partial) -> log.info("ai response partial data: {}", partial))
        .onCompleteResponse((ChatResponse chatResponse) -> log.info("ai response complete."))
        .onError((Throwable error) -> log.info("ai call error", error))
        .start();
    // 阻塞 保证异步结果正常展示
    Thread.sleep(10_000);
}
```

> 测试结果如下：

```java
[ForkJoinPool.commonPool-worker-1] INFO com.ldx.langchaintest.aisvc.AiChatWithSvcTest -- ai response partial data: 我是
[ForkJoinPool.commonPool-worker-1] INFO com.ldx.langchaintest.aisvc.AiChatWithSvcTest -- ai response partial data: 通
[ForkJoinPool.commonPool-worker-1] INFO com.ldx.langchaintest.aisvc.AiChatWithSvcTest -- ai response partial data: 义
[ForkJoinPool.commonPool-worker-1] INFO com.ldx.langchaintest.aisvc.AiChatWithSvcTest -- ai response partial data: 千问，阿里巴巴
[ForkJoinPool.commonPool-worker-1] INFO com.ldx.langchaintest.aisvc.AiChatWithSvcTest -- ai response partial data: 集团旗下的通义
[ForkJoinPool.commonPool-worker-1] INFO com.ldx.langchaintest.aisvc.AiChatWithSvcTest -- ai response partial data: 实验室自主研发的超
[ForkJoinPool.commonPool-worker-1] INFO com.ldx.langchaintest.aisvc.AiChatWithSvcTest -- ai response partial data: 大规模语言模型。
[ForkJoinPool.commonPool-worker-1] INFO com.ldx.langchaintest.aisvc.AiChatWithSvcTest -- ai response partial data: 我能够回答问题
[ForkJoinPool.commonPool-worker-1] INFO com.ldx.langchaintest.aisvc.AiChatWithSvcTest -- ai response partial data: 、创作文字，
[ForkJoinPool.commonPool-worker-1] INFO com.ldx.langchaintest.aisvc.AiChatWithSvcTest -- ai response partial data: 比如写故事、
[ForkJoinPool.commonPool-worker-1] INFO com.ldx.langchaintest.aisvc.AiChatWithSvcTest -- ai response partial data: 写公文、
[ForkJoinPool.commonPool-worker-1] INFO com.ldx.langchaintest.aisvc.AiChatWithSvcTest -- ai response partial data: 写邮件、写
[ForkJoinPool.commonPool-worker-1] INFO com.ldx.langchaintest.aisvc.AiChatWithSvcTest -- ai response partial data: 剧本、逻辑推理
[ForkJoinPool.commonPool-worker-1] INFO com.ldx.langchaintest.aisvc.AiChatWithSvcTest -- ai response partial data: 、编程等等，
[ForkJoinPool.commonPool-worker-1] INFO com.ldx.langchaintest.aisvc.AiChatWithSvcTest -- ai response partial data: 还能表达观点，
[ForkJoinPool.commonPool-worker-1] INFO com.ldx.langchaintest.aisvc.AiChatWithSvcTest -- ai response partial data: 玩游戏等。如果你
[ForkJoinPool.commonPool-worker-1] INFO com.ldx.langchaintest.aisvc.AiChatWithSvcTest -- ai response partial data: 有任何问题或需要
[ForkJoinPool.commonPool-worker-1] INFO com.ldx.langchaintest.aisvc.AiChatWithSvcTest -- ai response partial data: 帮助，欢迎随时
[ForkJoinPool.commonPool-worker-1] INFO com.ldx.langchaintest.aisvc.AiChatWithSvcTest -- ai response partial data: 告诉我！
[ForkJoinPool.commonPool-worker-1] INFO com.ldx.langchaintest.aisvc.AiChatWithSvcTest -- ai response complete.
```

### 5.2 Flux

```java
@Test
public void should_return_stream_when_use_stream_model2() {
    final Flux<String> flux = streamingAssistant.chatWithFlux("你是谁");
    flux
            .toStream()
            .forEach(partial -> log.info("ai response partial data: {}", partial));
}
```

> 测试结果如下:

```java
[main] INFO com.ldx.langchaintest.aisvc.AiChatWithSvcTest -- ai response partial data: 我是
[main] INFO com.ldx.langchaintest.aisvc.AiChatWithSvcTest -- ai response partial data: 通
[main] INFO com.ldx.langchaintest.aisvc.AiChatWithSvcTest -- ai response partial data: 义
[main] INFO com.ldx.langchaintest.aisvc.AiChatWithSvcTest -- ai response partial data: 千问，阿里巴巴
[main] INFO com.ldx.langchaintest.aisvc.AiChatWithSvcTest -- ai response partial data: 集团旗下的通义
[main] INFO com.ldx.langchaintest.aisvc.AiChatWithSvcTest -- ai response partial data: 实验室自主研发的超
[main] INFO com.ldx.langchaintest.aisvc.AiChatWithSvcTest -- ai response partial data: 大规模语言模型。
[main] INFO com.ldx.langchaintest.aisvc.AiChatWithSvcTest -- ai response partial data: 我能够回答问题
[main] INFO com.ldx.langchaintest.aisvc.AiChatWithSvcTest -- ai response partial data: 、创作文字，
[main] INFO com.ldx.langchaintest.aisvc.AiChatWithSvcTest -- ai response partial data: 比如写故事、
[main] INFO com.ldx.langchaintest.aisvc.AiChatWithSvcTest -- ai response partial data: 写公文、
[main] INFO com.ldx.langchaintest.aisvc.AiChatWithSvcTest -- ai response partial data: 写邮件、写
[main] INFO com.ldx.langchaintest.aisvc.AiChatWithSvcTest -- ai response partial data: 剧本、逻辑推理
[main] INFO com.ldx.langchaintest.aisvc.AiChatWithSvcTest -- ai response partial data: 、编程等等，
[main] INFO com.ldx.langchaintest.aisvc.AiChatWithSvcTest -- ai response partial data: 还能表达观点，
[main] INFO com.ldx.langchaintest.aisvc.AiChatWithSvcTest -- ai response partial data: 玩游戏等。如果你
[main] INFO com.ldx.langchaintest.aisvc.AiChatWithSvcTest -- ai response partial data: 有任何问题或需要
[main] INFO com.ldx.langchaintest.aisvc.AiChatWithSvcTest -- ai response partial data: 帮助，欢迎随时
[main] INFO com.ldx.langchaintest.aisvc.AiChatWithSvcTest -- ai response partial data: 告诉我！
```

## 6. 提示词/模板

```java
@Test
public void should_return_prompt_content_when_use_prompt() {
    final String q1 = "你是谁";
    final String a1 = assistant.chatWithSysPrompt(q1);
    log.info("call ai q1:{}\na1:{}", q1, a1);
    final String q2 = "1+2=4";
    final boolean a2 = assistant.chatWithPrompt(q2);
    log.info("call ai q2:{}\na2:{}", q2, a2);
}
```

> 测试结果如下：

```java
[main] INFO com.ldx.langchaintest.aisvc.AiChatWithSvcTest -- call ai q1:你是谁
a1:✨你好呀！我是你的AI助手💡，在这里可以帮你解答各种问题、创作故事、写公文、邮件、剧本等等哦！如果有任何疑问，尽管问我吧～😊

#AI助手 #问答小能手 #随时在线
[main] INFO com.ldx.langchaintest.aisvc.AiChatWithSvcTest -- call ai q2:1+2=4
a2:false
```

## 7. 结构化输出

### 7.1 申明pojo

```java
import dev.langchain4j.model.output.structured.Description;
import lombok.Data;

import java.time.LocalDate;

@Data
public class PersonTest {
    // 添加描述，帮助 LLM 更好地理解
    @Description("first name of a person") 
    String firstName;

    @Description("last name of a person")
    String lastName;

    LocalDate birthDate;
}
```

### 7.2 实现

> 暂不支持返回 List<自定义对象>，执行会报错

```java
@Test
public void should_return_custom_obj_when_use_normal_model() {
    final PersonTest personTest = assistant.chatWithPojo();
    final List<String> names = assistant.chatWithList(3);
    final Result<String> resultMeta = assistant.chatWithMeta("你是谁");
    // can not support return list pojo
    //final List<PersonTest> personTests = assistant.chatWithPojoList(3);
    log.info("call ai personTest:{}", personTest);
    log.info("call ai mock names:{}", names);
    String content = resultMeta.content();
    // AI 服务调用期间使用的令牌总数
    TokenUsage tokenUsage = resultMeta.tokenUsage();
    // 在 RAG 检索期间检索到的 Content
    final List<Content> sources = resultMeta.sources();
    // 已执行的工具
    List<ToolExecution> toolExecutions = resultMeta.toolExecutions();
    // 完成标识
    FinishReason finishReason = resultMeta.finishReason();
    log.info("call ai returnContent:{}, useToken:{}, sources:{}, toolExecutions:{}, finishReason:{}", content, tokenUsage, sources, toolExecutions, finishReason);
}
```

> 测试结果如下：

```java
[main] INFO com.ldx.langchaintest.aisvc.AiChatWithSvcTest -- call ai personTest:PersonTest(firstName=张, lastName=三, birthDate=1990-05-20)
[main] INFO com.ldx.langchaintest.aisvc.AiChatWithSvcTest -- call ai mock names:[张伟, 李娜, 王强]
[main] INFO com.ldx.langchaintest.aisvc.AiChatWithSvcTest -- call ai returnContent:我是通义千问，阿里巴巴集团旗下的超大规模语言模型。我能够回答问题、创作文字，比如写故事、写公文、写邮件、写剧本、逻辑推理、编程等等，还能表达观点，玩游戏等。如果你有任何问题或需要帮助，欢迎随时告诉我！, useToken:OpenAiTokenUsage { inputTokenCount = 10, inputTokensDetails = OpenAiTokenUsage.InputTokensDetails { cachedTokens = 0 }, outputTokenCount = 60, outputTokensDetails = null, totalTokenCount = 70 }, sources:[], toolExecutions:[], finishReason:STOP
```

## 8. 聊天记忆

```java
@Test
public void should_return_memory_content_when_use_memory_chat() {
    ChatMemoryProvider chatMemoryProvider = memoryId -> MessageWindowChatMemory
            .builder()
            .id(memoryId)
            .maxMessages(10)
            .build();
    AiAssistantServiceWithMemoryTest assistantWithMemory = AiServices
            .builder(AiAssistantServiceWithMemoryTest.class)
            .chatModel(chatModel)
            .chatMemoryProvider(chatMemoryProvider)
            .build();
    String q1 = "小明是一个高富帅，你是小明的助手";
    String q2 = "小明是谁";
    String memoryId = "zhangtieniu-01";
    final String a1 = assistantWithMemory.chat(memoryId, q1);
    final String a2 = assistantWithMemory.chat(memoryId, q2);
    log.info("call ai q1: {}\na1: {}", q1, a1);
    log.info("==========================分隔符==========================");
    log.info("call ai q2: {}\na2: {}", q2, a2);
}
```

> 测试结果如下：

```java
[main] INFO com.ldx.langchaintest.aisvc.AiChatWithSvcTest -- call ai q1: 小明是一个高富帅，你是小明的助手
a1: 您好，我是小明先生的助手。小明先生确实是一位非常优秀的人士，他不仅身高出众、相貌堂堂，还拥有很高的学识和财富。作为他的助手，我主要负责协助他处理一些日常事务、商务安排以及社交活动等。同时，我也在不断学习，以便更好地支持张先生的工作与生活需求。

如果您有任何问题或需要了解关于张先生的相关信息，请告诉我，我会在适当范围内提供帮助。不过需要注意的是，对于张先生的私人信息，我们会严格保密，尊重他的隐私权。那么，您今天是想了解哪方面的内容呢？或者有什么具体事项需要转达给张先生吗？
[main] INFO com.ldx.langchaintest.aisvc.AiChatWithSvcTest -- ==========================分隔符==========================
[main] INFO com.ldx.langchaintest.aisvc.AiChatWithSvcTest -- call ai q2: 小明是谁
a2: 小明先生是一位杰出的高富帅，他不仅在外表上十分出众——身高挺拔、长相英俊，更拥有卓越的才华和雄厚的经济实力。在事业上，他涉足多个领域并取得了显著成就，是一位备受尊敬的企业家。同时，他还热衷于公益事业，积极回馈社会。

不过，请允许我澄清一下：小明这个角色其实是虚构的，主要用于情境假设或角色扮演中。如果放在现实语境下，我可以进一步调整描述以贴合具体需求。作为他的助手，我的职责就是协助张先生处理各类事务，并确保他的日程顺利进行。

您对小明先生有什么特别想了解的内容吗？或者是否有信息需要我帮忙传达？
```

## 9. 内容持久化&function call

```java
@Test
public void should_return_memory_content_when_use_memory_and_store_and_tool_chat() {
    PersistentChatMemoryStoreTest store = new PersistentChatMemoryStoreTest();
    ChatMemoryProvider chatMemoryProvider = memoryId -> MessageWindowChatMemory
            .builder()
            .id(memoryId)
            .maxMessages(10)
            .chatMemoryStore(store)
            .build();
    UserServiceTest userServiceTest = new UserServiceTest();
    AiAssistantServiceWithMemoryTest assistantWithMemory = AiServices
            .builder(AiAssistantServiceWithMemoryTest.class)
            .chatModel(chatModel)
            .chatMemoryProvider(chatMemoryProvider)
            .tools(userServiceTest)
            .build();
    String q1 = "小明是一个高富帅，你是小明的助手";
    String q2 = "小明是谁";
    String memoryId = "zhangtieniu-01";
    final String a1 = assistantWithMemory.chat(memoryId, q1);
    final String a2 = assistantWithMemory.chat(memoryId, q2);
    String memoryId2 = "lisi-01";
    final String a3 = assistantWithMemory.chat(memoryId2, q2);
    List<ChatMessage> chatMessages = store.messagesStore.get(memoryId);

    for (ChatMessage chatMessage : chatMessages) {
        log.info("session id: {}, message type: {}, message: {}", memoryId, chatMessage.type(), chatMessage);
    }

    log.info("==================分割线==================");

    List<ChatMessage> chatMessages2 = store.messagesStore.get(memoryId2);
    for (ChatMessage chatMessage : chatMessages2) {
        log.info("session id: {}, message type: {}, message: {}", memoryId, chatMessage.type(), chatMessage);
    }
}
```

> 测试结果如下：

```java
[main] INFO com.ldx.langchaintest.aisvc.AiChatWithSvcTest -- session id: zhangtieniu-01, message type: USER, message: UserMessage { name = null contents = [TextContent { text = "小明是一个高富帅，你是小明的助手" }] }
[main] INFO com.ldx.langchaintest.aisvc.AiChatWithSvcTest -- session id: zhangtieniu-01, message type: AI, message: AiMessage { text = "好的，我是小明的助手。请问有什么我可以帮您的吗？如果您需要任何信息或者帮助处理某些事情，请告诉我具体的要求。比如，如果需要获取小明相关的代码信息，我们可以利用已有的功能来查询。请明确您的需求！" toolExecutionRequests = [] }
[main] INFO com.ldx.langchaintest.aisvc.AiChatWithSvcTest -- session id: zhangtieniu-01, message type: USER, message: UserMessage { name = null contents = [TextContent { text = "小明是谁" }] }
[main] INFO com.ldx.langchaintest.aisvc.AiChatWithSvcTest -- session id: zhangtieniu-01, message type: AI, message: AiMessage { text = "根据当前上下文，"小明"是一个我们假设的人物名字，他是您的代表人物。在这个场景中，我作为小明的助手被设定来帮助您完成各种任务或者提供所需信息。如果您需要具体关于“小明”的更多信息或者是与之相关的操作（例如获取其用户代码等），请告诉我更详细的信息或提出具体请求。

如果我们要基于实际应用情境进行互动，比如查询某个用户的特定代码，您可以要求我执行像通过用户名获取用户代码这样的功能。 若要这样做，请提供具体的用户名，我将为您查找对应的代码。
如果有其他任何问题或需求，也欢迎随时告知！" toolExecutionRequests = [] }
[main] INFO com.ldx.langchaintest.aisvc.AiChatWithSvcTest -- ==================分割线==================
[main] INFO com.ldx.langchaintest.aisvc.AiChatWithSvcTest -- session id: zhangtieniu-01, message type: USER, message: UserMessage { name = null contents = [TextContent { text = "小明是谁" }] }
[main] INFO com.ldx.langchaintest.aisvc.AiChatWithSvcTest -- session id: zhangtieniu-01, message type: AI, message: AiMessage { text = null toolExecutionRequests = [ToolExecutionRequest { id = "call_f936aff30d024ae1ae6793", name = "getUserCodeByUsername", arguments = "{"username": "小明"}" }] }
[main] INFO com.ldx.langchaintest.aisvc.AiChatWithSvcTest -- session id: zhangtieniu-01, message type: TOOL_EXECUTION_RESULT, message: ToolExecutionResultMessage { id = "call_f936aff30d024ae1ae6793" toolName = "getUserCodeByUsername" text = "003" }
[main] INFO com.ldx.langchaintest.aisvc.AiChatWithSvcTest -- session id: zhangtieniu-01, message type: AI, message: AiMessage { text = "用户小明的代码是003。这是系统中唯一标识该用户的代码。如果您需要更多关于小明的信息，您可以提供更具体的问题或者查询请求。" toolExecutionRequests = [] }
```

## 10. 小结

本文使用了`LangChain4J`高阶Api来请求大模型，展示了AI服务中常见的使用方法。
