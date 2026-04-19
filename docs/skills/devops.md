# ⚙️ DevOps — Skills Exercised

← [Back to README](../../README.md)

---

Running a live AI SaaS solo means I am product, engineering, *and* platform. The DevOps surface area here isn't decoration — it's the reason the product stays up.

## Cloud Run — revision pinning + rollback discipline

- **Every deploy names its Cloud Run revision in the changelog.** Example arc from the project history: revisions `00001–00003` crash-looped on a missing `tenacity` import chained to a `@retry` decorator; revision `00004-gl2` was the first stable cut. That chain is *documented*, not hidden.
- **Threshold tuning is versioned.** When I tuned the adjacent-band similarity threshold from 0.55 → 0.50 → 0.48, three separate Cloud Run revisions captured each step (`00025-lq5`, `00026-pd8`, `00027-cqx`). Every tuning decision has a named artifact to roll back to.
- **Traffic splits respected.** 100% traffic moves are intentional, documented, and smoke-tested before I walk away from the console.

## Cloud Build — reproducible container builds

- Containerized GraphRAG service builds via Cloud Build with pinned dependencies
- Caught a stale pin (`llama-index-embeddings-vertex==0.1.3` — version never published to PyPI) by running the actual Cloud Build rather than trusting local `pip install`
- Build ID + SHA → Cloud Run revision: every live revision is traceable to the exact image that produced it

## Firebase platform

- **Hosting + Functions on Blaze plan** with structured log routing
- **App Check (reCAPTCHA v3)** enforced on every callable function — not optional, not per-endpoint, global
- **Firestore Security Rules** scoped strictly:
  - Users own their profile and child-profile documents
  - Lessons readable by the owning parent only
  - `concept_library` and `worked_examples` are read-only for authenticated users; writes via Cloud Functions only
- **Storage rules** segregate `avatars/{userId}/`, `homework/{userId}/`, and `lessons/{userId}/` — parents never read across the tenant boundary
- **`defineSecret` bindings** on every callable — no env var leakage, no plaintext credentials in source

## Managed Neo4j (GraphRAG backend)

- Graph store runs on a managed Neo4j tier with GCP billing integration
- Ingestion scripts + graph-rebuild watchers operate out-of-band so the live query service is never blocked on write load
- Community-summary cache invalidation is tied to graph refresh — no manual cache-flush dance

## Observability

- **Structured logs** with correlation IDs across the Cloud Run GraphRAG service, Cloud Functions, and the frontend
- **Live log analysis surfaced real bugs:** the 8000 ms client-side timeout on GraphRAG cold-starts was identified by pattern-matching `DOMException [TimeoutError]` in production logs — not by theorizing. Fix raised the timeout to 25000 ms after measuring cold-start community-summary LLM latency at 10–15 s.
- **Frontend telemetry** (`auditDiagramData()`) warns on any malformed AI payload so schema-layer failures can't go silent

## End-to-end UX validation — Playwright

- **78 tests passing across desktop (1280×800 Chromium) + mobile (Pixel 7 Chromium) viewports**
- Covers 39 diagram-primitive fixtures × 2 viewports — full regression surface for the whiteboard renderer
- Baselines stored as PNG snapshots with `maxDiffPixelRatio: 0.02` tolerance
- Retries = 1 to absorb RAF + animation timing flakes without masking real regressions (observed ~5% flake rate, 0% post-retry)
- HTML + JSON reporters for human + CI consumption

## Release engineering

- **16 semver tags** (`v1.0.0` → `v3.5.4`) with dated changelog entries explaining the *why*, not just the *what*
- **Dual-branch publish** — every release merges to both `master` and `main`, tagged once, deployed once
- **Feature branches follow `feat/t{task-id}-{slug}`** convention for bisect-friendly history

## Incident learning

When Cloud Run revisions 00001–00003 crash-looped, the root cause (`@retry` decorator without matching imports, coupled with a stale dependency pin) got written into the changelog *with the revision range*. That's how a solo engineer builds the same post-mortem discipline a team gets from incident reviews — by externalizing the lesson rather than mentally filing it.

## What this rolls up to

Revision pinning + reproducible builds + strict IAM/security rules + observability + end-to-end UX validation + versioned release discipline. The product stays up because the platform underneath it is boring in all the right places.

→ Next skill: [🎯 AI Product Management](ai-product-management.md)
