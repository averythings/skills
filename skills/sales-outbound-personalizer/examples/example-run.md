# Example Run — sales-outbound-personalizer

> A full example of the skill being invoked to draft a cold email inviting a fictional seed-stage startup (**Lattice Forge**) to a startup fair with a demo table offer.

---

## User invocation

```
> Draft a cold outbound email inviting Lattice Forge to The Startup Fair.
> Offer them a demo table. Audience is ~400 founders/operators/seed
> investors in SF on June 14.
>
> Prospect: Maya Chen, co-founder & CTO, Lattice Forge (lattice-forge.dev)
> LinkedIn: linkedin.com/in/mayachen-lf
> Signals:
>   - LinkedIn post 6 days ago: "shipped a local-first sync engine for
>     CAD teams — 40ms p99 on 10k-element models"
>   - Hiring "Founding GTM" (opened 3 weeks ago)
>   - Raised $4M seed from Boldstart in Feb
> CRM: 0 prior touches, stage cold, no warm intro
> Sender: Avery Liu, organizer, averyliu.co
```

---

## Skill output (full return block)

```
RESEARCH NOTES (5–8 candidate hooks)
- H1 Shipped local-first sync for CAD teams, 40ms p99 on 10k-element
  models (source: LinkedIn post, 2026-04-22) — verified
- H2 Hiring "Founding GTM" (source: lattice-forge.dev/careers) — verified
- H3 $4M seed from Boldstart, Feb (source: Boldstart portfolio) — verified
- H4 Lattice Forge sells into CAD/hardware; fair audience skews software;
  partial overlap — note, may cut
- H5 Maya ex-Figma, shipped collab primitives (source: LinkedIn) — verified
- H6 "Founding GTM" still unfilled ⇒ Maya probably doing GTM herself —
  inference from H2, useful framing for the offer

CHOSEN HOOKS
- H1 — specific, recent, technical, verifiable (opener)
- H6 — bridges directly into the demo-table offer

SUBJECT LINES
1. 400 founders, june 14, sf
2. demo table for lattice forge?
3. saw the 40ms p99 post
4. sf, june 14 — want a table?
5. fair invite for lattice forge

EMAIL BODY
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

FOLLOW-UP (send 4–7 days later if no reply)
Bumping this — holding a table through Friday if you want it.

HOOKS USED
- Local-first sync post, 40ms p99 on 10k elements —
  source: linkedin.com/in/mayachen-lf (post 2026-04-22)
- Founding GTM role still open → inferred she's doing GTM —
  source: lattice-forge.dev/careers

OMITTED
- $4M seed from Boldstart — true but "congrats on the raise" reads
  generic / AI-tell
- Ex-Figma background — relevant but the email is already tight
- CAD/hardware vs software audience mismatch — would dilute the
  pitch to hedge; address in a reply if it comes up
```

---

## Why the email is the way it is

- **Two hooks, max.** H1 earns attention. H6 bridges to the offer. Everything else was cut, and the cuts are shown so the sender can sanity-check the decision.
- **Audience framing is around her, not us.** "the people you'd probably want demoing Lattice Forge to anyway" — not "we'll have 500 attendees."
- **One CTA.** "Would you want a demo table?" Not two asks, not a calendar link dump.
- **No AI tells.** No "I hope this finds you well," no "I was impressed," no tricolons, no em-dash essays.
- **Honest audience number.** "about 400" — not "500+".
- **Follow-up is 1 line.** References the first email, no new pitch.

---

## What a *bad* version of this email would look like (and why we don't ship it)

```
Hi Maya!

I hope this email finds you well. I came across your LinkedIn profile
and was really impressed by your journey from Figma to co-founding
Lattice Forge. Congratulations on your recent $4M seed from Boldstart!

I wanted to reach out about an exciting opportunity. We're hosting The
Startup Fair in San Francisco and we'd love to have Lattice Forge join
us! We'll have 500+ attendees including top founders, operators, and
investors — the perfect audience to help you scale, streamline, and
grow.

Would you be open to a quick 15-min call to discuss? I'd also love to
send over our deck. Let me know what works!

Best regards,
Avery
```

Every line of that is a violation: "I hope this finds you well," "came across your profile," "was really impressed," congrats-on-the-raise flattery, "500+" when the real number is ~400, "scale, streamline, and grow" tricolon, two CTAs, em-dash essay vibes, signs off with "Best regards."

The skill exists to not ship that.

---

## Why this is the whole example

One input block. One return block. The user reviews `HOOKS USED` + `OMITTED`, picks a subject, and sends. The skill is deliberately boring: the research is surfaced, the email is short, and every choice is explainable.
