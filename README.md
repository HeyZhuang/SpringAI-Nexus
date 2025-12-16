# 🚀 Spring AI MCP Nexus (RAG + Agent + Search)

> **基于 Spring AI 1.0 & Spring Boot 3.5 构建的下一代 AI 智能体架构演示。**

本项目集成 **RAG 知识库**、**MCP 工具协议**、**SearXNG 联网搜索**与 **SSE 流式对话**，展示了企业级 AI 应用的全链路实现。

---

## 🏗 系统架构 (System Architecture)

本项目采用 **双模块后端 + 静态前端** 的分层架构，清晰地分离了“智能决策层”与“工具服务层”。

```mermaid
graph TD
    User["用户 (Frontend)"] -- SSE / HTTP --> Client["MCP Client (智能决策层)"]
    
    subgraph "Core Logic (mcp-client)"
        Client -- 1. 聊天/推理 --> LLM["OpenAI / Compatible Model"]
        Client -- 2. 向量检索 --> Redis["Redis Vector Store"]
        Client -- 3. 联网搜索 --> SearXNG["SearXNG Engine"]
    end
    
    subgraph "Tool Services (mcp-server)"
        Client -- 4. MCP工具调用 --> Server["MCP Server (工具服务层)"]
        Server -- CRUD --> MySQL["MySQL Product DB"]
        Server -- SMTP --> Email["Email Service"]
    end


✨ 核心亮点 (Key Features)
1. 🧠 强大的 RAG 知识检索
不仅仅是简单的问答，实现了完整的 ETL 流程：

文档处理：支持文件上传与自动切分 (TokenTextSplitter)。

向量存储：使用 Redis 作为向量数据库，高性能存储 Embedding。

混合增强：查询时自动召回相关片段，并组装特定的 Prompt 注入上下文。

2. 🔌 标准化 MCP 工具链
利用 Spring AI 的 ToolCallbackProvider 实现了工具的自动挂载：

商品管理：集成 MyBatis-Plus，实现商品库的增删改查，模型可自主操作数据库。

邮件助手：支持发送 Markdown/HTML 格式邮件，模型可根据语义自动渲染邮件内容。

3. 🌐 实时联网搜索 (Internet Search)
集成 SearXNG 聚合搜索引擎：

当知识库无法回答时，Agent 可主动发起联网搜索。

对多源搜索结果进行重排序 (Re-rank) 和截断，作为上下文补充给模型。

4. ⚡️ 全链路流式响应 (Streaming)
拒绝等待。从后端 ChatClient 到前端 JS，全链路支持 SSE (Server-Sent Events)，实现“打字机”式的流畅体验。

💻 核心代码解析 (Code Insights)
1. 智能体编排与记忆 (ChatService)
使用 ChatClient 的 Fluent API 极简地实现了工具挂载与记忆管理。

Java

// mcp-client/src/main/java/com/itzixi/service/impl/ChatServiceImpl.java
public ChatServiceImpl(ChatClient.Builder chatClientBuilder, ToolCallbackProvider tools, ChatMemory chatMemory) {
    this.chatClient = chatClientBuilder
            .defaultToolCallbacks(tools)                 // 自动挂载所有 MCP 工具
            .defaultAdvisors(MessageChatMemoryAdvisor.builder(chatMemory).build()) // 注入会话记忆
            .build();
}
2. RAG 检索逻辑
实现了“检索-增强-生成”的标准范式，对“不知道”的情况做了兜底处理。

Java

// RAG Prompt 模板设计
private static final String ragPROMPT = """
  基于上下文的知识库内容回答问题：
  【上下文】
  {context}
  【问题】
  {question}
  【输出】
  如果没有查到，请回复：不知道。
  如果查到，请回复具体的内容。
  """;
3. 工具定义：商品管理
通过 @Tool 注解将 Java 方法暴露给 AI，结合 MyBatis-Plus 操作真实数据。

Java

// mcp-server/src/main/java/com/itzixi/mcp/tool/ProductTool.java
@Tool(description = "创建/新增商品信息记录")
public String createNewProduct(CreateProductRequest createProductRequest) {
    product.setProductId(RandomStringUtils.randomNumeric(12));
    productMapper.insert(product); // 实际写入 MySQL
    return "商品信息创建成功";
}
🛠️ 快速开始 (Quick Start)
环境准备
JDK: 21+

Maven: 3.9+

Middleware:

Redis (向量库, 默认端口 9379)

MySQL 8.0+ (库名: springai_items_mcp)

SearXNG (可选, 联网搜索服务)

1. 启动基础设施
确保 application-dev.yml 中的配置与本地环境一致：

Client 端: 检查 spring.ai.vectorstore.redis.uri 与 spring.ai.openai.api-key。

Server 端: 导入 springai_items_mcp 初始 SQL，检查数据库连接。

2. 启动 MCP 服务端 (Tools Provider)
Bash

cd SpringAI-MCP-RAG-Dev/mcp-server
mvn spring-boot:run -Dspring-boot.run.profiles=dev
# 端口默认为 9060
3. 启动 Client 客户端 (Brain)
Bash

cd SpringAI-MCP-RAG-Dev/mcp-client
mvn spring-boot:run -Dspring-boot.run.profiles=dev
# 端口默认为 9090
4. 前端体验
直接使用浏览器打开 spring-ai-frontend/spring-ai-best.html (建议使用 VSCode Live Server 运行以避免跨域问题)，即可开始对话。

🎯 面试官关注点 (Interview Cheat Sheet)
如果你在面试中展示此项目，建议重点阐述以下设计思想：

架构解耦
话术：“我将工具执行（MCP Server）与逻辑推理（MCP Client）分离，这符合微服务架构思想。未来如果需要接入 Python 编写的工具或扩展更多业务，只需在 Server 端新增，Client 端无需重构。”

RAG 落地细节
话术：“在处理 RAG 时，我使用了 Redis 作为向量库，因为它读写极快。我实现了完整的‘切片-向量化-存储-召回’闭环，并在 Prompt 中做了防幻觉处理（查不到即返回‘不知道’）。”

联网搜索增强
话术：“针对模型知识滞后的问题，我整合了 SearXNG。通过 API 获取实时信息后，我并不是直接扔给用户，而是将其作为 Context 喂给模型，让模型用自然语言总结后再输出。”

Java 生态的 AI 优势
话术：“相比于 Python (LangChain)，Spring AI 让 Java 开发者能利用现有的 Spring 生态（如 Bean 管理、AOP、事务控制）快速构建企业级 AI 应用，这个项目就是最好的证明。”

🤝 贡献 (Contribution)
欢迎提交 Issue 和 PR！

Author: HeyZhuang

License: MIT
