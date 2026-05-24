---
title: "Kalshi Review 2026: I Tested the CFTC-Regulated Prediction Market — Here's What You Need to Know"
date: 2026-05-23
tags: ["reviews", "prediction-markets", "fintech"]
categories: ["reviews"]
draft: false
---

## Quick Verdict

| Category | Rating |
|---|---|
| **Regulation & Trust** | ⭐⭐⭐⭐⭐ CFTC-regulated, US-legal in all 50 states |
| **Market Selection** | ⭐⭐⭐⭐ 100+ events, strong non-sports coverage |
| **Fees** | ⭐⭐⭐⭐⭐ No trading fees on most markets |
| **Liquidity** | ⭐⭐⭐ Strong during market hours, thin otherwise |
| **User Experience** | ⭐⭐⭐⭐ Clean UI, mobile app, API available |
| **Overall** | ⭐⭐⭐⭐ Best regulated prediction market for US traders |

**Bottom line up front:** Kalshi is the only CFTC-regulated prediction market where you can legally trade event contracts in all 50 US states — including California. If you want exposure to real-world events without touching Polymarket's crypto-only, geo-blocked platform, this is your best option. But it's not perfect: liquidity dries up outside US market hours, and sports multi-leg props are taking over the feed.

---

## What Is Kalshi?

Kalshi is a CFTC-regulated derivatives exchange where you can buy and sell event contracts — binary "yes/no" bets on real-world outcomes. Think of it as a regulated version of Polymarket, but with some key differences:

- **Regulated by the Commodity Futures Trading Commission (CFTC)**, which means full US legal compliance. No VPN required. No crypto wallet needed.
- **Event contracts settle in USD.** You deposit via ACH bank transfer, trade, and withdraw to the same bank account.
- **Each contract is binary**: a "yes" share pays $1.00 if the event happens, $0.00 if it doesn't. Price = probability implied by the market.
- **Available in all 50 states**, including California — which is rare for any form of event-based trading platform.

The platform launched in 2021 and has grown to feature 100+ markets across categories like politics, crypto prices, economic indicators, climate records, science breakthroughs, entertainment, and sports.

---

## How I Tested It

I wasn't going to write this review without putting real money on the line. Here's exactly what I did:

**Account Setup:** I created an account, completed KYC verification (standard for any CFTC-regulated exchange), and linked my bank account via Plaid for ACH deposits. The verification took about 5 minutes — faster than most crypto exchanges.

**Funding:** I deposited $10.02. Not a typo — ten dollars and two cents. I wanted to test the platform with a realistic small-account experience, which is how most people start. The ACH transfer cleared within 2 business days.

**Trading Activity:** Over two weeks, I placed orders across multiple markets:
- "Will Bitcoin close above $150,000 in May 2026?" — bought NO at $0.88
- "Fed funds rate target range in June 2026?" — spread across multiple outcomes
- A few short-term economic indicator markets

**API & CLI:** I set up kalshi-cli with RSA-signed API authentication. The CLI supports order management (place, cancel, list orders), market data queries, and position tracking. It works — but the documentation could be better. If you're comfortable with command-line tools, it's a solid way to interact with the exchange without the web UI.

**Total Time Invested:** Roughly 8–10 hours of active use across the web platform, mobile app, and CLI.

---

## Platform Walkthrough

### Web Platform

Kalshi's main interface is clean and intuitive. Here's what the category view looks like — note the clear navigation across Elections, Politics, Crypto, Climate, and more:

![Kalshi Crypto Markets](/images/kalshi-crypto.png)

And the Election markets page, showing political event contracts:

![Kalshi Election Markets](/images/kalshi-elections.png)

Here's a real market detail page — this was a live Trump impeachment contract showing the YES/NO pricing, price chart, and order placement interface:

![Kalshi Market Detail — Trump Impeachment](/images/kalshi-market-impeach.png)
The web interface is clean and functional. The main dashboard shows featured markets, your positions, and a market list you can filter by category. Order placement is straightforward: choose your side (YES/NO), set contract count and limit price, and submit.

The order book view is serviceable but not as rich as what you'd find on a traditional brokerage. You can see bid/ask spreads, recent trades, and depth — enough to make informed decisions, but market data nerds will want to use the API.

### Mobile App
Kalshi's mobile app (iOS and Android) mirrors the web experience reasonably well. It's useful for checking positions and placing quick orders. The push notification system for event settlements is reliable — I got notified within minutes when a market I was in resolved.

### API / kalshi-cli
This is where things get interesting for developers and automation-minded traders. Kalshi offers a REST API with RSA key-based authentication. The open-source `kalshi-cli` tool wraps this API and gives you:

- Market discovery and filtering
- Order placement, cancellation, and status
- Position and balance tracking
- Historical trade data

Authentication requires generating an RSA key pair and registering the public key with Kalshi. It's a one-time setup that takes about 10 minutes. Once configured, you can script trading strategies, build dashboards, or integrate Kalshi data into your own tools.

**The catch:** API rate limits are reasonable but not generous. If you plan to run high-frequency strategies, you'll hit walls quickly. This is a retail platform — it's not built for market-making bots.

---

## Pricing & Fees

This is one of Kalshi's strongest selling points:

| Fee Type | Cost |
|---|---|
| Trading fees | **$0** on most markets |

![Kalshi Fee Schedule — Zero Trading Fees on Most Markets](/images/kalshi-fees-table.png)
| Deposit (ACH) | **Free** |
| Withdrawal (ACH) | **Free** |
| Inactivity fee | **None** |
| Account minimum | **None** |

You read that right: zero trading commissions on most markets. Kalshi makes money on the spread — the difference between what buyers are willing to pay and what sellers are asking. For highly liquid markets (major political events, popular crypto price targets), the spread is tight — often 1–2 cents. For niche markets, spreads can be wider, which is effectively your "cost" to trade.

**Deposit note:** Only USD via ACH bank transfer is accepted. No crypto deposits. No credit cards. This is both a pro (regulatory compliance) and a con (slower funding than crypto platforms). If you're used to instant USDC deposits on Polymarket, the 1–2 day ACH delay will feel slow.

---

## Kalshi vs Competitors

### Kalshi vs Polymarket
This is the comparison everyone asks about.

| Feature | Kalshi | Polymarket |
|---|---|---|
| **Regulation** | CFTC-regulated | Unregulated (crypto) |
| **US Legal** | ✅ All 50 states | ❌ Geo-blocked in US |
| **Deposit** | USD (ACH) | USDC (crypto) |
| **Settlement** | Automatic, cash | Manual claim required |
| **Market Depth** | Moderate | Deeper for crypto/politics |
| **Fees** | None (spread only) | None |
| **Available In** | All 50 states incl. CA | Outside US only |

Polymarket has deeper liquidity on crypto and political markets — that's undeniable. But if you're in the US, Polymarket is geo-blocked, and using a VPN to access it puts your funds at risk. Kalshi's CFTC regulation means your money is held in segregated accounts with regulatory oversight.

### Kalshi vs PredictIt
PredictIt is the academic-focused prediction market run by Victoria University of Wellington. It operates under a CFTC "no-action" letter — not full regulation.

- **PredictIt limits:** $850 max position per market, limited market selection
- **PredictIt fees:** 10% on profits, plus withdrawal fees
- **Kalshi advantage:** No position limits (beyond your balance), no profit fees

### Kalshi vs Traditional Sports Betting
DraftKings and FanDuel take a 5–10% vig on every bet. Kalshi's zero-fee, spread-only model makes it fundamentally cheaper if you're trading, not gambling. But Kalshi is an exchange — you need a counterparty to fill your order. Sportsbooks always take the other side.

---

## Pros and Cons

### ✅ What I Liked

1. **Regulatory certainty.** As someone who's dealt with crypto exchange shutdowns and frozen accounts, knowing Kalshi is CFTC-regulated with proper segregation of customer funds is genuinely reassuring.

2. **Zero trading fees.** You keep 100% of your gains. The spread is your only cost, and on major markets, spreads are tight.

3. **Auto-settlement.** Markets resolve automatically based on the official data source. You don't need to "claim" anything — winning contracts are paid out in cash to your balance. Polymarket requires manual claiming, which is a bizarre UX choice.

4. **Diverse market categories.** Beyond the usual politics and crypto, Kalshi offers markets on climate records, science milestones, entertainment releases, and economic indicators. It's not just a betting site — it's a genuine event-based trading platform.

5. **Mobile app.** Functional, clean, push notifications work. Nothing revolutionary, but it does the job.

6. **API access.** The REST API and kalshi-cli tool make it possible to integrate Kalshi into automated workflows. Underrated feature for data-focused traders.

### ❌ What I Didn't Like

1. **Liquidity outside market hours.** This is the biggest practical problem. Non-sports event markets are most active during US market hours (Mon–Fri, roughly 9:30 AM–4 PM EST). Outside those hours — especially weekends — non-sports markets are thin. You'll see wide spreads, few resting orders, and it may take hours to fill a limit order at a fair price.

2. **Sports multi-leg props dominate the feed.** The main market feed is increasingly cluttered with complex sports prop bets (player X to score Y points + team Z to win, etc.). If you're here for economics, politics, or crypto markets, you have to filter past a lot of sports content. This isn't necessarily bad for the business (sports drive volume), but it dilutes the platform's identity as a prediction market.

3. **Weekend dead zone for non-sports.** Friday evening through Sunday, the non-sports markets are essentially on pause. Market makers pull back, spreads widen, and order books thin out. Plan your trading around market hours.

4. **ACH-only deposits.** I understand why — regulation requires it. But the 1–2 business day ACH delay feels archaic compared to crypto platforms. If you're trying to rush into a breaking-news trade, you need funds already on the platform.

5. **API documentation gaps.** The REST API works, but the official documentation is sparse in places. The kalshi-cli tool helps bridge the gap, but you'll likely need to read source code to understand some endpoints.

6. **Limited short-term markets.** Most markets have expiry dates weeks or months out. If you want same-day or next-day event contracts (like "Will X happen today?"), the selection is limited. Kalshi has been adding more short-duration markets, but the selection still lags behind Polymarket in this category.

---

## Who Should Use Kalshi?

### ✅ Good Fit For:
- **US-based traders** who want legal, regulated event-based trading
- **Macro/econ enthusiasts** who want to express views on economic indicators
- **Crypto traders** who want regulated exposure to price targets
- **Data-oriented people** who want API access to prediction market data
- **Small-account traders** — no minimums, no fees, $10 is enough to start

### ❌ Not A Good Fit For:
- **Non-US residents** — Polymarket or local alternatives may offer more markets
- **High-frequency traders** — API rate limits and market hours constrain strategies
- **Pure sports bettors** — DraftKings/FanDuel offer better liquidity and instant fills
- **Weekend warriors** — if you mainly trade on weekends, the liquidity situation will frustrate you

---

## A Note on Referrals

Kalshi runs an incentive program for referring new users. If you sign up through a referral link, both you and the referrer may receive bonus credits or cash rewards. The details change over time, so check the current promotion on the platform. I'm not going to hard-sell this — but if you're signing up anyway, you might as well claim any available sign-up bonus.

---

## Bottom Line

Kalshi is the best regulated prediction market available to US traders — and realistically, it's the *only* one with full CFTC oversight and 50-state availability.

For a $10.02 account, the platform works exactly as advertised. Zero fees mean your small balance doesn't get eaten by commissions. The auto-settlement system is reliable. The API gives you options beyond the web interface.

The liquidity situation is the real constraint. If you can trade during US market hours and focus on major event markets, you'll have a good experience. If you're trying to trade niche climate or science markets on a Saturday afternoon, you'll be staring at empty order books.

**My recommendation:** Start with $10–50. Trade a few markets to understand how event contracts work. Use the API if you're technically inclined — it's the most interesting part of the platform. And trade during market hours for the best fills.

[Try Kalshi →](https://kalshi.com)

*Disclaimer: I'm a Kalshi user with my own money on the platform. This review reflects my genuine experience. I may receive referral compensation if you sign up. Event contracts involve risk — only trade what you can afford to lose. Past results don't guarantee future outcomes.*
