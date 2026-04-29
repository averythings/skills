# UI Spec — llm-cost-guardrail

> Design anchor: **Linear** — `averythings/awesome-design-md/design-md/linear.app/DESIGN.md`
> Why Linear: the skill generates an `/admin/cost` endpoint; the natural home for that data is a founder-facing dashboard. Linear's dark, dense, monochrome system is built for data-heavy dev-tools surfaces with just enough accent color to flag what matters. Indigo for the kill switch, success green for "spend is healthy," everything else achromatic.

This document specifies a single page — `/admin/cost` — that visualizes the JSON produced by the skill's admin route. It is a **design spec**, not an implementation: the skill itself only produces JSON + markdown.

---

## 1. Surface

Dense single-page dashboard.

- Background: `#0f1011` (Linear "Panel Dark") — one step up from marketing black, conveys "utility surface"
- Content grid: 1200px max width, 24px gutter
- Typography: Inter Variable (`cv01`, `ss03`), Berkeley Mono for numbers
- Vertical rhythm: 24px between blocks, 12px within a card

---

## 2. Page regions

```
┌─────────────────────────────────────────────────────────────────┐
│  COST GUARDRAIL · patchnote        [ kill switch: OFF ] [ × ]   │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  $ 12.47   today      $ 6.02    24h avg     $ 50.00   daily cap │
│  ─────────                                                      │
│                                                                 │
│  ████████████░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░   25% of cap      │
│                                                                 │
├─────────────────────────────────────────────────────────────────┤
│  TOP USERS TODAY                                                │
│  ┌──────────────────────┬────────┬────────┬────────┬─────────┐  │
│  │ user                 │  calls │ tokens │   $    │  model  │  │
│  ├──────────────────────┼────────┼────────┼────────┼─────────┤  │
│  │ anon-203.0.113.42   │   142  │ 18.4k  │ $0.09  │ mini    │  │
│  │ user_42             │    88  │ 12.1k  │ $0.06  │ mini    │  │
│  │ user_17 ⚠ over cap  │   312  │ 47.9k  │ $0.50  │ mini    │  │
│  └──────────────────────┴────────┴────────┴────────┴─────────┘  │
├─────────────────────────────────────────────────────────────────┤
│  BY ENDPOINT                 │  BY MODEL                        │
│  ┌──────────────────┬──────┐ │  ┌────────────────┬───────────┐  │
│  │ /api/summarize   │ 9.10 │ │  │ gpt-4o-mini    │   $10.20  │  │
│  │ /api/classify    │ 2.61 │ │  │ claude-haiku   │    $1.87  │  │
│  │ /api/plan        │ 0.76 │ │  │ gpt-4o (esc.)  │    $0.40  │  │
│  └──────────────────┴──────┘ │  └────────────────┴───────────┘  │
├─────────────────────────────────────────────────────────────────┤
│  REFUSED IN LAST HOUR                                           │
│  14:02  over_budget  user_17  free tier  $0.50 > cap            │
│  13:47  over_budget  anon     $0.10 > cap                       │
│  13:32  kill_switch  user_99  global cap reached ($50.00)       │
└─────────────────────────────────────────────────────────────────┘
```

---

## 3. Component detail

### 3.1 Header bar

- Label "COST GUARDRAIL · `<project>`" — Inter 510, 11px uppercase, letter-spacing `0.12em`, `#62666d`
- **Kill-switch toggle**: Linear **Pill Button** spec
  - `OFF` state: border `rgb(35,37,42)`, text `#d0d6e0`, radius `9999px`, 12px 510
  - `ON` state: background `#5e6ad2`, text `#ffffff`, same shape
  - Toggling fires `POST /admin/cost/kill` — no confirm dialog; the mistake is cheap to undo
- Close/detach icon: Linear **Icon Button (Circle)** spec

### 3.2 Hero numbers row

Three numbers: today / 24h avg / cap.

- Number: Berkeley Mono 48px, weight 400, `#f7f8f8`
- Label under: Inter 510, 11px uppercase, `#62666d`
- Divider: 1px, `rgba(255,255,255,0.05)` under the numbers

### 3.3 Budget bar

- Track: `rgba(255,255,255,0.05)`, 6px tall, radius 3px
- Fill at <50%: `#10b981` (success green — the only saturated green on the page)
- Fill 50–85%: `#d0d6e0` (muted silver — deliberately *not* yellow)
- Fill 85–100%: `#5e6ad2` (brand indigo — "attention" without panic)
- Fill at 100%: indigo fills, small `×` glyph at end indicating kill-switch auto-trip

### 3.4 Top users table

- Linear **Card** spec as container: `rgba(255,255,255,0.02)` bg, `1px solid rgba(255,255,255,0.08)`, radius 8px
- Header row: Inter 510, 12px, `#8a8f98`, uppercase
- Body rows: Inter 400, 14px, `#d0d6e0`; numbers in Berkeley Mono 14px
- "over cap" tag: Linear **Subtle Badge** (`rgba(255,255,255,0.05)`, 10px 510, `⚠` glyph in `#5e6ad2`)
- Row hover: background shifts to `rgba(255,255,255,0.04)`
- Click user → detail drawer with their last 20 calls

### 3.5 Refusals log

- Each row monospaced: `Berkeley Mono 13px`
- Timestamp: `#62666d`
- Event type: `over_budget` in `#d0d6e0`, `kill_switch` in `#5e6ad2`
- Reason: `#8a8f98`

---

## 4. Component tokens (from Linear DESIGN.md)

| Token | Value | Usage |
|---|---|---|
| Panel Dark `#0f1011` | page bg | — |
| Level 3 `#191a1b` | card internal sections | — |
| Primary Text `#f7f8f8` | hero numbers, headings |
| Secondary Text `#d0d6e0` | table rows |
| Tertiary `#8a8f98` | meta, labels |
| Quaternary `#62666d` | timestamps, muted labels |
| Brand Indigo `#5e6ad2` | kill-switch active, attention states |
| Success `#10b981` | healthy-spend bar fill |
| Subtle border `rgba(255,255,255,0.05)` | internal dividers |
| Standard border `rgba(255,255,255,0.08)` | card outlines |
| Radius 8px | cards |
| Radius 9999px | kill-switch pill |
| Inter 510 | UI labels & table headers |
| Berkeley Mono | every number on the page |

---

## 5. States

### Healthy (the default)
Green fill on budget bar, no red/indigo accents except the dormant kill switch. Calm.

### Approaching cap (> 85% of daily)
Budget bar fills with indigo. A single row at the top reads, in Inter 510 15px `#f7f8f8`:
> "Daily cap in sight. Consider raising it, tightening per-user caps, or routing more traffic to cheaper models."

No modal. No color overload. One sentence, one line.

### Kill switch ON (manually or auto-tripped)
- Kill-switch pill is filled indigo.
- A thin indigo strip (`#5e6ad2`, 2px) appears at the very top of the page — the only intrusive element on the whole surface.
- Below the budget bar: "AI features are OFF. Users get 503 responses."
- A button **[ Turn back on ]** — Linear **Primary Brand Button** spec, the only filled button on the page.

---

## 6. Non-goals for this UI

- No charts over time. Minute/hour sparkline tempting, but the skill doesn't persist history beyond 48h (per `meter.ts`), and the chart would be dishonest over short windows.
- No forecasting. "You'll hit cap in 3 hours" would require trend modeling the skill doesn't do. Add later.
- No user management. The dashboard is read-only except for the kill switch.
- No dark/light toggle. This is a dark-only surface. Indie founders keep their terminals dark; this page lives in the same world.

---

## 7. Why this design matches the skill

The skill's stance is **boring on purpose** — circuit breakers, rolling windows, env-var kill switches. The UI mirrors that: one scroll, mostly monochrome, one accent color, one moment of green when you're safe. The design doesn't try to make cost management fun. It tries to make it unmissable and unscary, which is exactly what a solo founder needs.
