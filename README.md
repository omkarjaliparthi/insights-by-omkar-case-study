<h1 align="center">Insights by Omkar — Case Study</h1>

<p align="center">
  <b>How I shipped a production AI SaaS, solo, in 6 weeks.</b><br/>
  <i>Product, program, engineering, and compliance — end to end.</i>
</p>

<p align="center">
  <a href="https://www.insightsbyomkar.com"><img src="https://img.shields.io/badge/Live_Product-insightsbyomkar.com-181717?style=for-the-badge" /></a>
  <img src="https://img.shields.io/badge/Built_by-Omkar_Jaliparthi-6E56CF?style=for-the-badge" />
  <img src="https://img.shields.io/badge/Timeline-6_weeks-success?style=for-the-badge" />
  <img src="https://img.shields.io/badge/Status-Live_with_paying_users-brightgreen?style=for-the-badge" />
</p>

<p align="center">
  <i>This repo is a <b>public case study</b>. The production code lives in a <b>private</b> repo — shared here is architecture, decisions, and operating playbooks, with no proprietary code.</i>
</p>

---

## TL;DR

I built and launched **[insightsbyomkar.com](https://www.insightsbyomkar.com)** — a production AI SaaS platform — in **~6 weeks, solo**, under my LLC (**Omkar's Holistic Services LLC**, formed May 2023, DBA *Insights by Omkar*). End-to-end: product strategy, UX, full-stack engineering, payments, AI integration, email infrastructure, support operations, compliance, and launch.

**Why it matters for a PM/TPM role:** most candidates can talk about one layer of a product. This project is evidence I can own *all* of them — and that I know which layer to invest in based on business stage and risk.

---

## At a glance

| | |
|---|---|
| **Product** | AI-powered spiritual insights platform — tarot, spells, runes, dreams, numerology, and 7+ other modules |
| **Timeline** | v0.01 (initial scaffold) → v2.0.7 (production) in ~6 weeks (2026-03-05 → 2026-04-14) |
| **Release cadence** | 20+ versioned releases, full [CHANGELOG](https://github.com/omkarjaliparthi/insights-by-omkar) |
| **Stack** | Next.js 16 · React 19 · Supabase · TypeScript · Tailwind · Vercel |
| **Payments** | Dual rails — Stripe (live) + PayPal (live), 4-tier subscriptions + credit packs |
| **AI** | OpenAI + Anthropic, per-module model configuration, structured outputs |
| **Ops** | 5 Vercel cron jobs · Resend email (SPF/DKIM/DMARC) · Sentry · GSC + IndexNow |
| **Compliance** | Full RLS, consent logging, chargeback-defense data model, refund policy enforcement |
| **Surface area** | 30+ DB migrations · 20+ admin panels · 12+ product modules |

---

## 📑 Read the case study

1. **[Problem & users](./docs/01-problem-and-users.md)** — why this product exists, who it's for
2. **[Architecture](./docs/02-architecture.md)** — system design, data model, cron topology
3. **[Decision: dual payment rails](./docs/03-decision-payment-rails.md)** — why Stripe *and* PayPal
4. **[Decision: tiered AI support agents](./docs/04-decision-support-agent-tiering.md)** — 6 rotating Tier-1 + auto-escalation
5. **[Decision: chargeback defense](./docs/05-decision-chargeback-defense.md)** — how the law degree showed up
6. **[Operating rhythm](./docs/06-operating-rhythm.md)** — release discipline, pre-launch checklist, cron schedules
7. **[Outcomes & lessons](./docs/07-outcomes-and-lessons.md)** — what worked, what I'd do differently

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

**Omkar Jaliparthi** — 8+ years across Project/Program Management, Business Analysis, and full-stack shipping. Founder of **Omkar's Holistic Services LLC** (DBA *Insights by Omkar*) since May 2023. MS CS (Pace University, 2024), LLB (2021), ICWA Intermediate (2020). Based in San Jose, CA. Open to Senior PM, TPM, and Founding PM roles.

<p>
  <a href="https://github.com/omkarjaliparthi">GitHub</a> ·
  <a href="https://www.linkedin.com/in/jaliparthiomkar">LinkedIn</a> ·
  <a href="mailto:Jaliparthiomkar03@gmail.com">Email</a> ·
  <a href="https://www.insightsbyomkar.com">Live product</a>
</p>
