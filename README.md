# Rootly — Canadian Financial Health App for Newcomers

**Live:** [rootly.ca](https://rootly.ca) · [app.rootly.ca](https://app.rootly.ca)

---

## What is Rootly?

Rootly is a free financial health platform built specifically for newcomers to Canada. Unlike traditional credit bureaus that require months or years of Canadian credit history, Rootly generates a **CanadaReady Score™** based on actual banking behaviour — income consistency, cash flow, savings, and spending patterns.

---

## The Problem

When newcomers arrive in Canada, they face a catch-22:
- Banks won't approve credit without Canadian credit history
- You can't build credit history without being approved first

Rootly breaks this cycle by giving newcomers a verifiable, behaviour-based financial health score from day one.

---

## Tech Stack

| Layer | Technology |
|---|---|
| Frontend | Vanilla HTML/CSS/JavaScript (no frameworks) |
| Backend | Node.js v22 (no Express — pure http module) |
| Database | SQLite with AES-256-GCM encryption at rest |
| Auth | Passwordless magic link (JWT, httpOnly cookies) |
| Bank Data | Plaid open banking API (production approved, keys pending activation) |
| AI Scoring | Anthropic Claude (recommendations only) |
| Email | Resend API |
| Infrastructure | Ubuntu 24.04, Nginx, Systemd, DigitalOcean |
| SSL | Let's Encrypt (auto-renewing) |

---

## Architecture

```
rootly.ca          →  Static landing page (Nginx)
app.rootly.ca      →  Static frontend + API proxy (Nginx → Node.js :4001)
                       ├── Magic link auth
                       ├── Plaid bank connect
                       ├── Score engine
                       └── SQLite database
```

---

## Core Features

### Authentication
- Passwordless magic link flow — no passwords to leak or forget
- JWT sessions (1hr expiry) in httpOnly secure cookies
- 15-minute inactivity timeout (banking-grade)
- Page visibility detection — logs out when tab is switched for 15+ minutes
- Rate limited: 3 magic link requests per email per hour

### Plaid Bank Integration
- Plaid Link SDK for secure bank connection
- OAuth redirect support for Canadian banks (TD, RBC, BMO, CIBC, Scotiabank)
- Webhook handler for real-time transaction updates
- Update mode for expired bank sessions
- Transaction pagination (up to 5,000 transactions)
- Duplicate item detection
- AES-256-GCM encrypted access tokens at rest

### CanadaReady Score™ Engine
Privacy-preserving two-stage pipeline:

**Stage 1 — Local deterministic engine (no external API)**
Raw transactions → 9 anonymous aggregate metrics → weighted score (0-100)

Metrics calculated:
- Average monthly income
- Income consistency %
- Overdraft count
- Savings trend
- Recurring payment activity
- Discretionary spend ratio
- Minimum balance frequency
- Months of data
- Days of history

Scoring weights: Income consistency 25% · Avg income 20% · Recurring payments 20% · Min balance 15% · Overdrafts 10% · Discretionary 5% · Savings 5%

**Stage 2 — Claude AI (recommendations only)**
Only the 9 anonymous metrics are sent to Claude — no raw transaction data, no merchant names, no account numbers. Raw data is explicitly nulled after metric extraction.

### Security
- All PII encrypted with AES-256-GCM (names, Plaid tokens, score metrics, email addresses)
- HMAC-SHA256 email hashing for database lookups — emails stored encrypted, never in plaintext
- Magic link tokens hashed with SHA-256 before storage
- UFW firewall (SSH rate-limited, only 80/443 public)
- Nginx security headers (HSTS preload, X-Frame-Options, X-Content-Type-Options, Referrer-Policy)
- SSH password authentication disabled
- Monthly automated vulnerability scans (Lynis + npm audit)
- AES-256 encrypted daily database backups
- Anomalous access alerts (new IP/device detection)
- Zero secrets in source code — all in systemd environment variables

### Score Features
- Borrowell-style SVG line chart with score history
- 6 metric breakdown cards (Income Stability, Cash Flow, Savings, Recurring Payments, Overdraft Activity, Spending Discipline)
- 30-day minimum history threshold before first score
- Score smoothing (70/30 blend with previous score)
- Bi-weekly automated score refresh
- Email notifications only when score improves ≥3 points

### Partner Integrations
- **KOHO Financial** — score-tiered card recommendations (Core, Extra)
- Credit Cards page with eligibility badges based on CanadaReady Score™
- Affiliate disclosure compliant with PIPEDA and CASL

### Automated Jobs (Systemd timers)
| Job | Schedule | Purpose |
|---|---|---|
| Score refresh | 1st & 15th monthly | Recalculates scores for all connected users |
| Re-engagement | Daily 08:00 UTC | Emails users 7 days after bank disconnect |
| Vulnerability scan | 1st monthly | Lynis + npm audit + security report |
| DB backup | Daily 02:00 UTC | Encrypted SQLite backup, 7-day retention |

### Email Notifications (Resend API)
- Magic link login
- Score improved notification (no score revealed — drives back to app)
- First score ready
- Re-engagement (bank disconnected)
- Monthly security scan report

### Compliance
- PIPEDA compliant privacy policy
- CASL compliant email consent (express opt-in)
- AI-assisted scoring disclosure
- Affiliate disclosure
- Data deletion right (delete account feature)
- Terms of Service (Ontario law)

---

## Database Schema

9 SQLite tables, all sensitive columns encrypted:

- `users` — encrypted email, first name (Plaid-sourced), HMAC hash
- `magic_tokens` — SHA-256 hashed tokens, 15min expiry
- `plaid_connections` — encrypted access token, item_id, first name
- `score_cache` — encrypted metrics, breakdown, recommendations
- `score_history` — score over time, max 12 entries per user
- `disconnect_log` — tracks bank disconnects for re-engagement
- `access_log` — security audit log
- `known_ips` — device fingerprinting for anomaly detection
- `passports` — (reserved for future use)

---

## Pages

**rootly.ca**
- `/` — Landing page
- `/privacy` — PIPEDA/CASL privacy policy
- `/terms` — Terms of Service
- `/how-it-works` — Plain English score explainer

**app.rootly.ca**
- `/` — Magic link login
- `/dashboard` — Score dashboard
- `/score` — Full score overview (Borrowell-style)
- `/credit-cards` — KOHO card recommendations
- `/connect` — Plaid bank connection
- `/oauth-return` — OAuth redirect handler

---

## What's Next

- Ratehub.ca partnership (mortgage/insurance leads)
- Wise partnership (international money transfer)
- Plaid Returning User flow
- SQLite → PostgreSQL migration at scale

---

*Built by Jumbo · GoJumbo Digital Solutions Inc. · [hello@rootly.ca](mailto:hello@rootly.ca)*
