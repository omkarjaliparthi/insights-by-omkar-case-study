# 04 · Decision · Tiered AI Support Agents

## Context

v2.0.7 shipped a multi-tier AI support system — no hired rep. Requirements:

- Fast, accurate answers on billing, subscription, reading interpretation, account
- Escalate to a stronger model when Tier-1 stalls
- Never sound like the same bot twice across sessions
- Never confirm or deny being a bot — warm persona, not deceptive

## Design

### Tier-1 — six rotating agents

Six personas (Maya, Sofia, Priya, Zara, Leila, Cleo). Each has:
- Unique voice and persona card
- Same underlying model (OpenAI GPT-4 class), different system prompts
- **Weekly rotation per user** via deterministic hash of `(user_id, week_number)`

Why deterministic hash:
- Same user, same agent all week → session continuity
- Different users, different agents concurrently → load distribution
- Week rolls → new agent → freshness

### Tier-2 — escalation

**Nadia** on Anthropic Claude (different provider = redundancy + reasoning strength). Triggered after **3 pushback signals** in a session (negative sentiment, "this isn't helpful", repeated clarifications).

In-place handoff: same thread, soft UI transition (`"Connecting to a senior agent..."`, 4.5s). Context preserved.

### Message routing flow

```mermaid
flowchart TD
    Msg([User sends message]) --> SessionLoad[Load session state]
    SessionLoad --> EscCheck{Already escalated?}
    EscCheck -- Yes --> Nadia[Route to Nadia Tier-2<br>Anthropic Claude]
    EscCheck -- No --> PushbackCount[Count pushback signals]
    PushbackCount --> ShouldEsc{Pushback count >= 3?}
    ShouldEsc -- Yes --> EscEvent[Emit escalation event<br>UI: Connecting to senior agent]
    EscEvent --> Persist[Persist escalated_to_agent<br>on session and all future messages]
    Persist --> Nadia
    ShouldEsc -- No --> Hash[Deterministic hash<br>user_id + week]
    Hash --> Pick[Pick Tier-1 agent<br>Maya Sofia Priya<br>Zara Leila Cleo]
    Pick --> Tier1[Route to Selected Tier-1<br>OpenAI GPT-4 class]
    Tier1 --> Reply([Response + log])
    Nadia --> Reply

    style Nadia fill:#FFB366,stroke:#D97706,stroke-width:2px,color:#000
    style Tier1 fill:#A89CE8,stroke:#412991,stroke-width:2px,color:#000
    style EscEvent fill:#FF8888,stroke:#FF4444,color:#000
```

### Weekly rotation across users

```mermaid
flowchart LR
    subgraph Week_1
        U1a[User A] --> M1[Maya]
        U2a[User B] --> S1[Sofia]
        U3a[User C] --> P1[Priya]
    end

    subgraph Week_2
        U1b[User A] --> Z2[Zara]
        U2b[User B] --> L2[Leila]
        U3b[User C] --> C2[Cleo]
    end

    Week_1 -.->|next week<br>hash rotates| Week_2

    style M1 fill:#C4B8F0,color:#000
    style S1 fill:#C4B8F0,color:#000
    style P1 fill:#C4B8F0,color:#000
    style Z2 fill:#E0D4FF,color:#000
    style L2 fill:#E0D4FF,color:#000
    style C2 fill:#E0D4FF,color:#000
```

## Why this pattern

| Alternative | Rejected because |
|---|---|
| Single bot, always | Samey. No escalation recourse. No senior-agent trust signal. |
| Random agent per session | Breaks continuity mid-session. |
| Human escalation only | Not sustainable solo. Would have blocked launch. |
| Pure LLM routing / agent swarm | Over-engineered for v1 volume. Hard to debug. |

Weekly rotation + triggered escalation is **deterministic, debuggable, humane** — three things agentic systems typically aren't.

## Bug & fix · v2.0.7

Escalated sessions reverted to Tier-1 on the next message. Root cause: stale `escalated_to_agent` default on message insert.

Same-day fix. Affected sessions backfilled. CHANGELOG v2.0.7.

**Preventive:** added a 2-message escalation smoke test to the pre-launch checklist. Would have caught it.

## Governance built-in

- **"Are you a bot?"** — sidestepped warmly. We don't claim human status (deceptive); we don't volunteer disclosure (breaks immersion). ToS discloses AI use. LLB-reviewed.
- **Transcripts persist** in `chat_messages` — dispute defense if a user claims "the bot told me X"
- **UI event messages** (`Maya joined the chat`, `Connecting to senior agent...`) are filtered out of the OpenAI context — no reasoning pollution

## What this evidences

- **Product judgment** — six rotating personas is a real design call, not a coin-flip
- **Systems thinking** — deterministic hashing for distribution + consistency
- **Operational discipline** — same-release fix + preventive test
- **Governance awareness** — bot disclosure, transcript logging, ToS alignment
