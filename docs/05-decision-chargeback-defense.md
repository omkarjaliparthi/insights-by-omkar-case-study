# 05 · Decision · Chargeback Defense

## Context

High-emotion consumer purchases carry elevated chargeback rates — 0.5–1.5% vs. ~0.1% for normal e-commerce. Cross a processor's threshold (Stripe: 0.75%) and your merchant account is paused or terminated. Existential.

LLB training — contracts, consumer protection, evidentiary burden — shaped the design from v1.0, not a retrofit.

## Design

### Data model

Three tables form the defense layer:

| Table | Purpose |
|---|---|
| `payment_consents` | Which pricing page, what accepted, when, IP, user-agent |
| `chargeback_cases` | Dispute ID, status, evidence bundle, resolution |
| `email_log` | Every transactional email — sender, recipient, template, timestamp, HTML snapshot |

### Receipts as evidence artifacts

Every Stripe/PayPal receipt auto-injects:

1. **Amber "Questions about this charge?" callout** → support email + policy link (non-chargeback escape path)
2. **"Charged on [timestamp PDT]"** → proof of authorization
3. **Policy row** → for credits: *"Credits are non-refundable once added."* For appointments: *"Full refund >24 hrs · Non-refundable within 24 hrs."*
4. **Footer** → *"By completing this purchase you agreed to our Refund & Cancellation Policy"* with link to `/legal/refunds`

Not legal theater. When a dispute opens, the merchant has 7 days to submit evidence. Receipts archived + timestamped + policy-stamped **at purchase** win disputes that would otherwise lose.

### Per-product refund policies

Two tables — `credit_package_refund_policies` and `appointment_type_refund_policies` — store per-product rules. Referenced, not hard-coded:

- Policy changes don't require a code deploy
- Admin panel sets cancellation windows, refund percentages, fees per product
- What the user sees at checkout is what the webhook enforces

### Evidence chain — from purchase to dispute win

```mermaid
flowchart LR
    subgraph Purchase
        P1[User views pricing] --> P2[Policy shown inline]
        P2 --> P3[User clicks Pay]
        P3 --> P4[(payment_consents<br>IP · UA · policy · timestamp)]
    end

    subgraph Receipt
        R1[Payment webhook fires] --> R2[Receipt email sent]
        R2 --> R3[Auto-inject: timestamp<br>policy row + support callout]
        R3 --> R4[(email_log<br>HTML snapshot)]
    end

    subgraph Dispute
        D1[Processor dispute webhook] --> D2[Create chargeback_case]
        D2 --> D3[Bundle evidence:<br>consent + email + transcript]
        D3 --> D4[Submit within 7 days]
        D4 --> D5{Outcome}
        D5 -->|60% wins| Won([Dispute won])
        D5 -->|40% lost| Lost([Dispute lost])
    end

    P4 -.-> D3
    R4 -.-> D3

    style Won fill:#3FCF8E,stroke:#1F8F5F,color:#000
    style Lost fill:#FF8888,stroke:#CC4444,color:#000
    style P4 fill:#C4B8F0,stroke:#412991,color:#000
    style R4 fill:#F5D789,stroke:#8F7744,color:#000
```

**Key insight:** evidence must exist at purchase, not assembled after dispute. Every link in the chain is built to be evidentiary.

## Why this matters for PM/TPM

Most founders treat chargeback defense as an operational problem — handled when the first dispute arrives. By then, the evidence you need doesn't exist.

Treat it as a **product problem** and:

- Refund terms visible at checkout, in receipts, in-product — no "hidden terms" argument
- Every consent logged with context (IP, UA, timestamp, policy version)
- Every email becomes an evidentiary artifact
- Support agents see full purchase + communication timeline per complaint

Cross-functional thinking — product + engineering + legal + finance — is what senior PM/TPM roles hire for. The law degree saw the shape; the engineering built it.

## Engineering cost

| Work | Time |
|---|---|
| Migrations for `payment_consents`, `chargeback_cases`, `email_log` | 3 hours |
| Email template rewrite · 8 templates · injected evidence rows | 1 day |
| Refund policy table refactor | 0.5 day |
| **Total** | **~2 days** |

~2 engineering days protecting potentially the entire business.

## Would change

- **Dispute dashboard earlier.** Per-product dispute rates, refund-rate trends, threshold alerts. Next on roadmap.
- **Pre-dispute "save" flow.** Amber callout → routed support chat *before* user files bank dispute. Industry data: 30%+ conversion from would-be chargebacks to refunds or resolutions.
