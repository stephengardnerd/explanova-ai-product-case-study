# 04 — Content Pipeline

> How I built the 10,476-entry curriculum corpus that backs Explanova's retrieval layer.

← [Back to README](../README.md)

---

## Why the corpus matters

Retrieval is only as good as what's behind it. A graph that points at nothing is useless; embeddings over a thin corpus are worse than pure generation because they give the *impression* of grounding without the substance.

So I invested in the corpus first.

## End state — K–12+ coverage

| Collection | Entries | Contents |
|---|---|---|
| `concept_library` | 7,203 | Textbook chapters, video lesson concepts, reference material |
| `worked_examples` | 3,273 | Worksheets, problem sets, step-by-step solutions |
| **Total** | **10,476** | |

**Grade-band coverage:**

| Band | ~Entries | Example topics |
|---|---|---|
| K–2 Early Learner | ~175 | Counting, ten frames, basic addition, intro reading |
| 3–5 Elementary | ~2,100 | Fractions, multiplication, basic science |
| 6–8 Middle | ~2,400 | Pre-algebra, ratios, intro geometry |
| 9–10 High | ~2,600 | Algebra 1/2, geometry proofs |
| 11–12 High | ~2,000 | Pre-calc, calc 1, physics 1, chemistry 1 |
| College | ~1,200 | Calc 2/3, statistics, physics 2/3 |

## Ingestion pipeline — stages

Every source artifact (PDF chapter, video lesson, worksheet) flows through the same stages before landing in Firestore:

1. **Extract** — source-specific extractors normalize raw material into plain text + structural metadata (chapter/section/problem boundaries)
2. **Chunk** — chunk size tuned to retrieval granularity; boundaries respect semantic units (a worked example is never split mid-step)
3. **Classify** — each chunk gets a subject / topic / subtopic / grade-band tag from the classification task
4. **Embed** — Vertex AI text-embedding model; vectors stored alongside the chunk
5. **Validate** — quality gates check for duplicate fingerprints, off-topic drift, and grade-band mismatch
6. **Write** — Firestore insert with embedding, metadata, and provenance fields

Pipeline is idempotent — re-running on the same source yields the same Firestore state (by fingerprint), so I can re-ingest a corrected source without cascading duplicates.

## Source diversity — guardrail against monoculture

A single-source corpus overfits to one pedagogy. The corpus draws from multiple teaching traditions so the retrieval layer surfaces the method diversity the graph exposes:

- **Math Tutor DVD** full course bundle (K–12+ video lesson transcripts)
- **OpenStax** textbooks (college-level math + science)
- **MIT OpenCourseWare** (select college-level coverage)
- **Curated worksheet libraries** (K–12 problem sets)
- **Supplemental open curriculum sources** (grade-band-specific)

Each source gets a provenance tag on every chunk, so the graph can surface *"here's the area-model explanation from source A *and* source B"* when the methods diverge.

## Content-safety + child-appropriateness

Every chunk passes a safety classifier before it's eligible for retrieval. A corpus serving children has a higher bar than a generic RAG index — drift is measured conservatively, and anything flagged is quarantined for review rather than auto-deleted.

## Why Firestore

Firestore was the right store for this corpus because:

- **Read-mostly workload** — Firebase Security Rules make `concept_library` and `worked_examples` read-only for authenticated users, writes via Cloud Functions only
- **Tight integration with the rest of the stack** — same auth domain as the frontend, same App Check enforcement, same observability surface
- **Vector search** is handled outside Firestore (via the GraphRAG service on Cloud Run with Vertex embeddings), so Firestore holds the content and metadata; the graph holds the structure; Vertex handles the vectors. Each tool doing what it's best at.

→ Next: [05 — Versioning and CI/CD](05-versioning-and-cicd.md)
