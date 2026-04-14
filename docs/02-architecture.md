# 02 · Architecture

## System overview

```mermaid
flowchart TB
    User([User / Browser])

    subgraph Client["Client Layer"]
        direction LR
        Next["Next.js 16 App Router<br/>React 19 · TypeScript · Tailwind"]
    end

    subgraph Server["Server Layer (Vercel Edge + Node)"]
        direction LR
        RSC["Server Components<br/>+ Server Actions"]
        API["API Routes<br/>(webhooks, crons, admin)"]
    end

    subgraph Data["Data & Identity"]
        SB[("Supabase<br/>Auth · Postgres · Storage<br/>Row-Level Security")]
    end

    subgraph AI["AI Orchestration"]
        OAI["OpenAI<br/>(per-module models)"]
        ANT["Anthropic Claude<br/>(Tier-2 support)"]
    end

    subgraph Pay["Payment Rails"]
        ST["Stripe<br/>Subscriptions + Credits"]
        PP["PayPal<br/>Subscriptions + Orders"]
    end

    subgraph Comm["Email & Comms"]
        RS["Resend<br/>SPF · DKIM · DMARC<br/>Inbound routing"]
    end

    subgraph Ops["Ops & Observability"]
        direction LR
        CRN["5× Vercel Crons<br/>content · intelligence<br/>reports · re-engagement"]
        SEN["Sentry · GA4<br/>GSC · IndexNow"]
    end

    User <--> Next
    Next <--> RSC
    Next <--> API
    RSC --> SB
    API --> SB
    API --> OAI
    API --> ANT
    API --> ST
    API --> PP
    API --> RS
    ST -. webhooks .-> API
    PP -. webhooks .-> API
    CRN --> API
    API --> SEN

    classDef client fill:#6E56CF20,stroke:#6E56CF,color:#F5F0E6
    classDef server fill:#C8A96920,stroke:#C8A969,color:#F5F0E6
    classDef data fill:#3FCF8E20,stroke:#3FCF8E,color:#F5F0E6
    classDef ai fill:#41299120,stroke:#412991,color:#F5F0E6
    classDef pay fill:#635BFF20,stroke:#635BFF,color:#F5F0E6
    classDef comm fill:#FFCB1F20,stroke:#FFCB1F,color:#F5F0E6
    classDef ops fill:#FF444420,stroke:#FF4444,color:#F5F0E6

    class Next client
    class RSC,API server
    class SB data
    class OAI,ANT ai
    class ST,PP pay
    class RS comm
    class CRN,SEN ops
```

---

## Data model (high-level)

30+ migrations shipped, all changes versioned. Key domains:

| Domain | Core tables | Purpose |
|---|---|---|
| **Identity** | `profiles`, `ghost_profiles`, `guest_ip_rate_limits` | Signed-in + guest user flows, abuse prevention |
| **Content modules** | `tarot_readings`, `spells`, `rune_casts`, `dream_journal`, `numerology_profiles` | Per-module reading artifacts |
| **Monetization** | `subscriptions`, `credit_packages`, `impulse_packages`, `refund_policies` | Subscription tiers, credit economy, refund windows |
| **Compliance** | `payment_consents`, `chargeback_cases`, `email_log` | Consent proof, dispute defense, comms audit |
| **Support** | `support_agent_settings`, `support_agents`, `chat_sessions`, `chat_messages`, `support_tickets` | Multi-tier AI agent system + human escalation |
| **Content ops** | `content_pipeline`, `intelligence_layers`, `approval_workflow`, `blog` | Scheduled content generation + publishing |
| **Governance** | Row-Level Security on every user-scoped table | Service role gated to server-side only |

**Every user-scoped table has RLS.** The service-role key never touches the client bundle.

### Data model — ER diagram (simplified)

```mermaid
erDiagram
    PROFILES ||--o{ SUBSCRIPTIONS : has
    PROFILES ||--o{ TAROT_READINGS : creates
    PROFILES ||--o{ DREAM_JOURNAL : writes
    PROFILES ||--o{ RUNE_CASTS : casts
    PROFILES ||--o{ CHAT_SESSIONS : starts
    PROFILES ||--o{ PAYMENT_CONSENTS : grants
    PROFILES ||--o{ CHARGEBACK_CASES : "may trigger"
    PROFILES ||--o{ EMAIL_LOG : receives

    CHAT_SESSIONS ||--o{ CHAT_MESSAGES : contains
    CHAT_SESSIONS }o--|| SUPPORT_AGENTS : "assigned (T1 or T2)"

    SUBSCRIPTIONS }o--|| STRIPE_OR_PAYPAL : "provider"
    PAYMENT_CONSENTS }o--|| REFUND_POLICIES : "stamped with"
    CHARGEBACK_CASES ||--|| EMAIL_LOG : "evidence from"

    PROFILES {
        uuid id PK
        text email
        text display_name
        jsonb preferences
        timestamp created_at
    }
    SUBSCRIPTIONS {
        uuid id PK
        uuid user_id FK
        text provider
        text provider_subscription_id
        text state
        timestamp current_period_end
    }
    PAYMENT_CONSENTS {
        uuid id PK
        uuid user_id FK
        text policy_version
        text ip
        text user_agent
        timestamp consented_at
    }
    CHARGEBACK_CASES {
        uuid id PK
        uuid user_id FK
        text provider_dispute_id
        text status
        jsonb evidence_bundle
        text resolution
    }
    EMAIL_LOG {
        uuid id PK
        uuid user_id FK
        text template
        text html_snapshot
        timestamp sent_at
    }
    CHAT_SESSIONS {
        uuid id PK
        uuid user_id FK
        text escalated_to_agent
        timestamp started_at
    }
    SUPPORT_AGENTS {
        uuid id PK
        text tier
        text persona
        text provider
    }
```

---

## Cron topology

5 scheduled jobs run the content + ops engine:

| Schedule | Job | Purpose |
|---|---|---|
| Daily 06:00 UTC | `/api/cron/content-generate` | Generate next day's content (blog, rituals, daily card) |
| Daily 14:00 UTC | `/api/cron/content-publish` | Publish approved content, ping IndexNow, update sitemap |
| Weekly Mon 08:00 UTC | `/api/cron/intelligence` | Aggregate content intelligence, brainstorms, roadmap signals |
| Monthly 15th 10:00 UTC | `/api/cron/monthly-report` | Email monthly business report to admin |
| Daily 15:00 UTC | `/api/cron/re-engagement` | Re-engagement flows for dormant users |

Each is secured by a `CRON_SECRET` header check — Vercel Cron sends it, routes reject anything else. This pattern matters: **scheduled jobs are an attack surface if not signed**.

---

## AI provider strategy

Two providers wired in parallel — **OpenAI** and **Anthropic** — with per-module model env vars:

```
OPENAI_TAROT_MODEL
OPENAI_SPELL_MODEL
OPENAI_RUNE_MODEL
OPENAI_DREAM_MODEL
OPENAI_NUMEROLOGY_MODEL
ANTHROPIC_API_KEY            (Claude for support agents + escalation reasoning)
```

**Why both?** Three reasons:

1. **Redundancy.** If one provider has an outage or rate-limits, the product keeps working.
2. **Per-module model selection.** Claude is better for nuanced support; GPT-4 class is better for some structured outputs. Model swaps are env-var changes, not code changes.
3. **Cost control.** Swap to cheaper models on low-margin modules without a code deploy.

---

## Rendering & performance

- **App Router + RSC** — most pages are server-rendered; client hydration is scoped to interactive components
- **Turbopack** in dev
- **OG image auto-generation** per reading for shareable artifacts (every tarot reading has a unique Open Graph card)
- **Reduced-motion fallback** — 3D (`@react-three/*`, `gsap`) chambers gracefully degrade to static gradient backgrounds when OS reduced-motion is enabled

---

## What's *not* in the architecture

Decisions I explicitly didn't make yet, with reasoning in [07-outcomes-and-lessons.md](./07-outcomes-and-lessons.md):

- No microservices (monolith is right for this scale)
- No custom fine-tuned models (API-first)
- No native mobile (PWA-first)
- No event bus / queue (cron jobs + webhook handlers are sufficient for current volume)
