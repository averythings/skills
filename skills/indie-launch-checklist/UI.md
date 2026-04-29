# UI Spec — indie-launch-checklist

> Design anchor: **Linear** — `averythings/awesome-design-md/design-md/linear.app/DESIGN.md`
> Why Linear: dark-mode-native, monochrome + single indigo accent, ultra-thin borders. Matches the "calm, engineered, not marketing" tone this skill needs 30 minutes before a launch. A panicked founder does not want a rainbow of status lights.

This is a **design spec**, not an implementation. The agent skill produces `LAUNCH_READINESS.md` as markdown; this document describes how an optional web wrapper would render it so the output "ships with great UI."

---

## 1. Surface

Single page, single column, **no chrome**: no sidebar, no nav bar. The report *is* the UI.

- Background: `#08090a` (Linear "marketing black")
- Max content width: `720px`, centered
- Vertical rhythm: 32px between sections, 12px within a section
- Font: Inter Variable with `"cv01", "ss03"` on `body`
- Mono: Berkeley Mono (code blocks, commands)

---

## 2. Key screens

### 2.1 Header — status glance

```
┌─────────────────────────────────────────────────────────┐
│                                                         │
│   ●  launch readiness · patchnote.app                   │
│                                                         │
│   15  ⚠ 5  ✕ 4                                          │
│   ────────────                                          │
│   RED — 4 blockers, push the launch a day               │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

- Status dot color:
  - **GREEN** → `#10b981`
  - **YELLOW** → `#8a8f98` (muted — intentional; yellow-orange would scream)
  - **RED** → `#5e6ad2` (brand indigo, **not** warning red — see §5 tone)
- "launch readiness" in Inter 510, 13px, uppercase, letter-spacing `0.08em`, color `#8a8f98`
- Project name: Inter 510, 32px, `#f7f8f8`, letter-spacing `-0.704px`
- Counts: Berkeley Mono 48px, weight 400. Each count in its own cluster, 24px gap.
- The RED/YELLOW/GREEN verdict line: Inter 590, 17px, color = matching status color
- Advice sub-line: Inter 400, 15px, `#8a8f98`

### 2.2 Fix-Now card (one per blocker)

```
┌─────────────────────────────────────────────────────────┐
│  ✕  .env.production committed to git                    │
│                                                         │
│  Public if the repo is ever made public. Rotate every   │
│  key and scrub the history.                             │
│                                                         │
│  ┌───────────────────────────────────────────────────┐  │
│  │ git filter-repo --path .env.production \          │  │
│  │   --invert-paths                                  │  │
│  └───────────────────────────────────────────────────┘  │
│                                                         │
│  [ Mark fixed ]   [ Snooze ]                            │
└─────────────────────────────────────────────────────────┘
```

- Card container: `rgba(255,255,255,0.02)` bg, `1px solid rgba(255,255,255,0.08)` border, 8px radius
- `✕` icon: 14px, color `#5e6ad2`
- Title: Inter 510, 17px, `#f7f8f8`, letter-spacing `-0.165px`
- Body: Inter 400, 15px, `#d0d6e0`, line-height 1.6
- Command block: Berkeley Mono 14px, `#f7f8f8` on `rgba(255,255,255,0.03)`, 6px radius, 12px padding, one-click copy on hover
- Buttons: Linear **Subtle Button** spec (`rgba(255,255,255,0.04)` bg, 6px radius, 12px 510 text). "Mark fixed" is primary, which means *not* filled — it means slightly higher opacity background on hover.

### 2.3 Good-to-have list (for yellows)

Flat list, no card per item. Each row:

```
  ⚠  File-upload endpoint has no size cap.      add maxBodySize: "5mb"   →
  ⚠  Analytics upgrade-click event unverified.  confirm in Plausible      →
  ⚠  500 page is the Next.js default.           10 minutes on-brand       →
```

- Row height: 40px, border-bottom `rgba(255,255,255,0.05)`
- Left text: Inter 400, 15px, `#d0d6e0`
- Right hint: Inter 400, 13px, `#8a8f98`, Berkeley Mono if it's a code-ish fragment
- `⚠` glyph: 12px, `#8a8f98` (deliberately muted — yellow is not an emergency)
- Arrow on the right appears on hover only; clicking expands an inline detail drawer

### 2.4 Rollback plan — forced narrative box

```
┌─────────────────────────────────────────────────────────┐
│  ROLLBACK PLAN                                          │
│                                                         │
│  Previous stable deploy: patchnote-git-main-xyz...      │
│  Commit: ab12cd3                                        │
│                                                         │
│  $ vercel rollback patchnote-git-main-xyz.vercel.app   │
│                                                         │
│  Migration 20260427_add_teams.sql is additive.          │
│  Safe to roll back app without DB.                      │
└─────────────────────────────────────────────────────────┘
```

- Section label: Inter 510, 11px uppercase, letter-spacing `0.12em`, `#62666d`
- Content prose: Inter 400, 15px, `#d0d6e0`, line-height 1.6
- Command: Berkeley Mono, treated as block code

### 2.5 Traffic-Spike Posture — 2×2 grid

Four tiles: First bottleneck / Threshold / Tell / Lever. Each tile:

- `rgba(255,255,255,0.02)` bg, `1px solid rgba(255,255,255,0.05)` border, 8px radius, 16px padding
- Label: Inter 510 11px uppercase, `#62666d`
- Value: Inter 510 15px, `#f7f8f8`

---

## 3. Component choices from Linear DESIGN.md

| Element | Linear token used |
|---|---|
| Page background | Marketing Black `#08090a` |
| Primary text | `#f7f8f8` |
| Secondary text | `#d0d6e0` |
| Muted text / labels | `#8a8f98`, `#62666d` |
| Card surface | `rgba(255,255,255,0.02)` |
| Default border | `rgba(255,255,255,0.08)` |
| Subtle divider | `rgba(255,255,255,0.05)` |
| Accent (status RED verdict, ✕ icons) | `#5e6ad2` |
| Success dot (GREEN) | `#10b981` |
| Body font | Inter Variable @ weight 400 / 510 |
| Display | Inter Variable @ 510, letter-spacing per Linear display scale |
| Mono | Berkeley Mono (fallback: ui-monospace, SF Mono) |
| Card radius | 8px |
| Button radius | 6px |

---

## 4. Interaction notes

- **One action at a time**: only one card is "active" at any moment. Clicking "Mark fixed" collapses the card into a one-line confirmation; clicking "Snooze" hides it for this session only (no persistent state on a 30-min ritual).
- **Keyboard**: `j`/`k` to move between cards, `f` to mark fixed, `?` for help. Linear-like.
- **No notifications, no toasts, no modals.** The whole surface is the report.
- **Copy-on-hover** for any command block. A single flash of `#5e6ad2` at 0.3 opacity confirms copy.

---

## 5. Tone

The whole point of this UI is to **not cause panic**. The skill is already blunt in text. The visual design plays the opposite role: calm, monochrome, minimal. Red is indigo. Yellow is gray. Green is the only saturated color — and only for the checks that passed. The UI rewards a passing launch with the one moment of color on the page.

This is the opposite of a typical dashboard (green = success, red alarms everywhere). For a founder mid-launch, that pattern is noise. Linear's near-achromatic system is the right frame.

---

## 6. What this is not

- Not a persistent dashboard. No login. No history. Session-only.
- Not a CI status page. The skill runs on demand, and the UI is the report viewer for that one run.
- Not responsive-first. This is a desktop-at-the-kitchen-table tool; mobile is a graceful degradation (single column already, just reduce display size to 24px).
