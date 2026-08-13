# Connecting Free LLMs to Spring Boot using Spring AI

This guide provides the complete setup configurations, Maven dependencies, and code required to connect a free-of-charge Large Language Model (LLM) to a Spring Boot application using the native **Spring AI** framework.

---

## 🛠️ Approach 1: Local Deployment (100% Free & Unlimited)
Run open-source models (like Llama 3 or Mistral) locally on your own hardware using **Ollama**. This method requires no internet access after download and keeps all data completely private.

### 1. Download & Run Ollama
Execute the following command in your terminal to download and spin up the model engine locally:
```bash
ollama run llama3
```

### 2. Maven Dependencies (`pom.xml`)
Add the official Spring AI Ollama starter to your project configuration:
```xml
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-ollama-spring-boot-starter</artifactId>
</dependency>
```

### 3. Application Properties (`application.properties`)
Point your Spring Boot app to your local Ollama instance running on port `11434`:
```properties
spring.ai.ollama.base-url=http://localhost:11434
spring.ai.ollama.chat.options.model=llama3
```

---

## 🌐 Approach 2: Google Gemini API (Cloud Free-Tier)
Leverage cloud infrastructure without a powerful local GPU using Google AI Studio's free developer tier (e.g., up to 15 Requests Per Minute on Gemini 1.5 Flash).

### 1. Get Your Free API Key
Generate a free-tier API key from the [Google AI Studio Console](https://google.com).

### 2. Maven Dependencies (`pom.xml`)
Add the official Spring AI Vertex AI Gemini starter to your project:
```xml
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-vertex-ai-gemini-spring-boot-starter</artifactId>
</dependency>
```

### 3. Application Properties (`application.properties`)
Configure your key and target a lightweight model optimized for speed:
```properties
spring.ai.vertex.ai.gemini.api-key=YOUR_FREE_GEMINI_API_KEY
spring.ai.vertex.ai.gemini.chat.options.model=gemini-1.5-flash
```

---

## ⚡ Approach 3: Groq Cloud API (OpenAI-Compatible Free-Tier)
Access ultra-fast open-source model hosting on Groq's hardware for free. Because Groq exposes an OpenAI-compliant endpoint, we reuse the standard OpenAI starter.

### 1. Get Your Free API Key
Generate a free-tier developer API key from the [Groq Console](https://groq.com).

### 2. Maven Dependencies (`pom.xml`)
Add the standard Spring AI OpenAI starter:
```xml
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-openai-spring-boot-starter</artifactId>
</dependency>
```

### 3. Application Properties (`application.properties`)
Override the default OpenAI API base URL to redirect all network traffic to Groq's routing nodes:
```properties
spring.ai.openai.base-url=https://groq.com
spring.ai.openai.api-key=YOUR_FREE_GROQ_API_KEY
spring.ai.openai.chat.options.model=llama3-8b-8192
```

---

## 🚀 Unified Spring Boot Implementation Code
Regardless of which infrastructure approach you choose above, Spring AI uses a unified interface abstraction. You do not need to change a single line of Java code to swap models.

### 1. The REST Controller (`AiChatController.java`)
```java
package com.example.demo;

import org.springframework.ai.chat.client.ChatClient;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class AiChatController {

    private final ChatClient chatClient;

    // The generic ChatClient builder automatically binds to whichever starter is active
    public AiChatController(ChatClient.Builder builder) {
        this.chatClient = builder.build();
    }

    @GetMapping("/ai/generate")
    public String generate(@RequestParam(value = "message", defaultValue = "Tell me a joke") String message) {
        return this.chatClient.prompt(message)
                .call()
                .content();
    }
}
```

### 2. Testing Your Application
Run your Spring Boot application and execute a simple GET request via your browser or terminal to fetch your free model response:
```bash
curl http://localhost:8080/ai/generate?message=Explain+GitOps+in+one+sentence
```
