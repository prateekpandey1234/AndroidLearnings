# Elasticsearch for Vector Search — A Practical Primer

How "find me things similar to this" actually works: the storage model, the
maths, the algorithms, and the failure modes that bite in production.

Examples use a neutral domain (documents, products, articles). Formulas, JSON
and algorithm descriptions are exact — those are the transferable part.

---

## 1. Why a search index is a separate store

Most systems that do semantic search end up with **three** stores, each bad at
the others' job.

| Store | Role | Typical tech | Analogy (Android) |
|---|---|---|---|
| **Relational DB** | source of truth | Postgres, MySQL | Room / SQLite |
| **Search index** | fast retrieval | Elasticsearch, OpenSearch | AppSearch |
| **Cache** | repeat-request speed | Redis, Memcached | LruCache |

**The database owns correctness.** Exact lookups by key, transactions, foreign
keys, schema migrations, version history. If a row isn't here, the thing doesn't
exist.

```sql
SELECT * FROM products WHERE sku = 'ABC-123' AND status = 'ACTIVE';
```

What it's bad at: *"find the 10 products most similar in meaning to this one."*
That has no `WHERE` clause. You'd load every row and compare vectors in
application memory.

**The search index owns retrieval speed.** It stores a *derived projection* of
your data, shaped for exactly one query: nearest-neighbour ranking. It has no
transactions, no joins, no history. Delete it entirely and lose nothing — you
rebuild it from the database.

**The cache owns repeat speed.** Everything in it is recomputable from the other
two. A good test: if the cache is unreachable, the system should get *slower*,
never *wrong*. Many systems substitute a no-op cache client when none is
configured, precisely so this stays true.

### Why not collapse them?

- **DB only** — correct, but linear scans over vectors don't scale
- **Index only** — fast, but a failed write leaves a half-written document, and
  "what did this look like last month?" is unanswerable
- **Cache only** — fastest, and gone on restart

---

## 2. Vectors and embeddings

An **embedding** turns text into a fixed-length list of numbers — commonly 384,
768, or 1024 of them — such that things used in similar contexts land near each
other.

```
"backend engineer"  →  [0.031, -0.884, 0.220, ... ]   (1024 numbers)
"server developer"  →  [0.029, -0.871, 0.244, ... ]   ← nearly the same
"floral arranging"  →  [0.812,  0.104, -0.559, ... ]  ← very different
```

"Find similar" becomes geometry: *which stored points are closest to this
point?*

**Embeddings are computed at write time, not read time.** When a document is
created you call the model once and store the result. Search then reads a saved
vector — no model call on the request path. This is the single biggest
performance property of the design, and the reason a document without a stored
vector is unusable rather than merely slow.

---

## 3. Cosine similarity — the actual arithmetic

"Angle between vectors" is the intuition. Nothing measures an angle. What runs
is three passes and a division:

```
                a · b                    Σ(aᵢ × bᵢ)
cos(a, b) = ───────────── = ────────────────────────────────
             ‖a‖ × ‖b‖       √(Σaᵢ²) × √(Σbᵢ²)
```

```python
def cosine(a, b):
    dot   = sum(x*y for x, y in zip(a, b))   # multiply matching positions
    norm_a = sum(x*x for x in a) ** 0.5      # length of a
    norm_b = sum(y*y for y in b) ** 0.5      # length of b
    return dot / (norm_a * norm_b)
```

Worked with 2-number vectors so it's followable:

| pair | cosine | meaning |
|---|---|---|
| `[3,4]` vs `[3,4]` | **1.000** | identical direction |
| `[3,4]` vs `[4,3]` | 0.960 | very close |
| `[3,4]` vs `[-4,3]` | 0.000 | unrelated (perpendicular) |
| `[3,4]` vs `[-3,-4]` | −1.000 | opposite |

Real vectors use 1024 numbers. Same loop, longer.

Note cosine ignores magnitude — only direction matters. `[3,4]` and `[30,40]`
score 1.0. That's usually desirable for text: a long document and a short one
about the same topic should match.

### ⚠️ The rescale trap

Elasticsearch does **not** return raw cosine for `dense_vector` fields. It
rescales so scores are never negative:

```
score = (1 + cosine) / 2
```

| ES score | Actual cosine | Meaning |
|---|---|---|
| 1.0 | 1.0 | identical |
| 0.91 | **0.82** | closely related |
| **0.5** | **0.0** | **completely unrelated** |
| 0.0 | −1.0 | opposite |

**0.5 is the "no relationship" floor, not 0.** A result scoring 0.6 is barely
related, not "moderately related". Set thresholds accordingly — and if you
compute cosine yourself anywhere else in the system, apply the same rescale or
your two code paths will disagree.

---

## 4. The index mapping

Vector search is enabled by the field mapping, set when the index is created:

```json
PUT /products_v1
{
  "mappings": {
    "properties": {
      "vector": {
        "type": "dense_vector",
        "dims": 1024,
        "index": true,
        "similarity": "cosine"
      },
      "sku":      { "type": "keyword" },
      "category": { "type": "keyword" },
      "status":   { "type": "keyword" }
    }
  }
}
```

| Setting | Effect |
|---|---|
| `type: dense_vector` | this field holds an array of floats |
| `dims` | fixed length — every document must match exactly |
| `index: true` | **build an ANN graph**, don't merely store the numbers |
| `similarity` | `cosine`, `dot_product`, `l2_norm`, `max_inner_product` |

**`index: true` is the whole trick.** Without it, every search compares against
every document — correct but O(n). With it, Elasticsearch builds an HNSW graph
as documents are written, and search walks that graph instead.

`keyword` (not `text`) for filter fields matters: `keyword` is stored verbatim
for exact matching; `text` is tokenised and lowercased, so term filters on it
silently fail to match.

---

## 5. Anatomy of a kNN query

```json
POST /products_v1/_search
{
  "knn": {
    "field": "vector",
    "query_vector": [0.031, -0.884, ...],
    "k": 10,
    "num_candidates": 100,
    "filter": {
      "bool": {
        "must": [
          { "term": { "status": "ACTIVE" } },
          { "term": { "category": "footwear" } }
        ]
      }
    }
  },
  "size": 10,
  "_source": ["sku", "title"]
}
```

| Field | Meaning |
|---|---|
| `query_vector` | the point you're searching from |
| `k` | how many results to return |
| `num_candidates` | how many nodes to keep in play during the walk — the accuracy dial |
| `filter` | narrows the search **during** the walk, not after |
| `_source` | which stored fields to return; omit vectors unless needed — they're large |

A common pattern: `num_candidates = max(k × 10, floor)`. Larger means better
recall and higher latency.

---

## 6. Exact vs approximate (ANN)

**Nearest Neighbour**: given a point, which stored points are closest?

Exact means comparing against everything. At 1024 dims and 100k documents that's
~100 million multiplications per query, growing linearly forever.

**Approximate Nearest Neighbour (ANN)** trades a small chance of a wrong answer
for a very large speedup.

| | Exact | Approximate |
|---|---|---|
| Comparisons | all N | a few hundred |
| Correctness | guaranteed | very likely |
| Cost | O(n) | ~O(log n) |

ANN is the *category*. **HNSW** is the specific algorithm Elasticsearch (via
Lucene) uses.

What "approximate" actually costs: the search can miss a genuinely closer
document and return the second-best instead. Nothing errors. At typical
`num_candidates`, recall is in the high-90s%, but it's a probability, not a
promise. For small datasets, or when exactness matters, brute-force `script_score`
is still available and may be fast enough.

Measure it with **recall@k**: run exact search on a sample, run ANN on the same
queries, compute the overlap.

---

## 7. HNSW in depth

**H**ierarchical **N**avigable **S**mall **W**orld. Three ideas stacked — unpack
the name backwards.

### "Small World"

You can reach anyone on Earth through ~6 acquaintances: most of your friends are
local, but a few are far away, and those rare long links collapse the network's
diameter.

HNSW gives each node links to its nearest neighbours plus some longer-range
ones, so any node is reachable from any other in a few hops.

### "Navigable"

Because links point at *nearby* things, greedy walking works:

> At node A, target T. Neighbours: B (far), C (closer), D (far) → move to C.
> Repeat until no neighbour is closer.

No global map needed — each node locally knows enough to route.

### "Hierarchical"

Pure greedy walking gets stuck. So build layers, like zoom levels on a map:

```
Layer 2   A ──────────────────── M ──────────────────── Z     few nodes, huge jumps
          │                      │                      │
Layer 1   A ──── F ──── J ────── M ──── Q ──── U ────── Z     more nodes, medium
          │      │      │        │      │      │        │
Layer 0   A-B-C-D-E-F-G-H-I-J-K-L-M-N-O-P-Q-R-S-T-U-V-W-Z     everything, fine steps
```

Every node lives on layer 0; a random subset is promoted upward (probabilistically,
so higher layers are exponentially sparser).

### The search — two different algorithms

**Upper layers: greedy hill-climbing.**
Hold one node. Check neighbours. Move to the closest. Stop when none improves.
Drop a layer. No queue, no backtracking. Its only job is to hand layer 0 a good
entry point.

**Layer 0: bounded best-first search.**
This is `SEARCH-LAYER` from the original paper, and it uses **two heaps**:

- `candidates` — min-heap by distance to query (the frontier)
- `results` — max-heap, capped at `ef` (= `num_candidates`)

```
loop:
    c = pop nearest from candidates
    if dist(c) > dist(worst in results) and results is full:
        break                         # can't improve — stop
    for n in neighbours(c):
        if n not visited:
            mark visited
            if dist(n) < dist(worst in results) or results not full:
                push n onto candidates and results
                if len(results) > ef: evict worst
```

### Why this is best-first, **not** DFS

The distinction *is* the data structure:

| | Container | Expands next | Backtracks |
|---|---|---|---|
| DFS | **stack** | most recently discovered | yes, unwinds |
| Best-first | **priority queue** | globally closest unexplored | never — picks next-best |

A priority queue ordered by distance-to-target makes this the same family as
Dijkstra/A\*: greedy about *which node looks globally best*, not about *how deep
you've gone*.

**`ef` is what makes it approximate.** Unbounded, this becomes exact search.

### One-line summary

> Greedy hill-climb down the upper layers to find an entry point, then bounded
> best-first search (two heaps, capped at `ef`) at layer 0.

### Build-time parameters

| Parameter | Effect |
|---|---|
| `m` | links per node. Higher = better recall, more memory |
| `ef_construction` | search width while inserting. Higher = better graph, slower indexing |
| `ef` / `num_candidates` | search width at query time. The runtime dial |

`m` and `ef_construction` are fixed at index creation; changing them means
reindexing. `ef` is per-query.

---

## 8. Tuning

`k` and `num_candidates` do different jobs:

- **`k`** — how many results you want back
- **`num_candidates`** — how hard to look

```
num_candidates = 10   → one wrong turn and you're lost
num_candidates = 350  → many paths explored, nearly always correct
```

Cost is roughly linear in `num_candidates`; recall improves with sharply
diminishing returns. Tune empirically against a recall@k benchmark rather than
guessing.

**Over-fetching** is common: request more than you need so a post-processing
step has room to work. Re-ranking, deduplication and diversification all need
surplus candidates. If a step needs `k` final results and over-fetches ×5, then
`num_candidates` should be sized off the *fetched* count, not the final one.

---

## 9. Filtering

```json
"filter": {
  "bool": {
    "must": [
      { "term": { "status": "ACTIVE" } },
      { "term": { "category": "footwear" } }
    ]
  }
}
```

Filters are evaluated **during** the graph walk — non-matching nodes are skipped
as it traverses. That's why filtering is efficient here rather than a
post-processing pass.

### ⚠️ The "exclude one document" trap

A very common filter-builder helper looks like this:

```go
must := []map[string]any{}
for key, value := range filters {
    must = append(must, termClause(key, value))
}
return map[string]any{"bool": map[string]any{"must": must}}
```

It can only express **"results must have X."** There is no `must_not`, no ids
exclusion, no negation of any kind.

This becomes a real problem in one specific case: **searching an index using a
document that lives in that same index.** "Find products similar to product X"
returns X first, at a perfect score, every time — the query vector is byte-identical
to its stored vector.

Two workarounds:

1. **Over-fetch and drop** — request `k+1`, remove the source in application
   code. Match by **id or key, never by position**: if the source's vector is
   missing or stale it won't rank first, and a positional drop would then delete
   a real result.
2. **Add `must_not`** — an ids-exclusion clause in the query. Cleaner, but means
   changing a shared query builder.

Worth noting: if all your searches query a *different* index than the source
document came from, you never hit this — the exclusion is accidental and free.
The first same-index query is where it surfaces.

---

## 10. MMR — diversifying results

Nearest-neighbour search returns the *closest* results, which are frequently
near-duplicates of each other:

```
query: "running shoes"
  1. running shoes          0.98
  2. runner shoes           0.97     ← same thing
  3. shoes for running      0.97     ← same thing
  4. running footwear       0.96     ← same thing
  5. jogging shoes          0.95     ← same thing
```

All correct. All useless — nothing a user could actually act on.

**Maximal Marginal Relevance** picks results that are close to the query *and*
different from each other:

```
score = λ · relevance − (1 − λ) · maxSimilarityToAlreadySelected
```

```python
def mmr(hits, k, lam):
    selected, remaining = [], list(hits)
    while len(selected) < k and remaining:
        best, best_score = None, float("-inf")
        for c in remaining:
            penalty = max((cosine(c.vector, s.vector) for s in selected), default=0.0)
            score = lam * c.score - (1 - lam) * penalty
            if score > best_score:
                best, best_score = c, score
        selected.append(best)
        remaining.remove(best)
    return selected
```

| λ | Behaviour |
|---|---|
| 1.0 | pure relevance — MMR is a no-op |
| 0.7 | relevance-led, avoids near-duplicates |
| 0.0 | pure diversity — relevance ignored |

### Three properties worth internalising

**It is not a sort.** A sort needs a fixed comparator. Here the penalty depends
on what's *already selected*, so a candidate's score changes between rounds. It
is iterative greedy **selection** — `O(k·n)` with a re-scoring pass each round.
Formally, greedy submodular maximisation, the same shape as greedy set-cover.

**Round 1 always picks the top hit.** With nothing selected, the penalty is 0,
so the score is purely relevance. MMR can never diversify away the
highest-scoring result — which means it will never save you from the
self-match problem in §9.

**It requires over-fetching.** You cannot diversify a list of 7 if you only
fetched 7. A multiplier of ~5 is typical.

---

## 11. The dual-store sync problem

The hardest production issue with this architecture isn't performance. It's
**keeping two stores consistent.**

Your database and your index hold overlapping data, kept in sync by application
code. When a write path skips that code, they drift — silently.

### Three failure modes

**Row with no document.** The record exists; search can't find it.
Lookups by key succeed, then the vector fetch fails. Symptom: *"no stored vector
for id X"* — which confusingly arrives *after* a successful lookup.

**Document with no row.** The index returns results referencing records
that don't exist. Search finds them, every other part of the system 404s on
them. Often caused by a direct bulk load into the index, or by deletes that
didn't propagate.

**Divergent keys.** Both stores have the record under *different* identifiers,
so neither can find the other's. Typically an ingestion path that derives a key
one way while a second path derives it another.

All three are silent. Nothing errors at write time. You find out when a query
returns nothing, or returns something that can't be resolved.

### Defences

1. **One write path.** Every write goes through the same code that updates both
   stores. Direct bulk loads into either store are the usual root cause.
2. **Compensating writes.** If the index write fails after the DB write
   succeeds, flag the row (`vector_pending: true`) and reconcile later.
   **Caveat:** this only catches failures of *your own* write path. Records
   written by some other process carry no flag, so a reconciler keyed on it
   finds nothing.
3. **A consistency job that checks reality, not flags.** Walk the DB, probe the
   index by id, report both directions. Make dry-run the default: backfilling
   costs model spend and pruning deletes data.
4. **Make the index rebuildable.** If you can always regenerate it from the DB,
   drift is an inconvenience, not an incident.

---

## 12. Gotchas checklist

- [ ] `index: true` on the vector field — without it there's no ANN graph
- [ ] `dims` matches your model's output exactly; validate before writing
- [ ] Filter fields mapped as `keyword`, not `text`
- [ ] Scores are `(1+cos)/2` — **0.5 means unrelated, not 0**
- [ ] Searching an index with a document from that same index returns it first
- [ ] Naive filter builders have no `must_not`; exclusion may need code
- [ ] MMR round 1 always picks the top hit — it won't remove a self-match
- [ ] MMR needs over-fetch; `num_candidates` should be sized off the fetched count
- [ ] Exclude vectors from `_source` unless you need them — they're large
- [ ] Numeric types drift: a score may be `float32` fresh and `float64` after a
      JSON cache round-trip. Handle both
- [ ] `m` / `ef_construction` are fixed at index creation; changing them means reindex
- [ ] Deletes must propagate to the index, or you serve phantom results
- [ ] Measure recall@k against exact search before trusting ANN tuning

---

## 13. Where to go next

**Primary sources**

- *Efficient and robust approximate nearest neighbor search using Hierarchical
  Navigable Small World graphs* — Malkov & Yashunin (2016). The HNSW paper;
  §4 has the exact `SEARCH-LAYER` pseudocode.
- Elasticsearch docs → "k-nearest neighbor (kNN) search" and "dense_vector field type"
- Apache Lucene's `HnswGraphSearcher` — the implementation ES actually runs
- *The Use of MMR, Diversity-Based Reranking* — Carbonell & Goldstein (1998)

**Concepts worth reading up on**

| Topic | Why |
|---|---|
| **recall@k** | the only honest way to tune ANN |
| **IVF, Product Quantization** | alternative ANN families; different trade-offs |
| **Hybrid search (BM25 + vector)** | keyword and semantic together; usually beats either |
| **Reciprocal Rank Fusion (RRF)** | merging rankings from multiple retrievers |
| **Cross-encoder reranking** | slow, accurate second pass over top-k |
| **Quantization (`int8`, `bbq`)** | shrink vectors; big memory savings, small recall cost |
| **Chunking strategies** | how you split documents matters more than the model, often |
| **Embedding model choice** | dimension, domain fit, and cost dominate result quality |

**A good next exercise:** stand up a local Elasticsearch, index a few thousand
documents with embeddings, then measure recall@10 against brute force at
`num_candidates` of 10, 50, 100, 500. Seeing the recall-versus-latency curve
yourself makes every tuning decision afterwards obvious.
