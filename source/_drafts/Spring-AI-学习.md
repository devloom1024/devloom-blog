---
title: Spring AI 学习笔记
tags: [Spring AI, 学习笔记]
---



# 核心概念



## 模型 (Models)

- AI 模型是处理和生成信息的算法，通常模仿人类的认知功能。
- Spring AI 目前支持处理语言、图像和音频输入输出的模型，同时也支持将文本转换为数字表示（即“嵌入”）的模型。 

    ![模型分类](https://docs.spring.io/spring-ai/reference/1.0/_images/spring-ai-concepts-model-types.jpg)

- 像 GPT 这样的预训练模型，使得不具备深厚机器学习背景的开发者也能轻松使用 AI。



## 提示

- 提示是引导 AI 模型生成特定输出的语言基础输入。
- 一个有效的提示不仅仅是简单的字符串，可能包含多个带有“角色”（如 system 和 user）的文本输入，用以设定上下文和行为。



### 提示模板

其功能类似于 Spring MVC 中的“视图”，允许开发者创建请求的上下文，并用特定于用户输入的值来填充模板中的占位符。



## 嵌入

- 嵌入是文本、图像或视频的数字表示形式，它被转换为浮点数数组（即“向量”），用以捕捉输入内容之间的语义关系。
- 应用程序可以通过计算向量之间的数值距离来判断内容的相似性。
- 嵌入在“检索增强生成 (RAG)”等应用中至关重要，它能将数据表示为高维语义空间中的点，相似的内容在空间中的距离更近。


![](https://docs.spring.io/spring-ai/reference/1.0/_images/spring-ai-embeddings.jpg)


更通俗的理解可以参考 [大模型靠啥理解文字？通俗解释：词嵌入embedding](https://www.bilibili.com/video/BV1bfoQYCEHC/?share_source=copy_web&vd_source=9499333ae9d7c70c0bf6b29add6d04c2)。



## Tokens


- Tokens 是 AI 模型处理信息的基本单位；模型在输入时将文字转换为 Tokens，输出时再将 Tokens 转回文字。
- 在英语中，一个 Token 大约相当于 0.75 个单词。
- Tokens 的使用量直接关系到 AI 服务的费用，并且模型有“上下文窗口”的 Tokens 数量限制，超出部分不会被处理。

![](https://docs.spring.io/spring-ai/reference/1.0/_images/spring-ai-concepts-tokens.png)



## 结构化输出

- 传统上，即使要求 AI 模型返回 JSON，其输出也只是格式正确的字符串，而非可以直接在程序中使用的“数据结构”。
- Spring AI 提供了结构化输出转换器，通过精心设计的提示，将 AI 生成的简单字符串转换为应用程序可用的数据结构。

![](https://docs.spring.io/spring-ai/reference/1.0/_images/structured-output-architecture.jpg)



## 将您的数据和 API 引入 AI 模型


由于像 GPT-4 这类模型的知识有其时间限制（例如，数据截至 2021 年 9 月），为了让 AI 模型能够对于尚未训练的知识也能够很好的回答，有如下三种方式引入自有数据的技术：



### 微调 (Fine-tuning)

这是一种传统的机器学习技术，需要修改模型内部权重，过程复杂且资源消耗大。



### 提示填充 (Prompt Stuffing) / 检索增强生成 (RAG)


- 这是一种更实用的方法，将相关数据嵌入到提供给模型的提示中。
- RAG 是实现这一目标的成熟技术。
  - 它通过一个 ETL (提取、转换、加载) 流程，将非结构化文档进行语义分割，转换后存入向量数据库。
  - 当用户提问时，系统会从向量数据库中检索相似的内容片段，并连同问题一起放入提示中，以生成更准确的回答。

  ![](https://docs.spring.io/spring-ai/reference/1.0/_images/spring-ai-rag.jpg)



### 工具调用 (Tool Calling)

- 该技术允许开发者将自定义的服务（外部 API）注册为“工具”，让大型语言模型可以借此访问实时信息或执行外部操作。
- Spring AI 简化了这个流程，开发者可以通过 @Tool 注解来定义工具。模型会根据需要决定是否调用该工具，并将结果作为附加上下文，用于生成最终的响应。


![](https://docs.spring.io/spring-ai/reference/1.0/_images/tools/tool-calling-01.jpg)





## 评估 AI 响应

- 为了确保 AI 系统输出的准确性和实用性，对其响应进行有效评估至关重要。
- 一种方法是利用模型本身来评估其生成的响应是否与用户请求及提供的资料一致。
- Spring AI 提供了 Evaluator API 来帮助开发者评估模型的回应。



# 聊天客户端 API

> 详细内容参考：
>
> - [聊天客户端 API](https://www.spring-doc.cn/spring-ai/1.0.0/api_chatclient.html)
>
> - [通知器 (advisors)](https://www.spring-doc.cn/spring-ai/1.0.0/api_advisors.html)



`ChatClient` 是一个用于与 AI 模型进行交互的 Fluent API。它同时支持同步和流式（响应式）编程模型。



## 核心功能与概念


- **Fluent API**: `ChatClient` 提供了一套链式调用的方法，可以方便地构建向 AI 模型发送的 `Prompt`。一个 `Prompt` 由多条消息（用户消息和系统消息）组成。
- **创建 `ChatClient`**:
  - 可以通过 Spring Boot 自动配置的 `ChatClient.Builder` 来注入并构建一个 `ChatClient` 实例。
  - 在需要使用多个不同配置或不同类型的聊天模型（如 OpenAI, Anthropic）时，可以禁用自动配置 (`spring.ai.chat.client.enabled=false`)，然后以编程方式创建和注册多个 `ChatClient` Bean。
  - 文档还特别提到了如何使用 `mutate()` 方法来创建基于同一 `OpenAiChatModel` 但指向不同 API 端点（例如，一个指向 OpenAI，另一个指向 Groq）的客户端实例。
- **Prompt 模板**:
  - API 支持在用户和系统消息中使用模板，其中的占位符（默认为 {}）可以在运行时被动态替换。
  - 默认使用基于 `StringTemplate` 的 `StTemplateRenderer`，但用户可以自定义模板分隔符（例如，使用 `<` 和` >` 以避免与 JSON 语法冲突）或提供自己的 `TemplateRenderer` 实现。
- **处理响应**:
  - **同步调用 (`call()`)**: 可以返回多种格式的结果：
    - `content()`: 返回纯文本字符串响应。
    - `chatResponse()`: 返回包含元数据（如 token 使用量）的 `ChatResponse` 对象。
    -` entity()`: 将返回的字符串自动转换为指定的 Java 实体类（POJO），支持泛型集合。
  - **流式调用 (stream())**: 以异步方式返回响应：
    - `content()`: 返回一个 `Flux<String>` 的响应流。
    - `chatResponse()`: 返回 `Flux<ChatResponse>` 对象流。





## 高级功能

- **设置默认值**: 可以在构建 `ChatClient` 时通过 `ChatClient.Builder` 设置各种默认值，如默认的系统消息、模型选项 (`ChatOptions`)、函数调用、用户信息和 `Advisor`，从而简化运行时的代码。

- **通知器 (`Advisors`)**:
  - `Advisor` 是一种在生成最终 `Prompt` 之前对其进行修改或增强的机制。添加 `Advisor` 的顺序决定了它们的执行顺序。
  - 主要应用场景包括：
    - **检索增强生成 (RAG)**: 使用 `QuestionAnswerAdvisor` 将从外部数据源（如 `VectorStore`）检索到的相关信息附加到提示中，让模型能基于私有数据进行回答。
    - **对话历史记录**: `MessageChatMemoryAdvisor` 可以自动将之前的对话历史附加到请求中，使模型能够“记住”上下文。
    - **日志记录**: `SimpleLoggerAdvisor` 用于调试，可以记录请求和响应的详细信息。
  
- **聊天记忆 (`Chat Memory`)**:
  - `ChatMemory` 接口用于存储对话历史。
  - `MessageWindowChatMemory` 是一个内置实现，它维护一个固定大小的消息窗口，当消息超过限制时会自动移除旧消息，但会保留系统消息。它依赖于 `ChatMemoryRepository` 进行持久化存储，已有内存、JDBC、Cassandra 和 Neo4j 等多种实现。





## 重要实施说明

- `ChatClient` 的设计独特之处在于它混合了命令式和响应式编程模型。
- **依赖项**: 流式处理需要响应式技术栈（如 `spring-boot-starter-webflux`），而非流式处理则需要 Servlet 技术栈（如 `spring-boot-starter-web`）。因此，一个应用可能需要同时包含两者。
- **阻塞调用**: 工具（函数）调用本质上是命令式的，这可能会在响应式工作流中引入阻塞。
- **Spring Boot 3.4 Bug**: 文档提示，由于一个 Bug，必须设置属性 `spring.http.client.factory=jdk`，否则某些 AI 工作流可能会中断。





# 提示 (Prompt)



>  详细内容参考：[提示](https://www.spring-doc.cn/spring-ai/1.0.0/api_prompt.html)



## 核心交互流程



在 Spring AI 中，与 AI 模型交互的基本流程是：

- 调用 `ChatModel` 的 `call()` 方法。
- 该方法接收一个 `Prompt` 对象作为输入。
- 方法返回一个 `ChatResponse` 对象作为输出。



## Prompt：请求的容器



`Prompt` 是发送给 AI 模型的最终请求对象。它像一个容器，主要包含两部分：

1. **`List<Message>`**：一个由多条 Message 对象组成的列表，构成了对话的上下文。
2. **`ChatOptions`**：一个可选的配置对象，用于指定 AI 模型生成响应时的参数（如温度、最大Token数等）。



## Message：对话的基本单元

Message 是构成对话的最小单位，每条消息都包含内容、元数据和角色。

- **核心属性**:
  - `getContent()`: 获取消息的文本内容。
  - `getMetadata()`: 获取消息的元数据（一个 Map）。
  - `getMessageType()`: 获取消息的角色（Role）。
- **多模态消息**: 对于包含图片等多媒体内容的消息，还实现了 MediaContent 接口。
- **消息角色 (`MessageType`)**: 角色用于区分消息的来源和意图，指导 AI 如何理解和回应。主要有四种角色：
  - **SYSTEM (系统角色)**: 用于设定 AI 的行为、个性和响应风格，相当于给 AI 下达的全局指令。
  - **USER (用户角色)**: 代表最终用户的提问、指令或输入。
  - **ASSISTANT (助理角色)**: 代表 AI 自身的回复。在多轮对话中，将 AI 之前的回复作为历史消息传入，有助于保持对话的连贯性。它也可能包含对工具（Function）的调用请求。
  - **TOOL (工具角色)**: 当 AI 请求调用工具后，此角色用于返回工具执行的结果。



![Spring AI Message API](https://docs.spring.io/spring-ai/reference/1.0/_images/spring-ai-message-api.jpg)



## PromptTemplate：动态构建提示

`PromptTemplate` 是一个强大的工具类，用于方便地创建结构化和动态的提示。

- **核心功能**:
  - 它允许你定义一个包含占位符（如 {variable}）的模板字符串。
  - 通过 `render()` 方法，可以将一个 `Map` 对象中的值填充到模板的占位符中，生成最终的提示内容。
- **渲染引擎 (`TemplateRenderer`)**:
  - `PromptTemplate` 内部使用 `TemplateRenderer` 接口来执行模板渲染。
  - 默认实现是 `StTemplateRenderer`，它基于强大的 **`StringTemplate`** 引擎。
  - 你可以自定义渲染器，甚至可以配置模板的分隔符（默认是 {}，可以改为 <> 等）。
- **主要用法 (通过接口体现)**:
  `PromptTemplate` 实现了多个功能接口，使其能够生成不同类型的输出：
  1. **`PromptTemplateStringActions`**: 主要用于将模板渲染成一个最终的 **字符串**。
  2. **`PromptTemplateMessageActions`**: 用于根据模板创建一个 **`Message` 对象**，可以包含静态或动态内容。
  3. **`PromptTemplateActions`**: 功能最全面，用于根据模板直接创建出一个完整的 **`Prompt` 对象**（包含消息列表和可选的 `ChatOptions`），可以直接传递给 `ChatModel`。



## 创建有效的提示



在开发提示时，重要的是要集成几个关键组件以确保清晰度和有效性：

- **说明**：向 AI 提供清晰直接的指示，类似于您与人交流的方式。这种清晰度对于帮助 AI “理解” 预期内容至关重要。
- **外部背景**：必要时包括 AI 响应的相关背景信息或具体指导。这个 “外部上下文” 构建了提示并帮助 AI 掌握整体场景。
- **用户输入**：这是简单的部分 - 用户的直接请求或问题构成了提示的核心。
- **输出指示器**： 这方面可能很棘手。它涉及为 AI 的响应指定所需的格式，例如 JSON。但是，请注意，AI 可能并不总是严格遵守此格式。例如，它可能会在实际 JSON 数据之前预置“here is your JSON”之类的短语，或者有时会生成不准确的类似 JSON 的结构。

在制作提示时，向 AI 提供预期问答格式的示例可能非常有益。 这种做法有助于 AI “理解” 查询的结构和意图，从而获得更精确和相关的响应。 虽然本文档没有深入探讨这些技术，但它们为进一步探索 AI 提示工程提供了一个起点，更加详细的提示词工程可以学习：[Prompt-Engineering-Guide](https://github.com/dair-ai/Prompt-Engineering-Guide)。






# 结构化输出



>  详细内容参考：[结构化输出](https://www.spring-doc.cn/spring-ai/1.0.0/api_structured-output-converter.html)



Spring AI 提供了 `StructuredOutputConverter` 接口，旨在帮助开发者将大语言模型（LLM）的文本输出，转换为结构化的数据格式，如 JSON 或 Java 对象。这对于需要可靠的、可被下游应用程序直接使用的数据格式的场景至关重要。



## 工作原理



结构化输出转换器通过以下两个关键步骤实现其功能：

1. **调用 LLM 之前**：转换器会自动向用户提示（Prompt）中附加格式说明。这些说明会指导 AI 模型按照预期的结构（如 JSON Schema）生成文本。
2. **调用 LLM 之后**：转换器接收模型生成的文本输出，并将其解析、转换为指定的目标数据类型实例，例如一个 Java 类或一个 Map。

值得注意的是，这是一种“最大努力”的尝试，并不保证模型总能完美遵循格式要求。因此，建议实施额外的验证机制。



![img](https://docs.spring.io/spring-ai/reference/1.0/_images/structured-output-architecture.jpg)





## 主要的转换器实现





Spring AI 框架提供了一系列预置的转换器：

- **`BeanOutputConverter<T>`**: 最常用的转换器之一。它能够将 LLM 的输出直接转换为指定的 Java 类（Bean）的实例。它通过生成 JSON Schema 来指导模型，并使用 `ObjectMapper` 将返回的 JSON 字符串反序列化为 Java 对象。
- **`MapOutputConverter`**: 将模型的输出转换为 `java.util.Map<String, Object>` 实例。它会指导模型生成符合 RFC8259 规范的 JSON 响应。
- **`ListOutputConverter`**: 用于将模型的输出转换为 `java.util.List`。它会指导模型生成一个以逗号分隔的列表。



![](https://docs.spring.io/spring-ai/reference/1.0/_images/structured-output-hierarchy4.jpg)

## 内置 JSON 模式

一些 AI 模型提供专用的配置选项来生成结构化（通常是 JSON）输出，当使用这些模型时，可以通过配置相应的选项来强制模型输出严格合法的 JSON，这可以与 Spring AI 的转换器协同工作，以获得更可靠的结构化输出。

- [OpenAI 结构化输出](https://www.spring-doc.cn/spring-ai/1.0.0/api_chat_openai-chat.html#_structured_outputs)可以确保您的模型生成严格符合您提供的 JSON 架构的响应。您可以在`JSON_OBJECT`保证模型生成的消息是有效的 JSON 或`JSON_SCHEMA`使用提供的架构，保证模型将生成与您提供的架构 （`spring.ai.openai.chat.options.responseFormat`选项）。
- [Azure OpenAI](https://www.spring-doc.cn/spring-ai/1.0.0/api_chat_azure-openai-chat.html) - 提供`spring.ai.azure.openai.chat.options.responseFormat`选项指定模型必须输出的格式。设置为`{ "type": "json_object" }`启用 JSON 模式，该模式保证模型生成的消息是有效的 JSON。
- [Ollama](https://www.spring-doc.cn/spring-ai/1.0.0/api_chat_ollama-chat.html) - 提供`spring.ai.ollama.chat.options.format`选项以指定返回响应的格式。目前，唯一接受的值是`json`.
- [Mistral AI](https://www.spring-doc.cn/spring-ai/1.0.0/api_chat_mistralai-chat.html) - 提供`spring.ai.mistralai.chat.options.responseFormat`选项以指定返回响应的格式。将其设置为`{ "type": "json_object" }`启用 JSON 模式，该模式保证模型生成的消息是有效的 JSON。







# 多模态



> 详细内容参考：[多模态](https://www.spring-doc.cn/spring-ai/1.0.0/api_multimodality.html)



## 核心概念：多模态

多模态（Multimodality）是指大型语言模型（LLM）能够同时理解和处理来自多种不同来源或格式的信息的能力，这些信息包括文本、图像、音频和视频等。与只处理单一信息类型（如纯文本）的传统模型不同，多模态模型可以结合多种输入来生成更全面的文本响应。





## Spring AI 的实现方式

Spring AI 通过其 `Message` API 提供了支持多模态功能的抽象。具体实现于 `UserMessage` 类中，该类包含了两个关键字段：

- **`content`**: 用于存放主要的文本输入。
- **`media`**: 一个可选字段，允许开发者添加一个或多个非文本内容，如图像、音频或视频。通过 `MimeType` 来指定媒体的类型。



![](https://docs.spring.io/spring-ai/reference/1.0/_images/spring-ai-message-api.jpg)



**注意**：

- 目前，`media` 字段仅适用于用户输入消息 (`UserMessage`)。
- 模型的响应 (`AssistantMessage`) 仍然仅包含文本内容。如果需要生成非文本输出（如图、音频），则应使用专门的单模态模型。



## 代码示例



- **使用 `ChatClient` 的 Fluent API 示例：**

```java
String response = ChatClient.create(chatModel).prompt()
		.user(u -> u.text("Explain what do you see on this picture?") // 文本内容
				    .media(MimeTypeUtils.IMAGE_PNG, new ClassPathResource("/multimodal.test.png"))) // 媒体内容
		.call()
		.content();
```



- **直接使用 `UserMessage` 的示例：**

```java
var imageResource = new ClassPathResource("/multimodal.test.png");

var userMessage = new UserMessage(
	"Explain what do you see in this picture?", // content
	new Media(MimeTypeUtils.IMAGE_PNG, this.imageResource)); // media

ChatResponse response = chatModel.call(new Prompt(this.userMessage));
```







# 各种类型的模型



详细参考：https://www.spring-doc.cn/spring-ai/1.0.0/api_index.html





# 聊天记忆

> 详细内容参考：https://docs.spring.io/spring-ai/reference/api/chat-memory.html#_memory_in_chat_client



大型语言模型（LLMs）是无状态的，这意味着它们不会保留关于先前交互的信息。当您希望在多个交互中保持上下文或状态时，这可能是一个限制。为了解决这个问题，Spring AI 提供了聊天记忆功能，允许您在多个与 LLM 的交互中存储和检索信息。



## 核心概念



- **聊天记忆 (Chat Memory) vs 聊天记录 (Chat History)**:
  - **聊天记忆**: 指为了维持当前对话上下文，需要提供给 LLM 的相关信息。Spring AI 的 ChatMemory 抽象主要用于管理这部分内容。
  - **聊天记录**: 指用户与模型之间全部的、完整的对话记录，通常用于审计或存档。记录建议使用 Spring Data 等持久化方案来存储完整的聊天记录，而不是使用 ChatMemory。
- **核心抽象**:
  - ChatMemory: 定义了记忆管理的策略，例如决定保留哪些消息、何时移除旧消息（如保留最近的N条消息）。
  - ChatMemoryRepository: 负责记忆的底层存储和检索，是 ChatMemory 策略的具体实现。



## 记忆类型 (Memory Types)



### Message Window Chat Memory



这是 Spring AI 默认的记忆管理策略。它只保留一个固定大小的“消息窗口”（默认为20条），当新消息加入时，最老的消息会被移除，从而控制发送给 LLM 的上下文长度。



```java
MessageWindowChatMemory memory = MessageWindowChatMemory.builder()
    .maxMessages(10)
    .build();
```





## 记忆存储（Memory Storage）



Spring AI 提供了多种可插拔的 ChatMemoryRepository 实现，用于将聊天记忆持久化到不同地方：



- **InMemoryChatMemoryRepository**:
  - **默认实现**。将记忆存储在Java应用的内存中（使用 ConcurrentHashMap）。
  - **优点**: 配置简单，速度快。
  - **缺点**: 应用重启后数据会丢失。
- **JdbcChatMemoryRepository**:
  - 使用 JDBC 将聊天记忆存储在关系型数据库中（如 PostgreSQL, MySQL, SQL Server 等）。
  - **优点**: 数据持久化，适合生产环境。
  - **特性**: 支持多种数据库方言（Dialect），并能自动或手动管理数据库表的创建（通过 initialize-schema 属性）。
- **CassandraChatMemoryRepository**:
  - 使用 Apache Cassandra 数据库存储。
  - **优点**: 适合需要高可用、高可扩展性和 TTL（Time-To-Live，消息自动过期）功能的场景。
- **Neo4jChatMemoryRepository**:
  - 使用 Neo4j 图数据库存储，将记忆和会话存为图中的节点和关系。
  - **优点**: 可以利用图数据库的特性来分析和管理对话关系。



## Chat client 使用记忆的方式

主要有两种方式：通过高阶的 ChatClient 自动管理，或通过低阶的 ChatModel 手动管理。

- **通过 ChatClient 使用（推荐方式）**:

  - 通过配置 **通知器 (Advisors)** 来自动将记忆集成到 ChatClient 的调用流程中。

  - **内置的 Advisors**:

    - MessageChatMemoryAdvisor: 在每次请求时，从内存中检索历史记忆，并将其作为消息列表（`List<Message>`）附加到提示（Prompt）中。**这是最常用的方式**。

      ```java
      // 配置
      ChatMemory chatMemory = MessageWindowChatMemory.builder().build();
      
      ChatClient chatClient = ChatClient.builder(chatModel)
          .defaultAdvisors(MessageChatMemoryAdvisor.builder(chatMemory).build())
          .build();
      
      // 使用
      String conversationId = "007";
      
      chatClient.prompt()
          .user("Do I have license to code?")
          .advisors(a -> a.param(ChatMemory.CONVERSATION_ID, conversationId))
          .call()
          .content();
      ```

      

    - PromptChatMemoryAdvisor: 从内存中检索记忆，并将其作为纯文本附加到系统提示（System Prompt）中。

    - VectorStoreChatMemoryAdvisor: 从向量数据库中检索相关的记忆记录，并附加到系统提示中。

  - 在调用时，可以通过传递 conversationId 来区分和管理不同的对话会话。





- **通过 ChatModel 使用（手动方式）**:

  - 如果直接使用更底层的 ChatModel API，开发者需要**手动管理**记忆的整个生命周期。

  - **流程**:

    1. 将用户的消息添加到 ChatMemory。
    2. 从 ChatMemory 中获取当前会话的完整消息列表。
    3. 使用该消息列表创建 Prompt 并调用 ChatModel。
    4. 将模型的响应消息也添加到 ChatMemory 中，以便下次使用。

    ```java
    // Create a memory instance
    ChatMemory chatMemory = MessageWindowChatMemory.builder().build();
    String conversationId = "007";
    
    // First interaction
    UserMessage userMessage1 = new UserMessage("My name is James Bond");
    chatMemory.add(conversationId, userMessage1);
    ChatResponse response1 = chatModel.call(new Prompt(chatMemory.get(conversationId)));
    chatMemory.add(conversationId, response1.getResult().getOutput());
    
    // Second interaction
    UserMessage userMessage2 = new UserMessage("What is my name?");
    chatMemory.add(conversationId, userMessage2);
    ChatResponse response2 = chatModel.call(new Prompt(chatMemory.get(conversationId)));
    chatMemory.add(conversationId, response2.getResult().getOutput());
    
    // The response will contain "James Bond"
    ```

    







# 工具调用



## 什么是工具调用



工具调用（也称为函数调用）是一种常见的人工智能应用模式，它允许模型利用外部工具来完成自身无法独立完成的任务。主要分为两类：

*   **信息检索 (Information Retrieval)**：模型调用工具从外部数据源（如数据库、Web 服务、文件系统）获取它本身不知道的信息。例如，获取当前天气、查询最新的新闻或检索特定客户的订单记录。这是实现 **检索增强生成 (RAG)** 的关键。
*   **执行操作 (Taking Action)**：模型调用工具在软件系统中执行具体动作。例如，发送电子邮件、预订航班、在数据库中创建新记录或触发一个工作流。



**核心安全原则**：模型本身**永远不会**直接访问或执行任何 API。它只会生成一个“工具调用请求”，其中包含要调用的工具名称和所需的参数。实际的工具执行完全由您的 Spring 应用程序负责，应用程序执行后将结果返回给模型。这种分离是至关重要的安全保障。



## 工具调用的工作流程



整个过程遵循一个清晰的六步流程：

1.  **定义与请求**：应用程序在向 AI 模型发送请求时，会附上可用工具的定义（名称、描述、输入参数的 JSON Schema）。
2.  **模型决策**：AI 模型根据用户的提问和工具的描述，判断是否需要以及如何调用某个工具，并返回一个包含工具名称和参数的响应。
3.  **应用执行**：Spring AI 框架或您的应用程序代码接收到这个工具调用请求，并执行相应的 Java 方法或函数。
4.  **获取结果**：应用程序获得工具执行的结果（例如，一个 Java 对象）。
5.  **结果返回**：应用程序将执行结果序列化后，作为新的上下文信息发送回 AI 模型。
6.  **最终响应**：AI 模型利用工具返回的结果，生成一个更准确、更丰富的最终答案给用户。



![](https://docs.spring.io/spring-ai/reference/_images/tools/tool-calling-01.jpg)



## 如何定义工具



Spring AI 提供了多种灵活的方式来定义工具，主要围绕 `ToolCallback` 接口。



### 将方法 (Methods) 定义为工具



这是最常见的方式，适用于将任意 Java 方法暴露给 AI 模型。



#### 声明式 (`@Tool` 注解)



*   在任何 Java 方法上添加 `@Tool` 注解即可将其定义为一个工具。
*   `description` 属性至关重要，它告诉模型这个工具的用途，直接影响模型的调用决策。
*   使用 `@ToolParam` 注解为方法参数提供描述和是否必需的元数据。



```java
// 定义工具
class DateTimeTools {

    @Tool(description = "Get the current date and time in the user's timezone")
    String getCurrentDateTime() {
        return LocalDateTime.now().atZone(LocaleContextHolder.getTimeZone().toZoneId()).toString();
    }
    
    @Tool(description = "Set a user alarm for the given time")
    void setAlarm(@ToolParam(description = "Time in ISO-8601 format") String time) {
        LocalDateTime alarmTime = LocalDateTime.parse(time, DateTimeFormatter.ISO_DATE_TIME);
        System.out.println("Alarm set for " + alarmTime);
    }
}


// 添加工具
ChatClient.create(chatModel)
    .prompt("What day is tomorrow?")
    .tools(new DateTimeTools())
    .call()
    .content();


// 设置为默认工具
ChatModel chatModel = ...
ChatClient chatClient = ChatClient.builder(chatModel)
    .defaultTools(new DateTimeTools())
    .build();
```





#### 编程式 (`MethodToolCallback`)



*   通过 `MethodToolCallback.Builder` 以编程方式手动构建工具定义，提供更精细的控制。



```java
// 定义工具
Method method = ReflectionUtils.findMethod(DateTimeTools.class, "getCurrentDateTime");
ToolCallback toolCallback = MethodToolCallback.builder()
    .toolDefinition(ToolDefinition.builder(method)
            .description("Get the current date and time in the user's timezone")
            .build())
    .toolMethod(method)
    .toolObject(new DateTimeTools())  // 静态方法不需要这个步骤
    .build();

// 添加工具
ToolCallback toolCallback = ...
ChatClient.create(chatModel)
    .prompt("What day is tomorrow?")
    .tools(toolCallback)
    .call()
    .content();

// 设置为默认工具
ChatModel chatModel = ...
ToolCallback toolCallback = ...
ChatClient chatClient = ChatClient.builder(chatModel)
    .defaultTools(toolCallback)
    .build();
```





#### 方法工具限制



当前不支持的类型作为工具方法参数或返回类型如下：

- `Optional`

- Asynchronous types (e.g. `CompletableFuture`, `Future`)
  异步类型（例如 `CompletableFuture` , `Future` ）

- Reactive types (e.g. `Flow`, `Mono`, `Flux`)
  响应式类型（例如 `Flow` 、 `Mono` 、 `Flux` ）

- Functional types (e.g. `Function`, `Supplier`, `Consumer`).
  函数类型（例如 `Function` , `Supplier` , `Consumer` ）。

  

  

### 将函数 (Functions) 定义为工具



适用于将 `java.util.function` 包中的函数式接口（如 `Function`, `Supplier`, `Consumer`）定义为工具，特别适合与 Spring Bean 结合使用。





#### 编程式 (`FunctionToolCallback`)



*   使用 `FunctionToolCallback.Builder` 包装一个函数式接口实例。



```java
// 定义工具
public class WeatherService implements Function<WeatherRequest, WeatherResponse> {
    public WeatherResponse apply(WeatherRequest request) {
        return new WeatherResponse(30.0, Unit.C);
    }
}

public enum Unit { C, F }
public record WeatherRequest(String location, Unit unit) {}
public record WeatherResponse(double temp, Unit unit) {}

ToolCallback toolCallback = FunctionToolCallback
    .builder("currentWeather", new WeatherService())
    .description("Get the weather in location")
    .inputType(WeatherRequest.class)
    .build();


// 添加工具
ToolCallback toolCallback = ...
ChatClient.create(chatModel)
    .prompt("What's the weather like in Copenhagen?")
    .tools(toolCallback)
    .call()
    .content();

// 设置为默认工具
ChatModel chatModel = ...
ToolCallback toolCallback = ...
ChatClient chatClient = ChatClient.builder(chatModel)
    .defaultTools(toolCallback)
    .build();
```









#### 动态化 (`@Bean` 注解)



* 将一个 `Function` 类型的 Bean 注册到 Spring 容器中。

* Bean 的名称将作为工具的名称，可以使用 `@Description` 注解提供工具描述。

* 工具的输入参数的  JSON Schema 将自动生成，也可以使用 `@ToolParam` 注解来提供有关输入参数的额外信息。

  ```java
  record WeatherRequest(@ToolParam(description = "The name of a city or a country") String location, Unit unit) {}
  ```

*   Spring AI 会在运行时动态解析这些 Bean 作为可用工具。



```java
// 定义工具

@Configuration(proxyBeanMethods = false)
class WeatherTools {

    WeatherService weatherService = new WeatherService();

	@Bean
	@Description("Get the weather in location")
	Function<WeatherRequest, WeatherResponse> currentWeather() {
		return weatherService;
	}

}

// 添加工具
ChatClient.create(chatModel)
    .prompt("What's the weather like in Copenhagen?")
    .tools("currentWeather")
    .call()
    .content();


// 设置为默认工具
ChatModel chatModel = ...
ChatClient chatClient = ChatClient.builder(chatModel)
    .defaultTools("currentWeather")
    .build();
```





#### 函数工具限制





以下类型目前不能作为工具函数的输入或输出类型：

- Primitive types 基本类型
- `Optional`
- Collection types (e.g. `List`, `Map`, `Array`, `Set`)
  集合类型（例如 `List` 、 `Map` 、 `Array` 、 `Set` ）
- Asynchronous types (e.g. `CompletableFuture`, `Future`)
  异步类型（例如 `CompletableFuture` , `Future` ）
- Reactive types (e.g. `Flow`, `Mono`, `Flux`).
  响应式类型（例如 `Flow` 、 `Mono` 、 `Flux` ）。





## 工具规范核心概念





### 工具回调 (ToolCallback)

- **核心接口**：`ToolCallback` 是从头开始定义一个工具时需要实现的主要接口。它包含了工具的定义和执行逻辑。
- **主要方法**: 
  - `getToolDefinition()`: 获取工具的定义，供 AI 模型理解何时及如何调用该工具。
  - `getToolMetadata()`: 获取工具的元数据，提供额外处理信息。
  - `call()`: 执行工具并返回结果给 AI 模型。
- **内置实现**：Spring AI 为“方法 (Method)”和“函数 (Function)”提供了内置的 `ToolCallback` 实现 (`MethodToolCallback` 和 `FunctionToolCallback`)，简化了开发过程。





### 工具定义 (ToolDefinition)

- **目的**：提供 AI 模型关于工具的必要信息，包含名称、描述和输入参数的结构 (Schema)。
- **核心要素**：
  - **`name()` (名称)**：在同一组工具中必须是唯一的。
  - **`description()` (描述)**：向 AI 模型说明该工具的功能，帮助模型判断何时使用。
  - **`inputSchema()` (输入结构)**：使用 JSON Schema 格式定义调用工具时所需的参数。
- **自动生成**：当你从一个方法或函数创建工具时，Spring AI 会自动生成 `ToolDefinition`。你也可以使用 `ToolDefinition.Builder` 进行手动或定制化设置。





### JSON Schema 与参数定制

- **作用**：AI 模型需要通过 JSON Schema 来理解调用工具时输入参数的格式。Spring AI 内置了 `JsonSchemaGenerator` 来自动生成此结构。
- **定制选项**：
  - **`description` (描述)**：可以为每个输入参数添加描述，帮助模型更准确地传递参数。可使用 `@ToolParam`、`@JsonPropertyDescription` 等多种注解。
  - **`required` (是否必需)**：默认所有参数都是必需的。你可以通过 `@ToolParam(required = false)`、`@JsonProperty(required = false)` 或 `@Nullable` 等注解将其标记为选填。这对于避免 AI 模型在缺少信息时产生“幻觉”(hallucinations) 至关重要。





### 结果转换 (Result Conversion)

- **机制**：工具执行的结果会通过 `ToolCallResultConverter` 接口转换成字符串，然后再传回给 AI 模型。
- **默认行为**：默认使用 Jackson 将结果序列化为 JSON 格式。
- **定制化**：你可以提供自己的 `ToolCallResultConverter` 实现，来定制结果的序列化过程，这在声明式 (`@Tool` 注解) 和编程式 (Builder) 的方法中都支持。





### 工具上下文 (Tool Context)

- **功能**：允许在执行工具时传递额外的、由用户提供的上下文信息 (`ToolContext`)。这些信息**不会**被发送给 AI 模型，仅在工具内部使用。
- **应用场景**：例如，在多租户应用中，可以通过 `ToolContext` 传递 `tenantId` (租户ID)，而无需让模型知道这个概念。
- **设置方式**：可以通过 `ChatClient` 的 `.toolContext()` 方法或在 `ChatOptions` 中进行设置。





### 直接返回 (Return Direct)

- **默认行为**：工具执行的结果会先传回给 AI 模型，模型再根据结果决定下一步的回应。
- **`returnDirect` 功能**：在某些情况下，你可能希望工具的结果直接返回给最终的调用者，而不是再经过模型处理（例如：RAG 检索的结果或终止代理推理循环的工具）。
- **如何启用**：
  - **声明式**：在 `@Tool` 注解中设置 `returnDirect = true`。
  - **编程式**：通过 `ToolMetadata.builder().returnDirect(true).build()` 进行设置。
- **注意**：如果一次请求中有多个工具被调用，必须**所有**工具都设置了 `returnDirect = true`，结果才会直接返回给调用者。







## 工具执行



**核心管理器**：工具的执行生命周期由 `ToolCallingManager` 接口负责管理。它主要有两个职责：

1. 解析模型选项中的工具定义。
2. 执行 AI 模型请求的工具调用。

**默认实现**：如果你使用 Spring AI 的 Spring Boot Starter，框架会自动配置 `DefaultToolCallingManager` 作为默认实现。







### 框架控制的工具执行 (Framework-Controlled)



- **默认行为**：这是 Spring AI 的默认模式。整个过程对开发者是**透明的**。

- **流程**：

  1. 应用向 `ChatModel` 发起请求（`Prompt`），其中包含工具定义。

  2. AI 模型决定调用某个工具，并返回包含工具调用请求的响应（`ChatResponse`）。

  3. `ChatModel` 内部的 `ToolCallingManager` 自动拦截这个请求。

  4. `ToolCallingManager` 识别并执行相应的工具。

  5. 工具的执行结果返回给 `ToolCallingManager`，再由 `ChatModel` 传回给 AI 模型。

  6. AI 模型利用工具的结果作为上下文，生成最终的答复。

     ![](https://docs.spring.io/spring-ai/reference/_images/tools/framework-manager.jpg)

- **执行资格判断**：`ToolExecutionEligibilityPredicate` 接口用于判断是否应执行工具调用。默认情况下，只要 `internalToolExecutionEnabled` 为 `true`（默认值）且响应中包含工具调用，就会执行。



### 用户控制的工具执行 (User-Controlled)



- **适用场景**：当你希望自己完全控制工具的执行生命周期时使用。
- **如何启用**：在 `ToolCallingChatOptions` 中设置 `internalToolExecutionEnabled = false`。
- **开发者职责**：
  1. 调用 `ChatModel` 后，需要自己检查返回的 `ChatResponse` 是否包含工具调用 (`.hasToolCalls()`)。
  2. 如果包含，需要手动调用 `ToolCallingManager.executeToolCalls()` 来执行工具。
  3. 通常这需要一个 `while` 循环，直到 `ChatResponse` 中不再有工具调用为止。
  4. 在循环中，需要将工具执行的结果和历史对话重新包装成新的 `Prompt`，再次调用模型。
- **推荐做法**：即使在用户控制模式下，也推荐使用 `ToolCallingManager` 来处理工具调用，以便利用 Spring AI 的内置功能。文件中还提供了结合 `ChatMemory` API 进行多轮对话管理的示例。



```java
ToolCallingManager toolCallingManager = DefaultToolCallingManager.builder().build();
ChatMemory chatMemory = MessageWindowChatMemory.builder().build();
String conversationId = UUID.randomUUID().toString();

ChatOptions chatOptions = ToolCallingChatOptions.builder()
    .toolCallbacks(ToolCallbacks.from(new MathTools()))
    .internalToolExecutionEnabled(false)
    .build();
Prompt prompt = new Prompt(
        List.of(new SystemMessage("You are a helpful assistant."), new UserMessage("What is 6 * 8?")),
        chatOptions);
chatMemory.add(conversationId, prompt.getInstructions());

Prompt promptWithMemory = new Prompt(chatMemory.get(conversationId), chatOptions);
ChatResponse chatResponse = chatModel.call(promptWithMemory);
chatMemory.add(conversationId, chatResponse.getResult().getOutput());

while (chatResponse.hasToolCalls()) {
    ToolExecutionResult toolExecutionResult = toolCallingManager.executeToolCalls(promptWithMemory,
            chatResponse);
    chatMemory.add(conversationId, toolExecutionResult.conversationHistory()
        .get(toolExecutionResult.conversationHistory().size() - 1));
    promptWithMemory = new Prompt(chatMemory.get(conversationId), chatOptions);
    chatResponse = chatModel.call(promptWithMemory);
    chatMemory.add(conversationId, chatResponse.getResult().getOutput());
}

UserMessage newUserMessage = new UserMessage("What did I ask you earlier?");
chatMemory.add(conversationId, newUserMessage);

ChatResponse newResponse = chatModel.call(new Prompt(chatMemory.get(conversationId)));
```





### 异常处理



 当工具在执行过程中失败时，系统会抛出一个特定的异常，名为 `ToolExecutionException`。



**异常处理器 (`ToolExecutionExceptionProcessor`)**:

- 这是一个专门用来处理 `ToolExecutionException` 的接口。
- 它定义了两种可能的处理结果：
  1. **返回错误信息给 AI 模型**: 将异常转换为一个字符串消息，然后发送回 AI 模型。这能让模型知道工具调用失败，并可能根据此信息决定下一步行动。
  2. **抛出异常给调用者**: 不通知模型，而是直接将异常抛出，由应用程序的调用方代码来捕获和处理。



**默认行为**:

- 如果使用 Spring AI 的 Spring Boot Starter，框架会自动配置一个默认的处理器 `DefaultToolExecutionExceptionProcessor`。
- 这个默认处理器的行为是**将错误信息发送回 AI 模型**。



**如何定制行为**:

- 你可以通过自定义一个 `DefaultToolExecutionExceptionProcessor` Bean 来覆盖默认行为。
- 通过将其构造函数中的 `alwaysThrow` 参数设置为 `true`，就可以让处理器在遇到异常时总是**直接抛出异常**，而不是发送消息给模型。



**开发者的责任**:

- 如果你自己实现了 `ToolCallback` 接口来创建自定义工具，你必须确保在工具的 `call()` 方法内部，当发生错误时，主动抛出 `ToolExecutionException`。这能保证框架的异常处理机制能够正确地捕获和处理它。







# 提示词工程



> 详细内容参考：https://docs.spring.io/spring-ai/reference/api/chat/prompt-engineering-patterns.html
>
> 谷歌提示词工程文档：https://www.kaggle.com/whitepaper-prompt-engineering



## 配置



### LLM 输出配置



#### Temperature 温度

温度控制模型响应的随机性或“创造性”。



- **低值（0.0-0.3）**：更确定性、更集中的响应。适用于事实性问题、分类或一致性至关重要的任务。
- **中等值（0.4-0.7）**：在确定性和创造性之间取得平衡。适用于通用用例。
- **更高的值（0.8-1.0）**：更具创造性、多样化且可能令人惊喜的响应。更适用于创意写作、头脑风暴或生成多样化选项。



> 理解温度对于提示工程至关重要，因为不同的技术受益于不同的温度设置。



#### MaxTokens 最大令牌数



`maxTokens` 参数限制模型在其响应中可以生成的令牌（词块）数量。



- **低值（5-25）**：适用于单个词语、短句或分类标签。
- **中值（50-500）**：适用于段落或简短解释。
- **高值（1000+）**：适用于长篇内容、故事或复杂解释。



> 设置适当的输出长度非常重要，以确保您获得完整的响应而无不必要的冗长。



#### 采样控制（Top-K 和 Top-P）



这些参数让您可以对生成过程中的令牌选择过程进行细粒度控制。



- **Top-K**：将令牌选择限制在 K 个最可能的下一个令牌。较高的值（例如，40-50）引入更多多样性。
- **Top-P**：动态地从累积概率超过 P 的最小令牌集合中进行选择。0.8-0.95 这样的值很常见。



> **Top-K 和 Top-P 通俗易懂解释：**
>
> 想象一下，你让一个AI帮你写一句话：“今天天气真不错，我们去……”
>
> AI的下一步是预测接下来最可能出现的词。它的大脑里会有一张长长的候选词列表，每个词都有一个“可能性”分数。比如：
>
> - 公园 (40% a.k.a. 0.40)
> - 吃饭 (30% a.k.a. 0.30)
> - 逛街 (15% a.k.a. 0.15)
> - 游泳 (10% a.k.a. 0.10)
> - 写代码 (3% a.k.a. 0.03)
> - 睡觉 (1.5% a.k.a. 0.015)
> - 开飞船 (0.001% a.k.a. 0.00001)
> - ...还有成千上万个其他词...
>
> 现在，我们来看看 Top-K 和 Top-P 是怎么让 AI 从这个列表里选词的。
>
> ------
>
> ### Top-K：简单粗暴的“人气前几名”
>
> **K** 是一个具体的数字，比如 `K=3`。
>
> **Top-K 的工作方式是：**
>
> 1. 把所有候选词按“可能性”从高到低排序。
> 2. **只看排名前 K 个的词**。在这个例子里，就是只看前3名：“公园”、“吃饭”、“逛街”。
> 3. 然后，AI会从这3个词里随机选一个作为答案。那些排名靠后的词，比如“游泳”和“写代码”，就直接被忽略了，完全没有机会被选中。
>
> **通俗理解**： 这就像你去一家餐厅，服务员告诉你：“我们只提供今天最受欢迎的3道菜，你从这里面选一个吧。” 这种方法简单直接，能保证选出来的结果不会太离谱，但可能会错过一些同样不错但没那么热门的选项。
>
> - **K值越小**（比如K=1），AI就越“死板”，只会选最最可能的那一个词。
> - **K值越大**，AI的选择范围就越大，回答就可能更多样化。
>
> ------
>
> ### Top-P：更智能的“概率圈”
>
> **P** 是一个概率值（一个百分比），比如 `P=0.85` (也就是85%)。
>
> **Top-P 的工作方式是：**
>
> 1. 同样，把所有候选词按“可能性”从高到低排序。
> 2. 从可能性最高的词开始，把它们的概率一个个加起来，直到**总和刚刚超过你设定的 P 值**（这里是85%）。
> 3. 把这些加起来的词圈出来，形成一个选择范围。
>
> 在我们的例子里：
>
> - “公园” (40%)
> - 加上“吃饭” (30%) → 总和是 70%
> - 再加上“逛街” (15%) → 总和是 85%
>
> 好了，总和达到我们设定的85%了。所以，AI的选择范围就是“公园”、“吃饭”和“逛街”这3个词。然后AI会从这个圈子里选一个。
>
> **关键区别来了**：如果第一个词的概率本身就很高，比如“公园”的可能性是90%，那么只选这一个词就够了，选择范围就只有1个词。反之，如果每个词的概率都很低，AI就可能会圈进很多个词，直到概率总和达标。
>
> **通俗理解**： 这就像你有一个“预算”（P值），你要买东西。你从最想要的东西（概率最高的词）开始往购物车里放，直到购物车里东西的总价（总概率）达到你的预算为止。这种方法更灵活，选择范围的大小是动态变化的。
>
> - **P值越小**（比如0.5），AI的选择范围就越窄，回答更集中。
> - **P值越大**（比如0.95），AI的选择范围就越广，回答就越有创造性，因为它会考虑更多可能性不那么高的选项。
>
> ### 总结一下
>
> | 特性         | Top-K (人气前几名)                       | Top-P (概率圈)                                         |
> | ------------ | ---------------------------------------- | ------------------------------------------------------ |
> | **规则**     | 选出固定数量（K个）的最热门选项。        | 选出一批选项，它们的总概率加起来达到P。                |
> | **选择范围** | **固定的**，永远是K个词。                | **动态的**，有时多有时少。                             |
> | **优点**     | 简单、可控，能有效防止选到非常奇怪的词。 | 更智能、更灵活，能根据当前情况自动调整选择范围的大小。 |
>
> 在实际应用中，**Top-P 通常被认为是一种更先进、效果更好的方法**，因为它能更合理地平衡回答的准确性和创造性。很多时候，人们会将 Top-P 和一个较低的温度（Temperature）参数结合使用，以获得高质量且富有变化的AI回答。



## 提示词工程



### 零样本提示词



零样本提示词是指在不提供任何示例的情况下要求 AI 执行任务。这种方法测试模型从零开始理解和执行指令的能力。大型语言模型经过大量文本语料库的训练，使它们无需明确的演示即可理解“翻译”、“摘要”或“分类”等任务的含义。



零样本非常适合模型在训练期间可能见过类似示例的简单任务，以及当您希望最小化提示长度时。然而，性能可能因任务复杂性和指令表达的清晰程度而异。



```java
// Implementation of Section 2.1: General prompting / zero shot (page 15)
public void pt_zero_shot(ChatClient chatClient) {
    enum Sentiment {
        POSITIVE, NEUTRAL, NEGATIVE
    }

    Sentiment reviewSentiment = chatClient.prompt("""
            Classify movie reviews as POSITIVE, NEUTRAL or NEGATIVE.
            Review: "Her" is a disturbing study revealing the direction
            humanity is headed if AI is allowed to keep evolving,
            unchecked. I wish there were more movies like this masterpiece.
            Sentiment:
            """)
            .options(ChatOptions.builder()
                    .model("claude-3-7-sonnet-latest")
                    .temperature(0.1)
                    .maxTokens(5)
                    .build())
            .call()
            .entity(Sentiment.class);

    System.out.println("Output: " + reviewSentiment);
}
```



此示例展示了如何在不提供示例的情况下对电影评论情感进行分类。请注意使用较低的温度 (0.1) 以获得更确定性的结果，以及使用 `.entity(Sentiment.class)` 直接映射到 Java 枚举。



### 单样本与少样本提示词



少样本提示词为模型提供一个或多个示例，以帮助指导其响应，这对于需要特定输出格式的任务特别有用。通过向模型展示期望的输入-输出对示例，它可以在没有显式参数更新的情况下学习模式并将其应用于新的输入。

单样本提供一个示例，这在示例成本较高或模式相对简单时非常有用。少样本使用多个示例（通常 3-5 个）来帮助模型更好地理解更复杂任务中的模式或说明正确输出的不同变体。



```java
// Implementation of Section 2.2: One-shot & few-shot (page 16)
public void pt_one_shot_few_shots(ChatClient chatClient) {
    String pizzaOrder = chatClient.prompt("""
            Parse a customer's pizza order into valid JSON

            EXAMPLE 1:
            I want a small pizza with cheese, tomato sauce, and pepperoni.
            JSON Response:
            {
                "size": "small",
                "type": "normal",
                "ingredients": ["cheese", "tomato sauce", "pepperoni"]
            }
    
            EXAMPLE 2:
            Can I get a large pizza with tomato sauce, basil and mozzarella.
            JSON Response:
            {
                "size": "large",
                "type": "normal",
                "ingredients": ["tomato sauce", "basil", "mozzarella"]
            }

            Now, I would like a large pizza, with the first half cheese and mozzarella.
            And the other tomato sauce, ham and pineapple.
            """)
            .options(ChatOptions.builder()
                    .model("claude-3-7-sonnet-latest")
                    .temperature(0.1)
                    .maxTokens(250)
                    .build())
            .call()
            .content();
}
```



少样本提示词对于需要特定格式、处理边缘情况或在没有示例时任务定义可能模糊的任务尤其有效。示例的质量和多样性显著影响性能。





### 系统、上下文和角色提示词



#### 系统提示词



系统提示词为语言模型设置了总体上下文和目的，定义了模型应该做什么的“大局”。它为模型的响应建立了行为框架、约束和高级目标，与具体的用户查询是分开的。



系统提示词在整个对话过程中充当持久的“任务声明”，允许您设置全局参数，如输出格式、语调、伦理界限或角色定义。与专注于特定任务的用户提示词不同，系统提示词框架了所有用户提示词应如何被解释。



```java
// Implementation of Section 2.3.1: System prompting with JSON output
record MovieReviews(Movie[] movie_reviews) {
    enum Sentiment {
        POSITIVE, NEUTRAL, NEGATIVE
    }

    record Movie(Sentiment sentiment, String name) {
    }
}

MovieReviews movieReviews = chatClient
        .prompt()
        .system("""
                Classify movie reviews as positive, neutral or negative. Return
                valid JSON.
                """)
        .user("""
                Review: "Her" is a disturbing study revealing the direction
                humanity is headed if AI is allowed to keep evolving,
                unchecked. It's so disturbing I couldn't watch it.

                JSON Response:
                """)
        .call()
        .entity(MovieReviews.class);
```



系统提示对于多轮对话尤其有价值，可以确保跨多个查询的一致行为，并用于建立格式约束，例如应适用于所有响应的 JSON 输出。



#### 角色提示词



角色提示指示模型采用特定的角色或人格，这会影响其生成内容的方式。通过为模型分配特定的身份、专业知识或视角，您可以影响其响应的风格、语调、深度和框架。



角色提示词利用了模型模拟不同专业领域和沟通风格的能力。常见角色包括专家（例如，“你是一位经验丰富的数据科学家”）、专业人士（例如，“充当旅行指南”）或风格化角色（例如，“像莎士比亚那样解释”）。



```java
// Implementation of Section 2.3.2: Role prompting
public void pt_role_prompting_1(ChatClient chatClient) {
    String travelSuggestions = chatClient
            .prompt()
            .system("""
                    I want you to act as a travel guide. I will write to you
                    about my location and you will suggest 3 places to visit near
                    me. In some cases, I will also give you the type of places I
                    will visit.
                    """)
            .user("""
                    My suggestion: "I am in Amsterdam and I want to visit only museums."
                    Travel Suggestions:
                    """)
            .call()
            .content();
}
```



角色提示词可以通过风格指令增强



```java
// Implementation of Section 2.3.2: Role prompting with style instructions
public void pt_role_prompting_2(ChatClient chatClient) {
    String humorousTravelSuggestions = chatClient
            .prompt()
            .system("""
                    I want you to act as a travel guide. I will write to you about
                    my location and you will suggest 3 places to visit near me in
                    a humorous style.
                    """)
            .user("""
                    My suggestion: "I am in Amsterdam and I want to visit only museums."
                    Travel Suggestions:
                    """)
            .call()
            .content();
}
```



这项技术对于处理专业领域知识、在不同响应中保持一致的语调以及创建更具吸引力、个性化的用户交互特别有效。



#### 上下文提示词



上下文提示词通过传递上下文参数，为模型提供额外的背景信息。这项技术丰富了模型对特定情况的理解，从而能够提供更相关和更具针对性的响应，而不会使主要指令变得混乱。

通过提供上下文信息，您可以帮助模型理解与当前查询相关的特定领域、受众、约束或背景事实。这会带来更准确、更相关且框架适当的响应。



```java
// Implementation of Section 2.3.3: Contextual prompting
public void pt_contextual_prompting(ChatClient chatClient) {
    String articleSuggestions = chatClient
            .prompt()
            .user(u -> u.text("""
                    Suggest 3 topics to write an article about with a few lines of
                    description of what this article should contain.

                    Context: {context}
                    """)
                    .param("context", "You are writing for a blog about retro 80's arcade video games."))
            .call()
            .content();
}
```



### 回溯提示词



回溯提示词通过首先获取背景知识，将复杂的请求分解为更简单的步骤。这项技术鼓励模型首先从眼前的问题中“回溯”，考虑与问题相关的更广泛的上下文、基本原理或通用知识，然后再解决具体的查询。



通过将复杂问题分解为更易于管理的组成部分并首先建立基础知识，模型可以对难题提供更准确的响应。



```java
// Implementation of Section 2.4: Step-back prompting
public void pt_step_back_prompting(ChatClient.Builder chatClientBuilder) {
    // Set common options for the chat client
    var chatClient = chatClientBuilder
            .defaultOptions(ChatOptions.builder()
                    .model("claude-3-7-sonnet-latest")
                    .temperature(1.0)
                    .topK(40)
                    .topP(0.8)
                    .maxTokens(1024)
                    .build())
            .build();

    // First get high-level concepts
    String stepBack = chatClient
            .prompt("""
                    Based on popular first-person shooter action games, what are
                    5 fictional key settings that contribute to a challenging and
                    engaging level storyline in a first-person shooter video game?
                    """)
            .call()
            .content();

    // Then use those concepts in the main task
    String story = chatClient
            .prompt()
            .user(u -> u.text("""
                    Write a one paragraph storyline for a new level of a first-
                    person shooter video game that is challenging and engaging.

                    Context: {step-back}
                    """)
                    .param("step-back", stepBack))
            .call()
            .content();
}
```



回溯提示词对于复杂的推理任务、需要专业领域知识的问题以及当您想要更全面、更深思熟虑的响应而非即时答案时特别有效。









# 模型上下文协议 (MCP)



# 检索增强生成 (RAG)



# 模型评估 (Model Evaluation)



# 矢量数据库



# 可观察性



# 参考资料

- [Spring AI 中文版](https://www.spring-doc.cn/spring-ai/1.0.0/index.html)