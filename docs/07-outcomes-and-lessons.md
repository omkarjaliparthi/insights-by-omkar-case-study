# 07 · Outcomes & Lessons

## Shipped

In **6 weeks, solo**:

- Live, production-grade AI SaaS at [insightsbyomkar.com](https://www.insightsbyomkar.com)
- 12+ product modules · structured AI outputs · per-module schema + cost model
- Dual payment rails (Stripe + PayPal) with webhook reconciliation + chargeback case modeling
- Multi-tier AI support (6 rotating Tier-1 + Anthropic Tier-2)
- 30+ DB migrations · RLS on every user-scoped table
- 5 scheduled cron jobs · content, intelligence, reports, re-engagement
- 8 branded email templates with evidence-stamping
- 20+ admin panels · ops, content intelligence, support, observability
- 13-section pre-launch checklist · versioned CHANGELOG discipline

## What this evidences

I don't just manage shipping — I do the shipping when needed:

- **Strategize like a PM** — segment, price, decide what to cut
- **Execute like a TPM** — sequence, track, gate
- **Build like an engineer** — production-ready code across the stack
- **Think like legal/finance** — chargeback defense, unit economics, compliance

Most senior PM/TPM candidates do one or two. This project does all four, concurrently, on a commercial deadline.

---

## What worked

### 1. Ruthless scoping

Wrote a "not doing at launch" list on day one and held it:
- No mobile app
- No social features
- No custom models
- ≤5 modules at v1.0

Every "no" funded the "yes" list. The 12-module product today exists because v1.0 was 5 modules.

### 2. Compliance as product, not legal

Chargeback defense, consent, RLS, refund policy enforcement — designed as product requirements into data model and UX. Not bolted on post-legal review. Audit-ready day 1 without blocking velocity.

### 3. Versioned releases, always

Every change tagged. Every tag in the changelog. Six weeks in, "when did we ship X?" is a 10-second answer. That's operational leverage.

### 4. AI-native operating model

AI coding tools used as amplifiers, not shortcuts. `update-ai-context.sh` + `repomix-output.xml` kept the codebase compressible and queryable. A solo dev with AI can outpace a 5-person team — if context stays tight.

---

## What I'd change

### 1. Payment-provider abstraction at v1.0

Added Stripe at v1.2, PayPal at v2.0. Retrofitting rail-aware logic cost ~1.5 days. Cleaner: `PaymentProvider` interface from day one.

### 2. Staging smoke tests earlier

The v2.0.7 escalation bug was catchable with a 2-message test. I was smoke-testing against production. Works solo, won't scale.

### 3. Dispute dashboard on day 1

`chargeback_cases` is modeled. The dashboard isn't. Dispute rate, refund rate, per-product trends, threshold alerts. First item for week 7.

### 4. Pre-dispute save flow

The amber callout links to email support today. Better: in-product support chat with purchase context preloaded. Industry data: 30%+ conversion from would-be chargebacks.

---

## What this unlocks

This repo is a capability proof. The fits from here:

- **Founding PM / TPM at an AI startup** — end-to-end ownership at startup speed
- **Senior TPM at a consumer AI org** — scale the discipline across teams
- **Senior PM for monetization, payments, compliance** — the law + accounting + engineering combo is rare in regulated consumer AI

Not looking for "a PM role." Looking for a role where operator, builder, and governance-thinker all get used.

If that's your org — **[let's talk](mailto:admin@insightsbyomkar.com)**.

---

<p align="center">
  <a href="./06-operating-rhythm.md">← Operating Rhythm</a> · <a href="../README.md">Back to overview</a>
</p>
