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



好的，这是对您提供的 Spring AI "Tool Calling"（工具调用）参考文档的简体中文总结。

### Spring AI 工具调用 (Tool Calling) 核心功能概述

本文档详细介绍了 Spring AI 框架中强大的“工具调用”功能。该功能允许 AI 模型与外部世界进行交互，通过调用应用程序定义的 API 或“工具”来增强其能力。

---

#### 1. 什么是工具调用？

工具调用（也称为函数调用）是一种常见的人工智能应用模式，它允许模型利用外部工具来完成自身无法独立完成的任务。主要分为两类：

*   **信息检索 (Information Retrieval)**：模型调用工具从外部数据源（如数据库、Web 服务、文件系统）获取它本身不知道的信息。例如，获取当前天气、查询最新的新闻或检索特定客户的订单记录。这是实现 **检索增强生成 (RAG)** 的关键。
*   **执行操作 (Taking Action)**：模型调用工具在软件系统中执行具体动作。例如，发送电子邮件、预订航班、在数据库中创建新记录或触发一个工作流。

**核心安全原则**：模型本身**永远不会**直接访问或执行任何 API。它只会生成一个“工具调用请求”，其中包含要调用的工具名称和所需的参数。实际的工具执行完全由您的 Spring 应用程序负责，应用程序执行后将结果返回给模型。这种分离是至关重要的安全保障。

---

#### 2. 工具调用的工作流程

整个过程遵循一个清晰的六步流程：

1.  **定义与请求**：应用程序在向 AI 模型发送请求时，会附上可用工具的定义（名称、描述、输入参数的 JSON Schema）。
2.  **模型决策**：AI 模型根据用户的提问和工具的描述，判断是否需要以及如何调用某个工具，并返回一个包含工具名称和参数的响应。
3.  **应用执行**：Spring AI 框架或您的应用程序代码接收到这个工具调用请求，并执行相应的 Java 方法或函数。
4.  **获取结果**：应用程序获得工具执行的结果（例如，一个 Java 对象）。
5.  **结果返回**：应用程序将执行结果序列化后，作为新的上下文信息发送回 AI 模型。
6.  **最终响应**：AI 模型利用工具返回的结果，生成一个更准确、更丰富的最终答案给用户。

---

#### 3. 如何定义工具

Spring AI 提供了多种灵活的方式来定义工具，主要围绕 `ToolCallback` 接口。

##### **A. 将方法 (Methods) 定义为工具**

这是最常见的方式，适用于将任意 Java 方法暴露给 AI 模型。

*   **声明式 (`@Tool` 注解)**：
    *   在任何 Java 方法上添加 `@Tool` 注解即可将其定义为一个工具。
    *   `description` 属性至关重要，它告诉模型这个工具的用途，直接影响模型的调用决策。
    *   使用 `@ToolParam` 注解为方法参数提供描述和是否必需的元数据。
*   **编程式 (`MethodToolCallback`)**：
    *   通过 `MethodToolCallback.Builder` 以编程方式手动构建工具定义，提供更精细的控制。

##### **B. 将函数 (Functions) 定义为工具**

适用于将 `java.util.function` 包中的函数式接口（如 `Function`, `Supplier`, `Consumer`）定义为工具，特别适合与 Spring Bean 结合使用。

*   **编程式 (`FunctionToolCallback`)**：
    *   使用 `FunctionToolCallback.Builder` 包装一个函数式接口实例。
*   **动态化 (`@Bean` 注解)**：
    *   将一个 `Function` 类型的 Bean 注册到 Spring 容器中。
    *   Bean 的名称将作为工具的名称，可以使用 `@Description` 注解提供工具描述。
    *   Spring AI 会在运行时动态解析这些 Bean 作为可用工具。

---

#### 4. 核心概念详解

*   **`ToolDefinition`**：定义一个工具的元数据，包括**名称 (name)**、**描述 (description)** 和**输入模式 (inputSchema)**。这是模型理解工具的关键。
*   **JSON Schema**：Spring AI 会自动为工具的输入参数生成 JSON Schema。您可以使用 `@ToolParam` (Spring AI)、`@Schema` (Swagger) 或 `@JsonProperty` (Jackson) 等注解来定制化 Schema，例如提供参数描述或设置其为可选。
*   **`ToolContext`**：允许您在调用工具时传递额外的上下文数据（如 `tenantId`、用户认证信息），这些数据**不会**被发送给 AI 模型，仅在工具执行时可用，增强了安全性和多租户支持。
*   **`Return Direct` (直接返回)**：通过设置 `returnDirect = true`，可以让工具的执行结果直接返回给最终用户，而不是再次传递给 AI 模型进行处理。这对于结果本身就是答案的 RAG 场景或需要终止代理循环的场景非常有用。
*   **`Result Conversion` (结果转换)**：通过实现 `ToolCallResultConverter` 接口，可以自定义工具的返回值（如一个 POJO）如何被序列化成字符串再发送给模型。默认使用 Jackson 进行 JSON 序列化。

---

#### 5. 工具执行与控制

*   **`ToolCallingManager`**：管理工具执行生命周期的核心接口。
*   **框架控制的执行 (默认)**：默认情况下，Spring AI 会自动处理整个工具调用流程。您只需定义工具并将其提供给 `ChatClient` 即可。
*   **用户控制的执行**：通过设置 `internalToolExecutionEnabled(false)`，您可以接管工具调用的控制权。在这种模式下，模型会返回一个工具调用请求，您需要手动调用 `ToolCallingManager` 来执行工具，并管理对话历史，这为构建复杂的多步代理（Agent）提供了极大的灵活性。

---

#### 6. 其他重要主题

*   **异常处理**：工具执行失败会抛出 `ToolExecutionException`。您可以提供一个 `ToolExecutionExceptionProcessor` Bean 来自定义异常处理逻辑，例如是向模型返回错误信息还是向上抛出异常。
*   **工具解析**：当您只提供工具名称时，`ToolCallbackResolver` 负责在运行时查找并解析对应的 `ToolCallback` 实例。
*   **可观测性与日志**：Spring AI 为工具调用提供了内置的可观测性支持（tracing），并可在 `DEBUG` 级别下查看详细的日志信息。

总之，Spring AI 的工具调用功能是一个设计精良、功能全面且高度可扩展的系统，它使开发者能够轻松地将强大的 AI 模型与现有的业务逻辑和数据源无缝集成。







# 提示词工程



# 模型上下文协议 (MCP)



# 检索增强生成 (RAG)



# 模型评估 (Model Evaluation)



# 矢量数据库



# 可观察性



# 参考资料

- [Spring AI 中文版](https://www.spring-doc.cn/spring-ai/1.0.0/index.html)