---
title: "Copying DeepSeek's Core Logic — I Rewrote My Trading Strategy With Evolution"
subtitle: "Why 10,000 Lines of Code Lost to 30 Lines of Minimal Parameters"
date: 2026-05-23
draft: true
tags: ["trading", "AI", "quant", "jarvis"]
categories: ["jarvis-trading-series"]
description: "DeepSeek's philosophy applied to trading: don't teach AI how to think. Define a reward function and let compute power do the rest. Here's how a genetic algorithm in Go found a strategy no human would have imagined."
---

## Part 1: First, Some Context — I Wrote Tens of Thousands of Lines of "Clever" Code

Before I discovered evolutionary algorithms, my strategy looked like this:

```
Strategy 1: MACD golden cross + RSI oversold + Bollinger lower band → Long
Strategy 2: EMA triple alignment + volume breakout → Add position
Strategy 3: KDJ bullish divergence + Fibonacci 0.618 support → Bottom-fish
Strategy 4: ... (7 more strategies follow)
```

Eleven strategies. Each with 8-15 parameters. **Tens of thousands of lines of Python.** Dozens of technical indicators computed in real-time.

Backtest results? Gorgeous. Sharpe ratio 2.3, max drawdown 8%.

Live results? **Losing money.**

Why? Because I used tens of thousands of lines of code to fit noise.

---

## Part 2: DeepSeek's Insight — Never Teach AI *How* to Think

Cola AI Lab's fifth video is titled "Copying DeepSeek's Core Logic." In it, one idea hit me like a lightning bolt:

> **DeepSeek's core philosophy: Never teach AI how to think. Just define an extremely simple reward function, and let the machine go crazy with trial and error inside the compute.**

Translating this to quantitative trading:

❌ Old way: Design a strategy → Tune parameters → Backtest → Tweak parameters → Backtest → ... (You're *teaching* the system how to make money)

✅ New way: Define a goal (maximize Sharpe / minimize drawdown) → Feed in a few sparse parameters → Let compute find the optimal combination (You're *defining* what "good" means)

**The difference: the first way, you're guessing how to make money. The second way, you let evolution verify what can survive.**

---

## Part 3: Minimal Parameters — Compressing Complexity Into a Few Numbers

Step one of the DeepSeek philosophy: **compression.**

My 11 strategies had roughly 120 parameters. Now, I compressed the entire system into 7:

```python
# Minimal strategy parameter space
STRATEGY_PARAMS = {
    "lookback_period":  (10, 100),     # Lookback window (candlestick count)
    "entry_threshold":  (0.01, 0.10),  # Entry threshold (% price deviation)
    "exit_threshold":   (0.02, 0.15),  # Exit threshold
    "volume_filter":    (0.5, 3.0),    # Volume filter multiplier
    "volatility_filter":(0.5, 2.5),    # Volatility filter multiplier
    "position_size":    (0.1, 1.0),    # Position sizing
    "trend_weight":     (0.0, 1.0),    # Trend weight (0 = mean reversion, 1 = trend following)
}
```

**Seven parameters. Each a continuous range, not a binary switch.**

This means:
- Not "use MACD? yes/no" — it's `trend_weight = 0.47`, a continuous blend
- Not "stop-loss at 5%" — it's `exit_threshold = 0.067`, precise to three decimal places
- Not "buy on high volume" — it's `volume_filter = 1.73`, an exact multiplier

**This is "minimalism forces complexity" — a handful of parameters covering an infinite landscape of strategies.**

---

## Part 4: Genetic Algorithm — Letting Strategies Evolve in Compute

With the parameter space defined, the next step is evolution.

A Genetic Algorithm (GA) simulates natural selection:

```
Initialize: randomly generate 1,000 individuals (7 params each = 1 strategy)

Loop for 100 generations:

  Step 1 [Evaluate]: Backtest every individual on training data → fitness (Sharpe ratio)
  
  Step 2 [Select]: Top 20% by fitness survive. The rest: eliminated.
  
  Step 3 [Crossover]: Survivors pair up → swap parameter segments → produce offspring
  
  Step 4 [Mutate]: Offspring parameters get random tweaks (±5%) → introduce diversity
  
  Step 5 [Cull]: Add offspring to population, eliminate the worst 10%
  
  Repeat...
```

**The key isn't the algorithm itself — it's the philosophy:**

> You're not searching for the "highest peak." You're searching for "what can survive." Evolution is survival philosophy.

A parameter set that makes 500% in a bull market but loses 90% in a bear market? **It can't survive.**
A parameter set that makes 15% in both, with a 12% max drawdown? **It survives.**

**GA doesn't select the smartest strategy. It selects the most resilient one.**

---

## Part 5: Go With Concurrency — 1,000 Strategies Evolving Simultaneously

Python has a fatal flaw for GA: **it's slow.**

1,000 individuals × 100 generations × 3 years of backtest data per evaluation = astronomical compute. In single-threaded Python: roughly **47 hours**.

Cola AI Lab's solution: **rewrite the evolution engine in Go.**

Why Go?
1. **Native concurrency** — goroutines are lightweight threads. 1,000 individuals backtested simultaneously
2. **Compiled language** — identical logic runs 10-50× faster than Python
3. **Memory efficiency** — no DataFrame overhead, direct array operations

Architecture:

```go
// GA Evolution Engine (Go implementation)

type Genome struct {
    LookbackPeriod   float64
    EntryThreshold   float64
    ExitThreshold    float64
    VolumeFilter     float64
    VolatilityFilter float64
    PositionSize     float64
    TrendWeight      float64
    Fitness          float64  // Sharpe ratio
}

func Evolve(population []Genome, generations int, data []OHLCV) []Genome {
    for gen := 0; gen < generations; gen++ {
        // Parallel evaluation of all individuals
        var wg sync.WaitGroup
        results := make(chan Genome, len(population))
        
        for _, g := range population {
            wg.Add(1)
            go func(genome Genome) {
                defer wg.Done()
                genome.Fitness = backtest(genome, data)  // Independent goroutine
                results <- genome
            }(g)
        }
        
        wg.Wait()
        close(results)
        
        // Collect, select, crossover, mutate...
        population = selectCrossoverMutate(results)
    }
    return population
}
```

**1,000 simultaneous backtests. 47 hours compressed to 2.5 hours.**

---

## Part 6: The Reward Function Is the Soul

Here's the thing most people get wrong.

They spend 80% of their energy designing strategy logic (entry conditions, exit conditions, filters) and 20% defining "what good means."

**DeepSeek's philosophy says: flip it.**

80% of your energy should go into the reward function.

What makes a good reward function? Not simply "maximize total return."

```python
def fitness_function(trades, equity_curve):
    """
    Composite fitness: not "makes the most money" but "makes money most reliably."
    
    This function IS the soul of the GA.
    """
    # 1. Sharpe ratio (risk-adjusted return)
    returns = np.diff(equity_curve) / equity_curve[:-1]
    sharpe = np.mean(returns) / np.std(returns) * np.sqrt(252)
    
    # 2. Max drawdown penalty (bigger DD → bigger penalty)
    max_dd = calculate_max_drawdown(equity_curve)
    dd_penalty = max(0, max_dd - 0.15) * 2  # Heavy penalty beyond 15% DD
    
    # 3. Win-rate stability (don't want high variance in W/L ratio)
    win_rate_std = np.std(rolling_win_rate(trades, window=20))
    stability_bonus = 1.0 / (1.0 + win_rate_std)
    
    # 4. Trade frequency penalty (avoid overtrading)
    trade_count = len(trades)
    frequency_penalty = max(0, trade_count - 200) * 0.001
    
    # Composite score
    fitness = sharpe - dd_penalty + stability_bonus - frequency_penalty
    
    return fitness
```

**This function defines "what a good strategy is":**
- High Sharpe ratio (good risk-adjusted returns)
- Small max drawdown (you won't get shaken out mid-journey)
- Stable win rate (won't make you question your life choices)
- Low trade frequency (save fees, save sanity)

**Your reward function = your values. What you reward, the system evolves toward.**

---

## Part 7: My GA Evolution Results

I threw my 7 parameters into the Go engine for 100 generations. Here's what happened:

```
Generation 0:   Best fitness 0.47  | Average fitness 0.12
Generation 10:  Best fitness 0.83  | Average fitness 0.41
Generation 20:  Best fitness 1.12  | Average fitness 0.68
...
Generation 80:  Best fitness 1.47  | Average fitness 1.21
Generation 100: Best fitness 1.52  | Average fitness 1.28
```

The final evolved parameters:

```python
BEST_GENOME = {
    "lookback_period":   47.3,    # 47-candle lookback
    "entry_threshold":   0.034,   # 3.4% price deviation entry
    "exit_threshold":    0.071,   # 7.1% exit threshold
    "volume_filter":     1.73,    # 1.73× average volume filter
    "volatility_filter": 1.24,    # 1.24× average volatility filter
    "position_size":     0.62,    # 62% position sizing
    "trend_weight":      0.47,    # 47% trend + 53% mean reversion
}
```

**The fascinating one is `trend_weight = 0.47`.**

This value evolved on its own. Nobody told the system to use it. 0.47 means: **ride the trend, but keep half your conviction for mean-reversion plays.**

That's the beauty of evolutionary algorithms — they find combinations no human intuition would imagine.

---

## Part 8: Evolution Is Survival Philosophy

Let me share the line from Cola AI Lab's video that hit me hardest:

> **"The essence of evolution is survival philosophy, not peak-finding."**

This has three layers:

**Layer 1: In the market, survival is priority one.**

If a strategy gets liquidated in a bear market, its bull market returns don't matter — because it never lives to see the next bull. GA automatically eliminates high-drawdown strategies, not because they're unprofitable, but because they "can't survive."

**Layer 2: Don't seek the optimal solution. Seek the robust solution.**

The optimal solution is optimal only on historical data. The robust solution survives across all market regimes. GA's output isn't "the parameters that made the most money in the past." It's "the parameters that performed stably across different market slices."

**Layer 3: Diversity itself is value.**

GA preserves population diversity (through mutation and crossover). This prevents "over-specialization" to a single market regime. When market conditions shift, a diverse parameter space has more pathways to survival.

**Trading and evolution share a common truth: it's not the strongest or the smartest that survive — it's the most adaptable.**

---

## Part 9: Summary — Four Cognitive Leaps From Complex to Minimal

| Old Belief | New Belief |
|------------|------------|
| More complex strategies win | Fewer parameters win — minimalism forces complexity |
| Teach AI how to make money | Define "what's good" — let compute find it |
| Find historically highest-return parameters | Find parameters that survive all market conditions |
| Best strategy = highest return | Best strategy = longest survival |

**One sentence: stop teaching the system how to think. Define the reward function, then hand everything to evolution.**

---

## Coming Up: Part 8

**"Making Money While You Sleep — Jarvis Full Automation Goes Live"**

We'll connect every module from the first seven episodes:
- OpenClaw cron: scheduled scanning + auto order placement
- Real-time risk control module guarding every trade
- Telegram notifications + Obsidian daily reports
- 24/7 unattended operation

**Follow the series. Don't miss it.**

---

*This series draws on quant trading research from Cola AI Lab. GA evolution data is from real experimental results. Code samples are educational. Trading carries risk. This article does not constitute investment advice.*

---

> 💬 Have you tried evolutionary algorithms for strategy optimization? How did it go? Let's discuss.

**Series Index:**
1. ✅ Spot vs. Contract: The Hidden Math
2. ✅ One-Command Environment Setup
3. 🔜 Free APIs + AI = Your Personal Bloomberg Terminal
4. 🔜 Trading Strategies Aren't Born From Gut Feelings
5. 🔜 Backtests That Look Amazing, Live Trades That Don't
6. ✅ Vibe Coding My Stop-Loss Cost Me a Month's Profit
7. ✅ **[This Issue]** Copying DeepSeek's Core Logic — I Rewrote My Strategy With Evolution
8. 🔜 Making Money While You Sleep — Full Automation Goes Live
