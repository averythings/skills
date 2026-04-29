---
name: indie-launch-checklist
description: Pre-ship technical readiness skill for solo builders. Runs a 30-minute checklist before a public launch (Product Hunt, HN, X) covering env vars, abuse guards, observability, analytics, rollback, and scaling posture.
license: MIT
metadata:
  author: averythings
  version: "0.1.0"
  repository: https://github.com/averythings/skills
---

# Indie Launch Checklist

Agent skill that walks a solo builder through the **final 30 minutes before a public launch**. It is the opposite of a marketing launch plan: this is "did I break prod before the traffic arrives."

Marketing-side launch skills (go-to-market, pricing, pre-mortem) already exist in the ecosystem. This skill handles the *engineering* half that no one blogs about.

---

## When to invoke

Invoke this skill when the user says any of:

- "I'm launching on Product Hunt tomorrow"
- "About to tweet this, what did I forget"
- "Help me do a launch checklist"
- "Ship day readiness"
- "Pre-launch review"

Do **not** invoke for ordinary deploys — this is a public-traffic-spike checklist, not a release checklist.

---

## Inputs

The skill needs, in order of importance:

1. **Project root path** — to scan for `.env.example`, `package.json`, framework config.
2. **Production URL** — to run live HEAD checks and robots/headers audit.
3. **Hosting platform** — Vercel / Netlify / Fly / Railway / Render / bare VPS (affects the scaling questions).
4. **Analytics tool** — Plausible / PostHog / GA / none.
5. **Error tracker** — Sentry / Highlight / Axiom / none.
6. **Payment layer** — Stripe / Lemon Squeezy / none.
7. **Model provider** — if the product calls an LLM, which one (affects cost guard integration — see `llm-cost-guardrail` skill).

If any are missing, ask once, then proceed with best-effort defaults.

---

## Outputs

A single markdown report, `LAUNCH_READINESS.md`, with:

- **Green / Yellow / Red** status per check
- **Fix-now** actions (with commands) for every Red
- **Good-to-haves** for every Yellow
- **Rollback plan** section (1 paragraph, concrete)
- **Traffic-spike posture** section (what breaks first, at what RPS, how to tell)

No other side effects. The skill never deploys, never rotates keys, never posts anything.

---

## The Checklist (eight categories, 24 items)

### 1. Secrets & environment
- [ ] No `.env`, `.env.production`, or `secrets.json` in git history (`git log --all --full-history -- .env*`)
- [ ] `.env.example` is up to date with every required var
- [ ] Production env vars set in hosting dashboard match `.env.example` keys 1:1
- [ ] No test/staging API keys in production (grep for `sk_test_`, `_test_`, `localhost`)

### 2. Abuse & rate limits
- [ ] Every public POST endpoint has a rate limit (per-IP or per-user)
- [ ] Signup has email verification OR is rate-limited (stop bot signups draining free tier)
- [ ] LLM-calling endpoints have a per-request token cap and per-user daily cap *(hand off to `llm-cost-guardrail` skill)*
- [ ] File-upload endpoints have size + MIME allowlist

### 3. Observability
- [ ] Error tracker installed and receiving a test event from prod in the last 24h
- [ ] Uptime monitor on production URL (UptimeRobot / BetterStack / built-in)
- [ ] Log retention set to ≥ 7 days (hosting provider default is often 1 day)
- [ ] Alert channel confirmed — user has received at least one test alert on their phone

### 4. Analytics on the money funnel
- [ ] Top-of-funnel event fires (landing page view)
- [ ] Activation event fires (signup / first meaningful action)
- [ ] Conversion event fires (paid / upgrade / key feature used)
- [ ] Events visible in the analytics dashboard *right now* (not "should be")

### 5. Payments (skip if free)
- [ ] Stripe webhook endpoint responds 200 to a replay of `checkout.session.completed`
- [ ] Failed-payment email template exists and renders
- [ ] Customer portal link works end-to-end (cancel / update card)
- [ ] Refund path documented for self (even a note in Notion)

### 6. Rollback plan
- [ ] Previous deploy is still reachable (Vercel / Netlify / Fly keep the last N — user confirms which N)
- [ ] One-command rollback is documented in README or `docs/RUNBOOK.md`
- [ ] Database migrations are backwards-compatible OR a rollback migration exists

### 7. Scaling posture
- [ ] Know the first bottleneck: DB connections? Edge function concurrency? LLM rate limit?
- [ ] Know the threshold: "things break above ~X RPS / ~Y req/min"
- [ ] Know the tell: which dashboard shows it first
- [ ] Know the lever: "if we hit it, I can do Z in < 5 min" (scale up, disable feature, cache, queue)

### 8. Content & SEO sanity (cheap wins before traffic)
- [ ] OG image + title + description render correctly (`curl -I` + OG preview tool)
- [ ] `/robots.txt` exists and is not blocking everything
- [ ] `sitemap.xml` is valid and linked
- [ ] 404 and 500 pages exist and are on-brand

---

## How the agent should run it

1. Read the project root quickly (package.json / framework config / env files).
2. For each of the 24 checks, mark ✅ / ⚠️ / ❌ based on evidence. Prefer actual file/URL inspection over asking the user.
3. For ❌ items, write a **Fix-now** section with the exact command or 3-step fix.
4. For ⚠️ items, write a **Good-to-have** section — one line each, don't over-explain.
5. End with the **Rollback plan** and **Traffic-spike posture** sections — these are forced narrative so the user has to think, not just check boxes.
6. Never mark ✅ without evidence. "Probably fine" is a ⚠️.

---

## Composition

This skill is designed to compose with:

- **`llm-cost-guardrail`** (this repo) — handoff point for check 2.3 (LLM endpoint caps).
- **`phuryn/pre-mortem`** (external) — upstream; run their pre-mortem weeks before, this skill hours before.
- **`coreyhaines31/launch-strategy`** (external) — marketing-side launch plan runs in parallel, not as dependency.

---

## Non-goals

- Not a replacement for an SRE runbook on a mature product.
- Not a security audit — explicitly trust-but-verify level. For serious security posture, hand off to `trail-of-bits` or `prompt-security/clawsec`.
- Not a performance benchmark — does not run load tests, just predicts first-break.
