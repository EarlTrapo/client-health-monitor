# Client Health Score & Churn Prediction System

> Built with n8n + Google Sheets + Telegram.
> A modular signal collection and scoring system that tells you which clients are about to churn — before they do.

[Demo on X](https://x.com/0xTrapo/status/2030235770053632421)

---

## What It Does

Most service businesses find out a client is leaving when they get the cancellation email. This system catches it weeks earlier.

It pulls signals from every touchpoint — payments, support, email, calendar — runs them through a weighted scoring engine, and fires a Telegram alert when a client's health score drops below threshold. Your team gets context, not just a name.

**Core behaviour:**
- Four independent connectors collect signals on a schedule from Stripe, Chatwoot, Gmail, and Google Calendar
- The Scoring Engine aggregates all signals, applies weighted scoring logic, and writes results to Google Sheets
- At-risk clients trigger a Telegram alert with score breakdown and contributing signals
- Google Sheets acts as the persistent data layer — full score history, queryable, no extra database needed

---

## Architecture — How the Pieces Fit

This is a multi-workflow system. Each connector runs independently and feeds into the central Scoring Engine.

```
Connector - Stripe        (payment events: failed charges, downgrades, MRR changes)
Connector - Chatwoot      (support signals: ticket volume, resolution time, sentiment)
Connector - Gmail         (email signals: response rate, last contact recency)
Connector - Calendar      (engagement signals: meeting frequency, no-shows, cancellations)
         ↓
Scoring Engine
  → Weighted score calculation per client
  → Write scores + signals to Google Sheets
  → IF score < threshold → Telegram alert with breakdown
```

**Why modular connectors:**
Each data source is its own workflow. You can add, disable, or swap a connector without touching the scoring logic. Stripe goes down? The other three still run. New data source? Add a connector, feed it into the engine.

---

## Scoring Logic

Each signal type carries a weight. The engine combines them into a single 0–100 health score per client.

| Signal | Source | Weight |
|---|---|---|
| Payment health | Stripe | High |
| Support ticket load | Chatwoot | Medium |
| Email engagement | Gmail | Medium |
| Meeting cadence | Google Calendar | Medium |

Clients below the alert threshold get flagged automatically. Threshold is configurable in the Scoring Engine workflow.

---

## Stack

| Layer | Tool |
|---|---|
| Automation platform | n8n |
| Payment signals | Stripe API |
| Support signals | Chatwoot API |
| Email signals | Gmail API |
| Calendar signals | Google Calendar API |
| Data layer | Google Sheets |
| Alerts | Telegram Bot API |

---

## Repo Structure

```
client-health-monitor/
├── scoring-engine.json         ← core scoring + alert logic
├── connector-stripe.json
├── connector-chatwoot.json
├── connector-calendar.json
├── connector-gmail.json
├── README.md
└── assets/
```

---

## How to Import

1. Download all JSON files from this repo
2. Open your n8n instance → **Workflows → Import from file**
3. Import each file separately — start with the connectors, then the Scoring Engine
4. Set credentials for each connector: Stripe, Chatwoot, Gmail, Google Calendar, Telegram
5. Create a Google Sheet with your client list as the data layer
6. Update the score threshold in the Scoring Engine to match your churn risk tolerance
7. Activate connectors first, then activate the Scoring Engine

---

## What Makes This Production-Ready

- **Modular by design** — connectors are decoupled from the scoring logic. Swap data sources without rebuilding the engine.
- **Weighted scoring, not binary flags** — a single missed payment doesn't tank a score. The engine weighs context.
- **Google Sheets as the data layer** — no extra database to maintain. Full score history is queryable by anyone on the team.
- **Alert includes breakdown** — the Telegram message doesn't just say "client at risk." It shows which signals drove the score down.

---

## Built By

Elvis — AI automation builder. I build client-facing systems with n8n and Claude.

GitHub: [github.com/EarlTrapo](https://github.com/EarlTrapo)
X: [@0xTrapo](https://x.com/0xTrapo)
