---
title: "Free APIs + AI = Your Personal Bloomberg Terminal"
subtitle: "Building a Multi-Exchange Data Engine With Zero Cost"
date: 2026-05-23
draft: false
tags: ["trading", "AI", "quant", "jarvis", "data-engineering"]
categories: ["jarvis-trading-series"]
description: "How I built a real-time multi-exchange data engine using Kraken/Coinbase free APIs, Redis caching, PostgreSQL storage, OpenClaw Cron, and automatic Obsidian market snapshots — all for $0."
---

## Part 1: How Much Does Bloomberg Terminal Cost?

Let's start with some numbers:

- **Bloomberg Terminal**: $24,000/year/license. Exchange fees not included.
- **Reuters Eikon**: $22,000/year/license.
- **Wind (万得)**: ~$5,700/year in China.

What do these professional terminals actually provide? Three things:

1. **Real-time market data** — quotes, order books, and trade data from global exchanges
2. **Historical data** — pull up any instrument's historical candles for analysis
3. **Information aggregation** — auto-push for news, announcements, and research reports

Three things. $24,000 a year. That's roughly ¥170,000.

But here's the thing — in 2026, AI can do all three for you. And it costs **exactly $0 to start**.

---

## Part 2: Jarvis Data Engine Architecture — The "Three Layer" Design

Building on the Docker Compose foundation from Part 2, we're now giving Jarvis a brain.

```
┌─────────────────────────────────────────────────────┐
│                Jarvis Data Engine v1.0                │
├─────────────────────────────────────────────────────┤
│                                                       │
│  ┌─────────────┐   ┌─────────────┐   ┌────────────┐ │
│  │ Kraken API  │   │ Coinbase API│   │  Future:    │ │
│  │  (Free)      │   │  (Free)      │   │  Binance    │ │
│  │             │   │             │   │  OKX/Bybit  │ │
│  └──────┬──────┘   └──────┬──────┘   └────────────┘ │
│         │                 │                           │
│         └────────┬────────┘                           │
│                  ▼                                    │
│         ┌───────────────┐                            │
│         │  Redis Cache   │  ← Real-time (TTL=300s)    │
│         │  Sub-ms R/W    │                            │
│         └───────┬───────┘                            │
│                  │                                    │
│                  ▼                                    │
│         ┌───────────────┐                            │
│         │ PostgreSQL    │  ← Historical OHLCV          │
│         │  + TimescaleDB │    (Time-series optimized)  │
│         └───────┬───────┘                            │
│                  │                                    │
│         ┌────────┴────────┐                          │
│         ▼                 ▼                          │
│   ┌──────────┐     ┌──────────────┐                 │
│   │ Grafana  │     │  Obsidian    │                 │
│   │ Dashboard │     │  Snapshots   │                 │
│   └──────────┘     └──────────────┘                 │
│                                                       │
│         ┌─────────────────────────┐                 │
│         │  OpenClaw Cron          │                 │
│         │  Triggers every 5 min    │                 │
│         └─────────────────────────┘                 │
└─────────────────────────────────────────────────────┘
```

### Why This Architecture?

**Redis in front, PostgreSQL in back** — this is a classic "cache-through" pattern.

Kraken's BTC/USD ticker updates every second. If you query the database for every read, PostgreSQL's millisecond-level latency plus network overhead means stale data by the time your strategy engine sees it.

Redis reads and writes are **sub-millisecond**, entirely in-memory. Real-time quotes cached here mean near-zero latency for your strategy engine.

But Redis data expires (we set a 300-second TTL). Historical candles, trade records, and backtesting need persistent storage — that's PostgreSQL's job.

**Hot data first, cold data last. Speed first, permanence later.**

---

## Part 3: collector.py — The Full Implementation

The data collector is the "eyes" of the entire system. If the eyes go blind, every downstream analysis is worthless.

```python
"""
Jarvis Data Collector v2.0
============================
Kraken + Coinbase free APIs → Redis cache → PostgreSQL persistence

Upgrades (vs. Part 2 basic version):
- Added Coinbase source with cross-exchange spread comparison
- Added OHLCV (candlestick) collection into PostgreSQL
- Added anomaly detection with automatic alerts
- Command-line argument support for selecting sources

Usage:
    python src/data/collector.py              # Default: Kraken
    python src/data/collector.py --exchange coinbase
    python src/data/collector.py --all        # All exchanges
"""

import os, json, argparse, redis, ccxt, psycopg2
from datetime import datetime, timezone
from dotenv import load_dotenv

load_dotenv()

# Configuration
REDIS_HOST = os.getenv("REDIS_HOST", "localhost")
REDIS_PORT = int(os.getenv("REDIS_PORT", 6379))
PG_CONFIG = {
    "host": os.getenv("PG_HOST", "localhost"),
    "port": int(os.getenv("PG_PORT", 5432)),
    "user": os.getenv("PG_USER", "jarvis"),
    "password": os.getenv("PG_PASSWORD", "jarvis123"),
    "dbname": os.getenv("PG_DATABASE", "jarvis"),
}
SYMBOLS = ["BTC/USD", "ETH/USD"]
REDIS_TTL = 300
PRICE_ALERT_THRESHOLD = 0.05  # 5% move triggers alert

# Lazy-loaded exchange connections
_exchanges = {}

def get_exchange(name: str = "kraken") -> ccxt.Exchange:
    if name not in _exchanges:
        klass = getattr(ccxt, name)
        _exchanges[name] = klass({"enableRateLimit": True})
    return _exchanges[name]

def collect_ticker(exchange_name: str, symbol: str) -> dict:
    """Pull ticker for a single pair"""
    exchange = get_exchange(exchange_name)
    ticker = exchange.fetch_ticker(symbol)
    return {
        "exchange": exchange_name, "symbol": symbol,
        "last": ticker["last"], "bid": ticker["bid"],
        "ask": ticker["ask"], "volume_24h": ticker["baseVolume"],
        "change_24h": ticker["percentage"],
        "high_24h": ticker.get("high"), "low_24h": ticker.get("low"),
        "timestamp": datetime.now(timezone.utc).isoformat(),
    }

def collect_ohlcv(exchange_name: str, symbol: str, timeframe="5m", limit=12):
    """Pull OHLCV candle data"""
    exchange = get_exchange(exchange_name)
    return exchange.fetch_ohlcv(symbol, timeframe=timeframe, limit=limit)

def store_ticker_to_redis(r, data):
    key = f"ticker:{data['exchange']}:{data['symbol'].replace('/', '_')}"
    r.setex(key, REDIS_TTL, json.dumps(data))

def store_ohlcv_to_pg(data_list, exchange, symbol):
    """Batch insert OHLCV into PostgreSQL"""
    conn = psycopg2.connect(**PG_CONFIG)
    cur = conn.cursor()
    rows = []
    for o in data_list:
        ts = datetime.fromtimestamp(o[0] / 1000, tz=timezone.utc)
        rows.append((exchange, symbol, ts, o[1], o[2], o[3], o[4], o[5]))
    cur.executemany("""
        INSERT INTO ohlcv_5m (exchange, symbol, timestamp, open, high, low, close, volume)
        VALUES (%s,%s,%s,%s,%s,%s,%s,%s)
        ON CONFLICT (exchange, symbol, timestamp) DO NOTHING
    """, rows)
    conn.commit(); cur.close(); conn.close()

# ... (detect_price_anomaly, check_arbitrage, write_market_snapshot)
# Full source in GitHub repo — link at bottom.
```

### Core Upgrades (vs. Part 2 basic version)

| Feature | Part 2 | Part 3 |
|:--|:--|:--|
| Data Sources | Kraken only | Kraken + Coinbase |
| Storage | Redis only | Redis + PostgreSQL |
| Candles | ❌ | ✅ OHLCV (5m / 1h) |
| Anomaly Detection | ❌ | ✅ ±5% alerts |
| Arbitrage Scanning | ❌ | ✅ >0.5% spread flag |
| Obsidian Snapshots | ❌ | ✅ Daily markdown |
| CLI | No args | Full argparse |

---

## Part 4: OpenClaw Cron — 24/7 Automation

Code is written. But running `python collector.py` manually is not automation. You need a scheduler.

Traditional solution: Linux crontab. But it has pain points:
- Logs aren't aggregated — dig through `/var/log` when things break
- No alert integration — things die silently
- Configs scattered everywhere — collaboration nightmare

OpenClaw Cron bundles scheduling, logging, and alerting in one place:

```yaml
# openclaw-cron.yaml — Jarvis Data Engine Scheduling

- name: "Jarvis Ticker (every minute)"
  schedule: "* * * * *"
  command: "cd ~/jarvis-trading && python src/data/collector.py --all"
  notify_on_error: true

- name: "Jarvis OHLCV (every 5 min)"
  schedule: "*/5 * * * *"
  command: "cd ~/jarvis-trading && python src/data/collector.py --all --snapshot"
  notify_on_error: true

- name: "Jarvis Daily Snapshot"
  schedule: "0 0 * * *"
  command: "cd ~/jarvis-trading && python src/data/collector.py --all --snapshot"
  notify_on_success: false
```

**Three cadences, three purposes:**

1. **Every-minute ticker** — Lightweight. Pull latest price + 24h data into Redis.
2. **Every-5-minute OHLCV** — Heavier. Pull candle data + generate snapshot into PostgreSQL.
3. **Daily midnight snapshot** — Routine archiving. No notifications needed.

Clear separation of concerns. No blocking.

---

## Part 5: Obsidian Integration — Turning Data into Knowledge

This is my favorite feature.

Most quant traders have this problem: data flows in, but **nobody ever looks back**. What did the market look like six months ago? How did your strategy perform in that environment? You have no idea.

Jarvis solves this: **a human-readable market snapshot, generated daily, written into Obsidian**.

The generated Markdown looks like:

```markdown
# Market Snapshot — 2026-05-23

Recorded: 2026-05-23 22:00 UTC

## Live Prices

| Pair | Exchange | Last | 24h Change | 24h Volume |
|:--|:--|:--|:--|:--|
| BTC/USD | kraken | $97,231.40 | 📈 +2.34% | 12,456,789 |
| BTC/USD | coinbase | $97,150.00 | 📈 +2.28% | 10,234,567 |
| ETH/USD | kraken | $3,847.20 | 📉 -1.12% | 8,765,432 |

## Anomaly Alerts

(Price anomalies auto-logged here)
```

The value of this snapshot:

1. **Retrospective review** — Want to know "what was BTC at 3 months ago?" Search Obsidian, answer in seconds.
2. **Bidirectional linking** — Link your trading strategy notes and learning notes together.
3. **Zero cost** — One line of code to generate, fully automatic storage.

---

## Part 6: The NotebookLM Knowledge Lever — Mastering Any Data Source in One Hour

This section draws from *Cola AI Lab* videos 20 and 21 — using NotebookLM as a knowledge lever.

The data engine currently connects Kraken and Coinbase. But what about Binance, OKX, Bybit? Or on-chain data from Dune Analytics? Each exchange has hundreds of pages of API documentation. You can't read them all.

The correct approach:

```
Step 1: Gather sources (5 min)
    → Google "exchange name API documentation"
    → Find official docs
    → Find 3-5 related YouTube tutorials + GitHub projects

Step 2: Drop into NotebookLM (2 min)
    → Import all links, PDFs, YouTube links
    → 42 knowledge sources isn't enough? Delete old, add new.

Step 3: Generate mind map (1 min)
    → Click one button for API knowledge structure
    → Core concepts, auth methods, rate limits at a glance

Step 4: Generate quickstart report (2 min)
    → "How to connect Exchange X and pull BTC price using Python"
    → AI extracts complete code template from your sources

Step 5: CEO mode — ask targeted questions (5 min)
    → "How does this exchange's API differ from Kraken?"
    → "What pitfalls should I watch for when connecting?"
    → "How do rate limits affect my strategy?"

Total: 15 minutes to master a new data source.
```

**This is knowledge leverage.** Your time isn't spent on "learning tech" — it's spent on "making judgments."

In practice, every time I connect a new data source, I generate a dedicated NotebookLM notebook and store it in my knowledge base for future reference. All technical questions are outsourced to AI. I only think about: "What opportunity can this data source help me discover?"

---

## Part 7: Why the Data Engine Is Your Core Moat

People think "it's just pulling an API, what's so hard about that?"

The data engine has three underestimated challenges:

### Challenge 1: Data Quality

Exchange public APIs weren't built for quant trading. Disconnections, packet loss, latency, and data misalignment are the norm.

Kraken's `percentage` field is sometimes `None`. Coinbase's `bid/ask` occasionally exceeds the last price. If you don't handle these edge cases, your strategy gets misled by dirty data.

**Garbage in, garbage out.** Data cleaning code is often 3x the volume of collection code.

### Challenge 2: Clock Synchronization

Kraken and Coinbase's server clocks can differ by 1-2 seconds. For cross-exchange arbitrage, this 2-second misalignment turns an "arbitrage opportunity" into an "arbitrage trap" — the spread you see doesn't actually exist, it's just unsynchronized timestamps.

Jarvis uses UTC timestamps uniformly, with time correction at the strategy layer.

### Challenge 3: API Changes

Exchange APIs change constantly. Kraken modified their WebSocket interface 3 times in the first half of this year. Coinbase Advanced Trade and Coinbase Pro are completely different APIs.

**This is why the data collection layer must be modular** — one independent collector function per exchange. Changing one never breaks another.

---

## Part 8: Hands-On — Connecting a New Exchange From Scratch

Say you want to add Bybit. Traditional approach:

```
Read docs (2h) → Sign up (30m) → Apply API Key (15m)
→ Test sandbox (1h) → Production debug (2h) → Add to collector (30m)
Total: 6+ hours
```

NotebookLM + AI approach:

```
1. Drop Bybit API docs URL into NotebookLM, generate structure (5m)
2. Ask AI: "How does this differ from Kraken's API?" (2m)
3. Ask AI to directly generate the collector function (3m)
4. Copy, paste, run it, see what errors come up (2m)
5. Feed errors back to AI for fixes (3m)
Total: 15 minutes
```

**24x efficiency gain.**

This is the new "how to do things" in the AI era — no more pre-studying. Learn by doing.

---

## Part 9: Summary — Data Is More Than Data

Bloomberg Terminal doesn't sell data. It sells **information processing capability**.

The AI-era personal Bloomberg terminal is more than pulling data. It's:

1. **Auto-collection** — 24/7, no gaps
2. **Intelligent cleaning** — anomaly detection + multi-source validation
3. **Structured storage** — hot data in Redis, cold data in PostgreSQL
4. **Human-readable** — daily Obsidian snapshots, instant retrospection
5. **Arbitrage discovery** — automatic cross-exchange spread scanning

All of this costs **$0** (except your local electricity).

But the value is infinite — because a good data engine lets you see what others can't.

---

## Coming Up: Part 4

*"Trading Strategies Aren't Gut Feelings — From Intuition to PRD"*

We'll use:
- Natural language intuition → AI translates to quant logic
- Deep Research to pull academic finance models
- NotebookLM for knowledge synthesis
- Generate PRD document → Claude Code development

Turn your "I think" into "mathematically proven."

---

*This series is based on Cola AI Lab's quantitative trading research. All code is open source. This is not investment advice.*

---

> 💬 Is your data engine running? Questions? Share below.

**Series Index:**
1. ✅ Spot vs. Contracts — The Brutal Math
2. ✅ One-Click Environment Setup
3. ✅ **This Post** — Free APIs + AI = Your Bloomberg Terminal
4. 🔜 Trading Strategies Aren't Gut Feelings
5. 🔜 Backtests That Look Perfect, Live Accounts That Bleed
