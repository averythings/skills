---
name: llm-cost-guardrail
description: Runtime cost and budget guardrails for LLM-powered features. Adds per-request token caps, per-user daily spend limits, model-tier routing, and a kill switch before a product quietly bleeds money.
license: MIT
metadata:
  author: averythings
  version: "0.1.0"
  repository: https://github.com/averythings/skills
---

# LLM Cost Guardrail

Agent skill that installs **runtime guardrails** on any LLM-powered endpoint so that a scraper, a bug, or a viral Tuesday does not drain a founder's OpenAI / Anthropic / Gemini account.

The ecosystem has excellent **observability** (Datadog, Arize) and **evals** (hamelsmu). It is thin on **enforcement** — the thing that actually stops the bleeding. This skill closes that gap with a copy-paste-ready pattern in TypeScript or Python.

---

## When to invoke

Invoke when the user says any of:

- "Put a cost cap on my LLM endpoint"
- "I'm burning money on OpenAI, help"
- "Add a daily spend limit per user"
- "Route cheap queries to a cheaper model"
- "Kill switch for my AI feature"

Also invoke as a **sub-skill** from `indie-launch-checklist` for item 2.3 (LLM endpoint caps).

---

## Inputs

1. **Provider** — openai | anthropic | google | openrouter | together | other.
2. **Language** — typescript (node/next/bun) | python (fastapi/flask/django).
3. **Framework entry point** — the file/route where the LLM is called today.
4. **Current model** — e.g. `gpt-4o`, `claude-sonnet-4-5`, `gemini-2.5-pro`.
5. **Auth model** — anonymous allowed? logged-in only? tier-aware?
6. **State store** — redis | upstash | postgres | in-memory (dev only).
7. **Daily budget USD** — hard cap for the whole product.
8. **Per-user daily cap** — USD or token count.

Ask at most 2 of these at a time. Infer the rest from the codebase.

---

## Outputs

A patch to the user's project adding **four layers** of protection, composable and independently toggle-able:

### Layer 1 — Per-request token cap
Hard `max_tokens` + input-truncation. Refuses to call the model if the prompt already exceeds budget.

### Layer 2 — Per-user rolling window
Sliding window counter (default: last 24h) keyed on `user_id` (or IP for anonymous). Returns `429` with a retry-after header when exceeded.

### Layer 3 — Model-tier router
Task-complexity heuristic routes to cheapest viable model. Escalates to premium only when:
- explicit `quality: "high"` flag, OR
- cheap-model output fails a lightweight validator (schema / length / refusal pattern).

### Layer 4 — Global kill switch
A single boolean (env var OR feature flag) that short-circuits every LLM call with a graceful fallback response. Tripped automatically when daily spend exceeds the configured hard cap; resets at UTC midnight.

Plus a **/admin/cost** read-only page showing today's spend, top 10 users by cost, and the kill-switch state.

---

## Architecture

```
  request
    │
    ▼
[auth + rate-limit middleware]
    │
    ▼
[kill-switch check] ──tripped──► fallback response (202 + graceful message)
    │
    ▼
[per-user budget check] ──over──► 429 + retry-after
    │
    ▼
[input-truncate to max_tokens_in]
    │
    ▼
[model router]
    │
    ├──simple──► cheap model (e.g. gpt-4o-mini, haiku-4-5, gemini-flash)
    │              │
    │              ▼
    │        [validator]
    │              │
    │        ok ───┘
    │              │
    │        fail──► retry with premium model (counted as 1 premium call)
    │
    └──flagged high-quality──► premium model
    │
    ▼
[meter spend → state store]
    │
    ▼
  response
```

---

## Defaults the skill will apply (can be overridden)

| Knob | Default | Why |
|---|---|---|
| `MAX_TOKENS_IN` | 4000 | Covers 95% of real prompts; caller can bump per-endpoint |
| `MAX_TOKENS_OUT` | 1000 | Forces concise outputs; caller can raise for summarization |
| `ANONYMOUS_DAILY_CAP` | $0.10 | Generous enough for demo users, lethal to scrapers |
| `AUTHED_FREE_DAILY_CAP` | $0.50 | Enough for real usage on free tier |
| `AUTHED_PAID_DAILY_CAP` | $5.00 | Tweakable per plan |
| `GLOBAL_DAILY_HARD_CAP` | $50 | Founder-chosen. Kill-switch trips above this. |
| `WINDOW` | 24h rolling | Simpler than calendar day, avoids midnight-thundering-herd |

Costs are computed from the provider's published per-token rates, hardcoded in a `COSTS` table the skill ships with a link to update (provider pricing pages).

---

## Deliverables (files written to the user's repo)

```
lib/
  cost-guard/
    index.ts           # public API: guard(fn, opts)
    rate-limit.ts      # per-user rolling window
    router.ts          # model-tier selection
    kill-switch.ts     # global circuit breaker
    meter.ts           # spend accounting
    costs.ts           # provider price table
    README.md          # how to use, how to update price table
app/
  api/
    admin/
      cost/
        route.ts       # /admin/cost dashboard JSON
```

Python variant mirrors structure under `app/cost_guard/`.

The skill never replaces the user's existing LLM call — it wraps it. One-line change at the call site:

```ts
// before
const result = await openai.chat.completions.create({ ... });

// after
const result = await guard(
  (model) => openai.chat.completions.create({ model, ... }),
  { user: userId, task: "summarize", quality: "auto" }
);
```

---

## Non-goals

- Not an eval framework. Hand off to hamelsmu's eval skills for quality measurement.
- Not an observability dashboard. Complements Datadog/Arize — sends OTEL spans if present, doesn't replace them.
- Not a prompt-optimization tool. Use `skill-optimizer` or `hqhq1025/skill-optimizer` for prompt shrinkage.
- Does **not** negotiate provider contracts, enterprise quotas, or reserved capacity. This is app-level.

---

## Composition

- **Up-stream**: `indie-launch-checklist` calls into this skill at check 2.3.
- **Parallel**: pairs well with `zscole/model-hierarchy-skill` (broader routing strategy) and `hamelsmu/evaluate-rag` (for measuring quality after routing).
- **Down-stream**: the `/admin/cost` endpoint's JSON is a natural data source for a dashboard UI (see `UI.md`).
