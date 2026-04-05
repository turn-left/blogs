# 2026/4/4 Spring AI 实践

记录 Spring AI 框架的学习与实践，连接多种大语言模型。

## 概述

Spring AI 是 Spring 生态中用于构建 AI 应用的框架，提供了对各种大语言模型（LLM）的统一抽象。其核心设计理念是**解耦业务逻辑与 AI 模型提供商**，开发者只需编写一次代码，即可无缝切换不同的模型。

### 核心特性

- **统一 API**：ChatClient 提供跨模型的统一调用方式
- **多模型支持**：OpenAI、MiniMax、Azure OpenAI、Anthropic、阿里云等
- **Prompt 工程**：强大的模板系统和参数配置
- **结构化输出**：支持将 LLM 输出绑定到 POJO
- **RAG 支持**：集成向量存储和检索增强生成
- **Function Calling**：支持工具调用和函数绑定

---

## 环境准备

```bash
# 技术栈要求
Spring Boot 3.2+
Java 21+
Maven / Gradle
```

### Maven 依赖管理

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.ai</groupId>
            <artifactId>spring-ai-bom</artifactId>
            <version>1.1.4</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>

<dependencies>
    <!-- MiniMax 模型支持 -->
    <dependency>
        <groupId>org.springframework.ai</groupId>
        <artifactId>spring-ai-starter-model-minimax</artifactId>
    </dependency>
    
    <!-- OpenAI 模型支持（示例） -->
    <dependency>
        <groupId>org.springframework.ai</groupId>
        <artifactId>spring-ai-starter-model-openai</artifactId>
    </dependency>
    
    <!-- 向量数据库支持 -->
    <dependency>
        <groupId>org.springframework.ai</groupId>
        <artifactId>spring-ai-starter-store-pgvector</artifactId>
    </dependency>
</dependencies>
```

---

## 统一调用：ChatClient API

Spring AI 1.0 引入了 `ChatClient`，这是目前推荐的调用方式。它提供了流式调用、函数调用、结构化输出等能力。

### 基本调用

```java
@RestController
class AIController {

    private final ChatClient chatClient;

    public AIController(ChatClient.Builder builder) {
        this.chatClient = builder.build();
    }

    @GetMapping("/ai/chat")
    String chat(@RequestParam String message) {
        return chatClient.prompt()
            .user(message)
            .call()
            .content();
    }
}
```

### 流式响应

```java
@GetMapping(value = "/ai/stream", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
Flux<String> streamChat(@RequestParam String message) {
    return chatClient.prompt()
        .user(message)
        .stream()
        .content();
}
```

### 结构化输出

将 LLM 响应直接映射到 Java 对象：

```java
public record MovieReview(String title, int rating, String summary) {}

@GetMapping("/ai/review")
MovieReview generateReview(@RequestParam String movieName) {
    return chatClient.prompt()
        .user("请为电影《" + movieName + "》生成一个简短的评论")
        .call()
        .entity(MovieReview.class);
}
```

生成的 JSON 会自动映射到 `MovieReview` 对象。

---

## 多模型配置与切换

### MiniMax 配置

```yaml
spring:
  ai:
    minimax:
      api-key: ${MINIMAX_API_KEY}
      base-url: https://api.minimax.chat/v
    chat:
      options:
        model: abab6.5g-chat
        temperature: 0.7
        max-tokens: 2048
```

### OpenAI 配置

```yaml
spring:
  ai:
    openai:
      api-key: ${OPENAI_API_KEY}
      chat:
        options:
          model: gpt-4o
          temperature: 0.7
```

### 动态模型选择

```java
@Service
class MultiModelService {
    
    private final Map<String, ChatClient> modelClients;
    
    public MultiModelService(ClientBuilder builder) {
        this.modelClients = Map.of(
            "minimax", builder.build(MiniMaxChatModelFactory.class),
            "openai", builder.build(OpenAiChatModelFactory.class)
        );
    }
    
    public String chat(String model, String message) {
        return modelClients.get(model).prompt()
            .user(message)
            .call()
            .content();
    }
}
```

---

## 提示词模板

### PromptTemplate 基础

```java
@Bean
PromptTemplate jokeTemplate() {
    return new PromptTemplate(
        "请用{style}风格讲一个关于{topic}的笑话"
    );
}

@Test
void testPromptTemplate() {
    String result = jokeTemplate.render(
        Map.of("style", "幽默", "topic", "程序员")
    );
    assertTrue(result.contains("程序员"));
}
```

### System Prompt 配置

```java
String response = chatClient.prompt()
    .system("你是一个专业的Java后端工程师，用简洁的技术语言回答问题")
    .user("解释一下什么是依赖注入")
    .call()
    .content();
```

### Few-Shot Learning

```java
String response = chatClient.prompt()
    .system("""
        你是情感分析助手。
        示例：
        输入：今天天气真好！ 情绪：开心
        输入：丢了工作很沮丧 情绪：难过
        """)
    .user("终于放假了，太开心了")
    .call()
    .content();
```

---

## Function Calling

Function Calling 允许 LLM 调用外部工具或函数，实现更复杂的功能。

### 定义函数

```java
public class WeatherService {
    
    public record WeatherInfo(String city, String condition, int temperature) {}
    
    public WeatherInfo getWeather(String city) {
        // 调用天气API
        return new WeatherInfo(city, "晴天", 25);
    }
}
```

### 注册 Function

```java
@Bean
ChatClient chatClient(ChatModel chatModel, FunctionCallbackContext functionCallbackContext) {
    WeatherService weatherService = new WeatherService();
    
    return ChatClient.builder(chatModel)
        .defaultFunctions("getWeather", weatherService::getWeather)
        .build();
}
```

### 调用

```java
@GetMapping("/ai/weather")
String weather(@RequestParam String city) {
    return chatClient.prompt()
        .user("北京今天天气怎么样？")
        .call()
        .content();
}
```

---

## RAG：检索增强生成

### 架构概述

```
用户问题 → 向量检索 → 上下文拼接 → LLM 生成 → 最终回答
              ↓
        文档切分 → 向量化 → 存入向量数据库
```

### 文档处理

```java
@Service
class DocumentService {
    
    private final AiTemplate aiTemplate;
    private final VectorStore vectorStore;
    
    public void loadDocuments(Path filePath) {
        TextReader textReader = new TextReader(filePath);
        Document document = textReader.get();
        
        // 文档分割
        TokenTextSplitter splitter = new TokenTextSplitter();
        List<Document> chunks = splitter.apply(List.of(document));
        
        // 存入向量数据库
        vectorStore.add(chunks);
    }
}
```

### 检索增强查询

```java
@GetMapping("/ai/rag")
String ragQuery(@RequestParam String question) {
    // 1. 检索相关文档
    List<Document> relevantDocs = vectorStore.similaritySearch(question);
    
    // 2. 构建带上下文的 Prompt
    String context = relevantDocs.stream()
        .map(Document::getContent)
        .collect(Collectors.joining("\n"));
    
    // 3. 让 LLM 基于上下文回答
    return chatClient.prompt()
        .user("基于以下内容回答：" + context + "\n\n问题：" + question)
        .call()
        .content();
}
```

---

## 向量数据库集成

### PostgreSQL + pgvector

```yaml
spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/vector_db
    username: postgres
    password: secret
  
  ai:
    vectorstore:
      pgvector:
        dimensions: 1536
        distance-type: COSINE_DISTANCE
```

### Chromadb（轻量级）

```yaml
spring:
  ai:
    vectorstore:
      chromadb:
        path: ./data/chroma
```

---

## 常用配置参数

| 参数 | 说明 | MiniMax 默认值 | OpenAI 默认值 |
|------|------|----------------|---------------|
| `model` | 模型名称 | abab6.5g-chat | gpt-4o |
| `temperature` | 采样温度（0-1） | 0.7 | 0.7 |
| `maxTokens` | 最大生成 token 数 | 2048 | 4096 |
| `topP` | 核采样概率 | 1.0 | 1.0 |
| `frequencyPenalty` | 频率惩罚 | 0.0 | 0.0 |
| `presencePenalty` | 存在惩罚 | 0.0 | 0.0 |

---

## 可用模型列表

### MiniMax 模型

| 模型 | 说明 | 适用场景 |
|------|------|----------|
| `abab5.5-chat` | 基础模型 | 简单对话 |
| `abab5.5s-chat` | 高速版 | 实时交互 |
| `abab6.5-chat` | 增强版 | 复杂任务 |
| `abab6.5g-chat` | 联网版（推荐） | RAG、搜索增强 |
| `abab6.5t-chat` | 图像理解 | 多模态 |
| `abab6.5s-chat` | 超高速 | 客服场景 |

### OpenAI 模型

| 模型 | 说明 |
|------|------|
| `gpt-4o` | 最新全能模型 |
| `gpt-4-turbo` | 高速版 GPT-4 |
| `gpt-3.5-turbo` | 轻量级 |

---

## 实战案例

### 案例一：智能客服

```java
@Service
class CustomerService {
    
    private final ChatClient chatClient;
    private final VectorStore knowledgeBase;
    
    public String answer(String question) {
        // 1. 从知识库检索
        List<Document> docs = knowledgeBase.similaritySearch(question, 3);
        
        // 2. 构建上下文
        String context = docs.stream()
            .map(d -> "- " + d.getContent())
            .collect(Collectors.joining("\n"));
        
        // 3. 生成回答
        return chatClient.prompt()
            .system("你是一个专业的客服，基于给定的知识库回答客户问题")
            .user("知识库：\n" + context + "\n\n客户问题：" + question)
            .call()
            .content();
    }
}
```

### 案例二：代码审查助手

```java
@Service
class CodeReviewService {
    
    public String review(String code) {
        return chatClient.prompt()
            .system("""
                你是一个资深代码审查专家。
                审查维度：
                1. 代码安全性
                2. 性能优化
                3. 代码可读性
                4. 最佳实践
                
                以 Markdown 格式输出审查结果。
                """)
            .user(code)
            .call()
            .content();
    }
}
```

---

## 错误处理与最佳实践

### 异常处理

```java
@RestControllerAdvice
class AiExceptionHandler {
    
    @ExceptionHandler(AiException.class)
    public ResponseEntity<String> handleAiException(AiException e) {
        if (e.getCause() instanceof RateLimitException) {
            return ResponseEntity.status(429)
                .body("请求过于频繁，请稍后重试");
        }
        return ResponseEntity.status(500)
            .body("AI 服务异常：" + e.getMessage());
    }
}
```

### 重试机制

```java
@Configuration
class RetryConfig {
    
    @Bean
    public RetryTemplate retryTemplate() {
        return RetryTemplate.builder()
            .maxAttempts(3)
            .fixedBackoff(1000)
            .retryOn(AiException.class)
            .build();
    }
}
```

### 最佳实践建议

```markdown
1. Prompt 优化
   - 使用明确的系统提示词
   - 提供足够的上下文示例
   - 结构化输出优于自由文本

2. 性能优化
   - 启用流式响应改善用户体验
   - 合理设置 maxTokens 避免浪费
   - 使用缓存减少重复请求

3. 成本控制
   - 生产环境使用更轻量的模型
   - 实现请求去重和合并
   - 设置预算告警

4. 安全考虑
   - 永远不要在代码中硬编码 API Key
   - 实现用户输入过滤
   - 限制输出内容长度
```

---

## 常见问题

### Q：Spring AI 与 LangChain4j 的区别？

A：两者都是 Java AI 框架。Spring AI 与 Spring 生态深度集成，适合 Spring 项目；LangChain4j 更轻量，API 更灵活。选择取决于项目技术栈。

### Q：如何选择合适的模型？

A：简单任务用轻量模型（GPT-3.5、abab5.5），复杂推理用旗舰模型（GPT-4、abab6.5），RAG 场景用联网版。

### Q：Token 如何计算？

A：中文 1 Token ≈ 1-2 个字符，英文 1 Token ≈ 4 个字符。可用`tiktoken`库精确计算。

---

## 后续学习计划

- [ ] 多模型支持切换
- [ ] RAG 实现完善
- [ ] 向量数据库集成
- [ ] Function Calling 实战
- [ ] Agent 开发
- [ ] 图像处理（MiniMax Vision）

---

## 参考资源

- [Spring AI 官方文档](https://docs.spring.io/spring-ai/reference/)
- [MiniMax API 文档](https://www.minimaxi.com/document)
- [Spring AI Samples](https://github.com/spring-projects/spring-ai/tree/main/models/spring-ai-openai/src/test/java/org/springframework/ai/openai)

---

## 总结

Spring AI 提供了统一、简洁的 AI 应用开发体验。本文介绍了核心功能，包括：

- **ChatClient API**：统一调用方式
- **多模型支持**：MiniMax、OpenAI 等
- **Prompt 工程**：模板与 Few-Shot
- **Function Calling**：工具调用能力
- **RAG 架构**：检索增强生成

随着 AI 技术发展，Spring AI 将持续更新更多功能。建议持续关注官方 Release Notes。

---

*最后更新：2026/4/4*
