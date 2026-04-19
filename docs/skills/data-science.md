# 🧪 Data Science — Skills Exercised

← [Back to README](../../README.md)

---

Explanova is a data-science product disguised as a tutoring app. The retrieval layer, the grounding methodology, and the benchmark discipline are all data-science work. Specifically:

## Corpus engineering

- **Curated a 10,476-entry K–12+ corpus** from Math Tutor DVD, OpenStax, MIT OCW, and open curriculum sources. Balanced across six grade bands (K–2 through college) so the retrieval layer works at every level, not just the easy ones.
- **Source-diverse by design.** A single-source corpus overfits to one pedagogy. Multiple traditions mean multiple valid methods per topic.
- **Idempotent ingestion pipeline.** Re-running on a corrected source yields the same Firestore state. No cascading duplicates on re-ingest.
- **Provenance-tagged chunks.** Every chunk knows where it came from, so the graph can surface "method A from source X, method B from source Y" when pedagogies diverge.

## Embedding strategy

- **Vertex AI text-embedding model** for the vector layer. Tuned chunk-size boundary selection so a worked example is never split mid-step.
- **Per-topic community-summary cache** (cold → warm ~40× speedup). Deterministic given graph membership, invalidated on graph refresh.
- **Vector + graph hybrid retrieval.** Vector seed → graph expansion → community detection → LLM synthesis. Neither alone was enough; the combination is what ships.

## Knowledge-graph design

Modeled the K–12 math curriculum as a typed property graph in Neo4j:

- **6 grade bands × 30 quarters × 80 topics × 133 methods × 129 SOL codes**
- **445 typed relationships** across `HAS_QUARTER`, `COVERS`, `USES_METHOD`, `ALIGNS_TO`, `PREREQUISITE`
- Designed the edge vocabulary explicitly — typed edges, not generic "related to" — so the graph can drive pedagogically meaningful expansion (`USES_METHOD` surfaces sibling methods; `PREREQUISITE` surfaces gaps)

## Classification + grade-band calibration

- **Constrained subject taxonomy** (15 subjects, 7 grade bands) — bounded the classification problem enough that the task runs at ≥90% accuracy
- **Adjacent-band fallback** with a stricter similarity threshold (0.48) than exact-band (0.45), tuned empirically across three Cloud Run revisions to balance recall vs. false-positives. Tuning log was kept — **0.55 (too strict) → 0.50 (still missed the signal) → 0.48 (captures cross-band, stays above the exact-band floor)**.
- Cross-band matches are **demoted to `partially_grounded`** with a methodNote so the avatar frames age-appropriately, rather than silently teaching off-level.

## Benchmark methodology

Every prompt was scored against a held-out benchmark before promotion:

| Criterion | Weight |
|---|---|
| Correctness | 40% |
| Classification accuracy | 15% |
| Explanation quality | 20% |
| Grounding quality | 15% |
| Hint quality | 10% |

Success thresholds: ≥95% correctness, ≥90% classification, ≥85% strong-or-partial grounding, **zero incorrect answers with high confidence.**

**RAG vs. pure-generation ablation.** Ran the benchmark twice — with corpus retrieval and without — to quantify the delta. That delta is the measurable dollar value of the content library.

## Grounding provenance — observability into the model

Every response carries a `groundingQuality ∈ {grounded, partially_grounded, ungrounded}` tag. The frontend surfaces that to the parent. No grounded-looking answer ships from an ungrounded retrieval — epistemic honesty is a product feature.

## Output-schema discipline

Discriminated-union JSON schemas (`anyOf` per primitive type) eliminate the class of bug where the LLM produces syntactically-valid but semantically-empty output. Schema validation is the enforcement layer; the prompt is just the hint.

## What this rolls up to

Corpus engineering + embedding strategy + knowledge-graph design + classification calibration + benchmark discipline + grounding provenance + schema discipline — that's the data-science spine of the product. It's what makes the retrieval layer actually *work* rather than *appear* to work.

→ Next skill: [⚙️ DevOps](devops.md)
