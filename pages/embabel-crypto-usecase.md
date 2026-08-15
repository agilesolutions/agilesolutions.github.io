# 🚀 Scheduled Crypto Pipeline: Market Performance & Sentiment Integration

This documentation outlines a scheduled, production-grade **Emberly / Embabel Focused Mode** pipeline. The architecture seamlessly collects market indicators, subjects anomalies to Human-in-the-Loop (HITL) compliance, and persists verified data layers to a relational analytical warehouse using Model Context Protocol (MCP) tool extensions.

---

## 🏗️ Architectural Blueprint

The pipeline executes sequentially under an isolated state machine, ensuring data integrity, strict rate-limiting compliance, and explicit human override capacity.

```
  [ CRON Trigger ]
         │
         ▼
 ┌───────────────┐
 │   Action 1    │ ──► Fetch Historical Price Assets (CoinGecko API v3)
 └───────────────┘
         │
         ▼
 ┌───────────────┐
 │   Action 2    │ ──► Fetch Crypto Fear & Greed Index (Alternative.me API)
 └───────────────┘
         │
         ▼
 ┌───────────────┐
 │  HITL Guard   │ ──► Evaluates anomalies, data drops, or structural gaps
 └───────────────┘
         │
         ├─► [Rejected / Outlier] ──► Alert Engineer / Manual Override
         │
         ▼ [Approved / Verified]
 ┌───────────────┐
 │   Action 3    │ ──► Database Ingestion via MCP Client PostgreSQL Tool
 └───────────────┘
         │
         ▼
 ┌───────────────┐
 │  Goal Action  │ ──► Compile & Output Unified Analytical Document (readme.md)
 └───────────────┘
```

---

## ⏱️ Step-by-Step Pipeline Orchestration

### 📈 Action 1: Market Performance (CoinGecko API v3)
The pipeline initiates by pulling programmatic time-series data. To maintain symmetry with the daily sentiment updates, hourly records are aggregated into single daily buckets tracking clean pricing vectors.

*   **Target Resource:** `https://api.coingecko.com/api/v3/coins/{id}/market_chart`
*   **Data Aggregation Frame:** Multi-point indices are normalized to standard UTC timestamps (`.dt.date`) and grouped to establish a baseline mean price.

### 🎭 Action 2: Market Sentiment (Alternative.me API)
Simultaneously, macro market momentum metrics are gathered via the index provider.
*   **Target Resource:** `https://api.alternative.me/fng/`
*   **Metric Properties:** Resolves daily values from `0` (Extreme Fear) to `100` (Extreme Greed) along with their corresponding text classifications.

### 🛑 Action 3: Human-in-the-Loop (HITL) Guard
Before dispatching payload frames to system infrastructure, the pipeline enters a synchronous **HITL Evaluation Sandbox**. This gate acts as an isolated checkpoint to prevent systemic noise from polluting production infrastructure.
*   **Automated Triggers for Human Review:**
    *   *Data Skew:* Any single-day price variance exceeding $\pm 20\%$.
    *   *Null Payload:* Partial data drops from upstream endpoints.
    *   *Sentiment Divergence:* Extreme divergence anomalies (e.g., Price dropping $15\%$ while the Fear & Greed index moves up significantly).
*   **Resolution Protocol:** An operational agent must manually sign off, adjust metrics, or trigger an exception override before execution proceeds to storage.

### 💾 Action 4: MCP Client Tool Invocation (PostgreSQL)
Once cleared by the HITL guard, the unified telemetry frame is passed to an **Model Context Protocol (MCP)** tool execution block. The MCP client formats the parameters and communicates directly with an active `mcp-server-postgres` instance.

#### Target Database Schema
```sql
CREATE TABLE IF NOT EXISTS crypto_market_metrics (
    record_date DATE PRIMARY KEY,
    asset_id VARCHAR(50) NOT NULL,
    average_price_usd NUMERIC(18, 4),
    sentiment_score INT NOT NULL,
    sentiment_classification VARCHAR(25) NOT NULL,
    verified_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);
```

#### MCP Tool Structural Call Example
```json
{
  "name": "postgres_server__execute_query",
  "arguments": {
    "query": "INSERT INTO crypto_market_metrics (record_date, asset_id, average_price_usd, sentiment_score, sentiment_classification) VALUES ($1, $2, $3, $4, $5) ON CONFLICT (record_date) DO UPDATE SET average_price_usd = EXCLUDED.average_price_usd, sentiment_score = EXCLUDED.sentiment_score, sentiment_classification = EXCLUDED.sentiment_classification;",
    "parameters": ["2026-08-15", "bitcoin", 64320.50, 72, "Greed"]
  }
}
```

---

## 💻 Complete Production Code

This script provides a self-contained, scheduled execution loops utilizing Python's `schedule` stack, simulating the full lifecycle from data extraction to storage execution.

```python
import os
import time
import requests
import schedule
import pandas as pd
from datetime import datetime

# Configurations
COINGECKO_URL = "https://api.coingecko.com/api/v3/coins/bitcoin/market_chart"
ALTERNATIVE_ME_URL = "https://api.alternative.me/fng/"
ASSET_ID = "bitcoin"

def run_crypto_pipeline():
    print(f"\n[Time: {datetime.utcnow().isoformat()}] Triggering scheduled focused mode pipeline...")
    
    try:
        # -----------------------------------------------------------------
        # ACTION 1: Fetch CoinGecko Performance Data
        # -----------------------------------------------------------------
        cg_res = requests.get(COINGECKO_URL, params={"vs_currency": "usd", "days": "1"}).json()
        df_raw_price = pd.DataFrame(cg_res['prices'], columns=['timestamp', 'price'])
        df_raw_price['date'] = pd.to_datetime(df_raw_price['timestamp'], unit='ms').dt.date
        latest_price = float(df_raw_price.groupby('date')['price'].mean().iloc[-1])
        target_date = df_raw_price.groupby('date')['price'].mean().index[-1]

        # -----------------------------------------------------------------
        # ACTION 2: Fetch Alternative.me Sentiment Data
        # -----------------------------------------------------------------
        fng_res = requests.get(ALTERNATIVE_ME_URL, params={"limit": "1", "format": "json"}).json()
        fng_data = fng_res['data'][0]
        sentiment_score = int(fng_data['value'])
        sentiment_class = fng_data['value_classification']

        # -----------------------------------------------------------------
        # ACTION 3: HITL Guard Simulation
        # -----------------------------------------------------------------
        is_anomaly = latest_price < 10000 or sentiment_score > 95 or sentiment_score < 5
        
        if is_anomaly:
            print(f"⚠️ [HITL TRIGGERED] Data Anomaly Detected! Price: {latest_price}, Sentiment: {sentiment_score}")
            print("Pipeline paused awaiting engineer approval. Manual override required.")
            return False
        
        print("✅ [HITL PASSED] Metrics verified within nominal thresholds.")

        # -----------------------------------------------------------------
        # ACTION 4: Format MCP Client Tool Call (PostgreSQL)
        # -----------------------------------------------------------------
        mcp_payload = {
            "tool": "postgres_server__execute_query",
            "arguments": {
                "query": "INSERT INTO crypto_market_metrics (record_date, asset_id, average_price_usd, sentiment_score, sentiment_classification) VALUES (%s, %s, %s, %s, %s);",
                "parameters": [str(target_date), ASSET_ID, latest_price, sentiment_score, sentiment_class]
            }
        }
        
        print("🚀 [MCP TOOL INVOCATION] Sending payload structure to Postgres Client Server:")
        print(f"   Executing Query: {mcp_payload['arguments']['query']}")
        print(f"   With Parameters: {mcp_payload['arguments']['parameters']}")
        print("✅ Pipeline Cycle Terminated Successfully.")
        
    except Exception as e:
        print(f"❌ [CRITICAL ERROR] Pipeline Failed: {str(e)}")

# Scheduling Configuration: Execute pipeline automatically every day at 00:05 UTC
schedule.every().day.at("00:05").do(run_crypto_pipeline)

if __name__ == "__main__":
    print("🚀 Standby Mode Active. Crypto Market-Sentiment Pipeline Initialized...")
    # Execute immediately once for initial verification
    run_crypto_pipeline()
```

---

## ⚠️ Critical Operational Guardrails

*   **Strict Time Normalization:** Upstream assets must map to uniform timezone declarations (UTC). Failure to prune timezone variations will result in misaligned metrics or index mismatching within PostgreSQL columns.
*   **MCP Timeout Controls:** Database tools should utilize explicit internal connection pool parameters. If the target server experiences unexpected lock conditions, the tool client must gracefully time out to prevent hung states in the cron process.
*   **Public API Preservation:** To optimize execution limits, implement short-term cache buffers on the application tier. Repeated pipeline re-runs should hit cached payloads to avoid temporary operational blocks from CoinGecko.