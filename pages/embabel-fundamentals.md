# Multi-Staged Pipelines with the Embabel Blackboard

This guide explains how the **Blackboard pattern** functions as the architectural backbone for orchestrating complex, multi-staged pipelines within the [Embabel Agent Framework](https://github.com) (a JVM-native AI agent framework built by Rod Johnson).

---

## What is the Blackboard?

In the Embabel Agent Framework, a **Blackboard** is a **strongly typed, shared working memory system** that maintains process state throughout an agent's execution.

Unlike traditional pipelines that rely on rigid method chaining or passing unstructured "bags of strings" (like loose JSON maps), the Embabel blackboard matches and binds data **by its explicit Java/Kotlin data type**.

### Core Characteristics

* **Central State Repository:** Acts as the single source of truth storing initial user inputs, intermediate tool outputs, domain records, and the overall process state.
* **Type-Based Data Binding:** Actions automatically extract their required parameters from the blackboard by data type and publish their results back onto it as structured records.
* **Loose Coupling & Dynamic Planning:** Steps do not reference each other directly. Embabel uses **Goal-Oriented Action Planning (GOAP)** to inspect the blackboard's contents and dynamically determine which action to execute next.
* **Immutability & Versioning:** Appended objects are treated as immutable. If data is updated, a new version is added, and the runtime defaults to retrieving the latest candidate.
* **Condition Tracking:** Maintains boolean evaluations and system preconditions (e.g., `isUserVerified = true`) to guide dynamic branching.

---

## Installation & Setup

Embabel integrates natively with standard Spring Boot configurations.

### 1. Add Dependencies

Add the core Embabel starter dependency to your `pom.xml` (Maven) or `build.gradle` file.

**Maven (`pom.xml`):**
```xml
<dependency>
    <groupId>com.embabel.agent</groupId>
    <artifactId>embabel-agent-starter-shell</artifactId>
    <version>0.3.1</version>
</dependency>
```

**Gradle (`build.gradle`):**
```groovy
implementation 'com.embabel.agent:embabel-agent-starter-shell:0.3.1'
```

### 2. Configure Environment Keys

Set up your preferred LLM provider API credentials as environment variables before starting the application:

```bash
# Set your keys based on your preferred provider
export OPENAI_API_KEY="your-openai-api-key-here"
export ANTHROPIC_API_KEY="your-anthropic-api-key-here"
```

### 3. Bootstrap Embabel in Spring Boot

To enable Embabel's automatic action scanner and core registry, decorate your main Spring configuration class with `@EnableEmbabel`:

```java
package com.example.blog;

import com.embabel.annotations.EnableEmbabel;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
@EnableEmbabel // Activates Embabel's component registry and dynamic action scanner
public class BlogAgentApplication {
    public static void main(String[] args) {
        SpringApplication.run(BlogAgentApplication.class, args);
    }
}
```

---

## Local Testing with Ollama

You can completely replace cloud LLM providers with a locally running instance of **Ollama** for development and integration testing.

### 1. Start Ollama and Pull a Model
Ensure Ollama is installed and running on your local machine, then pull your model of choice (e.g., Llama 3 or Mistral):

```bash
ollama run llama3
```

### 2. Update `application.properties`
Configure Embabel to bypass standard cloud gateways and route traffic to your local Ollama server (`http://localhost:11434`). Add the following properties to your `src/main/resources/application.properties` file:

```properties
# Set the active Embabel LLM provider to Ollama
embabel.ai.provider=ollama
embabel.ai.ollama.url=http://localhost:11434

# Define the model to map for standard agent tasks
embabel.ai.ollama.model=llama3

# Optional: Adjust timeout parameters for larger local models
embabel.ai.timeout-seconds=60
```

*Note: Since the Blackboard relies heavily on structured output generation (JSON/Type binding under the hood), ensure the local model you choose is competent at following structural JSON constraints.*

---

## Code Example: Automated Blog Writing Pipeline

This example demonstrates how a multi-staged blog writing assistant is wired purely by declaring domain models and utilizing type-matching via the `@Action` annotation.

### 1. Define the Strongly-Typed Domain Models

Define clear, structured records representing each stage of your domain lifecycle:

```java
package com.example.blog.domain;

import java.util.List;

// Initial state submitted by the user
public record UserInput(String content) {}

// Intermediate state objects
public record BlogIdea(String coreAngle, List<String> targetKeywords) {}
public record BlogOutline(String title, List<String> sections) {}

// Final state object / Goal
public record FinalBlogPost(String title, String markdownContent) {}
```

### 2. Implement the Agent Actions

By declaring parameters and return types, Embabel infers the pipeline sequence seamlessly.

```java
package com.example.blog.agent;

import com.example.blog.domain.*;
import com.embabel.annotations.Action;
import com.embabel.annotations.Agent;
import com.embabel.annotations.AchievesGoal;
import com.embabel.api.AI;
import org.springframework.stereotype.Component;

@Agent(description = "Automates the generation of structured technical blog posts")
@Component // Integrates cleanly as a standard Spring bean
public class BlogWritingAgent {

    // STAGE 1: Consumes UserInput (from blackboard) -> Produces BlogIdea
    @Action(description = "Analyze user input and brainstorm a blog strategy")
    public BlogIdea understandIdea(UserInput userInput, AI ai) {
        return ai.withDefaultLlm()
                .creating(BlogIdea.class)
                .fromPrompt("Brainstorm keywords and a distinct angle for: " + userInput.content());
    }

    // STAGE 2: Consumes BlogIdea (from blackboard) -> Produces BlogOutline
    @Action(description = "Generate a structured outline based on the blog idea")
    public BlogOutline createOutline(BlogIdea idea, AI ai) {
        return ai.withDefaultLlm()
                .creating(BlogOutline.class)
                .fromPrompt("Create an outline focusing on the angle: " + idea.coreAngle());
    }

    // STAGE 3: Consumes BlogOutline (from blackboard) -> Produces FinalBlogPost
    @Action(description = "Write the final blog draft in markdown format")
    @AchievesGoal // Signals to Embabel that generating this type completes the pipeline execution
    public FinalBlogPost writeDraft(BlogOutline outline, AI ai) {
        return ai.withDefaultLlm()
                .creating(FinalBlogPost.class)
                .fromPrompt("Write a complete markdown post following this outline: " + outline.title());
    }
}
```

---

## Error Handling & Condition Routing

When running pipelines (especially via local LLMs), output parsing can occasionally fail structural validation, or business rules may dictate conditional routing. The Embabel Blackboard handles this gracefully via **Preconditions** and **Retry Policies**.

### 1. Conditional Routing using `@Prerequisite`

If you need to guide the path of the blackboard dynamically based on a state change, use custom condition evaluations. For example, if an outline must pass a human approval step or automated safety check before being drafted:

```java
import com.embabel.annotations.Prerequisite;

// This stage won't trigger until the "outlineApproved" flag is flag-marked True on the Blackboard
@Action(description = "Write the final blog draft in markdown format")
@Prerequisite(condition = "outlineApproved == true") 
public FinalBlogPost writeApprovedDraft(BlogOutline outline, AI ai) {
    // Pipeline logic goes here...
}
```

### 2. Validation & Self-Correction Loops

If an action returns a type that violates programmatic constraints, Embabel can capture the exception, look at the error details, feed the context back to the LLM, and update the Blackboard with the corrected object version automatically.

```java
import com.embabel.annotations.RetryOnFailure;
import com.embabel.api.exceptions.GenerationValidationException;

@Action(description = "Generate a structured outline based on the blog idea")
@RetryOnFailure(maxAttempts = 3, fallbackToHuman = false) // Re-runs step automatically if exception is thrown
public BlogOutline createValidatedOutline(BlogIdea idea, AI ai) {
    BlogOutline outline = ai.withDefaultLlm()
            .creating(BlogOutline.class)
            .fromPrompt("Create an outline focusing on the angle: " + idea.coreAngle());

    // Explicitly validate data properties
    if (outline.sections().isEmpty() || outline.title().isBlank()) {
        throw new GenerationValidationException("The generated outline must have a valid title and at least one section.");
    }

    return outline;
}
```

### 3. Global Exception Auditing

The Blackboard logs failed execution cycles explicitly. You can attach a generic error tracking subscriber to trace execution context bottlenecks:

```java
import com.embabel.api.blackboard.BlackboardListener;
import com.embabel.api.blackboard.BlackboardEvent;

public class ExecutionAuditLogger implements BlackboardListener {
    @Override
    public void onEvent(BlackboardEvent event) {
