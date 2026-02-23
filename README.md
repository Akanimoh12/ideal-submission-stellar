# iPredict

**Predict. Win or Lose — You Always Earn.**

A decentralized prediction market on **Stellar/Soroban** where users bet XLM on YES/NO outcomes, winners split the pool, and **every participant** earns points + IPREDICT tokens — whether they win or lose. Fully onchain referral system and leaderboard drive viral growth.

![CI/CD](https://github.com/Akanimoh12/iPredict/actions/workflows/ci.yml/badge.svg)
![Stellar](https://img.shields.io/badge/Stellar-Soroban-7C3AED?logo=stellar&logoColor=white)
![License](https://img.shields.io/badge/license-MIT-green)

---

## Overview

- Users bet XLM on YES/NO prediction markets
- Winners split the pool (minus 2% platform fee)
- **All participants earn rewards** — winners AND losers
- IPREDICT token minted as reward via inter-contract calls
- Onchain referral system: earn 2% of every bet your referrals place
- Onchain leaderboard ranked by points

### Why Stellar?

- **5-second finality** — bets confirm instantly
- **< $0.01 fees** — micro-bets of 1 XLM are practical
- **Rust/Soroban contracts** — type-safe, no reentrancy
- **Built-in asset support** — XLM native + custom tokens via SAC

---

## Reward System

Every user who participates in a resolved market earns rewards, regardless of outcome. This keeps users engaged and coming back.

| Outcome | Points | IPREDICT Tokens |
|---------|--------|-----------------|
| **Win** (correct prediction) | **30 points** | **10 IPREDICT** |
| **Loss** (wrong prediction) | **10 points** | **2 IPREDICT** |

### Payout Formula (Winners Only)

```
Total Pool   = All YES Bets + All NO Bets
Platform Fee = 2% of Total Pool
Winner Pool  = Total Pool - Platform Fee
User Payout  = (User Bet / Winning Side Total) × Winner Pool
```

### Worked Example

```
Market: "Will XLM hit $0.50 by Friday?"

YES bets: 500 XLM (10 users)    NO bets: 300 XLM (6 users)
Total Pool: 800 XLM → Fee: 16 XLM → Winner Pool: 784 XLM

Outcome: YES wins ✅

Alice bet 50 XLM on YES (winner):
  → Payout:  (50/500) × 784 = 78.4 XLM  (+56.8% profit)
  → Earns:   30 points + 10 IPREDICT tokens

Dave bet 30 XLM on NO (loser):
  → Payout:  0 XLM (lost his bet)
  → Earns:   10 points + 2 IPREDICT tokens  ← still rewarded!

Alice was referred by Bob:
  → Bob earned 1 XLM (2% of Alice's 50 XLM bet) automatically
```

---

## Core Features

| Feature | Description |
|---------|-------------|
| **Prediction Markets** | Admin creates YES/NO markets with deadlines |
| **XLM Betting** | Users stake XLM on their prediction |
| **Auto Payout** | Winners claim proportional share of the pool |
| **Points + Tokens for All** | Win = 30 pts + 10 IPREDICT, Lose = 10 pts + 2 IPREDICT |
| **IPREDICT Token** | Platform token minted via inter-contract call on claim |
| **Onchain Leaderboard** | Top 50 ranked by points, fully onchain |
| **Onchain Referrals** | Permanent 2% commission on all referred bets |
| **Market Browser** | Filter by active, ending soon, resolved |
| **Activity Feed** | Live stream of bets and claims |

---

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Smart Contracts | Rust / Soroban SDK 20.x |
| Frontend | React 18 + TypeScript + Vite |
| Wallet | Freighter, xBull, Albedo via `@creit.tech/stellar-wallets-kit` |
| Stellar SDK | `@stellar/stellar-sdk` |
| Hosting | Vercel |
| CI/CD | GitHub Actions |

---

## Smart Contract Architecture

Four Soroban contracts connected via inter-contract calls:

```
┌──────────────────────────────────────────┐
│            iPredict System               │
│                                          │
│  ┌──────────────────┐                    │
│  │ PredictionMarket │  (core logic)      │
│  └───────┬──────────┘                    │
│          │ inter-contract calls          │
│    ┌─────┼──────────┐                    │
│    ▼     ▼          ▼                    │
│  ┌─────┐ ┌────────┐ ┌──────────────┐    │
│  │Refer│ │Leader- │ │  IPREDICT    │    │
│  │ral  │ │board   │ │  Token       │    │
│  └─────┘ └────────┘ └──────────────┘    │
└──────────────────────────────────────────┘
```

### Inter-Contract Flow

```
place_bet(market_id, YES, 100 XLM)
  ├─ Transfer 100 XLM → contract (SAC)
  ├─ ReferralRegistry.credit(user, 2 XLM)     ← inter-contract
  └─ Store bet, update market totals

resolve_market(market_id, YES)
  └─ Mark market resolved onchain

claim(market_id)  ← called by ALL users (winners + losers)
  ├─ If winner: transfer payout XLM
  ├─ Leaderboard.add_pts(user, 30 or 10)      ← inter-contract
  └─ IPredictToken.mint(user, 10 or 2)        ← inter-contract
```

### Key Contract Functions

| Contract | Function | Description |
|----------|----------|-------------|
| **Market** | `create_market` | Admin creates YES/NO market |
| **Market** | `place_bet` | User bets XLM, auto-credits referral |
| **Market** | `resolve_market` | Admin declares outcome |
| **Market** | `claim` | User claims rewards (winners get XLM + tokens, losers get tokens) |
| **Token** | `mint` | Mint IPREDICT tokens to user (called by Market contract) |
| **Referral** | `register_referral` | Set referrer (one-time) |
| **Referral** | `credit` | Auto-pay 2% to referrer on each bet |
| **Leaderboard** | `add_pts` | Award 30 (win) or 10 (loss) points |
| **Leaderboard** | `get_top_players` | Get sorted top-N ranking |

---

## Frontend Pages

| Page | Purpose |
|------|---------|
| **Landing** | Hero, featured markets, how-it-works, stats |
| **Markets** | Browse/filter/search all markets |
| **Market Detail** | Bet panel, odds bar, countdown, activity feed |
| **Leaderboard** | Top 50 by points with win rates |
| **Profile** | Bet history, points, IPREDICT balance, referral link |
| **Admin** | Create markets, resolve outcomes, view fees |

---

## Deployment

```bash
# Build & deploy contracts
stellar contract build
stellar contract deploy --wasm target/.../prediction_market.wasm --network testnet
stellar contract deploy --wasm target/.../ipredict_token.wasm --network testnet
stellar contract deploy --wasm target/.../referral_registry.wasm --network testnet
stellar contract deploy --wasm target/.../leaderboard.wasm --network testnet

# Initialize with inter-contract links
stellar contract invoke --id $MARKET_ID --network testnet \
  -- initialize --admin $ADMIN --token $TOKEN_ID \
  --referral $REFERRAL_ID --leaderboard $LEADERBOARD_ID

# Frontend
npm install && npm run dev     # development
npm run build                  # production → Vercel auto-deploys
```

---

## User Acquisition (20 Users in 24h)

| Channel | Action |
|---------|--------|
| Twitter/X | Thread with screenshots + demo link |
| Stellar Discord | Share in #showcase |
| Referral chain | First 5 users refer 2+ each (2% incentive) |
| Telegram | Nigerian crypto groups |
| Direct | WhatsApp friends with faucet link |

**Seed markets before launch:** XLM price predictions, sports, pop culture, crypto events — things people have strong opinions on.

---

## Roadmap

- **MVP:** Markets, betting, claim, IPREDICT token rewards, referrals, leaderboard
- **v2:** User-created markets, oracle auto-resolution, categories
- **v3:** IPREDICT governance staking, mobile app, cross-chain deposits
- **v4:** Mainnet launch with real XLM

---

## License

MIT

---

*Built on Stellar/Soroban — Level 5 Black Belt MVP*  
*Author: Akanimoh | 2026*
