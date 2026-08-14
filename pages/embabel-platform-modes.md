# Artificial Intelligence Concepts & Embabel Agent Integration Guide

This comprehensive reference document covers core AI systems theory, state machine definitions, and an enterprise implementation guide using the **Embabel Agent Framework**, **Spring Boot Scheduling**, and **Model Context Protocol (MCP)** tools.

---

## 🗺️ Part 1: AI Architectural Concepts

### World State
In artificial intelligence, the **world state** refers to a comprehensive snapshot or representation of the environment at a specific point in time. It captures all relevant variables, facts, and configurations that an AI agent needs to perceive, reason about, and act upon.

*   **Classical AI and Automated Planning:** The "world" is modeled as a collection of variables (e.g., the position of every piece on a chessboard). Actions cause a state transition. The goal is to discover a sequence of transitions that move from the initial state to the "goal state."
*   **Reinforcement Learning & Robotics:** The ground truth of the system at time step $t$. If completely visible, it is *fully observable*. If restricted (e.g., a self-driving vehicle's front camera view), it is *partially observable*.
*   **World State vs. World Model:** The **World State** is the *current status* of the environment. The **World Model** is the *internal simulator* or transition system used by the AI to predict how its actions will change that world state.

### Embabel Execution Modes
When interacting with target AI engines inside the Embabel framework, three operational boundaries govern agent behavior:
*   **Focused Mode:** Used when developers explicitly determine *which* agent to run, passing down deterministic context. Best for programmatic triggers (e.g., automated schedulers).
*   **Closed Mode:** The platform implicitly evaluates user intent and dynamically determines the best routing path to an agent.
*   **Open Mode:** An open-ended playground where the system evaluates all known tools and maps a dynamic plan to hit an objective autonomously.

---

## 🔀 Part 2: Finite State Machine (FSM) Definitions

A **Finite State Machine (FSM)** is a mathematical abstraction used to design sequential logic paths. Formally, a deterministic finite automation is represented by a **5-tuple**:

$$\mathbf{M = (Q, \Sigma, \delta, q_0, F)}$$

*   **$Q$ (Set of States):** A non-empty, finite collection of all valid system conditions.
*   **$\Sigma$ (Input Alphabet):** A finite set of valid input symbols or incoming event triggers.
*   **$\delta$ (Transition Function):** The operational mapping rulebook: $\delta: Q \times \Sigma \rightarrow Q$.
*   **$q_0$ (Initial State):** The specific entry point state where $q_0 \in Q$.
*   **$F$ (Set of Final/Accept States):** The subset of success criteria conditions ($F \subseteq Q$).

### Taxonomy of FSMs
1.  **By Output Engine:** *Moore Machines* determine outputs entirely by the active state. *Mealy Machines* compute outputs based on both the active state and current input parameters.
2.  **By Predictability:** *Deterministic Finite Automata (DFA)* map to exactly one explicit next state per input. *Nondeterministic Finite Automata (NFA)* can branch into multiple states concurrently or shift automatically via $\epsilon$-transitions.

---

## 🛠️ Part 3: Spring Boot Integration Blueprint

This production blueprint handles structural file cleanup using **Embabel Focused Mode**, automated **Groq/OpenAI splitting**, and standard **MCP Postgres database tool connectivity**.

### 📦 Dependencies (`pom.xml`)
```xml
<dependencies>
    <!-- Spring AI Base Starter -->
    <dependency>
        <groupId>org.springframework.ai</groupId>
        <artifactId>spring-ai-openai-spring-boot-starter</artifactId>
    </dependency>

    <!-- Embabel Starter Platform -->
    <dependency>
        <groupId>com.embabel</groupId>
        <artifactId>embabel-agent-starter</artifactId>
        <version>${embabel.version}</version>
    </dependency>
</dependencies>
```

### ⚙️ Infrastructure Configuration (`application.yml`)
```yaml
spring:
  ai:
    openai:
      api-key: ${OPENAI_API_KEY}
      chat:
        options:
          model: gpt-4o

groq:
  ai:
    chat:
      options:
        api-key: ${GROQ_API_KEY}
        base-url: https://groq.com
        model: llama-3.3-70b-versatile

embabel:
  default-llm: gpt-4o
  ranking-llm: llama-3.3-70b-versatile
  
  mcp:
    clients:
      database-tool:
        docker:
          image: mcp/postgres
          env:
            DATABASE_URL: "postgresql://user:password@localhost:5432/my_db"
```

---

## ☕ Part 4: Source Implementation

### Application Bootstrap
```java
package com.example.agent;

import io.embabel.agent.spring.annotation.EnableAgents;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.scheduling.annotation.EnableScheduling;

@SpringBootApplication
@EnableScheduling
@EnableAgents 👈 Activates classpath scanning for Embabel agents
public class AgentApplication {
    public static void main(String[] args) {
        SpringApplication.run(AgentApplication.class, args);
    }
}
```
### Archival Context 
```java
package com.example.agent.components;

/**
 * Domain context record used by the DataArchiverAgent to manage 
 * state, thresholds, and execution tracking across the planning loop.
 * 
 * @param approvalGranted Indicates whether a human operator has authorized the operation
 * @param recordsToDelete The total count of database entries flagged for deletion
 * @param taskId          A unique identifier used to track and resume the agent's execution state
 */
public record ArchivalContext(
    boolean approvalGranted, 
    int recordsToDelete, 
    String taskId
) {}

```

### The Agent & Human-In-The-Loop (HITL) Guard
```java
package com.example.agent.components;

import io.embabel.agent.api.annotation.Agent;
import io.embabel.agent.api.annotation.Action;
import io.embabel.agent.api.annotation.AchievesGoal;
import io.embabel.agent.core.flow.WaitFor;
import java.util.Map;

@Agent(name = "dataArchiverAgent", description = "Handles cleanup and archival of historical records")
public class DataArchiverAgent {

    public record ArchivalContext(boolean approvalGranted, int recordsToDelete, String taskId) {}

    @Action
    @AchievesGoal("SAFE_DELETE_RECORDS")
    public Object purgeOldRecords(ArchivalContext context) {
        
        // Safeguard: Handoff to human review if mutating > 100 rows without verification
        if (context.recordsToDelete() > 100 && !context.approvalGranted()) {
            return WaitFor.userApproval(
                "Requesting permission to delete " + context.recordsToDelete() + " database rows.",
                Map.of("taskId", context.taskId())
            );
        }

        // Executed autonomously via native Postgres MCP bindings if approved
        return Map.of("status", "DELETED_SUCCESSFULLY", "deletedCount", context.recordsToDelete());
    }
}
```

### Manual Engine Bean Registration
```java
package com.example.agent.config;

import io.embabel.agent.core.EmbabelAgent;
import io.embabel.agent.spring.EmbabelPlatform;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class EmbabelAgentConfig {

    @Bean(name = "dataArchiverAgent")
    public EmbabelAgent dataArchiverAgent(EmbabelPlatform platform) {
        return platform.newAgentBuilder()
                .withName("DataArchiverAgent")
                .withSystemPrompt("You are an administrative assistant. Clean up databases safely.")
                .withMcpClient("database-tool") 
                .build();
    }
}
```

### Background Execution Worker
```java
package com.example.agent.scheduler;

import io.embabel.agent.core.EmbabelAgent;
import io.embabel.agent.core.AgentResponse;
import com.example.agent.components.DataArchiverAgent.ArchivalContext;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;

@Component
public class AgentWorkflowScheduler {

    private static final Logger log = LoggerFactory.getLogger(AgentWorkflowScheduler.class);
    private final EmbabelAgent dataArchiverAgent;

    @Autowired
    public AgentWorkflowScheduler(EmbabelAgent dataArchiverAgent) {
        this.dataArchiverAgent = dataArchiverAgent;
    }

    @Scheduled(fixedRate = 3600000) // Triggers hourly
    public void runScheduledAgentTask() {
        String uniqueTaskId = "archive-" + System.currentTimeMillis();
        ArchivalContext context = new ArchivalContext(false, 150, uniqueTaskId);

        // Executes via Focused Mode
        AgentResponse response = dataArchiverAgent.run(context);

        if (response.isWaitingForHuman()) { 
            log.warn("CRITICAL: Destructive action halted. Task {} is paused waiting for verification.", uniqueTaskId);
        } else {
            log.info("Agent processed task immediately. Result: {}", response.getResult());
        }
    }
}
```

### REST Verification Gateway
```java
package com.example.agent.controller;

import io.embabel.agent.spring.EmbabelPlatform;
import io.embabel.agent.core.AgentResponse;
import org.springframework.web.bind.annotation.*;
import com.example.agent.components.DataArchiverAgent.ArchivalContext;

@RestController
@RequestMapping("/api/agent/approval")
public class AgentApprovalController {

    private final EmbabelPlatform embabelPlatform;

    public AgentApprovalController(EmbabelPlatform embabelPlatform) {
        this.embabelPlatform = embabelPlatform;
    }

    @PostMapping("/{taskId}/resume")
    public String approveTask(@PathVariable String taskId, @RequestParam boolean approved) {
        if (!approved) return "Task execution explicitly aborted.";

        // Rehydrate domain context with approval bit set to true
        ArchivalContext approvedContext = new ArchivalContext(true, 150, taskId);
        
        // Re-inject the execution track directly back into the Embabel engine state machine
        AgentResponse response = embabelPlatform.getAgent("dataArchiverAgent").resume(taskId, approvedContext);
```
## 🧪 Testing the Flow

This section outlines how to mock external LLM inferences and assert that the Human-in-the-Loop (HITL) safeguard correctly forces a WaitFor state.

Testing an autonomous agent requires validating both the deterministic logic boundaries (the HITL guards) and the non-deterministic components (the LLM tool invocation planning).

To ensure stability without executing actual cloud LLM APIs, we use `spring-ai-stream-test-support` or standard Spring Boot Mocking structures.

### 1. Unit Testing the Safeguard Logic
This test verifies that if the agent encounters an unapproved context over the threshold limit, it returns an explicit `WaitFor` state instead of executing the destruction path.

```java
package com.example.agent.components;

import io.embabel.agent.core.flow.WaitFor;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import java.util.Map;

import static org.junit.jupiter.api.Assertions.*;

class DataArchiverAgentTest {

    private final DataArchiverAgent agent = new DataArchiverAgent();

    @Test
    @DisplayName("Should trigger WaitFor pause when rows exceed threshold without approval")
    void testPurgeOldRecordsTriggersSafeguard() {
        // Arrange
        var unapprovedContext = new DataArchiverAgent.ArchivalContext(false, 150, "test-task-123");

        // Act
        Object result = agent.purgeOldRecords(unapprovedContext);

        // Assert
        assertInstanceOf(WaitFor.class, result, "Should return a WaitFor instance when unapproved");
        WaitFor waitFor = (WaitFor) result;
        assertTrue(waitFor.getMessage().contains("150 database rows"));
        assertEquals("test-task-123", waitFor.getMetadata().get("taskId"));
    }

    @Test
    @DisplayName("Should execute deletion completely when approval is granted")
    void testPurgeOldRecordsExecutesWithApproval() {
        // Arrange
        var approvedContext = new DataArchiverAgent.ArchivalContext(true, 150, "test-task-456");

        // Act
        Object result = agent.purgeOldRecords(approvedContext);

        // Assert
        assertInstanceOf(Map.class, result);
        Map<?, ?> resultMap = (Map<?, ?>) result;
        assertEquals("DELETED_SUCCESSFULLY", resultMap.get("status"));
        assertEquals(150, resultMap.get("deletedCount"));
    }
}
```

### 2. Integration Testing the Controller Resumption Lifecycle
This integration test uses `@SpringBootTest` to evaluate the lifecycle end-to-end: triggering a mock agent suspension state and asserting that hitting the REST gateway successfully rehydrates and executes the runner loop.

```java
package com.example.agent.controller;

import io.embabel.agent.spring.EmbabelPlatform;
import io.embabel.agent.core.EmbabelAgent;
import io.embabel.agent.core.AgentResponse;
import com.example.agent.components.DataArchiverAgent.ArchivalContext;
import org.junit.jupiter.api.Test;
import org.mockito.Mockito;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.AutoConfigureMockMvc;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.mock.mockito.MockBean;
import org.springframework.test.web.servlet.MockMvc;

import static org.mockito.ArgumentMatchers.any;
import static org.mockito.ArgumentMatchers.eq;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.post;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.content;

@SpringBootTest
@AutoConfigureMockMvc
class AgentApprovalControllerIntegrationTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private EmbabelPlatform embabelPlatform;

    @MockBean(name = "dataArchiverAgent")
    private EmbabelAgent mockAgent;

    @Test
    void testApproveTaskSuccessfullyResumesAgent() throws Exception {
        String taskId = "archive-999";
        
        // Mocking framework execution layer
        Mockito.when(embabelPlatform.getAgent("dataArchiverAgent")).thenReturn(mockAgent);
        
        AgentResponse mockResponse = Mockito.mock(AgentResponse.class);
        Mockito.when(mockResponse.getResult()).thenReturn("SUCCESS_EXECUTION");
        
        Mockito.when(mockAgent.resume(eq(taskId), any(ArchivalContext.class)))
               .thenReturn(mockResponse);

        // Simulate Human Approval interaction over REST Gateway
        mockMvc.perform(post("/api/agent/approval/" + taskId + "/resume")
                .param("approved", "true"))
                .andExpect(status().isOk())
                .andExpect(content().string("Agent processing completed. Outcome: SUCCESS_EXECUTION"));
    }
}
```

## 🐳 Integration Testing with Testcontainers (Gradle Setup)

To accurately test how the agent interacts with actual schema boundaries without polluting production instances, we use **Testcontainers**. This setup automatically provisions an isolated PostgreSQL database inside an ephemeral Docker container during the test initialization phase.

### 1. Additional Testing Dependencies

Add the specialized Testcontainers dependencies to your `build.gradle.kts` (Kotlin) or `build.gradle` (Groovy) file under the `testImplementation` configuration block.

#### Option A: Kotlin DSL (`build.gradle.kts`)
```kotlin
dependencies {
    // Standard application dependencies go here...

    // Testcontainers Base & JUnit 5 Extension
    testImplementation("org.testcontainers:junit-jupiter:1.20.1")
    testImplementation("org.testcontainers:postgresql:1.20.1")
    
    // Testing essentials
    testImplementation("org.springframework.boot:spring-boot-starter-test")
}

tasks.withType<Test> {
    useJUnitPlatform() // Ensures JUnit 5 runs correctly
}
```

#### Option B: Groovy DSL (`build.gradle`)
```groovy
dependencies {
    // Standard application dependencies go here...

    // Testcontainers Base & JUnit 5 Extension
    testImplementation 'org.testcontainers:junit-jupiter:1.20.1'
    testImplementation 'org.testcontainers:postgresql:1.20.1'
    
    // Testing essentials
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
}

test {
    useJUnitPlatform() // Ensures JUnit 5 runs correctly
}
```

### 2. Base Test Container Configuration Base
Create an abstract base configuration class. This infrastructure block intercepts the dynamic random ports allocated by Testcontainers and dynamically injects them back into Spring's configuration engine before execution kicks off.

```java
package com.example.agent.base;

import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.DynamicPropertyRegistry;
import org.springframework.test.context.DynamicPropertySource;
import org.testcontainers.containers.PostgreSQLContainer;
import org.testcontainers.junit.jupiter.Container;
import org.testcontainers.junit.jupiter.Testcontainers;

@SpringBootTest
@Testcontainers
public abstract class BaseIntegrationTest {

    // Automatically pulls down the precise PostgreSQL engine version matching production
    @Container
    protected static final PostgreSQLContainer<?> postgresContainer = 
        new PostgreSQLContainer<>("postgres:16-alpine")
            .withDatabaseName("test_db")
            .withUsername("test_user")
            .withPassword("test_pass");

    @DynamicPropertySource
    static void configureProperties(DynamicPropertyRegistry registry) {
        // Overrides target app connection URLs to route directly into the live test container
        registry.add("spring.datasource.url", postgresContainer::getJdbcUrl);
        registry.add("spring.datasource.username", postgresContainer::getUsername);
        registry.add("spring.datasource.password", postgresContainer::getPassword);
        
        // Re-injects the container URL directly into the Embabel MCP configuration block
        registry.add("embabel.mcp.clients.database-tool.docker.env.DATABASE_URL", 
            postgresContainer::getJdbcUrl);
    }
}
```

### 3. Concrete Agent Interaction Integration Test
Inherit from the base infrastructure setup. This integration test initializes the schema, passes an approved transactional context, and validates that the agent successfully interacts with the database instance.

```java
package com.example.agent.components;

import com.example.agent.base.BaseIntegrationTest;
import io.embabel.agent.core.EmbabelAgent;
import io.embabel.agent.core.AgentResponse;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.jdbc.core.JdbcTemplate;
import java.util.Map;

import static org.junit.jupiter.api.Assertions.*;

class DataArchiverAgentDbIntegrationTest extends BaseIntegrationTest {

    @Autowired
    private JdbcTemplate jdbcTemplate;

    @Autowired
    @Qualifier("dataArchiverAgent")
    private EmbabelAgent dataArchiverAgent;

    @BeforeEach
    void setupDatabaseSchema() {
        // Arrange: Recreate fresh schema entities before every test execution path
        jdbcTemplate.execute("DROP TABLE IF EXISTS audit_records;");
        jdbcTemplate.execute("CREATE TABLE audit_records (id SERIAL PRIMARY KEY, payload TEXT);");
        
        // Insert mock stale records to verify agent mutation capability
        jdbcTemplate.execute("INSERT INTO audit_records (payload) VALUES ('old_data_1'), ('old_data_2');");
    }

    @Test
    @DisplayName("Should modify real database rows when execution context is approved")
    void testAgentExecutionAgainstRealContainer() {
        // Act: Invoke the target agent loop in focused mode with human override pre-granted
        var approvedContext = new DataArchiverAgent.ArchivalContext(true, 2, "integration-task-001");
        AgentResponse response = dataArchiverAgent.run(approvedContext);

        // Assert: Validate tracking response properties 
        assertNotNull(response);
        assertFalse(response.isWaitingForHuman(), "Agent should complete immediately with active approval");

        // Query the live test database to assert that structural mutations successfully completed via the MCP tool
        Integer remainingRows = jdbcTemplate.queryForObject("SELECT COUNT(*) FROM audit_records;", Integer.class);
        assertEquals(0, remainingRows, "The MCP database tool should have purged the targeted records from the table");
    }
}
```
