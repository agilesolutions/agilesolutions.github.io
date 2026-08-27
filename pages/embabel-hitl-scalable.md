# End-to-End Stateless Embabel Agent Integration with PostgreSQL Persistence

This blueprint outlines a complete, production-grade architecture for managing Human-in-the-Loop (HITL) workflows using **Embabel**. It avoids volatile in-memory state tracking by configuring a **PostgreSQL JDBC backend**, ensuring high-availability across stateless application instances.

The workflow leverages an asynchronous **Embabel Agent** that executes automated steps, pauses at a `waitFor` human verification checkpoint, and safely commits execution snapshots to a central database cluster. Concurrency race conditions are safely resolved using **JPA Optimistic Locking (`@Version`)**, auditing is tracked via **Spring Data Envers**, and API routing is fully guarded by **Spring Security OAuth2 JWT Resource Server** filters.

---

## 1. System Topology & Architectural Map

```
               +----------------------------------------+
               |       API Gateway / Front-End UI       |
               +----------------------------------------+
                     |                            |
          [POST] /confirm                      [GET] /history
                     |                            |
                     v                            v
  +-----------------------------------------------------------------+
  |                    Spring Boot Application                      |
  |                                                                 |
  |  +-----------------------------------------------------------+  |
  |  |                 Security & Telemetry Tier                 |  |
  |  |  - Spring Security OAuth2 Filter (JWT Context Extraction) |  |
  |  |  - Global Exception Advice (Micrometer Alert Counting)    |  |
  |  +-----------------------------------------------------------+  |
  |                                |                                |
  |                                v                                |
  |  +-----------------------------------------------------------+  |
  |  |                    Business Logic Tier                    |  |
  |  |  - HitlTransactionController (REST Endpoints)             |  |
  |  |  - TransactionReviewService (Transactional Coordinator)   |  |
  |  |  - EmbabelAgentWorker (Asynchronous Engine Orchestrator)   |  |
  |  +-----------------------------------------------------------+  |
  |                 |                              |                |
  +-----------------|------------------------------|----------------+
                    | (JPA Data Operations)        | (JDBC Driver)
                    v                              v
  +-----------------------------------------------------------------+
  |                     PostgreSQL Database                         |
  |                                                                 |
  |   +--------------------------+    +-------------------------+   |
  |   |    Business Schema       |    |   Embabel Core State    |   |
  |   | - transactions           |    | - embabel_workflow_state|   |
  |   | - transactions_aud       |    +-------------------------+   |
  |   | - custom_revinfo         |                                  |
  |   +--------------------------+                                  |
  +-----------------------------------------------------------------+
```

---

## 2. Global Database Migrations (`V1__init_architecture.sql`)

Place this script inside `src/main/resources/db/migration/` for **Flyway** to execute automatically on initialization.

```sql
-- Custom global revision tracker table for Hibernate Envers Auditing
CREATE TABLE custom_revinfo (
    rev INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    revtstmp BIGINT,
    user_id VARCHAR(50),
    client_ip VARCHAR(45)
);

-- Core business transactions tracking schema
CREATE TABLE transactions (
    transaction_id VARCHAR(50) PRIMARY KEY,
    amount DECIMAL(10, 2) NOT NULL,
    status VARCHAR(20) DEFAULT 'PENDING_REVIEW',
    reviewed_at TIMESTAMP WITH TIME ZONE,
    version BIGINT NOT NULL DEFAULT 0
);

-- Transaction history audit log (Managed by Envers)
CREATE TABLE transactions_aud (
    transaction_id VARCHAR(50),
    rev INTEGER REFERENCES custom_revinfo(rev),
    revtype SMALLINT,
    amount DECIMAL(10, 2),
    status VARCHAR(20),
    reviewed_at TIMESTAMP WITH TIME ZONE,
    version BIGINT,
    PRIMARY KEY (transaction_id, rev)
);

-- Embabel workflow persistence engine catalog schema
CREATE TABLE embabel_workflow_state (
    instance_id VARCHAR(50) PRIMARY KEY,
    workflow_id VARCHAR(50) NOT NULL,
    current_step VARCHAR(50) NOT NULL,
    context_data JSONB,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);
```

---

## 3. Configuration & Infrastructure Layer

### Application Properties (`application.yml`)
```yaml
spring:
  datasource:
    url: ${DB_URL:jdbc:postgresql://localhost:5432/transaction_db}
    username: ${DB_USER:postgres}
    password: ${DB_PASSWORD:secret}
    driver-class-name: org.postgresql.Driver
  
  flyway:
    enabled: true
    baseline-on-migrate: true
    locations: classpath:db/migration

  security:
    oauth2:
      resourceserver:
        jwt:
          issuer-uri: http://localhost:8080/realms/transaction-realm

management:
  endpoints:
    web:
      exposure:
        include: prometheus,health
```

### Spring IoC Config Class (`EmbabelConfig.java`)
```java
package com.example.config;

import javax.sql.DataSource;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.envers.repository.config.EnableEnversRepositories;
import org.springframework.data.jpa.repository.config.EnableJpaRepositories;
import org.springframework.scheduling.annotation.EnableScheduling;
import com.embabel.engine.EmbabelEngine;
import com.embabel.persistence.jdbc.JdbcStateRepository;

@Configuration
@EnableScheduling
@EnableEnversRepositories(basePackages = "com.example.repository")
@EnableJpaRepositories(basePackages = "com.example.repository")
public class EmbabelConfig {

    @Bean
    public JdbcStateRepository jdbcStateRepository(DataSource dataSource) {
        return new JdbcStateRepository(dataSource);
    }

    @Bean
    public EmbabelEngine embabelEngine(JdbcStateRepository stateRepository) {
        return EmbabelEngine.builder()
                .withStateRepository(stateRepository)
                .build();
    }
}
```

---

## 4. Security, Telemetry & Envers Custom Auditing

### Audit Payload Extension Envelopes (`CustomRevisionEntity.java` & `UserAuditRevisionListener.java`)
```java
package com.example.audit;

import jakarta.persistence.*;
import org.hibernate.envers.DefaultRevisionEntity;
import org.hibernate.envers.RevisionEntity;

@Entity
@Table(name = "custom_revinfo")
@RevisionEntity(UserAuditRevisionListener.class)
public class CustomRevisionEntity extends DefaultRevisionEntity {
    
    @Column(name = "user_id", length = 50)
    private String userId;

    @Column(name = "client_ip", length = 45)
    private String clientIp;

    public String getUserId() { return userId; }
    public void setUserId(String userId) { this.userId = userId; }
    public String getClientIp() { return clientIp; }
    public void setClientIp(String clientIp) { this.clientIp = clientIp; }
}
```

```java
package com.example.audit;

import jakarta.servlet.http.HttpServletRequest;
import org.hibernate.envers.RevisionListener;
import org.springframework.security.core.Authentication;
import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.security.oauth2.jwt.Jwt;
import org.springframework.web.context.request.RequestContextHolder;
import org.springframework.web.context.request.ServletRequestAttributes;

public class UserAuditRevisionListener implements RevisionListener {

    @Override
    public void newRevision(Object revisionEntity) {
        CustomRevisionEntity customRev = (CustomRevisionEntity) revisionEntity;
        
        // 1. Unpack Network Routing Parameters
        ServletRequestAttributes attributes = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
        if (attributes != null) {
            HttpServletRequest request = attributes.getRequest();
            String ip = request.getHeader("X-Forwarded-For");
            if (ip == null || ip.isEmpty() || "unknown".equalsIgnoreCase(ip)) {
                ip = request.getRemoteAddr();
            }
            customRev.setClientIp(ip);
        } else {
            customRev.setClientIp("SYSTEM_DAEMON");
        }

        // 2. Decode Secure Identity Context Footprint
        Authentication auth = SecurityContextHolder.getContext().getAuthentication();
        if (auth != null && auth.getPrincipal() instanceof Jwt jwt) {
            String userName = jwt.getClaimAsString("preferred_username");
            if (userName == null) {
                userName = jwt.getSubject();
            }
            customRev.setUserId(userName);
        } else {
            customRev.setUserId("SYSTEM_AGENT");
        }
    }
}
```

### Global Controller Exception Handler & Metric Publishing (`GlobalExceptionHandler.java`)
```java
package com.example.exception;

import io.micrometer.core.instrument.Counter;
import io.micrometer.core.instrument.MeterRegistry;
import org.springframework.http.HttpStatus;
import org.springframework.http.ProblemDetail;
import org.springframework.orm.ObjectOptimisticLockingFailureException;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;
import java.time.Instant;

@RestControllerAdvice
public class GlobalExceptionHandler {

    private final Counter lockingConflictCounter;

    public GlobalExceptionHandler(MeterRegistry meterRegistry) {
        this.lockingConflictCounter = Counter.builder("platform.transactions.lock.conflicts")
                .description("Total number of simultaneous data collisions on manual risk operations")
                .register(meterRegistry);
    }

    @ExceptionHandler(ObjectOptimisticLockingFailureException.class)
    public ProblemDetail handleOptimisticLockingFailure(ObjectOptimisticLockingFailureException ex) {
        lockingConflictCounter.increment();
        ProblemDetail pd = ProblemDetail.forStatusAndDetail(
                HttpStatus.CONFLICT, 
                "This transaction record was modified by another analyst or runtime thread simultaneously."
        );
        pd.setTitle("Concurrent Modification Error");
        pd.setProperty("timestamp", Instant.now());
        return pd;
    }

    @ExceptionHandler(WorkflowStateException.class)
    public ProblemDetail handleWorkflowStateError(WorkflowStateException ex) {
        ProblemDetail pd = ProblemDetail.forStatusAndDetail(HttpStatus.BAD_REQUEST, ex.getMessage());
        pd.setTitle("Embabel Workflow Alignment Mismatch");
        pd.setProperty("timestamp", Instant.now());
        return pd;
    }
}
```

---

## 5. Domain, Repositories, & DTO Contracts

### Domain Object Entity (`Transaction.java`)
```java
package com.example.domain;

import jakarta.persistence.*;
import org.hibernate.envers.Audited;
import java.math.BigDecimal;
import java.time.OffsetDateTime;

@Entity
@Table(name = "transactions")
@Audited
public class Transaction {

    @Id
    @Column(name = "transaction_id", length = 50)
    private String transactionId;

    @Column(name = "amount", nullable = false, precision = 10, scale = 2)
    private BigDecimal amount;

    @Column(name = "status", length = 20)
    private String status;

    @Column(name = "reviewed_at")
    private OffsetDateTime reviewedAt;

    @Version
    @Column(name = "version")
    private Long version;

    public Transaction() {}

    // Getters and Setters
    public String getTransactionId() { return transactionId; }
    public void setTransactionId(String transactionId) { this.transactionId = transactionId; }
    public BigDecimal getAmount() { return amount; }
    public void setAmount(BigDecimal amount) { this.amount = amount; }
    public String getStatus() { return status; }
    public void setStatus(String status) { this.status = status; }
    public OffsetDateTime getReviewedAt() { return reviewedAt; }
    public void setReviewedAt(OffsetDateTime reviewedAt) { this.reviewedAt = reviewedAt; }
    public Long getVersion() { return version; }
    public void setVersion(Long version) { this.version = version; }
}
```

### Repositories (`TransactionRepository.java`)
```java
package com.example.repository;

import com.example.domain.Transaction;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.repository.history.RevisionRepository;
import org.springframework.stereotype.Repository;

@Repository
public interface TransactionRepository extends 
        JpaRepository<Transaction, String>, 
        RevisionRepository<Transaction, String, Integer> {
}
```

### Data Transfer Objects (`DataContracts.java`)
```java
package com.example.dto;

import java.math.BigDecimal;

public record HumanReviewRecord(
    String transactionId,
    BigDecimal transactionAmount,
    String flaggedReason,
    String triggeredStep
) {}

public record HitlDecisionRequest(
    String workflowInstanceId, 
    boolean approved
) {}

public record AdvancedAuditLogDto(
    Integer revisionId, 
    String analystUserId, 
    String connectionIp, 
    String timestamp, 
    String statusCheckpoint
) {}
```

---

## 6. Asynchronous Embabel Agent Worker & HITL Loop

### Automated Orchestration Engine Worker (`EmbabelAgentWorker.java`)
```java
package com.example.service;

import com.example.domain.Transaction;
import com.example.dto.HumanReviewRecord;
import com.example.repository.TransactionRepository;
import com.embabel.engine.EmbabelEngine;
import com.embabel.context.WorkflowContext;
import org.springframework.scheduling.annotation.Async;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

@Service
public class EmbabelAgentWorker {

    private final EmbabelEngine embabelEngine;
    private final TransactionRepository transactionRepository;

    public EmbabelAgentWorker(EmbabelEngine embabelEngine, TransactionRepository transactionRepository) {
        this.embabelEngine = embabelEngine;
        this.transactionRepository = transactionRepository;
    }

    /**
     * Spins up an async thread execution segment mimicking an automated ingestion daemon.
     */
    @Async
    @Transactional
    public void initiateAutomatedRiskEvaluation(String transactionId, String workflowInstanceId) {
        Transaction transaction = transactionRepository.findById(transactionId)
                .orElseThrow(() -> new RuntimeException("Transaction target not found."));

        // Generate context state and map down initial properties
        WorkflowContext context = embabelEngine.createNewContext(workflowInstanceId, "TRANSACTION_RISK_WORKFLOW");
        context.setCurrentStep("AUTOMATED_SCAN");
        context.putData("amount", transaction.getAmount());

        // Process analytical automated rules...
        if (transaction.getAmount().compareTo(new java.math.BigDecimal("10000.00")) > 0) {
            context.setCurrentStep("PENDING_HUMAN_CHECKPOINT");
            
            // Build the record containing exact human-review contextual metadata
            HumanReviewRecord metadata = new HumanReviewRecord(
                transaction.getTransactionId(),
                transaction.getAmount(),
                "Transaction amount exceeds high-risk threshold limits",
                context.getCurrentStep()
            );

            // Halt current execution branch thread and wait for external signal activation
            context.waitFor(metadata);
            
            // Commit structural progress parameters directly to postgresql tables
            embabelEngine.saveAndAdvance(context);
        } else {
            transaction.setStatus("AUTO_APPROVED");
            context.setCurrentStep("COMPLETED");
            transactionRepository.save(transaction);
            embabelEngine.saveAndAdvance(context);
        }
    }
}
```

### Transaction Service Layer (`TransactionReviewService.java`)
```java
package com.example.service;

import com.example.domain.Transaction;
import com.example.exception.WorkflowStateException;
import com.example.repository.TransactionRepository;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;
import com.embabel.engine.EmbabelEngine;
import com.embabel.context.WorkflowContext;
import com.embabel.hitl.HitlConfirmation;
import java.time.OffsetDateTime;

@Service
public class TransactionReviewService {

    private final EmbabelEngine embabelEngine;
    private final TransactionRepository transactionRepository;

    public TransactionReviewService(EmbabelEngine embabelEngine, TransactionRepository transactionRepository) {
        this.embabelEngine = embabelEngine;
        this.transactionRepository = transactionRepository;
    }

    @Transactional
    public void confirmTransaction(String instanceId, String transactionId, boolean approved) {
        // 1. Fetch record inside transactional envelope boundary context
        Transaction transaction = transactionRepository.findById(transactionId)
                .orElseThrow(() -> new RuntimeException("Transaction signature target not found: " + transactionId));
        
        // 2. Hydrate stateless engine workflow definition tracking matrix directly from postgresql
        WorkflowContext context;
        try {
            context = embabelEngine.loadWorkflowContext(instanceId);
        } catch (Exception e) {
            throw new WorkflowStateException("Could not resolve backend workflow state for instance: " + instanceId, e);
        }
        
        // 3. Extract the active pending HITL signal frame
        HitlConfirmation confirmation = context.getHitlSignal();
        if (confirmation == null) {
            throw new WorkflowStateException("No active Human-In-The-Loop hook found waiting for input on instance: " + instanceId, null);
        }

        String note = approved ? "Approved manually by secure web client operations interface." 
                               : "Rejected manually due to structural fraud indicators.";
        if (approved) {
            confirmation.approve(note);
            transaction.setStatus("APPROVED");
        } else {
            confirmation.reject(note);
            transaction.setStatus("REJECTED");
        }
        
        // 4. Save updates and increment database tracking rows (@Version checked automatically here)
        transaction.setReviewedAt(OffsetDateTime.now());
        transactionRepository.save(transaction);
        
        try {
            // 5. Signal the agent to advance its workflow and flush state
            embabelEngine.saveAndAdvance(context);
        } catch (Exception e) {
            throw new WorkflowStateException("Database infrastructure failed to flash sync tracking changes back to cluster state nodes.", e);
        }
    }
}
```

---

## 7. API Rest Controller Tier

```java
package com.example.controller;

import com.example.dto.HitlDecisionRequest;
import com.example.dto.AdvancedAuditLogDto;
import com.example.domain.Transaction;
import com.example.repository.TransactionRepository;
import com.example.service.TransactionReviewService;
import org.springframework.data.history.Revisions;
import org.springframework.http.ResponseEntity;
import org.springframework.security.access.prepost.PreAuthorize;
import org.springframework.web.bind.annotation.*;
import com.example.audit.CustomRevisionEntity;
import java.util.List;
import java.util.stream.Collectors;

@RestController
@RequestMapping("/api/v1/transactions")
public class HitlTransactionController {

    private final TransactionReviewService reviewService;
    private final TransactionRepository transactionRepository;

    public HitlTransactionController(TransactionReviewService reviewService, TransactionRepository transactionRepository) {
        this.reviewService = reviewService;
        this.transactionRepository = transactionRepository;
    }

    @PostMapping("/{transactionId}/confirm")
    @PreAuthorize("hasAuthority('SCOPE_transaction:review')")
    public ResponseEntity<Void> confirmHitlCheckpoint(
            @PathVariable String transactionId,
            @RequestBody HitlDecisionRequest request) {
        
        reviewService.confirmTransaction(request.workflowInstanceId(), transactionId, request.approved());
        return ResponseEntity.ok().build();
    }

    @GetMapping("/{transactionId}/history")
    @PreAuthorize("hasAuthority('SCOPE_transaction:audit')")
    public ResponseEntity<List<AdvancedAuditLogDto>> getTransactionAuditTimeline(@PathVariable String transactionId) {
        Revisions<Integer, Transaction> revisions = transactionRepository.findRevisions(transactionId);
        
        List<AdvancedAuditLogDto> records = revisions.stream().map(rev -> {
            CustomRevisionEntity meta = (CustomRevisionEntity) rev.getMetadata().getDelegate();
            return new AdvancedAuditLogDto(
                    rev.getRequiredRevisionNumber(),
                    meta.getUserId(),
                    meta.getClientIp(),
                    rev.getRequiredRevisionInstant().toString(),
                    rev.getEntity().getStatus()
            );
        }).collect(Collectors.toList());

        return ResponseEntity.ok(records);
    }
}
```

---

## 8. Automated Maintenance Infrastructure Layer

### Stale State Eviction Engine Scheduled Daemon (`WorkflowStateCleanupTask.java`)
```java
package com.example.scheduler;

import javax.sql.DataSource;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;
import java.sql.Connection;
import java.sql.PreparedStatement;
import java.time.Instant;
import java.time.temporal.ChronoUnit;

@Component
public class WorkflowStateCleanupTask {

    private final DataSource dataSource;

    public WorkflowStateCleanupTask(DataSource dataSource) {
        this.dataSource = dataSource;
    }

    /**
     * Purges orphaned workflow tracking matrices that have lingered for over 30 days.
     */
    @Scheduled(cron = "0 0 2 * * *") // Daily execution trigger at 02:00 AM
    public void purgeExpiredEmbabelStateInstances() {
        String query = "DELETE FROM embabel_workflow_state WHERE updated_at < ?";
        Instant threshold = Instant.now().minus(30, ChronoUnit.DAYS);
        
        try (Connection conn = dataSource.getConnection();
             PreparedStatement stmt = conn.prepareStatement(query)) {
            stmt.setObject(1, java.sql.Timestamp.from(threshold));
            int impactedRows = stmt.executeUpdate();
            // System.out.println("Sweeper complete. Cleaned up " + impactedRows + " abandoned workflows.");
        } catch (Exception e) {
            // Handle logging framework exceptions gracefully in real runtime contexts
        }
    }
}
```

---

## 9. Testcontainers Concurrency Validation Engine

```java
package com.example.service;

import com.example.domain.Transaction;
import com.example.repository.TransactionRepository;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.orm.ObjectOptimisticLockingFailureException;
import org.springframework.test.context.DynamicPropertyRegistry;
import org.springframework.test.context.DynamicPropertySource;
import org.testcontainers.containers.PostgreSQLContainer;
import org.testcontainers.junit.jupiter.Container;
import org.testcontainers.junit.jupiter.Testcontainers;

import java.math.BigDecimal;
import java.util.concurrent.*;
import java.util.concurrent.atomic.AtomicInteger;

import static org.assertj.core.api.Assertions.assertThat;

@SpringBootTest
@Testcontainers
class TransactionReviewConcurrencyIT {

    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:16-alpine")
            .withDatabaseName("transaction_db")
            .withUsername("postgres")
            .withPassword("secret");

    @DynamicPropertySource
    static void configureProperties(DynamicPropertyRegistry reg) {
        reg.add("spring.datasource.url", postgres::getJdbcUrl);
        reg.add("spring.datasource.username", postgres::getUsername);
        reg.add("spring.datasource.password", postgres::getPassword);
    }

    @Autowired private TransactionReviewService reviewService;
    @Autowired private TransactionRepository transactionRepository;

    private final String txId = "TX-CONCURRENT-X2";
    private final String wfId = "WF-INSTANCE-MOCK";

    @BeforeEach
    void setupSandboxData() {
        transactionRepository.deleteAll();
        Transaction tx = new Transaction();
        tx.setTransactionId(txId);
        tx.setAmount(new BigDecimal("15000.00"));
        tx.setStatus("PENDING_REVIEW");
        transactionRepository.saveAndFlush(tx);
    }

    @Test
    void runConcurrentHitlResolutionsToVerifyLockingAndRollbackBoundaries() throws InterruptedException {
        int workers = 2;
        ExecutorService executor = Executors.newFixedThreadPool(workers);
        CountDownLatch gate = new CountDownLatch(1);
        
        AtomicInteger passes = new AtomicInteger(0);
        AtomicInteger lockConflicts = new AtomicInteger(0);

        for (int i = 0; i < workers; i++) {
            final boolean decision = (i == 0);
            executor.submit(() -> {
                try {
                    gate.await();
                    reviewService.confirmTransaction(wfId, txId, decision);
                    passes.incrementAndGet();
                } catch (ObjectOptimisticLockingFailureException ex) {
                    lockConflicts.incrementAndGet();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            });
        }

        gate.countDown();
        executor.shutdown();
        boolean completed = executor.awaitTermination(5, TimeUnit.SECONDS);

        assertThat(completed).isTrue();
        assertThat(passes.get()).isEqualTo(1);
        assertThat(lockConflicts.get()).isEqualTo(1);

        Transaction finalRecord = transactionRepository.findById(txId).orElseThrow();
        assertThat(finalRecord.getVersion()).isEqualTo(1L);
    }
}
```

---

## 10. Local Development Sandbox Manifests

### Docker Composition Architecture Script (`docker-compose.yml`)
```yaml
version: '3.8'

services:
  postgres:
    image: postgres:16-alpine
    container_name: transaction_postgres
    environment:
      POSTGRES_DB: transaction_db
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: secret
    ports:
      - "5432:5432"
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres -d transaction_db"]
      interval: 5s
      timeout: 5s
      retries: 5

  prometheus:
    image: prom/prometheus:latest
    container_name: transaction_prometheus
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - ./alert.rules.yml:/etc/prometheus/alert.rules.yml
    ports:
      - "9090:9090"
```

### Telemetry Target Definition Scraper (`prometheus.yml`)
```yaml
global:
  scrape_interval: 15s

rule_files:
  - "alert.rules.yml"

scrape_configs:
  - job_name: 'springboot-app'
    metrics_path: '/actuator/prometheus'
    static_configs:
      - targets: ['host.docker.internal:8080']
```

### Operational Prometheus Threshold Guard (`alert.rules.yml`)
```yaml
groups:
  - name: transaction-platform-alerts
    rules:
      - alert: HighLockingCollisionRate
        expr: rate(platform_transactions_lock_conflicts_total[1m]) > 0.1
        for: 30s
        labels:
          severity: warning
        annotations:
          summary: "High concurrency update lock collision volume detected"
          description: "Multiple analysts are overlapping workflow review submissions on the same datasets simultaneously."
```
