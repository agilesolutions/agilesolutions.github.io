# 📈 Crypto Market Performance & Sentiment Integration Guide

This guide details how to programmatically fetch, structure, and combine cryptocurrency price performance data from the **CoinGecko API v3** with market sentiment metrics from the **Alternative.me Crypto Fear & Greed Index API**.

---

## 🧭 Endpoint Overviews

### 1. Market Performance (CoinGecko API v3)
CoinGecko maps digital assets using strict internal string IDs (e.g., `bitcoin`, `ethereum`) rather than trading symbols/tickers (BTC, ETH).

*   **Live Snapshots:** Use `/coins/markets` to pull current pricing alongside percentage performance changes across multiple time windows (`1h,24h,7d,14d,30d,200d,1y`).
*   **Historical Time Series:** Use `/coins/{id}/market_chart` to fetch an array of historic prices paired with millisecond UNIX timestamps.

### 2. Market Sentiment (Alternative.me API)
Alternative.me provides a daily multi-factor macro-sentiment score ranging from `0` (**Extreme Fear**) to `100` (**Extreme Greed**).
*   **Endpoint:** `https://api.alternative.me/fng/`
*   **Historical Archive:** Pass the parameter `limit=0` to extract their complete historical database, or `limit=30` for the past 30 days.

---

## 💻 Python Integration Pipeline

Because CoinGecko updates continuously throughout the day while Alternative.me outputs a single macro data point at **00:00 UTC**, you must programmatically normalize and resample the data streams to align them accurately.

The script below extracts a 30-day window from both providers, flattens timestamps to matching daily calendar dates, and performs a clean data merge using the `pandas` library.

```python
import requests
import pandas as pd

# =====================================================================
# 1. FETCH HISTORICAL PRICE PERFORMANCES FROM COINGECKO
# =====================================================================
print("Fetching price history from CoinGecko...")
cg_url = "https://api.coingecko.com/api/v3/coins/bitcoin/market_chart"
cg_params = {
    "vs_currency": "usd", 
    "days": "30"
}
cg_res = requests.get(cg_url, params=cg_params).json()

# Parse CoinGecko response: returns structural lists of [unix_timestamp_ms, price]
df_price = pd.DataFrame(cg_res['prices'], columns=['timestamp', 'price'])
df_price['date'] = pd.to_datetime(df_price['timestamp'], unit='ms').dt.date

# Squash to daily average pricing to align with Alternative.me's daily cycle
df_price_daily = df_price.groupby('date')['price'].mean().reset_index()


# =====================================================================
# 2. FETCH HISTORICAL SENTIMENT FROM ALTERNATIVE.ME
# =====================================================================
print("Fetching sentiment history from Alternative.me...")
fng_url = "https://api.alternative.me/fng/"
fng_params = {
    "limit": "30", 
    "format": "json"
}
fng_res = requests.get(fng_url, params=fng_params).json()

# Parse Fear & Greed JSON payload
df_sentiment = pd.DataFrame(fng_res['data'])

# Alternative.me returns standard 10-digit UNIX timestamps as string objects
df_sentiment['date'] = pd.to_datetime(df_sentiment['timestamp'].astype(int), unit='s').dt.date

# Cast and clean up required numerical/categorical data fields
df_sentiment['fear_greed_score'] = df_sentiment['value'].astype(int)
df_sentiment['sentiment_class'] = df_sentiment['value_classification']
df_sentiment_clean = df_sentiment[['date', 'fear_greed_score', 'sentiment_class']]


# =====================================================================
# 3. MERGE PERFORMANCE DATASETS
# =====================================================================
# Inner join combines historical records where dates match up perfectly
combined_market_df = pd.merge(df_price_daily, df_sentiment_clean, on='date', how='inner')

print("\n--- Integrated Performance Dashboard ---")
print(combined_market_df.head(10))
```

---

## 📊 Unified Data Output Format

Once merged, your final data structure will look like this, providing a unified view of asset value versus crowd psychology:

| date | price (USD) | fear_greed_score | sentiment_class |
| :--- | :--- | :--- | :--- |
| `2026-08-10` | 64320.50 | 72 | Greed |
| `2026-08-11` | 65110.15 | 78 | Extreme Greed |
| `2026-08-12` | 61200.40 | 45 | Fear |
| `2026-08-13` | 59800.90 | 32 | Fear |

---

## ⚠️ Critical Engineering Guardrails

*   **Asset Scope Mismatch:** The Alternative.me Fear & Greed index is calculated using **Bitcoin market metrics** (dominance, global volatility, volume). If you map this data against an altcoin (e.g., Ethereum or Solana), you are analyzing that specific token's price performance against macro *crypto-wide market sentiment* rather than asset-specific sentiment.
*   **Time Zone Synchronization:** Ensure your pipelines explicitly convert all data into a shared UTC timestamp framework (`.dt.date`) before performing joins to prevent regional daylight lag distortions.
*   **API Key and Rate Limits:** Public endpoints for CoinGecko are rate-limited (~10–30 requests/minute). For production builds, switch the root domain to `https://pro-api.coingecko.com/api/v3/`, append your subscription key to the `x-cg-pro-api-key` header, and implement data caching.