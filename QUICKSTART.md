## 快速上手指南（SpringAI MCP + RAG）

本项目包含 Spring Boot 3.5 + Spring AI 1.0.0 的双模块后端（`mcp-client`、`mcp-server`）以及一个静态前端（`spring-ai-frontend`），演示了 RAG 检索增强、MCP 工具调用、SSE 流式对话与互联网搜索的整合方案。

### 架构速览
- `SpringAI-MCP-RAG-Dev/mcp-client`：对话与智能体入口，负责聊天流式输出、RAG 检索、互联网搜索聚合与向量库写入。
- `SpringAI-MCP-RAG-Dev/mcp-server`：MCP 工具服务，暴露日期/时间、邮件发送、商品库 CRUD 等工具。
- `spring-ai-frontend`：纯静态前端页面（示例页 `spring-ai-best.html`），通过 REST 与 SSE 调用后端。

### 亮点（可在面试中重点提及）
- 流式聊天 + 工具调用：`ChatClient` 默认挂载 MCP 工具与会话记忆。
- RAG 能力：上传文档 → 切分 → 写入 Redis 向量库 → 基于向量召回补充上下文回答。
- 互联网搜索：基于 SearXNG 的多源检索，重排序后作为上下文融合回答。
- MCP 工具服务：WebFlux 版 SSE MCP Server，内置时间查询、Markdown/HTML 邮件发送、商品库增删改查（MyBatis-Plus + MySQL）。

代码摘录（便于阐述设计点）：
```33:68:SpringAI-MCP-RAG-Dev/mcp-client/src/main/java/com/itzixi/service/impl/ChatServiceImpl.java
public ChatServiceImpl(ChatClient.Builder chatClientBuilder, ToolCallbackProvider tools, ChatMemory chatMemory) {
    this.chatClient = chatClientBuilder
            .defaultToolCallbacks(tools)                    // 默认启用 MCP 工具
            .defaultAdvisors(MessageChatMemoryAdvisor.builder(chatMemory).build()) // 带记忆
            .build();
}
```
```123:173:SpringAI-MCP-RAG-Dev/mcp-client/src/main/java/com/itzixi/service/impl/ChatServiceImpl.java
private static final String ragPROMPT = """
  基于上下文的知识库内容回答问题：
  【上下文】
  {context}
  
  【问题】
  {question}
  
  【输出】
  如果没有查到，请回复：不知道。
  如果查到，请回复具体的内容。不相关的近似内容不必提到。
  """;

public void doChatRagSearch(ChatEntity chatEntity, List<Document> ragContext) {
    // 根据向量检索到的上下文拼装 Prompt 并流式返回
}
```
```41:70:SpringAI-MCP-RAG-Dev/mcp-client/src/main/java/com/itzixi/service/impl/DocumentServiceImpl.java
CustomTextSplitter tokenTextSplitter = new CustomTextSplitter();
List<Document> list = tokenTextSplitter.apply(documentList);
// 向量存储
redisVectorStore.add(list);
```
```31:92:SpringAI-MCP-RAG-Dev/mcp-client/src/main/java/com/itzixi/service/impl/SesrXngServiceImpl.java
public List<SearchResult> search(String query) {
    HttpUrl url = HttpUrl.get(SEARXNG_URL)
            .newBuilder()
            .addQueryParameter("q", query)
            .addQueryParameter("format", "json")
            .build();
    // 调用 SearXNG 并按得分排序、截断
}
```
```32:79:SpringAI-MCP-RAG-Dev/mcp-server/src/main/java/com/itzixi/mcp/tool/ProductTool.java
@Tool(description = "创建/新增商品信息记录")
public String createNewProduct(CreateProductRequest createProductRequest) {
    product.setProductId(RandomStringUtils.randomNumeric(12));
    productMapper.insert(product); // MyBatis-Plus + MySQL
    return "商品信息创建成功";
}
```
```31:86:SpringAI-MCP-RAG-Dev/mcp-server/src/main/java/com/itzixi/mcp/tool/EmailTool.java
@Tool(description = "给指定邮箱发送邮件信息...")
public void sendMailMessage(EmailRequest emailRequest) {
    if (contentType == 1) {
        helper.setText(convertToHtml(emailRequest.getMessage()), true); // Markdown → HTML
    } else if (contentType == 2) {
        helper.setText(emailRequest.getMessage(), true);
    } else {
        helper.setText(emailRequest.getMessage());
    }
    mailSender.send(mimeMessage);
}
```

### 环境要求
- JDK 21、Maven 3.9+。
- Redis（用于向量库，默认 `host=127.0.0.1 port=9379 password=root`，见 `mcp-client` 的 `application-dev.yml`）。
- MySQL 8+（`springai_items_mcp` 库，见 `mcp-server` 的 `application-dev.yml`；需预先导入商品表结构与初始数据）。
- SearXNG（可选，用于互联网搜索，默认 `http://localhost:6080/search`）。
- OpenAI 或兼容模型服务：环境变量 `OPENAI_API_KEY`、`OPENAI_BASE_URL`、`OPENAI_MODEL`。

### 运行步骤（本地开发）
1) 启动依赖服务  
   - Redis：确保端口与密码与 `mcp-client/src/main/resources/application-dev.yml` 一致。  
   - MySQL：创建库 `springai_items_mcp`，导入商品表结构与数据。  
   - SearXNG：按 `application-dev.yml` 的地址启动（如不用互联网搜索，可先忽略）。

2) 启动 MCP 工具服务（server，端口默认 9060）  
```bash
cd SpringAI-MCP-RAG-Dev/mcp-server
mvn spring-boot:run -Dspring-boot.run.profiles=dev
```

3) 启动聊天/RAG 客户端（client，端口默认 9090）  
```bash
cd SpringAI-MCP-RAG-Dev/mcp-client
mvn spring-boot:run -Dspring-boot.run.profiles=dev
```

4) 前端示例  
   - 使用 VSCode Live Server 或任意静态服务器打开 `spring-ai-frontend/spring-ai-best.html`。  
   - 前端通过 REST/SSE 调用 `mcp-client`，配置的回调域在 `application-dev.yml` 的 `website.domain`。

### 常用接口速查
- 对话流式输出：`POST /chat/doChat`，body `{"currentUserName":"u1","message":"你好","botMsgId":"msg-1"}`。
- RAG 文档上传：`POST /rag/uploadRagDoc`，表单字段 `file`。  
- RAG 问答：`POST /rag/search`，body `{"currentUserName":"u1","message":"<你的问题>","botMsgId":"msg-2"}`。  
- 互联网搜索增强：`POST /internet/search`（查看 `InternetController`）。  
- MCP 工具（在对话中由模型自动调用）：时间查询、Markdown/HTML 邮件、商品 CRUD。

### 面试话术小抄
- “客户端用 Spring AI ChatClient 默认挂载 MCP 工具 + Memory，以最少代码实现工具编排和上下文记忆。”
- “RAG 采用 Redis Vector Store，上传即切分入库，查询时把召回片段拼进 Prompt，缺失时回落到‘不知道’。”
- “互联网搜索通过 SearXNG 拉取多源结果，二次排序后作为上下文融合回答。”
- “MCP Server 以 WebFlux SSE 暴露工具，包含 MySQL/MyBatis-Plus 商品库与 Markdown 邮件发送，工具注册用 `MethodToolCallbackProvider` 一键收敛。”

### 快速排错
- OpenAI 连接失败：检查 `OPENAI_BASE_URL`、`OPENAI_API_KEY` 是否生效。  
- 向量检索无结果：确认 Redis 连接与索引名 `lee-vectorstore`，以及上传文档是否成功写入。  
- SearXNG 报错：核对 `internet.websearch.searxng.url`，浏览器直接访问 `/search?q=test` 验证服务。  
- MySQL 工具报错：确认库/表存在且账号密码与 `application-dev.yml` 一致。

---
希望这份速记能帮助你在面试中快速呈现项目亮点。祝面试顺利！

