# Example Run — indie-launch-checklist

> A full example of the skill being invoked on a fictional Next.js + Supabase + Stripe side project called **Patchnote** the day before a Product Hunt launch.

---

## User invocation

```
> I'm launching Patchnote on Product Hunt at 6am PT tomorrow. Run the launch checklist on ~/code/patchnote. Prod is patchnote.app. Hosting is Vercel, analytics is Plausible, errors on Sentry, payments on Stripe.
```

## Skill output (summary, shown in chat)

```
Launch Readiness — patchnote
Status: RED · Score: 15 ✅ / 5 ⚠️ / 4 ❌ of 24

Fix now (4 blockers):

1. ❌ .env.production committed in commit 8a3f2b1 on 2026-03-12.
   Rotate every key in that file. Then:
     git filter-repo --path .env.production --invert-paths
   Then rotate Supabase service key, Stripe secret, and OpenAI key in their dashboards.

2. ❌ /api/summarize has no rate limit. At $0.004/call, 1000 anonymous hits = $4.
   Add upstash/ratelimit middleware; cap anonymous at 3/min, authed at 30/min.

3. ❌ Sentry has received zero events from prod in the last 24h.
   Likely DSN is set but init() isn't being called. Check app/layout.tsx.

4. ❌ No rollback plan documented.
   Vercel keeps the last 10 deploys. Add to README.md:
     "Rollback: vercel rollback <prev-deployment-url>"
   Current previous URL: patchnote-git-main-xyz.vercel.app

Report written to ~/code/patchnote/LAUNCH_READINESS.md

Fix these now, or launch anyway?
```

## Full `LAUNCH_READINESS.md` (written to disk)

```markdown
# Launch Readiness — patchnote

**Generated:** 2026-04-28T22:14:07Z
**Status:** RED
**Score:** 15 ✅ / 5 ⚠️ / 4 ❌ out of 24

## Fix Now (blockers)

### ❌ Secret committed to git history
`.env.production` appears in commit 8a3f2b1 (2026-03-12). Even though it's deleted from HEAD, the keys are public if the repo is ever made public.

**Fix:**
1. Rotate Supabase service role key in Supabase dashboard → Settings → API.
2. Rotate Stripe secret in Stripe dashboard → Developers → API keys → Roll.
3. Rotate OpenAI key at platform.openai.com → API keys.
4. Scrub history: `git filter-repo --path .env.production --invert-paths && git push --force`

### ❌ Unrated-limited LLM endpoint
`/api/summarize` calls OpenAI with no per-IP cap. A single scraper can drain the daily budget.

**Fix:** add `@upstash/ratelimit` with `Ratelimit.slidingWindow(3, "1 m")` for anonymous and `(30, "1 m")` for authed. See `skills/llm-cost-guardrail` for the full pattern.

### ❌ Sentry is silent
DSN is set in env but no events received. Init likely not running.

**Fix:** in `app/layout.tsx`, confirm `Sentry.init({ dsn: process.env.NEXT_PUBLIC_SENTRY_DSN })` runs at module load. Test with `throw new Error("launch-test")` on a protected route, confirm it appears in Sentry.

### ❌ No rollback plan
Not documented. Under launch-day adrenaline, you will not remember the command.

**Fix:** add to `README.md`:
```
## Rollback
vercel rollback patchnote-git-main-xyz.vercel.app
```

## Good to Have

- ⚠️ File-upload endpoint `/api/import` has no size cap. Add `maxBodySize: "5mb"`.
- ⚠️ Analytics: "upgrade clicked" event not verified in Plausible dashboard — confirm it fires.
- ⚠️ 500 page is the Next.js default. 10 minutes to make it on-brand.
- ⚠️ Uptime monitor not set. UptimeRobot free tier + 5 min checks is fine.
- ⚠️ Log retention on Vercel is 1 day by default. Pro tier = 30 days if you care.

## Rollback Plan

Previous stable deploy: `patchnote-git-main-xyz.vercel.app` (commit `ab12cd3`). Rollback command: `vercel rollback patchnote-git-main-xyz.vercel.app`. Database migration `20260427_add_teams.sql` is additive (new table, no drops) — safe to roll back app without reverting DB. If you need to revert the DB, use Supabase point-in-time recovery to 2026-04-27 18:00 UTC.

## Traffic-Spike Posture

**First bottleneck:** OpenAI rate limit on `/api/summarize` (3500 RPM on tier 2).
**Threshold:** ~60 concurrent users each summarizing once per minute.
**Tell:** Sentry surfaces `RateLimitError` before users complain.
**Lever:** In the admin route, flip `FEATURE_SUMMARIZE=off` — the UI already falls back to "temporarily unavailable" (verified in code).

## Green Checks

- ✅ .env.example up to date (8/8 keys match)
- ✅ No sk_test_ in production env
- ✅ Signup requires email verification
- ✅ Stripe webhook responds 200 on replay of checkout.session.completed
- ✅ Failed-payment email renders
- ✅ Customer portal link works
- ✅ Previous Vercel deploy is still reachable
- ✅ Migration is backwards-compatible (additive only)
- ✅ OG image renders at 1200×630
- ✅ robots.txt exists and is permissive
- ✅ sitemap.xml validates
- ✅ 404 page is on-brand
- ✅ Landing page view event fires
- ✅ Signup event fires
- ✅ Checkout.completed event fires
```

---

## Why this is the whole example

One run. One report. One question at the end. The skill is designed to be uninteresting in the best way — the user either fixes the Reds or consciously ships with them.
