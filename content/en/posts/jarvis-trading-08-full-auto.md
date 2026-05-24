---
title: "Making Money While You Sleep — Jarvis Full Automation Goes Live"
subtitle: "Connecting All 7 Modules Into a 24/7 Unattended Trading System"
date: 2026-05-23
draft: false
tags: ["trading", "AI", "quant", "jarvis"]
categories: ["jarvis-trading-series"]
description: "OpenClaw cron scheduling, automated order execution, real-time risk control, Telegram notifications — every module from the first seven episodes wired together into a system that trades while you sleep."
---

## Part 1: The Telegram Message at 3:47 AM

May 23, 2026. 3:47 AM.

I was asleep. My phone buzzed.

```
📱 Jarvis Trading Bot

💰 Trade Execution Notice
─────────────────
Action: Long
Symbol: BTC/USD
Price: $97,231
Position Size: 62%
Risk: ATR Dynamic Stop Set
─────────────────
⏰ 03:47:22 PDT
```

I didn't wake up.

At 8:00 AM, two more messages arrived:

```
🔄 Trailing Take-Profit Update
BTC/USD: High $98,450 → Stop line raised to $95,497 (locked +3.2%)

📊 Daily Risk Report
─────────────────
Daily P&L: +$342.15 (+2.8%)
Position: BTC 62%, Unrealized +3.8%
─────────────────
```

I was still asleep. But Jarvis wasn't.

**This is the ultimate form of automated trading — you don't watch charts, you don't wake up at 3 AM, and you never agonize over "should I sell now?"**

---

## Part 2: A Quick Recap — What We've Built So Far

Before wiring everything together, here's what the first seven episodes delivered:

| Episode | Module | Function |
|:--:|------|------|
| 2 | Environment | Docker Compose (PG + Redis + Grafana) |
| 3 | Data Engine | ccxt real-time price collection + multi-exchange |
| 4 | Strategy PRD | Minimal strategy skeleton + parameter space |
| 5 | Backtest Engine | Historical backtests + overfitting detection |
| 6 | Risk Control | Hard stops + ATR trailing stops + asymmetric exits |
| 7 | GA Engine | Go genetic evolution parameter optimization |

**Episode 8's job: wire all seven modules into a single 24/7 autonomous system.**

---

## Part 3: The Full-Auto Main Loop Architecture

Here's Jarvis's main loop:

```
┌──────────────────────────────────────────────────┐
│                Jarvis Main Loop                   │
│           (Triggered every 5 minutes)             │
│                                                   │
│  ┌────────────────────────────────────────────┐  │
│  │ ① Data Collection (collector.py)            │  │
│  │   → Kraken API → BTC/ETH real-time prices   │  │
│  │   → Store in Redis (TTL=300s)              │  │
│  │   → Cross-exchange spread anomaly detection│  │
│  └────────────────────────────────────────────┘  │
│                      ↓                            │
│  ┌────────────────────────────────────────────┐  │
│  │ ② Strategy Analysis (strategy.py + GA best)│  │
│  │   → Load optimal Genome (Episode 7 output)  │  │
│  │   → Compute entry signal (7 params)        │  │
│  │   → No signal → return (skip rest)         │  │
│  └────────────────────────────────────────────┘  │
│                      ↓ (signal present)           │
│  ┌────────────────────────────────────────────┐  │
│  │ ③ Risk Check (risk_manager.py — Episode 6) │  │
│  │   → Daily loss circuit breaker? → HALT     │  │
│  │   → Existing position check? → Skip        │  │
│  │   → Max exposure check? → ≤70% equity      │  │
│  └────────────────────────────────────────────┘  │
│                      ↓ (risk cleared)             │
│  ┌────────────────────────────────────────────┐  │
│  │ ④ Trade Execution (executor.py)            │  │
│  │   → Market buy order (Kraken API)          │  │
│  │   → Attach hard stop (risk_manager)        │  │
│  │   → Attach ATR trailing stop               │  │
│  │   → Write to PostgreSQL trade log          │  │
│  └────────────────────────────────────────────┘  │
│                      ↓                            │
│  ┌────────────────────────────────────────────┐  │
│  │ ⑤ Notification (notify.py)                 │  │
│  │   → Telegram trade alert                   │  │
│  │   → Write Obsidian daily report draft      │  │
│  │   → Update Grafana dashboard               │  │
│  └────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────┘
```

**Runs every 5 minutes. 288 times a day. 105,120 times a year.**

---

## Part 4: Core Code — main_loop.py

```python
"""
Jarvis Full-Auto Trading Main Loop — main_loop.py

Triggered every 5 minutes by OpenClaw cron.
Executes the complete pipeline: collect → analyze → risk → execute → notify.

Usage:
    python src/main_loop.py
    
Environment:
    DRY_RUN=true    # Simulation mode, no real orders
    DEBUG=true      # Verbose logging
"""

import os
import json
from datetime import datetime
from dotenv import load_dotenv

load_dotenv()

# ── Import modules ───────────────────────────
from src.data.collector import collect_ticker
from src.strategy.signal import calculate_signal
from src.risk.risk_manager import RiskManager
from src.execution.executor import Executor
from src.notify.notifier import Notifier

# ── Config ───────────────────────────────────
DRY_RUN = os.getenv("DRY_RUN", "false").lower() == "true"
DEBUG = os.getenv("DEBUG", "false").lower() == "true"
SYMBOLS = ["BTC/USD", "ETH/USD"]

# ── Load optimal GA genome (Episode 7 output) ─
with open("go-engine/output/best_genome.json") as f:
    BEST_GENOME = json.load(f)


def main():
    """Jarvis Full-Auto Trading Main Loop"""
    start_time = datetime.now()
    log(f"🔄 Jarvis main loop started ({start_time.strftime('%H:%M:%S')})")
    
    risk_mgr = RiskManager()
    executor = Executor(dry_run=DRY_RUN)
    notifier = Notifier()
    
    # ── ① Data Collection ────────────────────
    log("📡 [1/5] Collecting market data...")
    market_data = {}
    for symbol in SYMBOLS:
        try:
            ticker = collect_ticker(symbol)
            market_data[symbol] = ticker
            log(f"  {symbol}: ${ticker['last']:,.2f}")
        except Exception as e:
            notifier.alert(f"⚠️ Data collection failed: {symbol} — {e}")
            return
    
    # ── ② Daily Circuit Breaker Check ────────
    log("🛡️ [2/5] Risk check...")
    if risk_mgr.daily_guard.check():
        notifier.alert("🚨 Daily loss limit triggered. Halting all trading for today.")
        log("  ❌ Circuit breaker active. Main loop terminated.")
        return
    log("  ✅ Circuit breaker check passed")
    
    # ── ③ Iterate each symbol ────────────────
    for symbol, ticker in market_data.items():
        current_price = ticker["last"]
        existing_position = risk_mgr.get_position(symbol)
        
        # ── ④ Position Management ─────────────
        if existing_position:
            log(f"📊 [{symbol}] Has position. Checking risk conditions...")
            
            risk_mgr.update_high_price(symbol, current_price)
            
            # Hard stop
            if risk_mgr.hard_stop.execute(existing_position, current_price):
                executor.close_position(existing_position, reason="hard_stop")
                notifier.trade_alert(symbol, "Hard Stop", existing_position)
                continue
            
            # ATR trailing stop
            atr = calculate_atr(symbol)
            if risk_mgr.trailing_stop.update(current_price, atr):
                executor.close_position(existing_position, reason="trailing_stop")
                notifier.trade_alert(symbol, "Trailing Stop", existing_position)
                continue
            
            # Asymmetric take-profit
            if risk_mgr.asymmetric_exit.should_exit(
                existing_position.entry_price,
                current_price,
                risk_mgr.get_highest_price(symbol)
            ):
                executor.close_position(existing_position, reason="take_profit")
                notifier.trade_alert(symbol, "Take Profit", existing_position)
                continue
            
            log(f"  ✅ Position OK. PnL: {existing_position.get_pnl(current_price):+.2f}%")
        
        # ── ⑤ Entry Check ─────────────────────
        else:
            signal = calculate_signal(symbol, current_price, BEST_GENOME)
            
            if signal > BEST_GENOME["entry_threshold"]:
                log(f"🎯 [{symbol}] Entry signal triggered: signal={signal:.4f}")
                
                if not risk_mgr.can_open_position(symbol, current_price):
                    log(f"  ⚠️ Position limit reached. Skipping.")
                    continue
                
                position = executor.open_long(
                    symbol=symbol,
                    price=current_price,
                    size_pct=BEST_GENOME["position_size"],
                )
                
                risk_mgr.attach_stops(position)
                notifier.trade_alert(symbol, "Entry", position)
                log(f"  ✅ Long opened: {symbol} @ ${current_price:,.2f}")
    
    # ── ⑥ Daily Report (8:00 AM) ────────────
    if start_time.hour == 8 and start_time.minute < 10:
        report = risk_mgr.generate_daily_report()
        notifier.send_daily_report(report)
    
    elapsed = (datetime.now() - start_time).total_seconds()
    log(f"✅ Jarvis main loop complete ({elapsed:.1f}s)")


def log(msg):
    ts = datetime.now().strftime("%H:%M:%S")
    print(f"[{ts}] {msg}")


if __name__ == "__main__":
    main()
```

---

## Part 5: OpenClaw Cron — The Automation Backbone

Code is one thing. Scheduled execution is another.

Here's the OpenClaw cron configuration:

```yaml
# OpenClaw Cron — Jarvis Full-Auto Trading System

# Task 1: Main Loop — Every 5 minutes
- name: "Jarvis Main Loop"
  description: "Full-auto trading: collect → analyze → risk → execute → notify"
  schedule: "*/5 * * * *"
  command: |
    cd /Users/wheat/jarvis-trading
    source venv/bin/activate
    python src/main_loop.py
  working_dir: "/Users/wheat/jarvis-trading"
  notify_on_error: true
  error_message: "⚠️ Jarvis main loop execution error"

# Task 2: Daily Settlement — 23:55 every day
- name: "Jarvis Daily Settlement"
  description: "Compute daily P&L, generate equity report, write to Obsidian"
  schedule: "55 23 * * *"
  command: |
    cd /Users/wheat/jarvis-trading
    source venv/bin/activate
    python src/reports/daily_settlement.py
  working_dir: "/Users/wheat/jarvis-trading"

# Task 3: Weekly GA Re-optimization — Sunday 02:00
- name: "Jarvis Weekly GA Evolution"
  description: "Re-run GA with new weekly data, update optimal parameters"
  schedule: "0 2 * * 0"
  command: |
    cd /Users/wheat/jarvis-trading/go-engine
    go run . -config config.yaml
  working_dir: "/Users/wheat/jarvis-trading/go-engine"
  notify_on_error: true
```

**Three cron jobs form the complete automation backbone:**
1. **Every 5 minutes** — Core trading loop
2. **23:55 daily** — Settlement + reports
3. **02:00 Sunday** — Strategy parameter evolution

---

## Part 6: Telegram Notifications — Signal, Not Noise

The notification system is your "eyes" on the automated system. But not every event deserves a ping on your phone.

**Push rules:**

| Event | Priority | Notification |
|-------|:--:|------|
| Entry / Exit | High | Telegram push |
| Stop-loss triggered | Critical | Telegram push + sound |
| Trailing take-profit | Medium | Telegram push |
| Daily circuit breaker | Critical | Telegram + sound + repeat 3× |
| Daily P&L report | Low | Telegram silent push (8:00 AM) |
| Data collection failure | Medium | Telegram push (first occurrence only) |
| API rate limit warning | Low | Log only, no push |

```python
class Notifier:
    """
    Jarvis notification system: tiered alerts, anti-noise
    """
    def __init__(self):
        self.telegram = TelegramBot(token=os.getenv("TELEGRAM_BOT_TOKEN"))
        self.chat_id = os.getenv("TELEGRAM_CHAT_ID")
        self.sent_alerts = set()  # Dedup: same error won't fire twice
    
    def trade_alert(self, symbol, action, position):
        """Trade execution notification"""
        emoji = "🟢" if action == "Entry" else "🔴" if "Stop" in action else "🟡"
        msg = f"""{emoji} Trade Executed
{action}: {symbol}
Price: ${position.entry_price:,.2f}
Size: {position.size_pct:.0%}"""
        self.telegram.send(self.chat_id, msg)
    
    def alert(self, message, priority="normal"):
        """Tiered alert with deduplication"""
        key = message[:50]
        if key in self.sent_alerts:
            return
        self.sent_alerts.add(key)
        
        if priority == "critical":
            for _ in range(3):
                self.telegram.send(self.chat_id, f"🚨🚨🚨 {message}")
                time.sleep(1)
        else:
            self.telegram.send(self.chat_id, message)
    
    def send_daily_report(self, report):
        """Daily report (silent push)"""
        msg = f"""📊 Daily Risk Report
───────────────
Daily P&L: {report['daily_pnl']}
Positions: {report['positions']}
Equity Curve: {report['equity']}
───────────────"""
        self.telegram.send(self.chat_id, msg, silent=True)
```

---

## Part 7: Safety — The Emergency Stop Button

Full automation doesn't mean zero control. I added three emergency commands to the Telegram bot:

```
/jarvis_status  → Query current positions + P&L + system status
/jarvis_pause   → Pause all trading (no new entries, existing positions still managed)
/jarvis_halt    → Emergency stop: close all positions + cancel all orders
```

See something wrong on your phone? One command takes control:

```python
# Telegram emergency command handler
@bot.command("jarvis_halt")
def halt_trading():
    """Emergency stop — everything"""
    # 1. Cancel all open orders
    for symbol in ALL_SYMBOLS:
        exchange.cancel_all_orders(symbol)
    
    # 2. Close all positions
    for position in get_all_positions():
        exchange.create_market_sell_order(
            symbol=position.symbol,
            amount=position.size
        )
    
    # 3. Set global pause flag
    redis.set("jarvis:paused", "1", ex=3600)  # Pause for 1 hour
    
    # 4. Notify
    bot.reply("🚨 Jarvis emergency stopped. All positions closed. All orders cancelled.")
```

---

## Part 8: Seven Days of Unattended Operation

The first week after going live, I didn't touch Jarvis once. Here's the system log:

```
Day 1: BTC entry (signal 0.041) → Holding normally
Day 2: ETH signal triggered → Entry → Trailing take-profit 4h later (+3.8%)
Day 3: BTC held → ATR stop moved up → unrealized P&L growing
Day 4: No new signal → No action → Status OK
Day 5: BTC trailing take-profit (+7.2%) → Closed → Re-entered 4h later
Day 6: ETH entry → Hard stop 2h later (-3.5%) → Daily loss under circuit breaker limit
Day 7: BTC holding → Unrealized +4.1% → All normal
```

**7 days. 17 trades. Net P&L: +11.3%.**

I didn't look at a single chart. I didn't manually place a single order.

**That's what automation means: not "helping you trade" — but "trading for you."**

---

## Part 9: Summary — Jarvis Isn't a Trading Tool. Jarvis Is a Trader.

| Manual Trading | Jarvis Full-Auto |
|----------------|-----------------|
| You watch the charts | It watches for you |
| You agonize over exits | It follows risk rules |
| You wake up at 3 AM to check | It runs 24/7 + Telegram notifies |
| You tweak parameters by gut feel | GA engine auto-optimizes |
| You manually calculate P&L | It generates daily reports |

**Jarvis isn't a tool that helps you trade. Jarvis is a trader that works for you.**

Your only jobs:
1. Check the daily report (8:00 AM Telegram)
2. Glance at the weekly GA evolution results
3. Peek at the Grafana dashboard occasionally

Everything else belongs to the code.

---

## Coming Up: Part 9

**"Wake Up to an AI-Written Trading Report Every Morning"**

We'll cover:
- How Jarvis auto-generates daily trading reports
- AI analysis of every trade — what worked and what didn't
- Auto-archiving knowledge in Obsidian
- Mining strategy improvements from daily reports

---

*This series draws on quant trading research from Cola AI Lab. Code samples are educational. Fully test any automated trading system before going live. Trading carries risk. This article does not constitute investment advice.*

---

> 💬 Would you trust AI to trade fully automated? What's your biggest concern? Let's discuss.

**Series Index:**
1. ✅ Spot vs. Contract: The Hidden Math
2. ✅ One-Command Environment Setup
3. 🔜 Free APIs + AI = Your Personal Bloomberg Terminal
4. 🔜 Trading Strategies Aren't Born From Gut Feelings
5. 🔜 Backtests That Look Amazing, Live Trades That Don't
6. ✅ Vibe Coding My Stop-Loss Cost Me a Month's Profit
7. ✅ Copying DeepSeek's Core Logic — I Rewrote My Strategy With Evolution
8. ✅ **[This Issue]** Making Money While You Sleep — Full Automation Goes Live
9. 🔜 Wake Up to an AI-Written Trading Report Every Morning
10. 🔜 Jarvis Is More Than a Trading Tool
