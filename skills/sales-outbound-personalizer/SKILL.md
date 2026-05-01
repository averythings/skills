---
name: sales-outbound-personalizer
description: Personalize cold emails using LinkedIn/browser research + CRM context
license: MIT
metadata:
  author: averythings
  version: "0.1.0"
  repository: https://github.com/averythings/skills
---

# Sales Outbound Personalizer

Agent skill that drafts **founder-to-founder cold outbound emails** personalized from LinkedIn / browser research and (optional) CRM context.

Primary scenario: **inviting startups to a startup fair**, offering them a **demo table** where builders from that startup get in front of **200–600 people they actually want to meet**.

The skill's job is to turn raw research notes + an optional CRM dump into a short, specific, non-generic email with a single CTA — no flattery, no AI tells, no overclaiming.

---

## When to use

Invoke this skill when the user says any of:

- "Write a cold email to <founder> at <startup>"
- "Invite these founders to the fair"
- "Personalize these outbound emails from LinkedIn"
- "Draft a demo-table pitch for <company>"
- "Here's my CRM notes on this prospect — write the email"
- "Batch of 10 founders, fair invite, personalize each"

Do **not** invoke for generic marketing copy, newsletter blasts, warm intros (a warm intro doesn't need hooks — just context), or inbound responses.

---

## Inputs

Accept as much as the user provides. If something critical is missing, ask once — then proceed.

**Prospect**
1. Prospect name
2. Role / title
3. Company
4. Company URL
5. LinkedIn URL (profile)
6. Company LinkedIn URL (optional)
7. Recent signals — raw notes, links, or paste of posts/articles (optional but strongly preferred)

**CRM context (optional, structured paste)**
8. Prior touches (how many, which channels)
9. Stage (cold / replied / meeting / stalled)
10. Owner notes (free text from the rep)
11. Warm intro path if any (who knows whom)
12. Last contact date

**Event**
13. Event name (e.g. "The Startup Fair")
14. Date
15. City / venue
16. Expected audience size range — default **200–600**
17. Audience composition — who is in the room (e.g. "operators, angels, PMs at YC-stage startups, a few partners from seed funds")

**Sender**
18. Sender name
19. Sender company / role
20. Signature preference (minimal vs. full)

If the user pastes a LinkedIn URL but no extracted content, run the **Research checklist** below before drafting.

---

## Research checklist

Before writing a word of email, the agent runs this in order. Every output hook must map back to one of these sources.

1. **LinkedIn profile**
   - Current headline + current role tenure
   - Most recent 3 posts / reposts (topic, stance, any specific claim)
   - Prior companies — especially any that map to the sender's network or the event audience
   - Education / unusual background (only if it's load-bearing — no "went to Stanford" filler)

2. **Company site / product**
   - One-line of what they actually ship (not the homepage tagline — the *product*)
   - Most recent release, changelog entry, or launch post
   - Pricing or customer tier (self-serve? enterprise? dev tool? consumer?)
   - Who they sell to — and whether that audience overlaps with the fair's 200–600

3. **Recent news / signals**
   - Funding announcement in the last 6 months
   - Hiring signals (open roles — especially GTM / founding engineer)
   - Press, podcast, or Substack appearance by the founder
   - Shipped feature or major update in the last 30 days

4. **Shared context**
   - Mutual connections (especially investors or ex-colleagues the sender actually knows)
   - Prior CRM touches — don't re-pitch what's already been said
   - Warm intro path if one exists — if yes, the email should probably be routed through that person, not cold

**Output of research phase:** a short bullet list of **5–8 candidate hooks**, each with a source link. The agent then picks **1–2** for the email. Everything else is discarded.

---

## Drafting rules

Hard rules. Do not violate.

1. **Two hooks, maximum.** One is usually enough. Three is a LinkedIn stalker energy.
2. **Every hook must be verifiable.** If the agent can't point at a URL or a CRM line, the hook is cut.
3. **State the offer in one sentence:** a demo table at the fair, in front of 200–600 people that match the audience composition.
4. **Frame the audience around them, not us.** Not "we'll have 500 people." But: "the room is ~400 operators and seed-stage founders — the people *you're already trying to reach on LinkedIn*."
5. **One CTA.** Usually: "worth a 10-min call this week?" or "want me to hold a table?" Never two asks.
6. **Founder-to-founder register.** Lowercase subject is fine. Contractions. No "I hope this finds you well." No "I came across your profile." No "I was impressed by..."
7. **Length: 60–110 words in the body.** If it's longer, cut.
8. **No AI tells.** No em-dash essays, no "In today's fast-paced world," no "I'd love to," no tricolons ("streamline, scale, and succeed"), no "delve," no "leverage."
9. **No overclaiming about the event.** If the audience size is a range, say the range. If composition is uncertain, hedge honestly.
10. **Subject line variants: 3–5.** Short (≤ 6 words), lowercase-ok, no clickbait, no emoji, no brackets like `[invite]`.
11. **Follow-up is 1 line.** Sent 4–7 days later if no reply. No new pitch — just a ping that references the first email.

---

## Output format

The skill returns one block, in this order:

```
SUBJECT LINES
1. <variant>
2. <variant>
3. <variant>
4. <variant>  (optional)
5. <variant>  (optional)

EMAIL BODY
<60–110 words, plain text, no markdown>

FOLLOW-UP (send 4–7 days later if no reply)
<1 line>

HOOKS USED
- <hook 1> — source: <URL or CRM note>
- <hook 2> — source: <URL or CRM note>  (if two)

OMITTED (candidates that failed verification)
- <candidate hook> — reason
```

The `HOOKS USED` and `OMITTED` sections are shown to the sender, not included in the email. They exist so the sender can sanity-check before hitting send.

---

## Worked example

**Inputs (pasted by user)**

```
Prospect: Maya Chen, co-founder & CTO, Lattice Forge (lattice-forge.dev)
LinkedIn: linkedin.com/in/mayachen-lf
Recent signals:
  - Posted on LinkedIn 6 days ago: "we just shipped a local-first sync
    engine for CAD teams, early numbers: 40ms p99 on 10k-element models"
  - Hiring: "Founding GTM" role opened 3 weeks ago
  - Raised $4M seed from Boldstart in Feb
CRM:
  - 0 prior touches
  - Stage: cold
  - No warm intro
  - Last contact date: n/a
Event: The Startup Fair, June 14, SF (SoMa), ~400 attendees,
  mostly seed-to-Series-A founders, operators, and 20 angel/seed
  investors. Audience skew is technical/product people.
Sender: Avery Liu, organizer, averyliu.co
```

**Research notes → candidate hooks (5–8)**

- H1: Shipped local-first sync for CAD teams — LinkedIn post, 6 days ago (source verified)
- H2: Hiring "Founding GTM" (source: company careers page, verified)
- H3: Raised $4M seed from Boldstart (source: Boldstart portfolio page, verified)
- H4: Lattice Forge sells to hardware / CAD teams — audience composition at the fair skews software; only partial overlap (noted, might cut)
- H5: Maya previously shipped collab primitives at Figma (source: LinkedIn, verified)
- H6: GTM hire is unfilled → she probably *is* doing GTM right now (inference, useful framing)

**Chosen hooks**

- H1 (specific, recent, technical, verifiable) — the opener
- H6 (relevant to the offer — if she's doing GTM herself, a room of buyers is valuable) — the bridge

**Subject lines (3–5)**

```
1. 400 founders, june 14, sf
2. demo table for lattice forge?
3. saw the 40ms p99 post
4. sf, june 14 — want a table?
5. fair invite for lattice forge
```

**Email body**

```
Hey Maya —

Saw the local-first sync post last week. The 40ms p99 on a 10k-element
model is the specific number I was hoping someone would publish, so
thanks for that.

Running a startup fair in SF on June 14 — about 400 people in the room,
mostly seed-to-Series-A founders, operators, and ~20 seed investors.
Skew is technical, which maps to the people you'd probably want demoing
Lattice Forge to anyway.

Would you want a demo table? I noticed the Founding GTM role is still
open, and a full afternoon of warm conversations might be a decent
shortcut while you're still the one doing GTM.

Happy to hold one if it's a yes.

Avery
```

**Follow-up (send 4–7 days later if no reply)**

```
Bumping this — holding a table through Friday if you want it.
```

**Hooks used**
- Local-first sync post (40ms p99) — source: linkedin.com/in/mayachen-lf (post 2026-04-22)
- Founding GTM hire open → sender reasonably infers she's doing GTM — source: lattice-forge.dev/careers

**Omitted**
- $4M seed from Boldstart — true but generic ("congrats on the raise" is an AI tell)
- Ex-Figma — relevant but the email is already tight; cut
- CAD/hardware vs. software audience mismatch — honest hedge would dilute the pitch; either address head-on in a reply or cut

---

## Reusable prompt template

Copy-paste the block below, fill in the `<...>` fields, and run the skill.

```
You are the sales-outbound-personalizer skill. Draft a cold outbound
email inviting this startup to a startup fair with a demo table offer.

PROSPECT
- name: <prospect name>
- role: <role>
- company: <company>
- company URL: <url>
- LinkedIn: <url>
- recent signals: <paste posts, launches, hires, funding, etc.>

CRM (optional)
- prior touches: <n or none>
- stage: <cold / replied / meeting / stalled>
- owner notes: <free text>
- warm intro path: <name, or none>
- last contact: <date or n/a>

EVENT
- name: <event name>
- date: <date>
- city / venue: <location>
- audience size: <e.g. 200–600>
- audience composition: <who is in the room>

SENDER
- name: <your name>
- company/role: <your company or role>
- signature preference: <minimal | full>

RULES
- Follow every rule in SKILL.md "Drafting rules".
- Do not fabricate facts. If a hook can't be verified against the
  inputs above, omit it and list it under OMITTED.
- Return the exact output format specified in SKILL.md "Output format":
  subject lines, email body, follow-up, hooks used, omitted.
- Body length: 60–110 words.
- One CTA. Founder-to-founder register. No AI tells.
- Frame the audience around what *they* want, not what we have.
```

---

## Guardrails

- **Never fabricate** a post, number, funding round, customer, mutual connection, or quote. If the only source is "probably true," cut the hook.
- **If a hook cannot be verified** from the inputs provided (pasted notes, CRM, a URL the agent can read), it goes under `OMITTED` with a reason. Do not silently drop it.
- **No fake urgency.** Don't invent "we only have 2 tables left" unless the user explicitly tells you that's true.
- **No name-dropping other attendees** unless the user confirms those attendees are confirmed and OK to be named.
- **Respect CRM state.** If CRM says "stage: stalled, owner notes: asked us to stop emailing" — refuse to draft and tell the user why.
- **If a warm intro path exists**, recommend routing through that person instead of cold. The warm version is almost always higher ROI than a better-personalized cold email.
- **Audience framing is honest.** If the fair is 200–600, say 200–600 (or the specific expected number if known). Don't round up. Don't quote "500+" if the range is 200–600.
- **One email, one CTA.** If the user asks for two CTAs in one email, push back once, then comply if they insist.

---

## Non-goals

- Not a CRM. Does not store prospects, does not track sends, does not schedule.
- Not a scraper. Does not log into LinkedIn. The user pastes research notes or URLs the agent can read.
- Not a deliverability tool. Does not warm up inboxes, does not check SPF/DKIM.
- Not a generic copywriter. This skill is scoped to **cold outbound for event invites with a demo-table offer**. For newsletter, lifecycle, or product copy — use a different skill.
