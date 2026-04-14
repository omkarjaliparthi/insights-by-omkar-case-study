# 03 · Decision · Dual Payment Rails

## Context

At v1.2 (mid-March 2026), payments stack decision. Two revenue paths:

- **Subscriptions** — 4 tiers (Lucky Pro M/A, Lucky Max M/A)
- **Credit packs + impulse packages** — pay-per-reading

## Options considered

| Option | Pros | Cons |
|---|---|---|
| **Stripe only** | Best-in-class DX, subscription primitives, great dashboard | Some segments (older users, non-US) prefer PayPal; Stripe account reviews can freeze accounts |
| **PayPal only** | Broad trust, international reach, lower friction for first-time online buyers | Subscription API is worse, developer ergonomics are painful, reconciliation harder |
| **Stripe + PayPal (dual rails)** | Meet users where they already have a wallet; redundancy if one account gets held | 2× webhook surfaces, 2× reconciliation, 2× fraud models, more test matrix |
| **Lemon Squeezy / Paddle MoR** | Handles tax globally, simpler compliance | Higher fees, less control, less credible signal to enterprise buyers later |

### Scoring the options

| Dimension (weight) | Stripe only | PayPal only | **Dual rails** | MoR (Paddle) |
|---|:-:|:-:|:-:|:-:|
| User trust / conversion (×3) | 🟡 6 | 🟢 8 | 🟢 **9** | 🟡 7 |
| DX / velocity (×2) | 🟢 9 | 🔴 4 | 🟡 7 | 🟢 8 |
| Chargeback resilience (×3) | 🔴 4 | 🔴 4 | 🟢 **9** | 🟡 6 |
| Unit economics (×2) | 🟢 9 | 🟡 7 | 🟢 8 | 🔴 4 |
| Ops complexity (×1, inverted) | 🟢 9 | 🟡 7 | 🟡 6 | 🟢 9 |
| **Weighted score** | 65 | 55 | **78** | 64 |

### Decision flow

```mermaid
flowchart TD
    Start([Payments stack decision]) --> Q1{Need global tax<br>handling at v1?}
    Q1 -- Yes --> MoR[Merchant of Record<br>Paddle or Lemon Squeezy]
    Q1 -- No --> Q2{Single processor<br>risk acceptable?}
    Q2 -- Yes --> Q3{Audience skews<br>younger or global?}
    Q2 -- No --> Dual[DUAL RAILS<br>Stripe + PayPal]
    Q3 -- Yes --> StripeOnly[Stripe only]
    Q3 -- No --> PayPalPref[PayPal preferred<br>older first-time buyers]
    PayPalPref --> Dual

    style Dual fill:#3FCF8E,stroke:#1F8F5F,stroke-width:3px,color:#000
    style MoR fill:#FF8888,stroke:#CC4444,color:#000
    style StripeOnly fill:#C8A969,stroke:#8F7744,color:#000
```

## Decision

**Dual rails — Stripe primary, PayPal secondary.** Highest weighted score. Strongest chargeback resilience, which was the gating concern.

## Rationale

1. **Trust surface.** Skeptical buyers — especially older demographics — have PayPal accounts and won't enter card details on new sites. PayPal at checkout measurably lowers friction.
2. **Processor redundancy.** High-emotion purchases carry elevated dispute risk. One processor can't kill the business. If Stripe freezes for review, PayPal keeps revenue flowing.
3. **Unit economics hold.** Fee differential (Stripe 2.9%+30¢ vs PayPal 3.49%+49¢) is small vs. the cost of a lost customer at checkout.

## Tradeoffs accepted

- **2× webhook surface.** Both endpoints need signature verification + idempotency. Covered in pre-launch checklist.
- **Reconciliation complexity.** Different provider IDs. Schema abstracts `provider_*` columns so app code doesn't branch on rail.
- **State normalization.** PayPal's subscription state machine is thinner than Stripe's. Both map into one internal state (`active | past_due | canceled | refunded | disputed`).

## Testing

Pre-launch (v1.4.1):

- Full test loop both rails: checkout → webhook → credits delta → refund → reversal
- Dispute simulation — test chargeback, verified `chargeback_cases` row + evidence-stamped email logged
- Subscription lifecycle: upgrade, downgrade, cancel → webhook → UI state

## Would change

Build `PaymentProvider` abstraction at v1.0 even with only Stripe implemented. Retrofitting at v2.0 cost ~1.5 days.

---

*Artifacts referenced:* `payment_consents` · `chargeback_cases` · `STRIPE_WEBHOOK_SECRET` · `PAYPAL_WEBHOOK_ID`. See [pre-launch checklist](./06-operating-rhythm.md#pre-launch-checklist).
