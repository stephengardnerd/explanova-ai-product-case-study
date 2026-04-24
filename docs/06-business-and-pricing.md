# 06 — Business, Pricing, and Cost Discipline

> Stripe billing, SendGrid deliverability, tiered SKU strategy, and the cost discipline that makes a multi-model AI product economically viable.

← [Back to README](../README.md)

---

## Why this is in the case study

A lot of AI demos are technically impressive and economically broken — they burn $5 of inference per user session and have no path to viability. Explanova's billing, email, and tier architecture exist because the product has to *work as a business*, not just as a tech demo.

This doc covers the business-side decisions: payment infrastructure, email deliverability, tier strategy, and the cost discipline that keeps the unit economics defensible.

## Stripe — payment infrastructure

- **Stripe SDK pinned to API version `2025-03-31.basil`** — explicit version pinning means platform updates are intentional, not accidental
- **Three secrets, three roles:**
  - `STRIPE_PUBLISHABLE_KEY` (frontend, safe to ship)
  - `STRIPE_SECRET_KEY` (Cloud Functions only, via Cloud Secret Manager)
  - `STRIPE_WEBHOOK_SECRET` (Cloud Functions, validates inbound Stripe events)
- **Customer + subscription IDs persisted on the user record** (`stripe_customer_id`, `stripe_subscription_id`) so the dashboard can render plan state without round-tripping Stripe on every load
- **Subscription-metadata mutations are scripted, not manual** — when the `family` plan was renamed to `crew` mid-cycle, a migration script updated both Firestore docs *and* Stripe subscription metadata in lockstep, with per-subscription logging and rollback receipts. Plan renames break things; scripts make the rename auditable.
- **Webhook handler is a separate Cloud Function** with App Check exemption (Stripe can't carry an App Check token) and webhook-secret signature validation as the auth layer instead

### Why Stripe (per `DECISIONS.md`)

> *"Stripe.js — React-native Stripe Elements integration is first-class. Flutter's Stripe plugin lags the JS SDK in features."*

The platform decision (web-first, deferring Flutter to Phase 2) was partially driven by Stripe's better web-SDK feature parity. Payment infrastructure influenced the platform call, not the other way around.

## SendGrid — email deliverability

Sending email from a startup domain to Apple, Google, Microsoft, and Yahoo without ending up in spam is harder than building the AI pipeline. The deliverability work was its own engineering project:

- **Custom sending domain:** `em1318.explanova.ai` (subdomain delegation per SendGrid best practice)
- **DNS configured in Cloudflare** with explicit SPF, DKIM, DMARC records
- **Domain authentication verified** before going live to any real users
- **`SENDGRID_API_KEY` provisioned via Firebase Secret Manager** (key prefix gotcha caught: the wrong-prefix key (`AQ.`) was rejected silently; correct `SG.` prefix is now part of the secret-rotation checklist)

### Real ops vignette — the Apple/iCloud SPF fix

Early invite emails were landing fine for Gmail and Outlook but bouncing for Apple/iCloud recipients. Root cause: the SPF record was missing `sendgrid.net`, so Apple's stricter DMARC alignment was rejecting the messages. Fix was a one-line Cloudflare DNS change. Lesson: **deliverability is a per-recipient-domain QA problem,** not a binary "email works / doesn't work" question.

## Tiered subscription model

Four production SKUs, each with a documented purpose:

| Tier | Purpose |
|---|---|
| `free` | Activation surface — parent can experience the product before paying |
| `plus` | Single-child individual plan |
| `family_premium` | Multi-child family plan with up to 6 account managers (parents, grandparents, babysitters, tutors) |
| `premium_visual` | Adds the visual-learner whiteboard primitives + future P2–P5 features as they ship |

**Mid-cycle rename:** `family_premium` was previously named `family`. The rename to `crew` (in some cohorts) was driven by user research showing "crew" tested better with the target demographic. The rename was executed with a migration script that updated Firestore docs *and* Stripe subscription metadata in lockstep — never manually, never piecemeal.

## Community pricing posture

Pricing is a values statement, not just a margin lever. Community discounts (military families, teachers, low-income households) were designed into the **Stitch ideation phase**, before any payment integration. That posture exists because:

- The founder is a Navy veteran. Military-family pricing isn't a marketing add-on; it's a personal commitment.
- The whole product premise is that **the $60–100/hr private-tutor gap shouldn't decide who falls behind.** Charging full price for the families with the least access would defeat the premise.

Community pricing is a market-positioning decision *and* a unit-economics decision. The blended ARPU is lower, but the addressable market is larger and the brand signal is stronger.

## Cost discipline — the AI economics layer

Multi-model routing isn't just an engineering decision; it's a cost decision:

| Decision | Cost effect |
|---|---|
| **Gemini 3 Pro Preview** for `solveHomework` (premium reasoning) | Higher per-call cost — but only for the user-facing call where quality matters |
| **Gemini 3 Flash Preview** for corpus ingestion | ~10× cheaper per call than Pro — used for the high-volume, low-stakes work |
| **Gemini 2.5 Flash** for GraphRAG community summaries | Cheap enough to be cached cheaply; cold call ~10s, warm call ~0.25s after caching |
| **Per-community summary cache** | ~40× speedup *and* ~40× cost reduction on warm calls |
| **`tts_quota` Firestore collection** | Daily TTS usage tracked per-user; runaway usage is detectable before the bill arrives |
| **4,000-character TTS rate limit per call** | Cloud TTS hard limit, but also a natural per-step usage cap |
| **Temperature 0.2 for tasks 1–4** | Deterministic outputs are cheaper to validate (no re-prompting on flaky output) |
| **32K thinking budget on Gemini 3 Pro** | Explicit budget — not "let it think as long as it wants" |

**Cost-per-completed-lesson is the real unit-economics number.** It's the sum of: extraction call + classification call + retrieval (vector + graph + community summary) + explanation call + practice call + avatar-script call + N × TTS calls + avatar-video composition. Every line in that sum has been chosen to optimize for *production cost*, not just *demo quality*.

## Family-account model — the business side

Up to 6 account managers per family is a billing decision as much as a UX decision. The premise: a single family unit (parents + grandparents + tutors + babysitters) shares one paid subscription rather than buying six. That's:

- **Better for the family** (one bill, shared visibility, consistent child experience)
- **Better for retention** (six adults invested in keeping the subscription active is harder to churn than one)
- **Worse for naive ARPU** (one paid seat instead of six) — but better for **household lifetime value** (the metric that actually matters)

The bet is that household LTV beats per-seat ARPU for a child-education product. Pricing model designed around that bet.

## Auditability + receipts

Every billing-relevant change (tier migration, plan rename, refund, comp grant) is scripted and logged. No production change to a customer's billing state is ever made manually in the Stripe dashboard. That discipline is what makes the system auditable when (not if) a customer disputes a charge — there's a script with a timestamp and a per-record outcome log for every change.

## What this rolls up to

Stripe pinned + secret-rotated + webhook-validated + scripted-migration · SendGrid with custom domain auth + per-recipient-domain deliverability QA · four-tier SKU strategy with intentional renames · community pricing as values + market positioning · multi-model cost routing + caching + quota tracking. **The product is engineered to work as a business at scale, not just to demo well.**

→ Back to [README](../README.md)
