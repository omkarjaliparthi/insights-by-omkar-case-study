<h1 align="center">Insights by Omkar · Case Study</h1>

<p align="center">
  <b>A production AI SaaS — shipped solo, in 6 weeks.</b><br/>
  <i>Product, program, engineering, compliance — end to end.</i>
</p>

<p align="center">
  <a href="https://www.insightsbyomkar.com"><img src="https://img.shields.io/badge/Live_Product-insightsbyomkar.com-181717?style=for-the-badge" /></a>
  <img src="https://img.shields.io/badge/Built_by-Omkar_Jaliparthi-6E56CF?style=for-the-badge" />
  <img src="https://img.shields.io/badge/Ship-6_weeks-success?style=for-the-badge" />
  <img src="https://img.shields.io/badge/Status-Live_with_paying_users-brightgreen?style=for-the-badge" />
</p>

<p align="center">
  <i>Public case study. Production code private. What's shared: architecture, decisions, operating playbooks — no proprietary code.</i>
</p>

---

## TL;DR

Built and launched **[insightsbyomkar.com](https://www.insightsbyomkar.com)** — a production AI SaaS — in **6 weeks, solo**, under **Omkar's Holistic Services LLC** (formed May 2023, DBA *Insights by Omkar*). End to end: product, UX, full-stack engineering, payments, AI, email, support, compliance, launch.

Most PM/TPM candidates own one layer. This project evidences all four — and the judgment to know which layer to invest in at each stage.

---

## At a glance

| | |
|---|---|
| **Product** | AI consumer SaaS · 12+ modules (tarot, spells, runes, dreams, numerology, …) |
| **Timeline** | v0.01 → v2.0.7 · 6 weeks · 2026-03-05 → 2026-04-14 |
| **Releases** | 20+ versioned · full [CHANGELOG](https://github.com/omkarjaliparthi/insights-by-omkar) |
| **Stack** | Next.js 16 · React 19 · Supabase · TypeScript · Tailwind · Vercel |
| **Payments** | Stripe + PayPal (both live) · 4 subscription tiers + credit packs |
| **AI** | OpenAI + Anthropic · per-module models · structured outputs |
| **Ops** | 5 Vercel crons · Resend (SPF/DKIM/DMARC) · Sentry · GSC · IndexNow |
| **Compliance** | RLS · consent logging · chargeback defense · enforced refund policy |
| **Surface** | 30+ DB migrations · 20+ admin panels · 12+ product modules |

---

## 📑 Read the case study

1. **[Problem & users](./docs/01-problem-and-users.md)** — why it exists, who it serves
2. **[Architecture](./docs/02-architecture.md)** — system, data model, cron topology
3. **[Decision · payment rails](./docs/03-decision-payment-rails.md)** — Stripe + PayPal, with tradeoff matrix
4. **[Decision · tiered support agents](./docs/04-decision-support-agent-tiering.md)** — 6 rotating Tier-1 + auto-escalation
5. **[Decision · chargeback defense](./docs/05-decision-chargeback-defense.md)** — governance as product
6. **[Operating rhythm](./docs/06-operating-rhythm.md)** — releases, checklists, crons
7. **[Outcomes & lessons](./docs/07-outcomes-and-lessons.md)** — what worked, what to change
8. **[User flows](./docs/08-user-flows.md)** — signup, support escalation, refund/chargeback, content automation

---

## Skills this project evidences

<table>
<tr>
<th>Product</th>
<th>Program</th>
<th>Engineering</th>
<th>Business</th>
</tr>
<tr>
<td valign="top">

- 0 → 1 product discovery
- Structured-output AI UX
- Pricing & packaging
- Feature scoping & cut lists
- Release sequencing

</td>
<td valign="top">

- Versioned release discipline
- Pre-launch checklists
- Incident + RCA playbooks
- Cron/ops scheduling
- Observability setup

</td>
<td valign="top">

- Full-stack TS/Next.js
- Supabase + RLS design
- Payment webhook integration
- Multi-provider AI orchestration
- Production deployment

</td>
<td valign="top">

- Unit economics & pricing
- Chargeback defense (LLB + ICWA)
- Refund policy design
- Subscription modeling
- Legal/compliance sequencing

</td>
</tr>
</table>

---

## Who I am

**Omkar Jaliparthi** · Product & Program leader · 8+ years across PM, BA, and full-stack shipping.
Founder of **Omkar's Holistic Services LLC** (DBA *Insights by Omkar*) since May 2023.
MS CS · LLB · ICWA Intermediate · San Jose, CA · Open to Senior PM / TPM / Founding PM.

<p>
  <a href="https://github.com/omkarjaliparthi">GitHub</a> ·
  <a href="https://www.linkedin.com/in/jaliparthiomkar">LinkedIn</a> ·
  <a href="mailto:Jaliparthiomkar03@gmail.com">Email</a> ·
  <a href="https://www.insightsbyomkar.com">Live product</a>
</p>
