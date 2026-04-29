# Example — TypeScript / Next.js / Upstash Redis

End-to-end example: wrapping an existing `app/api/summarize/route.ts` that calls OpenAI.

## Before

```ts
// app/api/summarize/route.ts
import OpenAI from "openai";
const openai = new OpenAI();

export async function POST(req: Request) {
  const { text } = await req.json();
  const r = await openai.chat.completions.create({
    model: "gpt-4o",
    messages: [{ role: "user", content: `Summarize: ${text}` }],
  });
  return Response.json({ summary: r.choices[0].message.content });
}
```

## After (1-line change at call site)

```ts
// app/api/summarize/route.ts
import OpenAI from "openai";
import { guard } from "@/lib/cost-guard";

const openai = new OpenAI();

export async function POST(req: Request) {
  const { text } = await req.json();
  const userId = /* from your auth */ "anon-" + req.headers.get("x-forwarded-for");

  const r = await guard(
    (model) => openai.chat.completions.create({
      model,
      messages: [{ role: "user", content: `Summarize: ${text}` }],
    }),
    { user: userId, task: "summarize", quality: "auto" }
  );

  return Response.json({ summary: r.choices[0].message.content });
}
```

## What the skill generates

### `lib/cost-guard/costs.ts`
```ts
// Provider pricing as of 2026-04-28. Update from:
//   https://openai.com/pricing
//   https://anthropic.com/pricing
//   https://ai.google.dev/pricing
// Prices are USD per 1M tokens. Keep integer micro-cents for safety.

export const COSTS = {
  "gpt-4o":         { in: 2_500_000, out: 10_000_000 }, // micro-cents per 1M tokens
  "gpt-4o-mini":    { in:   150_000, out:    600_000 },
  "claude-sonnet-4-5":{ in: 3_000_000, out: 15_000_000 },
  "claude-haiku-4-5": { in:   250_000, out:  1_250_000 },
  "gemini-2.5-pro": { in: 1_250_000, out:  5_000_000 },
  "gemini-2.5-flash":{ in:   100_000, out:    400_000 },
} as const;

export type Model = keyof typeof COSTS;

export function costMicroCents(model: Model, inTok: number, outTok: number) {
  const p = COSTS[model];
  return Math.ceil((inTok * p.in + outTok * p.out) / 1_000_000);
}
```

### `lib/cost-guard/router.ts`
```ts
import type { Model } from "./costs";

export function pickModel(task: string, quality: "auto" | "high"): Model {
  if (quality === "high") {
    if (task === "code") return "claude-sonnet-4-5";
    return "gpt-4o";
  }
  // cheap defaults by task
  const map: Record<string, Model> = {
    summarize: "gpt-4o-mini",
    classify:  "gemini-2.5-flash",
    extract:   "gpt-4o-mini",
    code:      "claude-haiku-4-5",
  };
  return map[task] ?? "gpt-4o-mini";
}
```

### `lib/cost-guard/kill-switch.ts`
```ts
import { redis } from "./redis";

const HARD_CAP_MICROCENTS = Number(process.env.LLM_DAILY_HARD_CAP_USD ?? 50) * 100_000_000;

export async function isKilled(): Promise<boolean> {
  if (process.env.LLM_KILL_SWITCH === "true") return true;
  const dayKey = `spend:global:${new Date().toISOString().slice(0, 10)}`;
  const spent = Number((await redis.get(dayKey)) ?? 0);
  return spent >= HARD_CAP_MICROCENTS;
}
```

### `lib/cost-guard/rate-limit.ts`
```ts
import { redis } from "./redis";

const LIMITS = {
  anon:  10 * 100_000_000,   // $0.10
  free:  50 * 100_000_000,   // $0.50
  paid: 500 * 100_000_000,   // $5.00
} as const;

export async function checkBudget(user: string, tier: keyof typeof LIMITS) {
  const key = `spend:user:${user}:${new Date().toISOString().slice(0, 10)}`;
  const spent = Number((await redis.get(key)) ?? 0);
  if (spent >= LIMITS[tier]) {
    return { ok: false, retryAfter: secondsUntilUtcMidnight() } as const;
  }
  return { ok: true } as const;
}

function secondsUntilUtcMidnight() {
  const now = new Date();
  const midnight = new Date(Date.UTC(now.getUTCFullYear(), now.getUTCMonth(), now.getUTCDate() + 1));
  return Math.floor((+midnight - +now) / 1000);
}
```

### `lib/cost-guard/meter.ts`
```ts
import { redis } from "./redis";
import { costMicroCents, Model } from "./costs";

export async function meter(user: string, model: Model, inTok: number, outTok: number) {
  const cost = costMicroCents(model, inTok, outTok);
  const day = new Date().toISOString().slice(0, 10);
  await Promise.all([
    redis.incrby(`spend:user:${user}:${day}`, cost),
    redis.incrby(`spend:global:${day}`,       cost),
    redis.expire(`spend:user:${user}:${day}`, 86_400 * 2),
    redis.expire(`spend:global:${day}`,       86_400 * 2),
  ]);
  return cost;
}
```

### `lib/cost-guard/index.ts` (public API)
```ts
import { pickModel } from "./router";
import { isKilled } from "./kill-switch";
import { checkBudget } from "./rate-limit";
import { meter } from "./meter";
import type { Model } from "./costs";

type GuardOpts = {
  user: string;
  task: string;
  quality?: "auto" | "high";
  tier?: "anon" | "free" | "paid";
  maxTokensOut?: number;
};

type Completion = {
  choices: { message: { content: string | null } }[];
  usage?: { prompt_tokens: number; completion_tokens: number };
};

export async function guard(
  call: (model: Model) => Promise<Completion>,
  opts: GuardOpts
): Promise<Completion> {
  if (await isKilled()) {
    console.warn(JSON.stringify({ evt: "kill_switch", user: opts.user }));
    throw new Response("AI features temporarily unavailable", { status: 503 });
  }

  const tier = opts.tier ?? (opts.user.startsWith("anon-") ? "anon" : "free");
  const budget = await checkBudget(opts.user, tier);
  if (!budget.ok) {
    console.warn(JSON.stringify({ evt: "over_budget", user: opts.user, tier }));
    throw new Response("Daily AI budget reached", {
      status: 429,
      headers: { "retry-after": String(budget.retryAfter) },
    });
  }

  const model = pickModel(opts.task, opts.quality ?? "auto");
  const r = await call(model);
  const u = r.usage ?? { prompt_tokens: 0, completion_tokens: 0 };
  const cost = await meter(opts.user, model, u.prompt_tokens, u.completion_tokens);
  console.info(JSON.stringify({
    evt: "llm_call", user: opts.user, model, task: opts.task,
    in: u.prompt_tokens, out: u.completion_tokens, microcents: cost,
  }));
  return r;
}
```

## Test it

```bash
# 1. normal call
curl -X POST http://localhost:3000/api/summarize \
  -H 'content-type: application/json' -d '{"text":"hi"}'

# 2. trips per-user cap for an anonymous IP ($0.10)
for i in {1..200}; do
  curl -sX POST http://localhost:3000/api/summarize \
    -H 'content-type: application/json' -d '{"text":"hi"}' > /dev/null
done
# the last ones return 429 with retry-after

# 3. trips global kill switch
LLM_KILL_SWITCH=true npm run dev
curl -X POST http://localhost:3000/api/summarize ... # -> 503
```

## What you'll see in logs

```
{"evt":"llm_call","user":"user_42","model":"gpt-4o-mini","task":"summarize","in":120,"out":85,"microcents":687}
{"evt":"llm_call","user":"user_42","model":"gpt-4o-mini","task":"summarize","in":140,"out":100,"microcents":810}
...
{"evt":"over_budget","user":"user_42","tier":"free"}
...
{"evt":"kill_switch","user":"user_99"}
```

That's the whole loop: every call is priced, every user has a rolling 24h ceiling, and a single env var turns the feature off if anything goes wrong.
