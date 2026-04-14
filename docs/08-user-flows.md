# 08 · User Flows

The happy paths and the recovery paths — how real users move through the product, and what the system does at each step.

---

## 1. First-time user: signup → first reading → purchase

The acquisition-to-activation-to-monetization journey.

```mermaid
flowchart TD
    Land([Land on site<br/>via organic / social]) --> Guest{Signed in?}
    Guest -- No --> FreeCard["Free daily card / hook reading<br/>(no signup required)"]
    FreeCard --> Value{Found value?}
    Value -- No --> Exit([Exit — GA event<br/>tracked for funnel analysis])
    Value -- Yes --> Signup["Signup prompt<br/>email / social"]
    Signup --> Verify["Email verification"]
    Verify --> FirstReading["First full reading<br/>(3 free credits granted)"]
    FirstReading --> Artifact["Reading saved to library<br/>unique OG image generated"]
    Artifact --> Hit{Hit paywall?<br/>(credits exhausted)}
    Hit -- No --> Loop([Loop: more readings])
    Hit -- Yes --> Pricing["Pricing page<br/>(4 tiers + credit packs)"]
    Pricing --> Policy["Consent row captured<br/>policy version + IP + UA"]
    Policy --> Checkout{Chosen rail?}
    Checkout -- Stripe --> StripeFlow["Stripe Checkout"]
    Checkout -- PayPal --> PayPalFlow["PayPal Checkout"]
    StripeFlow --> Webhook["Webhook signature verified<br/>credits provisioned"]
    PayPalFlow --> Webhook
    Webhook --> Receipt["Receipt email<br/>evidence-stamped"]
    Receipt --> Active([Active paying user<br/>→ retention flows])

    style Exit fill:#FF444430,color:#F5F0E6
    style Active fill:#3FCF8E40,color:#F5F0E6
    style Policy fill:#6E56CF30,color:#F5F0E6
    style Webhook fill:#C8A96930,color:#F5F0E6
```

**Design principles baked in:**
- Free hook before signup — reduces acquisition friction
- 3 free credits after signup — lets user experience structured readings before paywall
- Consent captured at checkout — evidentiary layer from the first purchase
- Evidence-stamped receipts — chargeback defense starts at purchase confirmation

---

## 2. Support journey with AI escalation

What happens when a user needs help.

```mermaid
sequenceDiagram
    actor User
    participant UI as Chat Widget
    participant Router as Agent Router
    participant T1 as Tier-1 Agent<br/>(Maya / Sofia / ...)
    participant T2 as Tier-2 Agent<br/>(Nadia — Anthropic)
    participant DB as Supabase

    User->>UI: Opens chat
    UI->>Router: Load session
    Router->>DB: Read escalated_to_agent
    DB-->>Router: null (not escalated)
    Router->>T1: Route via hash(user_id, week)
    T1-->>User: "Hi, I'm Maya — how can I help?"

    User->>T1: Question
    T1-->>User: Response 1

    User->>T1: Still not clear
    T1-->>User: Response 2 (pushback: 1)

    User->>T1: This isn't helping
    T1-->>User: Response 3 (pushback: 2)

    User->>T1: I want a human
    Note over Router: pushback: 3 — escalate
    Router->>DB: Set escalated_to_agent = nadia
    Router->>UI: Event: "Connecting to senior agent..."
    Router->>T2: Route + full context
    T2-->>User: "Hi, I'm Nadia — let me help with this."

    User->>T2: Continue conversation
    Note over Router,DB: All future messages route to T2<br/>escalated_to_agent persisted correctly
```

**Why it matters:**
- Users get consistency within a session, variety across weeks
- Escalation is data-driven (pushback count) not keyword-based
- Handoff is soft — same thread, no context loss
- Full transcripts logged for compliance + dispute defense

---

## 3. Refund / chargeback recovery flow

What happens when a user wants their money back — and how we stay off processor watch-lists.

```mermaid
flowchart TD
    Unhappy([User regrets purchase]) --> Channel{How do they<br/>reach out?}

    Channel -- "Opens support chat<br/>(low friction)" --> Support["Tier-1 or Tier-2 agent<br/>with full purchase context"]
    Support --> Policy["Apply per-product refund policy<br/>(cancellation window, fee rules)"]
    Policy --> Eligible{Refund<br/>eligible?}
    Eligible -- Yes --> Refund["Process refund<br/>via Stripe / PayPal API"]
    Refund --> Resolved([Refund issued<br/>credits clawed back])
    Eligible -- No --> Explain["Explain policy<br/>offer partial / alternative"]
    Explain --> UserAccepts{User<br/>accepts?}
    UserAccepts -- Yes --> Resolved
    UserAccepts -- No --> FileDispute

    Channel -- "Emails support<br/>(via receipt callout)" --> Support

    Channel -- "Skips support<br/>files bank dispute" --> FileDispute["Processor dispute opens"]
    FileDispute --> Webhook["Dispute webhook fires<br/>chargeback_case created"]
    Webhook --> Evidence["Assemble evidence bundle<br/>• payment_consents row<br/>• email_log HTML snapshot<br/>• support transcript<br/>• reading records"]
    Evidence --> Submit["Submit to processor<br/>within 7 days"]
    Submit --> Outcome{Outcome}
    Outcome -- Won --> Won([Dispute won<br/>~60%+ rate])
    Outcome -- Lost --> Lost([Dispute lost<br/>+ fee charged<br/>→ investigate gap in evidence])

    style Resolved fill:#3FCF8E40,color:#F5F0E6
    style Won fill:#3FCF8E40,color:#F5F0E6
    style Lost fill:#FF444430,color:#F5F0E6
    style FileDispute fill:#FF444420,color:#F5F0E6
    style Support fill:#6E56CF30,color:#F5F0E6
```

**The most important design decision:** the **receipt callout** ("Questions about this charge?") routes unhappy users into support *before* they reach for a chargeback. That single step converts a ~$30 lost revenue event (refund) into a preserved relationship, or at worst a $15 fee event (lost dispute) into a $0 refund event.

Industry data: a well-designed pre-dispute flow converts **30%+ of would-be chargebacks** into refunds or resolutions.

---

## 4. Content publishing flow (automated)

How the content side of the platform operates autonomously via crons.

```mermaid
flowchart LR
    subgraph Daily["Daily cycle"]
        direction TB
        Gen["06:00 UTC<br/>content-generate cron"]
        Gen --> Brainstorm[("content_pipeline<br/>drafts created")]
        Brainstorm --> Review{Admin<br/>review needed?}
        Review -- No --> Auto["Auto-approve<br/>(trusted templates)"]
        Review -- Yes --> Queue["Admin queue<br/>approval workflow"]
        Queue --> Approve[Admin approves or edits]
        Auto --> Publish
        Approve --> Publish
        Publish["14:00 UTC<br/>content-publish cron"]
        Publish --> Live[Live on /blog, /rituals, etc.]
        Live --> Ping["Ping IndexNow + GSC<br/>sitemap updated"]
    end

    subgraph Weekly["Weekly"]
        Intel["Mon 08:00 UTC<br/>intelligence cron"]
        Intel --> Signals["Aggregate signals<br/>(brainstorms, performance,<br/>search trends)"]
        Signals --> Roadmap[Next-week roadmap generated]
    end

    subgraph Monthly["Monthly"]
        Report["15th 10:00 UTC<br/>monthly-report cron"]
        Report --> Email[Email to admin]
    end

    style Gen fill:#C8A96930,color:#F5F0E6
    style Publish fill:#C8A96930,color:#F5F0E6
    style Intel fill:#6E56CF30,color:#F5F0E6
    style Report fill:#3FCF8E30,color:#F5F0E6
```

---

## What these flows evidence

For a PM/TPM audience, these aren't "product screenshots" — they're **operating models**. Each diagram encodes:

1. **User-centered thinking** — where does the user want to go, what obstacles do they hit?
2. **System thinking** — what does the backend do at each decision point?
3. **Governance thinking** — where does consent, evidence, or compliance enter the flow?
4. **Recovery thinking** — what happens when the happy path breaks?

A PM who can think in these four layers simultaneously is rare. This is what the case study is trying to show.

---

<p align="center">
  ← <a href="./07-outcomes-and-lessons.md">Outcomes & Lessons</a> · <a href="../README.md">Back to overview</a>
</p>
