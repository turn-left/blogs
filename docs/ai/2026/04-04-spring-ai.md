# 2026/4/4 Spring AI 实践

记录 Spring AI 框架的初次实践。

## 概述

Spring AI 是 Spring 生态中用于构建 AI 应用的框架，提供了对各种大语言模型（LLM）的统一抽象。

## 环境准备

```bash
# Spring Boot 3.2+
# Java 21
```

## 快速开始

### 1. 添加依赖

```xml
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-starter</artifactId>
    <version>1.0.0</version>
</dependency>
```

### 2. 配置 OpenAI

```yaml
spring:
  ai:
    openai:
      api-key: ${OPENAI_API_KEY}
      chat:
        options:
          model: gpt-4o
```

### 3. 基本使用

```java
@RestController
class AIController {

    private final ChatClient chatClient;

    public AIController(ChatClient.Builder chatClientBuilder) {
        this.chatClient = chatClientBuilder.build();
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

## 进阶功能

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

### 提示词模板

```java
@Bean
PromptTemplate promptTemplate() {
    return new PromptTemplate("请用{style}风格介绍{topic}");
}

String result = promptTemplate.render(
    Map.of("style", "简洁", "topic", "Spring AI")
);
```

## 总结

Spring AI 简化了 AI 应用的开发流程，提供了统一的 API 接口，后续计划尝试：
- [ ] 多模型支持
- [ ] RAG 实现
- [ ] 向量数据库集成
