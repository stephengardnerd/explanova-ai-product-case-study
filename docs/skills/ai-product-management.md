# 🎯 AI Product Management — Skills Exercised

← [Back to README](../../README.md)

---

Building an AI product is product management first and engineering second. The engineering fails if the product decisions don't hold up. Here's what the AI PM surface looked like on Explanova.

## Phased roadmap — plan.md per phase

Every phase of the build had a written `plan.md` *before* execution:

| Phase | Scope |
|---|---|
| 00 — Content pipeline | Corpus sourcing, normalization, ingestion discipline |
| 01 — GitHub setup | Repo structure, branch model, release policy |
| 02 — Stitch | UI-first ideation, nine-screen journey, subscription tiering |
| 03 — AI Studio | 5-task pipeline methodology, retrieval-first principle, benchmark criteria |
| 04 — Firebase Studio | Platform selection rationale |
| 05 — Firebase | Production hosting, functions, rules, auth providers |
| 06 — HeyGen | Face-shot avatar generation |
| 07 — Seedance | Hands-on-whiteboard avatar generation |

Writing the plan forces you to discover the decisions. "Decide the retrieval contract before writing the retrieval code" is the single highest-leverage product-management habit I exercised.

## Retrieval-first as a product principle

The single biggest product decision: **the AI never answers from pure generation alone.**

This is a product decision disguised as an architectural one. It shapes:

- The trust posture the parent experiences (grounded badges, epistemic hedges)
- The benchmark criteria (grounding quality is weighted in success scoring)
- The corpus investment (retrieval is only as good as what's behind it, so the corpus had to come first)
- The failure UX (when retrieval can't ground, the frontend tells the parent — rather than fake confidence)

Product decisions that are also architectural decisions are the most expensive ones to get wrong. This one I got right.

## Provider strategy — Gemini + Claude + Perplexity

- **Gemini 3 (primary)** — temperature 0.2 for tasks 1–4, 0.4 for task 5 (the warmth task)
- **Claude Sonnet (failover)** — defined trigger conditions (timeout, low confidence, out-of-curriculum classification)
- **Perplexity (external grounding)** — bounded to current-events / NASA-style queries with mandatory citations

Triage rules are *explicit*, not emergent. If a provider fails, the fallback is deterministic rather than a pager alert.

## Subscription tiering + community pricing

Pricing was designed in the Stitch ideation phase, before any payment integration:

- **Four-SKU tier model:** `free | plus | family_premium | premium_visual`
- **Stripe SDK pinned to API version `2025-03-31.basil`** — explicit version pinning, scripted migrations (e.g., `family` → `crew` rename executed across Firestore *and* Stripe subscription metadata in lockstep)
- **Webhook handler** with App Check exemption + Stripe webhook-secret signature validation
- **Community-pricing posture** for military families, teachers, and low-income households — baked into the pricing UX from day one, not bolted on later

That posture exists because I'm a Navy veteran and the product is about access. Pricing is a values statement.

→ Full business architecture: [docs/06-business-and-pricing.md](../06-business-and-pricing.md)

## Cost discipline — AI economics as a PM responsibility

Multi-model routing isn't just engineering — it's PM:

| Decision | Why it's a PM call |
|---|---|
| Gemini 3 Pro Preview only on the user-facing call | Spend reasoning tokens where the user notices |
| Gemini 3 Flash Preview for ingestion | 10× cheaper for high-volume, low-stakes work |
| Per-community summary cache | 40× cost reduction on warm calls |
| Daily TTS quota tracking in `tts_quota` Firestore | Detect runaway usage before the bill arrives |
| 32K thinking budget on Gemini 3 Pro | Explicit budget — not "let it think as long as it wants" |
| Family-account model (up to 6 managers per family) | Bet on household LTV beating per-seat ARPU for child-education |

Cost-per-completed-lesson is the unit-economics number that makes or breaks the business. Every model choice, cache layer, and quota gate above is a PM decision dressed as an engineering decision.

## Email + deliverability — the boring infra that makes onboarding work

- SendGrid with custom sending subdomain (`em1318.explanova.ai`)
- Cloudflare DNS with explicit SPF, DKIM, DMARC records
- Per-recipient-domain QA — caught Apple/iCloud rejection (missing `sendgrid.net` in SPF) before it became a churn problem
- API key managed via Firebase Secret Manager with rotation discipline (key-prefix gotcha now part of the rotation checklist)

Invite emails that land in spam are an onboarding failure dressed as a user-acquisition cost. Boring infra; high product impact.

## Child-safety + privacy-consent UX

The product serves children. The Stitch journey included a privacy-consent surface from the start, not retrofitted for App Store review. Child-appropriate content filtering runs on every retrieval layer. Parent controls sit at the top of the dashboard, not buried in settings.

## Grade-band strategy — K–2 Early Learner as a first-class band

Most education products treat K–2 as an afterthought. Explanova added **K–2 Early Learner as a first-class grade band end-to-end** — classification taxonomy, corpus coverage (~175 entries), visual primitives (ten-frames, counter grids), and avatar-script vocabulary all adapt. That was a product call, not an engineering one.

## Visual-learner initiative — P1 → P5 roadmap

Shipped P1 (six K–5 whiteboard primitives with entrance animations) under versioned milestones (`v3.3.0`, `v3.3.1`, `v3.3.3`) with live K–2 smoke-test feedback loops between each release. Planned P2–P5 extend the primitive vocabulary through middle school, algebra, calculus, and engineering-level diagrams.

Phased shipping is the antidote to big-bang product failure. Each P-release proves a hypothesis before the next one starts.

## Encourage Mode — tone as a first-class feature

Explanova has an **Encourage Mode** flag that propagates through all 5 AI tasks — extra warmth, warm-up questions, confidence-building framing. Tone isn't a prompt tweak; it's a product capability with a flag, a name, and documented behavior across the whole pipeline.

## Version-stamped milestones

Every product-visible change is tagged (`v1.0.0` → `v3.5.4`, 16 releases). Every tag has a dated changelog entry that names the user-facing capability it shipped. A roadmap you can't point recruiters at is not a roadmap.

## Family-account model

Up to 6 account managers per family — parents, grandparents, babysitters, tutors — can share the dashboard. That's a product insight (the real unit of learning is the family, not the user) turned into a data-model decision.

## What this rolls up to

Retrieval-first principle + written phase plans + provider triage rules + values-driven pricing + child-safety posture + grade-band strategy + phased visual-learner shipping + encourage-mode feature + family-account model + version-stamped milestones. Each one is a **product decision** that shaped the **engineering decisions** downstream — not the other way around.

→ Back to [README](../../README.md)
