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

