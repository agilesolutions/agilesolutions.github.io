# Embabel HITL & Transaction Review System
### Spring Boot 4.0 | Spring Framework 7.0 | Jakarta EE 11 Baseline

This blueprint provides a production-grade, stateless architecture for managing Human-in-the-Loop (HITL) transaction review workflows using **Embabel Agent core features**.

By replacing in-memory state tracking with a **PostgreSQL JDBC backend**, this solution ensures seamless horizontal scaling across application instances. It features race-condition safety via **Optimistic Locking**, automated audit trails via **Spring Data Envers** initialized with an OAuth2 JWT principal identity listener, and real-time operational monitoring with **Micrometer**.

---

## 1. Dependency Management & Build Files

### `gradle/libs.versions.toml` (Version Catalog)
```toml
[versions]
springBoot = "4.0.0"
springFramework = "7.0.0"
jakartaEe = "11.0.0"
hibernate = "7.0.0.Beta1"
postgresql = "42.7.2"
embabel = "2.1.0"
testcontainers = "1.20.1"

[plugins]
springBoot = { id = "org.springframework.boot", version.ref = "springBoot" }
springDependencyManagement = { id = "io.spring.dependency-management", version = "1.1.4" }
java = { id = "org.java" }

[libraries]
spring-boot-starter-web = { module = "org.springframework.boot:spring-boot-starter-web" }
spring-boot-starter-data-jpa = { module = "org.springframework.boot:spring-boot-starter-data-jpa" }
spring-boot-starter-security = { module = "org.springframework.boot:spring-boot-starter-security" }
spring-boot-starter-oauth2-resource-server = { module = "org.springframework.boot:spring-boot-starter-oauth2-resource-server" }
spring-boot-starter-actuator = { module = "org.springframework.boot:spring-boot-starter-actuator" }
micrometer-registry-prometheus = { module = "io.micrometer:micrometer-registry-prometheus" }

flyway-core = { module = "org.flywaydb:flyway-core" }
flyway-database-postgresql = { module = "org.flywaydb:flyway-database-postgresql" }
postgresql = { module = "org.postgresql:postgresql", version.ref = "postgresql" }
hibernate-envers = { module = "org.hibernate.orm:hibernate-envers", version.ref = "hibernate" }
spring-data-envers = { module = "org.springframework.data:spring-data-envers" }

embabel-core = { module = "com.embabel:embabel-core", version.ref = "embabel" }
embabel-persistence-jdbc = { module = "com.embabel:embabel-persistence-jdbc", version.ref = "embabel" }

# Testing
spring-boot-starter-test = { module = "org.springframework.boot:spring-boot-starter-test" }
testcontainers-junit-jupiter = { module = "org.testcontainers:junit-jupiter", version.ref = "testcontainers" }
testcontainers-postgresql = { module = "org.testcontainers:postgresql", version.ref = "testcontainers" }
```

### `build.gradle` (Groovy Syntax)
```groovy
plugins {
    alias(libs.plugins.springBoot)
    alias(libs.plugins.springDependencyManagement)
    id 'java'
}

group = 'com.example'
version = '1.0.0-SNAPSHOT'
sourceCompatibility = JavaVersion.VERSION_21

repositories {
    mavenCentral()
}

dependencies {
    // Spring Boot & Jakarta Baseline
    implementation libs.spring.boot.starter.web
    implementation libs.spring.boot.starter.data-jpa
    implementation libs.spring.boot.starter.security
    implementation libs.spring.boot.starter.oauth2.resource-server
    implementation libs.spring.boot.starter.actuator
    implementation libs.micrometer.registry.prometheus

    // Persistency & Audit Trailing
    implementation libs.flyway.core
    implementation libs.flyway.database.postgresql
    runtimeOnly libs.postgresql
    implementation libs.hibernate.envers
    implementation libs.spring.data.envers

    // Embabel Platform SDK
    implementation libs.embabel.core
    implementation libs.embabel.persistence.jdbc

    // Testing Scope
    testImplementation libs.spring.boot.starter.test
    testImplementation libs.testcontainers.junit-jupiter
    testImplementation libs.testcontainers.postgresql
}

test {
    useJUnitPlatform()
}
```

---

## 2. Infrastructure Setup & Configurations

### `src/main/resources/application.yml`
```yaml
spring:
  application:
    name: transaction-review-service
  datasource:
    url: ${DATABASE_URL:jdbc:postgresql://localhost:5432/transaction_db}
    username: ${DATABASE_USER:postgres}
    password: ${DATABASE_PASSWORD:secret}
    driver-class-name: org.postgresql.Driver
  
  flyway:
    enabled: true
    baseline-on-migrate: true
    locations: classpath:db/migration

  security:
    oauth2:
      resourceserver:
        jwt:
          issuer-uri: ${OAUTH2_ISSUER:http://localhost:8080/realms/platform}

management:
  endpoints:
    web:
      exposure:
        include: health, info, prometheus
```

### Flyway Schema Migration (`V1__init_architecture.sql`)
```sql
-- Core business transactions table
CREATE TABLE transactions (
    transaction_id VARCHAR(50) PRIMARY KEY,
    amount DECIMAL(10, 2) NOT NULL,
    status VARCHAR(20) DEFAULT 'PENDING_REVIEW',
    reviewed_at TIMESTAMP WITH TIME ZONE,
    version BIGINT NOT NULL DEFAULT 0
);

-- Custom global revision tracker table for Envers
CREATE TABLE custom_revinfo (
    rev INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    revtstmp BIGINT NOT NULL,
    user_id VARCHAR(50),
    client_ip VARCHAR(45)
);

-- Transaction audit mirror table matching Envers naming specifications
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

-- Embabel execution state table managed by JDBC persistence layer
CREATE TABLE embabel_workflow_state (
    instance_id VARCHAR(50) PRIMARY KEY,
    workflow_id VARCHAR(50) NOT NULL,
    current_step VARCHAR(50) NOT NULL,
    context_data JSONB,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);
```

---

## 3. Configuration & Platform Core Setup

### `com.example.config.EmbabelConfig`
```java
package com.example.config;

import javax.sql.DataSource;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.envers.repository.config.EnableEnversRepositories;
import org.springframework.data.jpa.repository.config.EnableJpaRepositories;
import org.springframework.scheduling.annotation.EnableAsync;
import org.springframework.scheduling.annotation.EnableScheduling;
import com.embabel.engine.EmbabelEngine;
import com.embabel.persistence.jdbc.JdbcStateRepository;

@Configuration
@EnableAsync
@EnableScheduling
@EnableEnversRepositories
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

### `com.example.config.SecurityConfig`
```java
package com.example.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.web.SecurityFilterChain;

@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            // Spring Security 7.0 mandates manual handling for Stateless API endpoints
            .csrf(csrf -> csrf.disable())
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/management/**", "/actuator/**").permitAll()
                .requestMatchers("/api/v1/hitl/transactions/**").hasAuthority("SCOPE_transaction:review")
                .requestMatchers("/api/v1/audit/**").hasAuthority("SCOPE_transaction:audit")
                .anyRequest().authenticated()
            )
            .oauth2ResourceServer(oauth2 -> oauth2.jwt(jwt -> {}));
            
        return http.build();
    }
}
```

---

## 4. Advanced Audit Trailing & Audit Models

### `com.example.audit.CustomRevisionEntity`
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

### `com.example.audit.UserAuditRevisionListener`
```java
package com.example.audit;

import jakarta.servlet.http.HttpServletRequest;
import org.hibernate.envers.RevisionListener;
import org.springframework.security.core.Authentication;
import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.security.oauth2.jwt.Jwt;
import org.springframework.web.context.request.RequestContextHolder;
import org.springframework.web.context.request.ServletRequestAttributes;

