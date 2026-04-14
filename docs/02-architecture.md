# 02 · Architecture

## System overview

```mermaid
flowchart TB
    User([User / Browser])
    Next[Next.js 16 App Router<br>React 19 · TypeScript · Tailwind]

    subgraph Server [Server Layer · Vercel]
        RSC[Server Components<br>+ Server Actions]
        API[API Routes<br>webhooks · crons · admin]
    end

    SB[(Supabase<br>Auth · Postgres · RLS)]

    subgraph AI [AI Orchestration]
        OAI[OpenAI<br>per-module models]
        ANT[Anthropic Claude<br>Tier-2 support]
    end

    subgraph Pay [Payment Rails]
        ST[Stripe]
        PP[PayPal]
    end

    RS[Resend<br>SPF · DKIM · DMARC]

    subgraph Ops [Ops and Observability]
        CRN[5 Vercel Crons]
        SEN[Sentry · GA4 · GSC]
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
    CHAT_SESSIONS }o--|| SUPPORT_AGENTS : assigned

    SUBSCRIPTIONS }o--|| PAYMENT_PROVIDER : uses
    PAYMENT_CONSENTS }o--|| REFUND_POLICIES : stamped
    CHARGEBACK_CASES ||--o{ EMAIL_LOG : references

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

5 scheduled jobs drive the content + ops engine:

| Schedule | Job | Purpose |
|---|---|---|
| Daily 06:00 UTC | `content-generate` | Generate next-day content (blog, rituals, daily card) |
| Daily 14:00 UTC | `content-publish` | Publish approved content, ping IndexNow, update sitemap |
| Mon 08:00 UTC | `intelligence` | Aggregate signals, brainstorms, roadmap |
| 15th 10:00 UTC | `monthly-report` | Email business report |
| Daily 15:00 UTC | `re-engagement` | Dormant-user flows |

Each signed with `CRON_SECRET`. Routes reject anything else. **Scheduled jobs are an attack surface if not signed.**

---

## AI provider strategy

Two providers in parallel — OpenAI + Anthropic — with per-module model env vars:

```
OPENAI_TAROT_MODEL
OPENAI_SPELL_MODEL
OPENAI_RUNE_MODEL
OPENAI_DREAM_MODEL
OPENAI_NUMEROLOGY_MODEL
ANTHROPIC_API_KEY   (Claude for support + escalation reasoning)
```

Why both:

1. **Redundancy** — one outage or rate-limit doesn't stop the product
2. **Per-module selection** — Claude is better for nuanced support, GPT-4 class for some structured outputs. Swaps are env changes, not code.
3. **Cost control** — cheaper models on low-margin modules, no deploy

---

## Rendering & performance

- **App Router + RSC** — server-rendered by default, client hydration scoped to interactive components
- **Turbopack** in dev
- **Auto-generated OG images** — every reading has a unique Open Graph card
- **Reduced-motion fallback** — 3D chambers degrade to static gradients when OS reduced-motion is on

---

## Not in the architecture

Explicit non-choices, with reasoning in [07-outcomes-and-lessons.md](./07-outcomes-and-lessons.md):

- No microservices — monolith is right at this scale
- No custom fine-tuned models — API-first
- No native mobile — PWA-first
- No event bus — crons + webhooks are sufficient for current volume
