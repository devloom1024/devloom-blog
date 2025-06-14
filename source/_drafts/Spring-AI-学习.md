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

详细内容参考：

- [聊天客户端 API](https://www.spring-doc.cn/spring-ai/1.0.0/api_chatclient.html)

- [通知器 (advisors)](https://www.spring-doc.cn/spring-ai/1.0.0/api_advisors.html)



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



详细内容参考：[提示](https://www.spring-doc.cn/spring-ai/1.0.0/api_prompt.html)





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



# 多模态 API



# 各种类型的模型



# 聊天记忆



# 工具调用



# 提示词工程



# 模型上下文协议 (MCP)



# 检索增强生成 (RAG)



# 模型评估 (Model Evaluation)



# 矢量数据库



# 可观察性



# 参考资料

- [Spring AI 中文版](https://www.spring-doc.cn/spring-ai/1.0.0/index.html)