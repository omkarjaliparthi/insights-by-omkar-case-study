# 09 · Decision Log

Every meaningful decision on this program is captured as a `project_plan` row (jsonb) or a `project_decision` row (data_before / data_after). This is how program state stays auditable.

Below: five pivotal strategic documents from the early launch phase — structure preserved, implementation-level details abstracted where sensitive.

---

## Plan 1 · Launch Priority Schedule — 5-Batch Rollout

**Type:** `strategy` · **Status:** `approved`

Phased vertical launch sequenced by **readiness × search volume × revenue potential**. Core rule: never launch more than one vertical per week, and intelligence must approve each launch (see Vertical Readiness scoring).

| Batch | Scope | Rationale |
|---|---|---|
| **0** | Intelligence only — 4 crons active, no new public pages, 1-2 weeks of observation | Turn on the brain before shipping more surface |
| **1** | Angel Numbers → Moon Phases → Chakras → Candle Colors → Custom Spell Generator (first paid AI feature) | High-search, already-built, low risk |
| **2** | Herbs + Crystals content generation in house voice | Fills empty data files, no new surface |
| **3** | Dream Journal + Rune Cast AI features, each with reference content | AI feature ships WITH its reference pages |
| **4** | Auras + Numerology + Charms + Numerology Calculator | Lowest search volume, last priority |

**Explicit rules baked into the plan:**
- Never launch more than one vertical per week
- Intelligence (Vertical Readiness scoring) must approve each launch
- Monitor Core Web Vitals after each launch
- Add 301 redirects at launch time for URL migrations
- Each launch gets 3-5 social posts auto-generated through the content pipeline

*Why this matters:* prevents excitement-driven launches that over-reach capacity. Past pattern most solo founders fall into — ship everything, lose quality, chase fires. This plan pre-commits to restraint.

---

## Plan 2 · Performance Audit — April 8, 2026

**Type:** `implementation_plan` · **Status:** `completed`

**Problem:** Dev server loading in 40s.

**Root causes identified:**
1. Duplicate routes — 8 verticals existed under both `/spells/X` and `/X` → 89 pages compiling
2. Sitemap module-scope imports → ~1MB of data arrays loading at module scope on every build
3. Root layout DB query → support agent settings query hitting Supabase on every page render
4. Heavy animation deps — three.js, framer-motion, gsap, rive loaded in bundle

**Fixes shipped:**
- ✅ Deleted duplicate `/spells/` routes → 89 → 81 pages
- ✅ Converted sitemap to async with dynamic imports via `Promise.all`
- ✅ Added dev-mode skip for support chat widget DB query

**Explicitly deferred:**
- ⬜ Audit animation libraries — determine which are actually used
- ⬜ Add `googleapis` to `serverExternalPackages` in `next.config.ts`
- ⬜ Consider route-level code splitting for heavy admin pages

*Why this matters:* the deferred items are **named**, not glossed over. Deferred work that isn't documented becomes technical debt that disappears into memory. Deferred work that's documented becomes a queue.

---

## Plan 3 · Full Status Audit — April 8, 2026

**Type:** `brainstorm` · **Status:** `approved`

Comprehensive snapshot of all 13 content verticals and every system:

| Status | Verticals | Count |
|---|---|---|
| Live | Tarot Cards (78), Tarot Spreads (10) | 88 pages |
| Ready to launch | Angel Numbers (30), Moon Phases (8), Chakras (7), Candle Colors (15) | 60 entries |
| Building — empty data files | Herbs, Crystals | 0 entries |
| Not started | Dream Symbols, Runes, Auras, Numerology, Charms | — |

**System status:**
- Intelligence system: **100% built** — 7 files, 1,753 lines (Chandra, Surya, Brahma, Vishnu, Shiva, Triad, Vertical-Readiness)
- Admin: 95% built
- Legal: ToS + Disclaimer + Privacy + Refund Policy all updated to cover all 13 verticals
- URL structure: top-level migrations complete, 301 redirects queued

*Why this matters:* the audit isn't vibes-based. Every vertical has a page count, every system has a completion %, every gap has a clear next action. **This is what "knowing the state of your program" looks like in practice.**

---

## Plan 4 · AI-Powered Paid Features — Credits Monetization

**Type:** `system_design` · **Status:** `approved`

Five AI features, each with a specified cost, route, table, and shared implementation pattern.

| Feature | Credits | Route | Table |
|---|---|---|---|
| Custom Spell Generator | 2 | `/spells/create` | `custom_spells` |
| Spell Refinement | 1 | `/spells/create` (follow-up, versioned) | same |
| Dream Journal / Interpreter | 2 | `/dreams/journal` | `dream_journal_entries` |
| Rune Cast | 1 | `/runes/cast` | `rune_cast_results` |
| Numerology Calculator | 1 | `/numerology/calculator` | `numerology_profiles` |

**Shared contract across all 5 (written before any were built):**

1. Supabase migration → table with RLS (user sees own rows, admin sees all)
2. API route → auth check → credit check → AI generation → credit deduction → save result → return
3. Client page → input form → loading state → result display → save confirmation
4. Dashboard tab → list saved results → expand to view full detail → re-generate option
5. Credit ledger entry → stable `reason` string (e.g. `chamber.spell-create`)

**AI model strategy:** Claude Sonnet primary (better for spiritual/creative content), GPT-4o fallback (if Claude rate-limited or down). Every generation emits `model_used` for audit.

**Numerology is uniquely architected:** calculator is deterministic math (runs client-side for instant feedback), AI interpretation is the *paid* part. Separates free from paid by actual value.

*Why this matters:* writing the shared pattern before building the first feature prevented five different implementations. Shared pattern = one contract to audit, refactor, and extend. Staff-engineer move.

---

## Plan 5 · Approval Workflow — Manual vs Auto Mode

**Type:** `implementation_plan` · **Status:** `approved`

**Problem:** Content pipeline auto-publishes after passing all 5 AI layers with zero human review. Intelligence recommendations auto-apply without admin awareness. Risky for first-time launch.

**Solution:** two toggle settings + a three-phase rollout ramp.

### Two governance toggles

| Setting | Default | Behavior when ON |
|---|---|---|
| `require_content_approval` | TRUE | Content stops at `awaiting_approval` after Triad; admin reviews and clicks approve/reject |
| `require_intelligence_approval` | TRUE | Recommendations appear as `pending`; admin reviews each before apply |

### Three-phase confidence-building workflow

| Phase | When | Settings | Posture |
|---|---|---|---|
| **Phase 1** | Now | Both ON | Review everything manually. Build confidence. |
| **Phase 2** | After 2-4 weeks | Intelligence approval OFF | Let Tier-1 recs auto-apply. Still review content. |
| **Phase 3** | When comfortable | Both OFF (for routine post types only) | Automate what's been seen to work. |

**Hard rule:** *"Never turn off approval for new content types or verticals you haven't seen output from."*

*Why this matters:* most people either (a) trust the AI immediately and get burned, or (b) never trust it and waste the automation. This plan **encodes a trust-building ramp** as a product setting, not a vibe. The rule at the end — never skip phase 1 for new content — is a safety invariant that survives across team changes and phase transitions.

---

## What this log evidences

Every plan above is a concrete artifact — stored, versioned, timestamped, auditable — not a slide deck or a memory.

For a PM/TPM audience, these demonstrate:

1. **Phased rollout thinking** — sequencing by risk × readiness × value, not by excitement
2. **Root-cause discipline** — performance problems traced to four causes, each named and addressed or explicitly deferred
3. **Written shared contracts before implementation** — five features, one contract
4. **Governance as a setting, not a meeting** — approval toggles and phase rules encoded in the product
5. **Hard invariants that survive state changes** — rules that protect future-you from present-you

All plans were written before the work they describe was shipped. All decisions they reference are stored in the `project_plans` and `project_decisions` tables with full audit history.

---

<p align="center">
  <a href="./08-user-flows.md">← User Flows</a> · <a href="../README.md">Back to overview</a>
</p>
