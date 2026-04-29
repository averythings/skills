# Gap Analysis — `awesome-agent-skills`

> Source: [`averythings/awesome-agent-skills`](https://github.com/averythings/awesome-agent-skills) — ~1,100+ skills across ~50 curated sections (as of April 2026).
> Method: categorized every `### Skills by …` and `<summary>` section in the README into coarse domains, counted representative skills per bucket, and looked for obvious white space from a **solo builder / indie dev** perspective.

---

## 1. Domain map of existing skills

The catalog is large but lopsided. Grouping the ~27 official vendor sections plus ~7 community super-sections into domains:

| Domain | Representative sections | Approx. share | Notes |
|---|---|---|---|
| **Backend / infra / data** | Supabase, Neon, ClickHouse, Tinybird, DuckDB, MongoDB, HashiCorp, Cloudflare, Netlify, Vercel, Firebase, qdrant | ~25% | Heavily served — every major database and edge platform has official skills. |
| **AI model providers & SDKs** | Anthropic, OpenAI, Gemini, fal.ai, Replicate, MiniMax, Hugging Face, x.ai, Together | ~15% | Saturated on "how to call the API"; thin on *operating* LLMs in production. |
| **Dev workflow & code quality** | CodeRabbit, Trail of Bits, obra/superpowers, NeoLabHQ, mattpocock, Matteo Collina, LambdaTest, callstack | ~18% | Strong on TDD, review, upgrades; weak on release/launch orchestration. |
| **Frontend & design** | Angular, Figma, GSAP, Remotion, SwiftUI, taste-skill, ibelick/ui-skills, apple-hig-skills | ~10% | Design-system fluency is good; *ship with great UI* wrappers for skills are absent. |
| **Mobile / native** | Expo, Flutter, React Native (callstack), iOS simulator, App Store tooling | ~7% | Covered. |
| **Marketing / content / PM** | Corey Haines, Kim Barrett, Dean Peters, Paweł Huryn, Typefully, Resend | ~10% | Strong strategy skills; light on *daily operator* rituals (status updates, changelogs, pricing-page copy). |
| **Finance / legal / specialized** | EveryInc CFO, openaccountants, awesome-legal-skills, Coinbase, Binance, helius (Solana) | ~5% | Niche but present. |
| **Security** | Trail of Bits, VibeSec, prompt-security, 753-skill cybersecurity pack | ~4% | Covered. |
| **Automation / glue** | n8n, Zapier-adjacent, Composio, Courier | ~3% | Covered. |
| **Research / science / specialty** | K-Dense-AI, materials-sim, genealogy, health, music | ~3% | Niche but present. |

**Headline observation**: the catalog is **deep on *building* (SDKs, components, frameworks)** and **thin on *operating* (cost, launch day, support inbox, release rhythm)**. For a solo builder who has to be PM + eng + ops + marketer, the gap is in the "operating" half.

---

## 2. Identified gaps (high-leverage for indie devs)

Ranked by leverage × absence:

### Gap A — Pre-launch / ship-day technical readiness checklist  *(TOP PICK)*
Marketing-side launch skills exist (`coreyhaines31/launch-strategy`, `phuryn/gtm-strategy`, `phuryn/pre-mortem`). **No skill runs the pre-ship technical checklist** a solo founder does 30 minutes before posting to Product Hunt / HN / X:
- env vars sanity-checked in prod
- rate limits + abuse guards on public endpoints
- error tracking wired
- analytics events on the top 3 funnel steps
- rollback plan
- "I got to the front page" scaling posture

Nothing in the catalog bundles these into one callable skill.

### Gap B — LLM runtime cost/budget guardrails  *(TOP PICK)*
There are **LLM observability** skills (Datadog, Arize-via-Azure) and **eval** skills (hamelsmu, `evaluate-rag`), but **no skill that enforces a runtime cost budget** — the thing indie devs actually burn cash on. Missing:
- per-request token budget / max-cost caps
- model-routing by task complexity (only `zscole/model-hierarchy-skill` touches this, narrowly)
- automatic degrade to cheaper model when daily spend crosses a threshold
- cost attribution per user / per feature
- "shut the feature off if it's losing money" kill-switch pattern

This is the #1 thing that kills small AI products in week 2.

### Gap C — Support-inbox triage for indie devs
`notion`, `linear-claude-skill`, `coderabbit` handle *internal* workflow. **No skill** ingests a founder's unified support inbox (email + Discord + X DMs) and triages into: bug / feature-request / billing / churn-risk / duplicate. `obra/superpowers` has generic productivity; none handle the founder-specific triage loop.

### Gap D — Changelog-first release pipeline
`phuryn/release-notes` generates user-facing notes from tickets. There's no skill that **chains**: merged PRs → grouped changelog entry → in-app "what's new" card → email/X post → changelog.md commit. Every serious indie product (Linear, Cal, Raycast) ships on this rhythm; the skill catalog does not.

### Gap E — Pricing-page + paywall copy generator
`phuryn/pricing-strategy` and `deanpeters/finance-based-pricing-advisor` cover **strategy**. No skill writes the *page + paywall* copy (tier names, feature rows, FAQs, upgrade-modal microcopy) tied to a Stripe products.json. This is Friday-afternoon-indie work.

---

## 3. What gets built in this PR

To stay disciplined (quality > quantity), this PR ships the **top 2 gaps**:

1. **`indie-launch-checklist`** — addresses **Gap A**
2. **`llm-cost-guardrail`** — addresses **Gap B**

Each ships with `SKILL.md`, `prompt.md`, `examples/`, and a `UI.md` spec referencing **Linear's DESIGN.md** from `averythings/awesome-design-md`.

### Why Linear as the design anchor?
- Of the ~60 DESIGN.md files in `awesome-design-md`, Linear is the closest match for a solo-builder/dev-tools aesthetic: dark-mode-native, ultra-thin borders, Inter 510 as the workhorse weight, monochrome-plus-one-accent system.
- It's opinionated enough to copy-paste from, simple enough that a solo builder can ship a UI wrapper in an evening.
- Vercel and Raycast were the runner-ups; both skew heavier (Vercel = marketing surface, Raycast = native command palette). Linear is the right middle.

---

## 4. Gaps explicitly *not* addressed here (parking lot)

- **Gap C** (support-triage) — needs inbox integrations (Gmail/Discord/X APIs); out of scope for a docs-first skill.
- **Gap D** (changelog pipeline) — overlaps with `phuryn/release-notes`; would need a follow-up that glues PRs + email + in-app.
- **Gap E** (pricing-page copy) — could be a third skill later; Stripe already publishes `stripe-best-practices`, so there's prior art to lean on.

These are tracked here so future PRs can pick them up without re-running this analysis.
