---
title: "Trading Strategies Aren't Gut Feelings — From Intuition to PRD"
subtitle: "A Four-Step Pipeline to Turn Your 'I Feel' Into Mathematical Proof"
date: 2026-05-23
draft: true
tags: ["trading", "AI", "quant", "jarvis", "strategy", "PRD"]
categories: ["jarvis-trading-series"]
description: "How to transform a raw trading intuition into a mathematically rigorous, machine-readable PRD using Gemini Deep Research, NotebookLM, and AI multi-agent roundtables — then hand it to Claude Code for execution."
---

## Part 1: Where 99% of Traders Die

Here's a sample of the messages I get:

> "Dude, I *feel* like BTC is about to pump. Should I chase?"
>
> "ETH is bouncing off the 20-day MA. *Looks like* a dip-buying opportunity?"
>
> "Some KOL said SOL is crashing. I shorted. Got liquidated..."

They all share one trait: **Every decision is based on "I feel."**

"I feel," "I think," "someone said" — in quantitative finance, these phrases have a formal academic name:

**Noise.**

Yes — the exact same high-order noise term from Part 3's Taylor expansion. Your "gut feeling" is mathematically equivalent to the discarded ε (error term) that wrecks your backtest.

But here's the real question: if gut feelings are useless, where do strategies actually come from?

That's what Part 4 is about. A complete pipeline to **translate intuition into mathematics, and mathematics into code.**

---

## Part 2: The Four-Step Pipeline — Intuition to PRD

Here's the full process:

```
                    ┌──────────────┐
                    │ ①Raw Intuition │
                    │ "I think shorting│
                    │  pumped altcoins │
                    │  could work"      │
                    └──────┬───────┘
                           │
                           ▼
                    ┌──────────────┐
                    │ ②Deep Research │
                    │ Search academic │
                    │ literature for   │
                    │ quant models     │
                    └──────┬───────┘
                           │
                           ▼
                    ┌──────────────┐
                    │ ③NotebookLM   │
                    │ Knowledge synth │
                    │ + AI roundtable  │
                    │ discussion       │
                    └──────┬───────┘
                           │
                           ▼
                    ┌──────────────┐
                    │ ④PRD Document │
                    │ Machine-readable│
                    │ spec → Claude   │
                    │ Code executes    │
                    └──────────────┘
```

---

### Step 1: Describe Your Intuition in Natural Language

I'll use a real case from Cola AI Lab video #6.

A friend of Cola's had a high-win-rate shorting strategy. The core intuition:

> "Find altcoins that pumped insanely in the short term. Once their momentum dies, open a low-leverage short. The underlying philosophy: all altcoins trend toward zero."

Good intuition? Yes. It has a clear **trading logic** (mean reversion) and an **underlying thesis** (zero-convergence hypothesis).

But also bad. It doesn't answer:
- "How much is 'pumped insanely'?" — no quantitative threshold
- "How do you define 'momentum dying'?" — no technical indicator
- "How much is 'low leverage'?" — no position sizing
- "When do you exit?" — no exit conditions

**These four sentences can make money — luck money. Quant converts it to certainty money.**

Feed the intuition to an AI assistant for initial decomposition:

```
Me: I need to build a quant strategy. The idea: find altcoins that have
pumped extremely in the short term. When their momentum dies, open a
low-leverage short. Philosophy: all altcoins trend toward zero.

AI:
This is a "shorting bubble mean-reversion strategy."

Mathematical Engine — Quantifying momentum exhaustion:
1. Pump threshold: >X% gain in past 3 or 7 days
2. Momentum exhaustion triggers (right-side entry, never short during pump):
   - Volume-price divergence: price at new highs but volume shrinking
   - Derivatives data: extremely positive funding rate + declining OI growth

Risk Layer:
3. Leverage × stop-loss mathematical coupling
4. Maximum acceptable loss per trade
```

One sentence of intuition, decomposed into **actionable technical dimensions**. But not yet precise enough. Next step: Deep Research.

---

### Step 2: Deep Research — Mining Academic Quant Models

This is the most critical step. You're not asking AI to make up indicators. You're asking it to pull **real quant models from academic finance literature**.

Prompt:

```
Below is my trading strategy framework. From mathematical and financial
perspectives, search academic literature for quant models to validate
and refine this strategy:

Core strategy: Short recently-pumped altcoins, betting on mean reversion
to zero. Underlying philosophy: exponential decay + Log-Periodic Power Law
(LPPL) model.

Specific requests:
1. Search mean-reversion strategy academic literature for best quant metrics
2. Search LPPL model applications in financial bubble prediction
3. Analyze mathematical definition of volume-price divergence — specifically
   the derivative-angle method within the same time window
4. Find academic support for funding rate as a short signal
5. Provide complete mathematical model formulas
```

Deep Research spent about 3 minutes pulling from Google Scholar, arXiv, and SSRN.

**Key findings:**

1. **LPPL Model (Log-Periodic Power Law)** — the bubble prediction model by Didier Sornette (ETH Zurich):

```
ln(p(t)) ≈ A + B(t_c - t)^m × [1 + C cos(ω ln(t_c - t) + φ)]
```

Where:
- `t_c` = critical (crash) time
- `m` = power law exponent (0 < m < 1; closer to 0 = more like exponential blowoff)
- `C` = log-periodic oscillation amplitude
- `ω` = log-angular frequency

Before a bubble crashes, prices exhibit accelerating oscillations (log-periodic fluctuations) — the optimal short signal.

2. **Volume-price divergence — derivative-angle method**:

```
Divergence = arctan(V'(t) / P'(t))
```

When price hits new highs but volume growth stalls, `V'(t)` → 0 while `P'(t)` remains positive. `arctan(0/P')` → 0°, near-zero angle = divergence. During healthy rallies, volume and price move together, producing larger angles.

3. **Funding rate + Open Interest** — persistently positive funding rate with stagnant OI growth signals crowded longs without fresh capital, a textbook short setup.

---

### Step 3: NotebookLM Synthesis + AI Multi-Agent Roundtable

Package all research results (paper abstracts, formulas, backtest data) into NotebookLM.

NotebookLM does three things for you:

**① Mind Map** — one-click knowledge structure of your entire strategy. Core concepts, quant metrics, and risk parameters interconnected at a glance.

**② CEO-speed reading** — "Give me a 10-minute strategy overview."

**③ Multi-agent roundtable** — the crown jewel from Cola AI Lab video #6.

Embed AI personas in your quant system for roundtable discussions:

```
Buffett (Value Investing):
"Shorting bubble altcoins is fundamentally shorting human greed. This is
the same logic I used to short the 'tronics boom' in the 1960s. Only
difference: that bubble lasted 3 years. Crypto bubbles compress into 3
weeks. Margin for error is far smaller. My advice: never size a position
larger than what lets you sleep at night."

Gauss (Mathematics):
"The beauty of the LPPL model is that it doesn't predict direction — it
predicts instability. As m → 0, the frequency of log-periodic oscillations
increases dramatically — the system is approaching a critical point. It's
not 'will it crash' but 'when will it crash' with increasing certainty.
But note: LPPL's out-of-sample accuracy is only 60-70%. There's still a
30% false-positive rate."

Naval (Philosophy + Entrepreneurship):
"All these models do the same thing — find certainty inside uncertainty.
But here's my warning: the most expensive lesson isn't the money you lose.
It's the wrong lesson you learn from losing. Lose once and quit shorting?
That's throwing the baby out with the bathwater. Lose once and double
leverage to recover? That's gambling. Lose once and improve your stop-loss
+ shrink position sizes + tighten filters? That's iteration."
```

**Newton (Final Judge):**

"Synthesizing all three: I determine the current X-factor value at 0.73 (1.2 standard deviations from mean). Recommendation: observation mode, no live trading. First verify your LPPL m-value is truly between 0 and 0.5. If sample m exceeds 0.5, you may be fitting a bubble that never existed."

This multi-agent roundtable isn't just entertaining — it addresses **cognitive bias**. Everyone (including AI) has blind spots. Running your strategy through Buffett's lens, Gauss's lens, and Naval's lens simultaneously prevents "resonant self-deception" — believing your own story because no one challenged it.

---

### Step 4: Generate PRD → Claude Code Development

The final step: crystallize all analysis into a **machine-readable PRD document**.

Regular requirements docs are for humans. A PRD for Claude Code must satisfy three criteria:

> 1. **Precise problem statement** — no "feels like," "maybe," "probably"
> 2. **Clear architectural constraints** — what can change, what can't, which modules to interface
> 3. **Quantifiable acceptance criteria** — what counts as "success"

Here's a standard strategy PRD template, ready to hand to Claude Code:

```markdown
# PRD: Bubble Altcoin Short Strategy v1.0

## Background
Under the mean-reversion hypothesis, altcoins that pump aggressively
in the short term will retrace once momentum exhausts. This strategy
identifies bubble-top signals and opens shorts.

## Core Hypotheses
- H1: Altcoin long-term value converges toward zero
- H2: LPPL model reliably predicts bubble-top time windows
- H3: Volume-price divergence + extreme funding rate = high-probability short

## Quantitative Rules

### Entry Conditions (ALL must be met)
1. LPPL m-value < 0.5 AND t_c - now < 48 hours
2. Volume-price divergence (arctan method) < 15°
3. Funding rate > 0.1% (annualized > 87.6%)
4. Open Interest growth rate < 5% / 24h
5. Market cap > $100M (exclude dust coins)

### Position Sizing
- Initial position: 2% of total capital
- Max leverage: 2x
- Max loss per trade: 1% of total capital

### Exit Conditions (ANY trigger)
1. Take profit: price closes below 7-day MA
2. Stop loss: price breaches prior high + 1%
3. Time stop: position held > 72 hours
4. Funding rate reverts to zero or negative

### Risk Circuit Breakers
- Daily max loss > 5% → halt trading today
- Weekly max loss > 10% → pause for the week
- Max concurrent positions: 3
- Blacklist: 3 consecutive stops on same ticker → permanent exclusion

### Acceptance Criteria
- Backtest win rate > 55%
- Profit factor > 1.5:1
- Max drawdown < 15%
- Sharpe ratio > 1.0
```

The power of this PRD:
- Claude Code reads it with **zero ambiguity** — every condition has a precise numeric threshold
- Every metric has a **mathematical origin** (LPPL from Sornette papers, divergence from derivative-angle method)
- Acceptance criteria are **quantitatively testable** — run the backtest and you immediately know pass/fail

---

## Part 3: PRD Essence — Writing "Legal Code" for AI

Many people misunderstand PRDs as "detailed development docs." Wrong.

A PRD's essence is **constraint**. It tells AI (and future you):

- ✅ **What you CAN do**: free creativity within these boundaries
- ❌ **What you CANNOT do**: these red lines are absolute
- 📐 **What success looks like**: acceptance criteria = passing grade

A good PRD reads like legal code — no ambiguity, no gray areas. Any two rational people reach the same understanding.

A bad PRD reads like poetry — full of metaphors, white space, and "you know what I mean."

**In the Vibe Coding era, PRD quality directly determines code quality.** Feed AI a fuzzy requirement and you get fuzzy code. Feed it a precisely defined PRD and the output can exceed human programmer quality.

---

## Part 4: A Worked Example — BTC Spot Strategy PRD

So you have something actionable immediately, I ran the pipeline to generate a **BTC spot DCA + trend enhancement strategy** PRD:

```markdown
# PRD: BTC Spot DCA + Trend Enhancement v1.0

## Background
Based on Part 1's 5,000-evolution-experiment conclusion: spot-only,
long-only, simplest strategy wins.

## Core Rules

### DCA Layer (70% of capital)
- Fixed weekly BTC DCA purchase
- Execution: every Monday UTC 00:00
- Rebalancing: quarterly check

### Trend Enhancement Layer (30% of capital)
Entry (ALL must be met):
1. BTC price > 50-day MA
2. Volume > 20-day average × 1.2
3. 30-day volatility < 60-day volatility (volatility convergence)
4. 2 consecutive up-close days

Exit (ANY trigger):
1. BTC drops below 50-day MA → sell all enhancement position
2. Volume shrinks below 20-day average × 0.5
3. Max hold: 14 days
4. Enhancement entry: 10% per signal

### Risk Controls
- Total position never exceeds 100%
- Drop below 200-day MA → pause DCA, resume on reclaim

### Acceptance Criteria
- 12-month backtest > DCA baseline + 15%
- Max drawdown < 20%
- Trade count > 12 (excludes "lucky one-shot")
```

---

## Part 5: Why Most People Get Stuck at Step 2

Observing the quant traders around me, nearly all of them get stuck at the same place:

**Step 2: Using Deep Research to mine quant models.**

It's not a capability problem. It's an **impatience** problem.

Most people jump straight into Cursor and start coding. Three lines in, they switch strategies. Five strategies written in a week. Zero backtest results worth looking at.

What's wrong? **No knowledge accumulation upfront.**

Cola AI Lab video #6 has a line I strongly endorse:

> "Don't open Cursor and blindly write code. First describe your trading intuition in natural language. Then use Deep Research to pull professional quant metrics and financial logic for dimensionality reduction. Finally, crystallize these rigorous rules into an AI-readable PRD."

The time allocation for these four steps should be:

```
Intuition articulation: 10% (30 min)
Deep Research: 50% (2-3 hours) ← the most important investment
NotebookLM + roundtable: 25% (1 hour)
PRD document: 15% (30 min)
```

Most people invert this ratio: 10% thinking about strategy, 90% writing code + debugging.

**Strategy design is intellectual labor. Code implementation is manual labor. AI has already automated the manual labor. You should spend all your brainpower on the first three steps.**

---

## Part 6: Strategy PRD vs. Code — Which Is More Valuable?

Question:

The "Bubble Altcoin Short PRD" (above) vs. its corresponding `collector.py` code file. Which is more valuable?

Answer: **the PRD.**

Why:
- **PRD is reusable knowledge IP.** Today's altcoin short strategy can become tomorrow's meme-stock short or small-cap A-share short — same logic, different tickers.
- **Code is consumable.** Exchange API changes → code breaks. Dependency updates → errors. Strategy logic changes → rewrites.
- **PRD is directly "understood" and executed by AI.** Hand the PRD to Claude Code and it can implement the strategy on any exchange. But hand the code to another AI and ask it to "understand the strategy logic" — that's a reading comprehension test, not a translation.

**My advice: manage your strategy PRDs as IP.**

Store each strategy PRD in Obsidian, tag it, review periodically. A good PRD cross-markets, cross-cycles reusable. That's true "knowledge compounding."

---

## Part 7: Summary — Four Steps from "I Feel" to "Mathematically Proven"

| Step | What | Tool | Time |
|:--|:--|:--|:--|
| ① Intuition | Describe strategy in plain language | Any LLM | 30 min |
| ② Deep Research | Mine academic models + quant metrics | Deep Research | 2-3 hours |
| ③ Knowledge Synthesis | Mind map + CEO-speed-read + AI roundtable | NotebookLM | 1 hour |
| ④ PRD Document | Crystallize into machine-readable spec | Claude Code | 30 min |

Total: 4-5 hours. Output: a trading strategy implementable on any market, any exchange.

**This isn't a gut feeling. This is mathematics.**

---

## Coming Up: Part 5

*"Backtests Look Perfect, Live Accounts Bleed — Why 90% Die at This Step"*

We'll use Taylor expansion to expose backtest overfitting traps:
- Constant noise → you've handled it
- First/second derivative noise → strategy captures core signals
- High-order noise → **this is what kills your live account**

And: **How to make Jarvis auto-detect if your strategy is overfitting.**

---

*This series is based on Cola AI Lab's quantitative trading research. All code is open source. This is not investment advice.*

---

> 💬 How do you develop your trading strategies? Gut feeling? Indicators? Math? Share below.

**Series Index:**
1. ✅ Spot vs. Contracts — The Brutal Math
2. ✅ One-Click Environment Setup
3. ✅ Free APIs + AI = Your Bloomberg Terminal
4. ✅ **This Post** — Strategies Aren't Gut Feelings
5. 🔜 Backtests That Look Perfect, Live Accounts That Bleed
