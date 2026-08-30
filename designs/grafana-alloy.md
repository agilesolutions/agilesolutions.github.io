# Spring Boot 4 Microservices — OpenTelemetry / Grafana Observability Architecture

```mermaid
flowchart TB

    %% ============================================================
    %% APPLICATION LAYER
    %% ============================================================

    subgraph APP["Application Layer — Spring Boot 4 Microservices"]

        MS1["Spring Boot 4<br/>Vergunning Service"]
        MS2["Spring Boot 4<br/>Zakenregistratie Service"]
        MS3["Spring Boot 4<br/>Other Microservice"]

        MS1 -->|OTLP| OTEL1["OpenTelemetry SDK<br/>Micrometer / OTEL"]
        MS2 -->|OTLP| OTEL2["OpenTelemetry SDK<br/>Micrometer / OTEL"]
        MS3 -->|OTLP| OTEL3["OpenTelemetry SDK<br/>Micrometer / OTEL"]

    end


    %% ============================================================
    %% OBSERVABILITY COLLECTION
    %% ============================================================

    subgraph COLLECT["Telemetry Collection Layer"]

        ALLOY["Grafana Alloy<br/><br/>OpenTelemetry Collector<br/>OTLP Receiver"]

    end

    OTEL1 -->|OTLP/gRPC<br/>4317| ALLOY
    OTEL2 -->|OTLP/gRPC<br/>4317| ALLOY
    OTEL3 -->|OTLP/gRPC<br/>4317| ALLOY


    %% ============================================================
    %% GRAFANA OBSERVABILITY BACKENDS
    %% ============================================================

    subgraph BACKENDS["Grafana Observability Backends"]

        MIMIR["Grafana Mimir<br/><br/>Metrics"]
        TEMPO["Grafana Tempo<br/><br/>Distributed Traces"]
        LOKI["Grafana Loki<br/><br/>Logs"]

    end


    ALLOY -->|Metrics<br/>Prometheus / OTLP| MIMIR
    ALLOY -->|Traces<br/>OTLP| TEMPO
    ALLOY -->|Logs<br/>OTLP| LOKI


    %% ============================================================
    %% GRAFANA
    %% ============================================================

    subgraph GRAFANA["Grafana Visualization & Alerting"]

        GF["Grafana"]

        DASH["Dashboards<br/><br/>• JVM<br/>• HTTP<br/>• Business<br/>• Kubernetes<br/>• Infrastructure"]

        ALERT["Alerting<br/><br/>• SLO / SLA<br/>• Error Rate<br/>• Latency<br/>• Availability<br/>• Resource Usage"]

        GF --> DASH
        GF --> ALERT

    end


    MIMIR -->|PromQL| GF
    TEMPO -->|TraceQL| GF
    LOKI -->|LogQL| GF


    %% ============================================================
    %% USERS / OPERATIONS
    %% ============================================================

    subgraph USERS["Consumers"]

        DEV["Developer"]
        OPS["Operations"]
        ARCH["Solution Architect"]

    end

    DEV -->|Investigate<br/>application telemetry| GF
    OPS -->|Monitor / Respond<br/>to alerts| GF
    ARCH -->|Architecture / SLO<br/>observability| GF


    %% ============================================================
    %% CORRELATION
    %% ============================================================

    MS1 -.->|trace_id / span_id| MS2
    MS2 -.->|trace_id / span_id| MS3

    TEMPO -.->|Trace correlation| GF
    LOKI -.->|Trace ID correlation| TEMPO
    MIMIR -.->|Exemplars / Trace ID| TEMPO
```

---

## Telemetry flow

The overall flow is:

```text
Spring Boot 4
     │
     │ OpenTelemetry
     │ OTLP
     ▼
Grafana Alloy
     │
     ├──────────────► Mimir
     │                  │
     │                  └── Metrics
     │
     ├──────────────► Tempo
     │                  │
     │                  └── Traces
     │
     └──────────────► Loki
                        │
                        └── Logs

             Mimir ─────┐
             Tempo ─────┼──► Grafana
             Loki ──────┘
                           │
                    ┌──────┴──────┐
                    ▼             ▼
                Dashboards      Alerts
```

## Spring Boot 4 observability responsibilities

Each microservice should expose telemetry through the application instrumentation layer:

```text
Spring Boot 4
     │
     ├── Micrometer
     │      │
     │      └── Metrics
     │
     └── OpenTelemetry
            │
            ├── Traces
            ├── Logs
            └── Metrics
```

For distributed tracing, propagate the trace context between services:

```text
Vergunning Service
        │
        │ HTTP / REST
        │ traceparent
        ▼
Zakenregistratie Service
        │
        │ HTTP / REST
        │ traceparent
        ▼
Other Service
```

This allows Grafana/Tempo to reconstruct the complete distributed transaction.

---

## Grafana's role

Grafana is **not the telemetry collector or primary telemetry store** in this architecture.

Its role is primarily:

```text
                  Grafana
                     │
       ┌─────────────┼─────────────┐
       ▼             ▼             ▼
     Mimir          Tempo         Loki
    Metrics        Traces         Logs
       │             │             │
       └─────────────┼─────────────┘
                     ▼
              Unified View
```

This gives operators a single place to correlate:

**Metrics → Logs → Traces**

For example:

```text
HTTP 500 spike
     │
     ▼
Mimir
     │
     │ investigate time range
     ▼
Grafana
     │
     ▼
Trace
     │
     ▼
Tempo
     │
     │ trace_id
     ▼
Logs
     │
     ▼
Loki
```

---

## Recommended architecture principle

The important separation of responsibilities is:

| Component     | Responsibility                          |
| ------------- | --------------------------------------- |
| Spring Boot   | Generate application telemetry          |
| Micrometer    | Application metrics / observation       |
| OpenTelemetry | Telemetry standard and instrumentation  |
| Grafana Alloy | Collect, process and route telemetry    |
| Mimir         | Long-term metrics backend               |
| Tempo         | Distributed tracing backend             |
| Loki          | Log aggregation backend                 |
| Grafana       | Visualization, correlation and alerting |

This is a good **cloud-native observability architecture** because the application services remain largely independent of the actual observability storage implementation.

For example:

```text
Spring Boot
     │
     │ OTLP
     ▼
   Alloy
     │
     ├── Mimir
     ├── Tempo
     └── Loki
```

The applications therefore don't need to know whether the backend is Grafana, another observability platform, or a future replacement.
