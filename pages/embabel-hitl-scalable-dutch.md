# Embabel Agent & PostgreSQL State Persistence Architectuurgids
Dit document biedt een complete, end-to-end blauwdruk voor het implementeren van een stateless **Embabel Agent**-integratie binnen een **Spring Boot 3.x**-applicatie. Het elimineert in-memory state-trackers door gebruik te maken van een **PostgreSQL JDBC-backend**, biedt Human-in-the-Loop (HITL) workflows, garandeert data-integriteit via **Optimistic Locking**, registreert geavanceerde auditlogs met **Spring Data Envers** (inclusief OAuth2 JWT-identiteitsextractie), en bevat volledige telemetrie en integratietesten met **Testcontainers**.

---

## 1. Architectuuroverzicht & Database Schema

Door de runtime-state van de Embabel-workflow in PostgreSQL op te slaan, worden applicatie-instanties volledig stateless. Dit maakt horizontaal schalen over Kubernetes-clusters mogelijk zonder risico op state-verlies.

### Flyway Migratiescript (`V1__init_embabel_and_transactions.sql`)

```sql
-- Globale audit-revisietabel (Aangepast voor Envers met extra context)
CREATE TABLE custom_revinfo (
    rev INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    revtstmp BIGINT NOT NULL,
    user_id VARCHAR(50),
    client_ip VARCHAR(45)
);

-- Kernbedrijfstransactiestabel met Optimistic Locking-ondersteuning
CREATE TABLE transactions (
    transaction_id VARCHAR(50) PRIMARY KEY,
    amount DECIMAL(10, 2) NOT NULL,
    status VARCHAR(20) DEFAULT 'PENDING_REVIEW',
    reviewed_at TIMESTAMP WITH TIME ZONE,
    version BIGINT NOT NULL DEFAULT 0
);

-- Transactie-audittabel beheerd door Hibernate Envers
CREATE TABLE transactions_aud (
    transaction_id VARCHAR(50),
    rev INTEGER REFERENCES custom_revinfo(rev),
    revtype SMALLINT, -- 0 (ADD), 1 (MOD), 2 (DEL)
    amount DECIMAL(10, 2),
    status VARCHAR(20),
    reviewed_at TIMESTAMP WITH TIME ZONE,
    version BIGINT,
    PRIMARY KEY (transaction_id, rev)
);

-- Embabel workflow-state tabel (beheerd door de JDBC persistence backend)
CREATE TABLE embabel_workflow_state (
    instance_id VARCHAR(50) PRIMARY KEY,
    workflow_id VARCHAR(50) NOT NULL,
    current_step VARCHAR(50) NOT NULL,
    context_data JSONB,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

-- Indexen voor geoptimaliseerde runtime-lookups
CREATE INDEX idx_embabel_workflow_perf ON embabel_workflow_state(workflow_id, current_step);
```

---

## 2. Spring IoC Component & Security Configuratie

### Applicatie-instellingen (`application.yml`)

```yaml
spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/transaction_db
    username: postgres
    password: secret
    driver-class-name: org.postgresql.Driver
  jpa:
    hibernate:
      ddl-auto: validate
    properties:
      org.hibernate.envers.audit_table_suffix: _aud
  flyway:
    enabled: true
    locations: classpath:db/migration
  security:
    oauth2:
      resourceserver:
        jwt:
          issuer-uri: http://localhost:8080/realms/master

server:
  port: 8081
```

### Embabel & Audit Infrastructuur Beans (`EmbabelConfig.java`)

```java
package com.example.config;

import javax.sql.DataSource;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.envers.repository.config.EnableEnversRepositories;
import org.springframework.data.jpa.repository.config.EnableJpaRepositories;
import com.embabel.engine.EmbabelEngine;
import com.embabel.persistence.jdbc.JdbcStateRepository;

@Configuration
@EnableEnversRepositories
@EnableJpaRepositories(basePackages = "com.example.repository")
public class EmbabelConfig {

    @Bean
    public JdbcStateRepository jdbcStateRepository(DataSource dataSource) {
        // Koppelt de Embabel-persistentie rechtstreeks aan de Spring Boot DataSource
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

## 3. Domeinlaag & Geavanceerde Audit Logs

### JPA Entiteit met Locking en Auditing (`Transaction.java`)

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

    // Getters en Setters
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

### Custom Envers Revisie-entiteit (`CustomRevisionEntity.java`)

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

### OAuth2 & Servlet Context Aware Audit Listener (`UserAuditRevisionListener.java`)

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
        
        // 1. Extraheer Client IP uit de actieve HTTP-aanvraag
        ServletRequestAttributes attributes = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
        if (attributes != null) {
            HttpServletRequest request = attributes.getRequest();
            String ip = request.getHeader("X-Forwarded-For");
            if (ip == null || ip.isEmpty() || "unknown".equalsIgnoreCase(ip)) {
                ip = request.getRemoteAddr();
            }
            customRev.setClientIp(ip);
        } else {
            customRev.setClientIp("SYSTEM_BACKGROUND_JOB");
        }

        // 2. Extraheer Authentieke Gebruikers-ID uit de OAuth2 Security Context JWT
        Authentication auth = SecurityContextHolder.getContext().getAuthentication();
        if (auth != null && auth.getPrincipal() instanceof Jwt jwt) {
            String username = jwt.getClaimAsString("preferred_username");
            if (username == null) {
                username = jwt.getSubject();
            }
            customRev.setUserId(username);
        } else {
            customRev.setUserId("UNAUTHENTICATED_SYSTEM");
        }
    }
}
```

### Spring Data Spring Data Repositories

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

---

## 4. Embabel Agent & Human-in-the-Loop (HITL) Logica

### Onveranderlijk (Immutable) Beoordelingsrecord (`HumanReviewRecord.java`)

```java
package com.example.dto;

import java.math.BigDecimal;
import java.time.OffsetDateTime;

public record HumanReviewRecord(
    String transactionId,
    BigDecimal amount,
    String originalStatus,
    String workflowInstanceId,
    OffsetDateTime capturedAt
) {}
```

### Asynchrone Embabel Agent met `waitFor`-logica (`EmbabelAgentService.java`)

```java
package com.example.service;

import com.example.domain.Transaction;
import com.example.repository.TransactionRepository;
import com.example.exception.TransactionNotFoundException;
import com.example.exception.WorkflowStateException;
import org.springframework.scheduling.annotation.Async;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;
import com.embabel.engine.EmbabelEngine;
import com.embabel.context.WorkflowContext;

@Service
public class EmbabelAgentService {

    private final EmbabelEngine embabelEngine;
    private final TransactionRepository transactionRepository;

    public EmbabelAgentService(EmbabelEngine embabelEngine, TransactionRepository transactionRepository) {
        this.embabelEngine = embabelEngine;
        this.transactionRepository = transactionRepository;
    }

    @Async
    @Transactional
    public void executeTransactionAnalysisWorkflow(String transactionId, String instanceId) {
        Transaction transaction = transactionRepository.findById(transactionId)
                .orElseThrow(() -> new TransactionNotFoundException(transactionId));

        WorkflowContext context = embabelEngine.loadWorkflowContext(instanceId);

        // Geautomatiseerde evaluatiestappen van de agent...
        boolean isSuspicious = transaction.getAmount().compareTo(new java.math.BigDecimal("5000.00")) > 0;

        if (isSuspicious) {
            // Pauzeer de execution state engine en open een HITL-checkpoint via PostgreSQL JDBC
            context.waitFor("HUMAN_RISK_ANALYST_CONFIRMATION");
            embabelEngine.saveAndAdvance(context);
        } else {
            transaction.setStatus("AUTO_APPROVED");
            transactionRepository.save(transaction);
            context.completeStep("AUTO_PASS");
            embabelEngine.saveAndAdvance(context);
        }
    }
}
```

### Transaction Review Service (`TransactionReviewService.java`)

```java
package com.example.service;

import com.example.domain.Transaction;
import com.example.exception.TransactionNotFoundException;
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
        Transaction transaction = transactionRepository.findById(transactionId)
                .orElseThrow(() -> new TransactionNotFoundException(transactionId));
        
        WorkflowContext context;
        try {
            context = embabelEngine.loadWorkflowContext(instanceId);
        } catch (Exception e) {
            throw new WorkflowStateException("Kon workflow-state niet ophalen voor instance: " + instanceId, e);
        }
        
        HitlConfirmation confirmation = context.getHitlSignal();
        if (confirmation == null) {
            throw new WorkflowStateException("Geen actieve Human-In-The-Loop actie vereist voor instance: " + instanceId, null);
        }

        if (approved) {
            confirmation.approve("Handmatig goedgekeurd via risk-dashboard.");
            transaction.setStatus("APPROVED");
        } else {
            confirmation.reject("Handmatig afgewezen via risk-dashboard.");
            transaction.setStatus("REJECTED");
        }
        
        transaction.setReviewedAt(OffsetDateTime.now());
        transactionRepository.save(transaction); // Verhoogt @Version en triggert Envers audit log
        
        try {
            embabelEngine.saveAndAdvance(context); // Wast en beëindigt de workflow stap in DB
        } catch (Exception e) {
            throw new WorkflowStateException("Fout bij het opslaan van Embabel workflow-voortgang", e);
        }
    }
}
```

---

## 5. Web- & Telemetrielaag (REST Controller, Exception Handling & Micrometer)

### REST API Gateway endpoints (`HitlTransactionController.java`)

```java
package com.example.controller;

import com.example.service.TransactionReviewService;
import org.springframework.http.ResponseEntity;
import org.springframework.security.access.prepost.PreAuthorize;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/v1/hitl/transactions")
public class HitlTransactionController {

    private final TransactionReviewService reviewService;

    public HitlTransactionController(TransactionReviewService reviewService) {
        this.reviewService = reviewService;
    }

    @PostMapping("/{transactionId}/confirm")
    @PreAuthorize("hasAuthority('SCOPE_transaction:review')")
    public ResponseEntity<Void> confirm(
            @PathVariable String transactionId,
            @RequestBody HitlDecisionRequest request) {
        
        reviewService.confirmTransaction(request.workflowInstanceId(), transactionId, request.approved());
        return ResponseEntity.ok().build();
    }
}

record HitlDecisionRequest(String workflowInstanceId, boolean approved) {}
```

### Global Controller Exception Handler met Micrometer Telemetrie (`GlobalExceptionHandler.java`)

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
        // Registreert een custom teller voor Prometheus-metrische monitoring
        this.lockingConflictCounter = Counter.builder("platform.transactions.lock.conflicts")
                .description("Aantal gelijktijdige conflicten gedetecteerd tijdens HITL-validaties")
                .register(meterRegistry);
    }

    @ExceptionHandler(ObjectOptimisticLockingFailureException.class)
    public ProblemDetail handleOptimisticLockingFailure(ObjectOptimisticLockingFailureException ex) {
        lockingConflictCounter.increment(); // Verhoog teller voor alerting doeleinden
        
        ProblemDetail problemDetail = ProblemDetail.forStatusAndDetail(
                HttpStatus.CONFLICT, 
                "Deze transactie is gewijzigd door een andere behandelaar of proces."
        );
        problemDetail.setTitle("Gelijktijdige Wijziging Gedetecteerd");
        problemDetail.setProperty("timestamp", Instant.now());
        return problemDetail;
    }

    @ExceptionHandler(TransactionNotFoundException.class)
    public ProblemDetail handleNotFound(TransactionNotFoundException ex) {
        return ProblemDetail.forStatusAndDetail(HttpStatus.NOT_FOUND, ex.getMessage());
    }

    @ExceptionHandler(WorkflowStateException.class)
    public ProblemDetail handleWorkflowStateError(WorkflowStateException ex) {
        return ProblemDetail.forStatusAndDetail(HttpStatus.BAD_REQUEST, ex.getMessage());
    }
}
```

---

## 6. Automatische Dead-Letter Opschoon-Worker

Voorkom database-vervuiling door afgebroken of wegrottende workflows periodiek te verwijderen uit `embabel_workflow_state`.

```java
package com.example.worker;

import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;
import org.springframework.transaction.annotation.Transactional;

@Component
public class WorkflowMaintenanceWorker {

    private final JdbcTemplate jdbcTemplate;

    public WorkflowMaintenanceWorker(JdbcTemplate jdbcTemplate) {
        this.jdbcTemplate = jdbcTemplate;
    }

    // Draait elke nacht om 02:00 uur
    @Scheduled(cron = "0 0 2 * * ?")
    @Transactional
    public void purgeAbandonedWorkflowStates() {
        String SQL = "DELETE FROM embabel_workflow_state WHERE updated_at < NOW() - INTERVAL '30 days'";
        int deletedRows = jdbcTemplate.update(SQL);
        // Log actie via SLF4J logger...
    }
}
```

---

## 7. Concurrency Integratietest met Testcontainers

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
class TransactionReviewServiceConcurrencyIT {

    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:16-alpine")
            .withDatabaseName("transaction_db")
            .withUsername("postgres")
            .withPassword("secret");

    @DynamicPropertySource
    static void configureProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", postgres::getJdbcUrl);
        registry.add("spring.datasource.username", postgres::getUsername);
        registry.add("spring.datasource.password", postgres::getPassword);
    }

    @Autowired
    private TransactionReviewService reviewService;

    @Autowired
    private TransactionRepository transactionRepository;

    @BeforeEach
    void setup() {
        transactionRepository.deleteAll();
        Transaction initialTx = new Transaction();
        initialTx.setTransactionId("TX-100");
        initialTx.setAmount(new BigDecimal("6000.00"));
        initialTx.setStatus("PENDING_REVIEW");
        transactionRepository.saveAndFlush(initialTx);
    }

    @Test
    void testConcurrentHitlReviewsTriggersOptimisticLockingException() throws InterruptedException {
        int threadCount = 2;
        ExecutorService executor = Executors.newFixedThreadPool(threadCount);
        CountDownLatch latch = new CountDownLatch(1);
        
        AtomicInteger successCounter = new AtomicInteger(0);
        AtomicInteger conflictCounter = new AtomicInteger(0);

        for (int i = 0; i < threadCount; i++) {
            executor.submit(() -> {
                try {
                    latch.await(); // Synchroniseer start van beide threads
                    reviewService.confirmTransaction("WF-INSTANCE-123", "TX-100", true);
                    successCounter.incrementAndGet();
                } catch (ObjectOptimisticLockingFailureException ex) {
                    conflictCounter.incrementAndGet();
                } catch (Exception e) {
                    // Fail-safe handling
                }
            });
        }

        latch.countDown(); // Start de threads tegelijkertijd
        executor.shutdown();
        executor.awaitTermination(5, TimeUnit.SECONDS);

        assertThat(successCounter.get()).isEqualTo(1);  // Slechts 1 behandelaar mag slagen
        assertThat(conflictCounter.get()).isEqualTo(1); // De 2e moet falen met een lock-conflict
    }
}
```

---

## 8. Sandbox Productieomgeving & Prometheus Alerting Rules

### `docker-compose.yml`

```yaml
version: '3.8'
services:
  postgres:
    image: postgres:16-alpine
    container_name: local_postgres
    environment:
      POSTGRES_DB: transaction_db
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: secret
    ports:
      - "5432:5432"

  prometheus:
    image: prom/prometheus:latest
    container_name: local_prometheus
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - ./alert.rules.yml:/etc/prometheus/alert.rules.yml
    ports:
      - "9090:9090"
```

### Prometheus Alerting Rules (`alert.rules.yml`)

```yaml
groups:
  - name: transaction_alert_rules
    rules:
      - alert: HighLockingConflictsDetected
        expr: rate(platform_transactions_lock_conflicts_total[5m]) > 2
        for: 2m
        labels:
          severity: warning
        annotations:
          summary: "Te veel gelijktijdige updates gedetecteerd op transactie-data"
          description: "De applicatie ondervindt meer dan 2 locking-conflicten per minuut over de afgelopen 5 minuten. Dit kan duiden op dubbele dashboard-behandelingen."
```
