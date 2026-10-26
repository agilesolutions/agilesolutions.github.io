# GitHub Copilot – Quickstart User Guide

A practical guide for using GitHub Copilot as an AI-assisted software engineering partner, with examples focused on Java, Spring Boot, Kubernetes, Terraform, GitOps and solution architecture.

## Table of Contents

- [1. What is GitHub Copilot?](#1-what-is-github-copilot)
- [2. Recommended Setup](#2-recommended-setup)
- [3. First Interaction](#3-first-interaction)
- [4. Code Completion](#4-code-completion)
- [5. Copilot Chat](#5-copilot-chat)
- [6. Effective Prompting](#6-effective-prompting)
- [7. Repository Analysis](#7-repository-analysis)
- [8. Testing](#8-testing)
- [9. Debugging](#9-debugging)
- [10. Refactoring](#10-refactoring)
- [11. Solution Architecture](#11-solution-architecture)
- [12. Spring Boot](#12-spring-boot)
- [13. Kubernetes](#13-kubernetes)
- [14. Terraform](#14-terraform)
- [15. Copilot Agent Workflow](#15-copilot-agent-workflow)
- [16. Repository Instructions](#16-repository-instructions)
- [17. Daily Development Workflow](#17-daily-development-workflow)
- [18. Golden Rules](#18-golden-rules)
- [19. Official References](#19-official-references)

---

## 1. What is GitHub Copilot?

GitHub Copilot is an AI coding assistant that can help with:

- Code generation
- Inline code completion
- Code explanation
- Refactoring
- Test generation
- Debugging
- Documentation
- Repository analysis
- Pull-request assistance
- Command-line assistance
- Agentic software-development workflows

The most effective use is not simply asking Copilot to "write code", but using it as an engineering pair:

```text
Understand → Analyze → Plan → Implement → Test → Review → Refactor
```

---

## 2. Recommended Setup

For Java/Spring Boot development:

```text
GitHub
   │
   └── Repository
          │
          ├── Java
          ├── Spring Boot
          ├── Gradle
          ├── Tests
          └── Documentation
                 │
                 ▼
          GitHub Copilot
                 │
        ┌────────┼────────┐
        ▼        ▼        ▼
      Chat   Completion  Agent
        │        │        │
        ▼        ▼        ▼
     Explain   Generate   Implement
```

Recommended IDE options include:

- IntelliJ IDEA
- Visual Studio Code
- Visual Studio
- Eclipse

For Java/Spring Boot development, IntelliJ IDEA with the GitHub Copilot plugin is a strong combination.

### IntelliJ installation

```text
Settings
  → Plugins
  → Marketplace
  → GitHub Copilot
  → Install
```

Then sign in with GitHub and authorize Copilot.

---

## 3. First Interaction

Open a Java class and use Copilot Chat.

Start with:

```text
Explain this class.
```

Then progressively ask:

```text
What are the responsibilities of this class?
```

```text
Identify potential design problems.
```

```text
Suggest improvements without changing the public API.
```

This provides much better results than starting with a generic:

```text
Write some Java code.
```

---

## 4. Code Completion

Copilot can generate code while you type.

Example:

```java
@Service
public class CustomerService {

    private final CustomerRepository repository;

    public CustomerService(CustomerRepository repository) {
        this.repository = repository;
    }

    public Optional<Customer> findById(Long id) {
```

Copilot can suggest the implementation.

In supported IDE integrations, inline suggestions can generally be accepted with `Tab`.

---

## 5. Copilot Chat

Useful commands include:

```text
Explain this class.
```

```text
Explain how this service interacts with the rest of the application.
```

```text
Find potential bugs in this implementation.
```

```text
Suggest unit tests for this class.
```

```text
Refactor this method while preserving its behavior.
```

```text
Explain the Spring annotations used here.
```

```text
Review this code for maintainability problems.
```

---

## 6. Effective Prompting

One of the most important Copilot skills is providing sufficient context.

Instead of:

```text
Create a controller.
```

provide:

```text
Create a Spring Boot REST controller for Customer.

Requirements:
- Java 25
- Spring Boot 4
- constructor injection
- RESTful API
- Bean Validation
- ProblemDetail for errors
- service layer
- no business logic in controller
- write JUnit 5 tests
```

### Recommended prompt structure

```text
CONTEXT
What are we working on?

GOAL
What do I want?

CONSTRAINTS
What must / must not happen?

TECHNOLOGY
Which versions and frameworks?

DESIGN
Which architectural principles apply?

OUTPUT
What exactly should Copilot produce?

VALIDATION
How should the result be verified?
```

### Example

```text
Context:
This is a Spring Boot REST microservice.

Goal:
Implement a customer creation endpoint.

Technology:
Java 25
Spring Boot 4
Spring Framework 7
PostgreSQL
JPA
Flyway
JUnit 5
Testcontainers

Constraints:
- constructor injection
- no business logic in controller
- use DTOs
- Bean Validation
- return ProblemDetail for errors
- transactional service layer

Output:
Generate:
1. Controller
2. DTO
3. Service
4. Repository
5. Entity
6. Exception handling
7. Unit tests
8. Integration tests

Validation:
Make sure the generated code compiles and tests cover validation
and duplicate customers.
```

---

## 7. Repository Analysis

For an existing repository, do not immediately ask Copilot to modify files.

Start with analysis:

```text
Analyze this repository.

Identify:
- application architecture
- main modules
- REST APIs
- persistence
- messaging
- external integrations
- security
- testing strategy
- configuration
- deployment architecture

Do not modify any files.
```

Then:

```text
Create a concise architecture summary based on your analysis.
```

Next:

```text
Identify areas that could cause maintainability problems.
Do not modify anything.
```

Then:

```text
Propose a refactoring plan.
Do not implement it yet.
```

Finally:

```text
Implement step 1 of the approved refactoring plan.
```

### Recommended workflow

```text
Analyze
   ↓
Plan
   ↓
Review plan
   ↓
Implement
   ↓
Test
   ↓
Review diff
```

---

## 8. Testing

### Unit tests

```text
Generate comprehensive JUnit 5 tests for this service.

Cover:
- happy path
- validation
- not found
- duplicate entity
- repository failure
- boundary conditions

Use Mockito where appropriate.
Do not test implementation details.
```

### Integration tests

```text
Create a Spring Boot integration test using Testcontainers
with PostgreSQL.

Use:
- JUnit 5
- @SpringBootTest
- Testcontainers
- Flyway

Verify the complete persistence flow.
```

A good principle is:

```text
Unit tests
    ↓
Fast feedback

Integration tests
    ↓
Real infrastructure behavior

End-to-end tests
    ↓
System behavior
```

---

## 9. Debugging

Avoid vague questions such as:

```text
Why doesn't this work?
```

Provide evidence:

```text
This Spring Boot application fails during startup.

Exception:
[paste exception]

Expected:
Application starts normally.

Environment:
Java 25
Spring Boot 4
PostgreSQL

Relevant class:
[paste class]

Configuration:
[paste relevant configuration]

Analyze the root cause.
Do not propose changes until you have identified
the likely cause.
```

Then:

```text
Give me the smallest safe fix.
```

Finally:

```text
Generate a regression test that would have caught this problem.
```

---

## 10. Refactoring

Use a controlled refactoring workflow:

```text
1. Explain
       ↓
2. Identify problems
       ↓
3. Propose alternatives
       ↓
4. Select approach
       ↓
5. Implement
       ↓
6. Run tests
       ↓
7. Review diff
       ↓
8. Commit
```

Example:

```text
Analyze this class for SOLID violations.
Do not change the code.
```

Then:

```text
Propose a refactoring that improves SRP while keeping
the external API unchanged.
```

Then:

```text
Implement the refactoring.
```

Then:

```text
Generate tests that demonstrate that behavior has not changed.
```

---

## 11. Solution Architecture

Copilot can be particularly useful as an architecture review assistant.

Example:

```text
Analyze this repository from a solution architecture perspective.

Evaluate:
- bounded contexts
- service boundaries
- coupling
- cohesion
- dependency direction
- synchronous communication
- asynchronous communication
- persistence ownership
- security boundaries
- observability
- resilience
- scalability
- deployment topology

Return:
1. Current architecture
2. Architectural risks
3. Recommendations
4. Priority
5. Suggested target architecture
```

You can also ask:

```text
Generate a C4 Container diagram using Mermaid.
```

or:

```text
Generate a C4 Component diagram for this Spring Boot service.
```

### Architecture review checklist

```text
Architecture
├── Bounded contexts
├── Service boundaries
├── Dependencies
├── Coupling / cohesion
├── Data ownership
├── APIs
├── Messaging
├── Security
├── Resilience
├── Scalability
├── Observability
└── Deployment topology
```

---

## 12. Spring Boot

Useful prompts:

```text
Explain the Spring Boot application startup sequence.
```

```text
Review this Spring configuration for unnecessary complexity.
```

```text
Identify incorrect Spring bean scopes.
```

```text
Review this application for circular dependencies.
```

```text
Review this REST API against REST principles.
```

```text
Review this Spring Security configuration.
```

```text
Identify possible transaction-boundary problems.
```

```text
Review this code for N+1 query problems.
```

```text
Review this code for improper use of @Transactional.
```

### Recommended Spring architecture

```text
REST Controller
       │
       ▼
Application / Service Layer
       │
       ▼
Domain Logic
       │
       ├──────────► Repository
       │
       ├──────────► External API
       │
       └──────────► Messaging
```

Keep controllers thin and keep business rules outside controllers.

---

## 13. Kubernetes

Copilot can review Kubernetes manifests and Helm charts.

Example:

```text
Review this Kubernetes deployment.

Check:
- resource requests
- resource limits
- probes
- securityContext
- topologySpreadConstraints
- PodDisruptionBudget
- rolling update strategy
- service account
- container security
- configuration
- secrets

Identify problems and recommend improvements.
```

For Helm:

```text
Review this Helm chart against Kubernetes production best practices.
```

For FluxCD:

```text
Generate a FluxCD HelmRelease for this Helm chart.

Requirements:
- FluxCD
- HelmRepository
- namespace isolation
- configurable values
- health checks
```

### Kubernetes review areas

```text
Deployment
├── Resources
├── Probes
├── SecurityContext
├── Scheduling
├── Availability
├── Scaling
├── Configuration
├── Secrets
├── Networking
└── Observability
```

---

## 14. Terraform

Use Copilot to review infrastructure code:

```text
Review this Terraform module.

Check:
- module boundaries
- variables
- outputs
- sensitive values
- state management
- dependency handling
- provider configuration
- naming
- lifecycle
- idempotency

Do not modify anything.
```

Then:

```text
Propose improvements.
```

And:

```text
Implement only the approved improvements.
```

### Terraform review principles

```text
Root module
   │
   ├── Platform modules
   │
   ├── Infrastructure modules
   │
   └── Service modules
```

Keep modules focused and avoid unnecessary coupling between infrastructure components.

---

## 15. Copilot Agent Workflow

Modern Copilot workflows can go beyond autocomplete and chat. Depending on the Copilot environment and enabled features, agentic workflows can research a repository, create a plan, modify code and prepare changes for review.

A useful workflow is:

```text
Issue
  │
  ▼
Copilot investigates repository
  │
  ▼
Copilot creates plan
  │
  ▼
Developer reviews plan
  │
  ▼
Copilot implements
  │
  ▼
Tests
  │
  ▼
Review diff
  │
  ▼
Pull Request
  │
  ▼
Human review
```

The key principle:

> Let Copilot accelerate implementation, but keep architectural decisions and final code review human-controlled.

---

## 16. Repository Instructions

For serious projects, create project-specific Copilot instructions.

A common location is:

```text
.github/
└── copilot-instructions.md
```

Example:

```markdown
# Project Coding Guidelines

## Java

- Use Java 25.
- Prefer records for immutable DTOs.
- Use constructor injection.
- Do not use field injection.
- Prefer immutable objects.

## Spring Boot

- Use Spring Boot 4.
- Controllers must not contain business logic.
- Business logic belongs in services.
- Use ProblemDetail for REST errors.

## Testing

- Use JUnit 5.
- Use Testcontainers for integration tests.
- Do not mock repositories in integration tests.
- Maintain meaningful test coverage.

## Architecture

- Follow hexagonal architecture.
- Keep domain independent from infrastructure.
- Do not introduce unnecessary abstractions.

## Observability

- Use Micrometer/OpenTelemetry.
- Include meaningful metrics and tracing.
- Do not log secrets or personal data.

## Security

- Use OAuth2/OIDC.
- Never hard-code credentials.
- Never commit secrets.
```

Project instructions reduce the need to repeat the same requirements in every prompt.

---

## 17. Daily Development Workflow

A practical development workflow:

```text
Morning
   │
   ├── Ask Copilot to summarize current work
   │
   ├── Investigate issue
   │
   ├── Ask for implementation plan
   │
   ├── Review plan
   │
   ├── Implement
   │
   ├── Generate/update tests
   │
   ├── Run tests
   │
   ├── Ask Copilot to review the diff
   │
   └── Create PR
```

Before merging:

```text
Review this change as a senior Java/Spring Boot architect.

Check:
- correctness
- maintainability
- security
- performance
- resilience
- observability
- test quality
- API compatibility
- architectural consistency

Do not change anything.
Return findings grouped by severity.
```

---

## 18. Golden Rules

### 1. Do not blindly accept generated code

Copilot generates suggestions. It does not guarantee correctness.

### 2. Give Copilot context

Bad:

```text
Create a service.
```

Better:

```text
Create a Spring Boot service using Java 25,
constructor injection, PostgreSQL and JUnit 5.
Keep business logic outside the controller.
```

### 3. Separate planning from implementation

Prefer:

```text
Analyze → Plan → Review → Implement
```

rather than immediately generating code.

### 4. Ask for tests

Whenever Copilot changes behavior:

```text
Generate tests that verify the intended behavior.
```

### 5. Review the diff

Always inspect:

```text
git diff
```

before committing AI-generated changes.

### 6. Protect secrets

Never provide Copilot with:

- passwords
- API keys
- private keys
- access tokens
- credentials
- unnecessary personal data

### 7. Keep humans responsible for architecture

Copilot can suggest architectures, but the developer/architect remains responsible for:

- architectural decisions
- security
- compliance
- correctness
- operational consequences
- production readiness

---

## 19. Official References

- GitHub Copilot documentation: https://docs.github.com/en/copilot
- GitHub Copilot quickstart: https://docs.github.com/en/copilot/get-started/quickstart
- GitHub Copilot setup: https://docs.github.com/en/copilot/how-tos/set-up/set-up-for-self
- GitHub Copilot overview: https://docs.github.com/en/copilot/get-started/what-is-github-copilot

---

## Recommended Mental Model

Use GitHub Copilot as:

```text
                 ┌──────────────────────┐
                 │     SOFTWARE TEAM    │
                 └──────────┬───────────┘
                            │
                 Human architecture
                            │
                            ▼
                 ┌──────────────────────┐
                 │   GitHub Copilot     │
                 │                      │
                 │ Analyze              │
                 │ Explain              │
                 │ Plan                 │
                 │ Generate             │
                 │ Refactor             │
                 │ Test                 │
                 │ Review               │
                 └──────────┬───────────┘
                            │
                            ▼
                 ┌──────────────────────┐
                 │ Human verification   │
                 │                      │
                 │ Architecture         │
                 │ Security             │
                 │ Tests                │
                 │ Code review          │
                 │ Production readiness │
                 └──────────────────────┘
```

**Best practice:** treat Copilot as a highly productive engineering pair, not as an autonomous replacement for software engineering judgment.
