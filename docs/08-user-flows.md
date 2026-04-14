# 08 · User Flows

The happy paths and the recovery paths — how real users move through the product, and what the system does at each step.

---

## 1. First-time user: signup to first reading to purchase

The acquisition → activation → monetization journey.

```mermaid
flowchart TD
    Land([Land on site]) --> Guest{Signed in?}
    Guest -- No --> FreeCard[Free daily card reading<br>no signup required]
    FreeCard --> Value{Found value?}
    Value -- No --> Exit([Exit - GA event tracked])
    Value -- Yes --> Signup[Signup prompt]
    Signup --> Verify[Email verification]
    Verify --> FirstReading[First full reading<br>3 free credits granted]
    FirstReading --> Artifact[Reading saved to library<br>unique OG image generated]
    Artifact --> Hit{Hit paywall?}
    Hit -- No --> Loop([Loop - more readings])
    Hit -- Yes --> Pricing[Pricing page<br>4 tiers + credit packs]
    Pricing --> Policy[Consent row captured<br>policy version + IP + UA]
    Policy --> Checkout{Which rail?}
    Checkout -- Stripe --> StripeFlow[Stripe Checkout]
    Checkout -- PayPal --> PayPalFlow[PayPal Checkout]
    StripeFlow --> Webhook[Webhook verified<br>credits provisioned]
    PayPalFlow --> Webhook
    Webhook --> Receipt[Receipt email<br>evidence-stamped]
    Receipt --> Active([Active paying user])

    style Exit fill:#FF8888,color:#000
    style Active fill:#3FCF8E,color:#000
    style Policy fill:#C4B8F0,color:#000
    style Webhook fill:#F5D789,color:#000
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
    participant T1 as Tier-1 Agent
    participant T2 as Nadia Tier-2
    participant DB as Supabase

    User->>UI: Opens chat
    UI->>Router: Load session
    Router->>DB: Read escalated_to_agent
    DB-->>Router: null - not escalated
    Router->>T1: Route via hash
    T1-->>User: Hi, I am Maya. How can I help?

    User->>T1: Question
    T1-->>User: Response 1

    User->>T1: Still not clear
    T1-->>User: Response 2 (pushback 1)

    User->>T1: This is not helping
    T1-->>User: Response 3 (pushback 2)

    User->>T1: I want a human
    Note over Router: pushback 3 - escalate
    Router->>DB: Set escalated_to_agent to nadia
    Router->>UI: Event - connecting to senior agent
    Router->>T2: Route with full context
    T2-->>User: Hi, I am Nadia. Let me help.

    User->>T2: Continue conversation
    Note over Router,DB: Future messages stay with T2
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
    Unhappy([User regrets purchase]) --> Channel{How reached out?}

    Channel -- Opens support chat --> Support[Tier-1 or Tier-2 agent<br>with full purchase context]
    Channel -- Emails support --> Support
    Channel -- Files bank dispute --> FileDispute[Processor dispute opens]

    Support --> Policy[Apply refund policy]
    Policy --> Eligible{Refund eligible?}
    Eligible -- Yes --> Refund[Process refund via API]
    Refund --> Resolved([Refund issued<br>credits clawed back])
    Eligible -- No --> Explain[Explain policy<br>offer partial or alternative]
    Explain --> UserAccepts{User accepts?}
    UserAccepts -- Yes --> Resolved
    UserAccepts -- No --> FileDispute

    FileDispute --> Webhook[Dispute webhook fires<br>chargeback_case created]
    Webhook --> Evidence[Bundle evidence:<br>consent + email + transcript + reading]
    Evidence --> Submit[Submit within 7 days]
    Submit --> Outcome{Outcome}
    Outcome -- Won --> Won([Dispute won - 60% rate])
    Outcome -- Lost --> Lost([Dispute lost + fee])

    style Resolved fill:#3FCF8E,color:#000
    style Won fill:#3FCF8E,color:#000
    style Lost fill:#FF8888,color:#000
    style FileDispute fill:#FFB366,color:#000
    style Support fill:#C4B8F0,color:#000
```

**The most important design decision:** the **receipt callout** ("Questions about this charge?") routes unhappy users into support *before* they reach for a chargeback. That single step converts a ~$30 lost revenue event (refund) into a preserved relationship, or at worst a $15 fee event (lost dispute) into a $0 refund event.

Industry data: a well-designed pre-dispute flow converts **30%+ of would-be chargebacks** into refunds or resolutions.

---

## 4. Content publishing flow (automated)

How the content side of the platform operates autonomously via crons.

```mermaid
flowchart LR
    Gen[06:00 UTC<br>content-generate cron] --> Brainstorm[(content_pipeline<br>drafts created)]
    Brainstorm --> Review{Admin review needed?}
    Review -- No --> Auto[Auto-approve<br>trusted templates]
    Review -- Yes --> Queue[Admin queue<br>approval workflow]
    Queue --> Approve[Admin approves or edits]
    Auto --> Publish
    Approve --> Publish[14:00 UTC<br>content-publish cron]
    Publish --> Live[Live on blog and rituals]
    Live --> Ping[Ping IndexNow and GSC<br>sitemap updated]

    Intel[Mon 08:00 UTC<br>intelligence cron] --> Signals[Aggregate signals]
    Signals --> Roadmap[Next-week roadmap generated]

    Report[15th 10:00 UTC<br>monthly-report cron] --> Email[Email to admin]

    style Gen fill:#F5D789,color:#000
    style Publish fill:#F5D789,color:#000
    style Intel fill:#C4B8F0,color:#000
    style Report fill:#3FCF8E,color:#000
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
  <a href="./07-outcomes-and-lessons.md">← Outcomes and Lessons</a> · <a href="../README.md">Back to overview</a>
</p>
