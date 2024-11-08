---
marp: true
footer: '[@cdxker](htttps://x.com/cdxker) - denzell.f@trieve.ai'
transition: fade
theme: gaia
---

<!-- _footer: 'https://github.com/cdxker/scaling-search' -->

![bg left](https://cdn.trieve.ai/scaling-search-presentation/sharded-cubism.jpeg)

# Scaling Retrieval using Rust

### 1.2B Chunks at <100ms

---

![bg right:40%](https://avatars.githubusercontent.com/u/32852291?v=4)

## Denzell Ford
### Founder/CTO </br>[trieve](https://trieve.ai)

<i class="fa-brands fa-twitter"></i> Twitter: [@cdxker](https://x.com/cdxker)
<i class="fa-brands fa-linkedin"></i> LinkedIn: - [denzell-ford](https://www.linkedin.com/in/denzell-ford/)
<i class="fa-brands fa-github"></i> GitHub: [github.com/cdxker](https://github.com/cdxker)

---

![bg left](https://cdn.trieve.ai/scaling-search-presentation/cubism-clock)

# Agenda

- Introduction
- Ingestion
	- Finding Bottlenecks
- Explainable Search Results
	- Fast Typo Correction
	- Highlights
- Sharding your search index
  
---

# Introduction

*def: AI Search is the use of vector embeddings to search for N similar items to a given query.*

AI Search introduces a new set of challenges:

1) Efficient Ingestion.
2) Low latency retrieval.
3) Relevant and explainable search results.
4) Scalability.

---

# Why ingestion is hard

- Vector DB's are **not** the Primary DB. (no explicit relations)
- Calculating embeddings are expensive.
- Finding Bottle necks is hard.
- Iteration cycle is slower.

<!-- _footer: '[github.com/trieve/ingestion-worker](https://github.com/devflowinc/trieve/blob/main/server/src/bin/ingestion-worker.rs)' -->

![bg left:30%](https://cdn.trieve.ai/scaling-search-presentation/katy-fwy-ingest.jpeg)

--- 

# Ingestion Frameworks

- Single Thread, scale processes. (*horizontal scaling*)
- Multi Thread, scale threads. (*vertical scaling*)
- Multi Thread, scale threads **and** processes. (*hybrid scaling*) 

![bg left:30% height:100%](https://cdn.trieve.ai/scaling-search-presentation/ingestion-strats.png)

---

<img width="100%" height="100%" src="https://trieve.b-cdn.net/scaling-search-presentation/trieve-queue-diagram.png" />

---

# Finding Ingestion Bottlenecks

- Need to profile queue size changes
<img width="100%" src="https://cdn.trieve.ai/scaling-search-presentation/hackernews-ingestion.png" />
- Need to profile individual spans

---

# Bottlenecks

- Inserts into Vector DB
- Vector Embedding Compute
- Inserts into Postgres

---

# Vector Embedding Compute Latency

<!-- _footer: '[docs.trieve.ai/vector-inference](https://docs.trieve.ai/vector-inference)' -->

![bg right:30%](https://cdn.trieve.ai/scaling-search-presentation/sand-timer.jpeg)

- Cloud API's take ~150ms, but 15ms possible w/ Candle
- Ingesting 1,000,000 chunks (33k batches)
    - ~80mins using OpenAI
    - ~8mins on Candle
- LTR plugins usually <10ms

---

# Latency by layer

1. Inference vectors for query ~15ms
2. Typo detection+correction ~1ms
3. Score docs in search index ~10ms
4. Re-rank (LTR/cross-encode) ~15ms
5. First LLM token (TTFT) ~250ms

--- 

# Explainable Search Results

- Semantic Search doesn't care about keywords. (highlights needed)
- Semantic Search is bad at typos.

Lets look at highlights on hackernews with an example query: ["Tools to Track spans"](https://hn.trieve.ai/?score_threshold=0.5&page_size=30&prefetch_amount=30&rerank_type=none&highlight_delimiters=+%2C-%2C_%2C.%2C%2C&highlight_threshold=0.85&highlight_max_length=50&highlight_max_num=50&highlight_window=0&recency_bias=0&highlight_results=true&use_quote_negated_terms=true&q=Tools+to+Track+spans&storyType=story&matchAnyAuthorNames=&matchNoneAuthorNames=&popularityFilters=%7B%7D&sortby=relevance&dateRange=all&searchType=semantic&page=1&getAISummary=false)

---

### Without Highlights

![bg width:100% left:70%](https://cdn.trieve.ai/scaling-search-presentation/semantic-no-highlights.png)

---

### With Highlights

![bg width:100% left:70%](https://cdn.trieve.ai/scaling-search-presentation/semantic-with-highlights.png)

Semantic Search biases towards larger content

---

# Fast Typo Correction

<!-- _footer: '[trieve.ai/building-blazingly-fast-typo-correction-in-rust](https://trieve.ai/building-blazingly-fast-typo-correction-in-rust)' -->

![bg left:40%](https://cdn.trieve.ai/scaling-search-presentation/cubism-crab.jpeg)

- Many vector db's don't correct typos
    - Harder when multi-tenant
- BK-Tree vs. SymSpell Data Structures
    - [SymSpell is 100x faster](https://seekstorm.com/blog/symspell-vs-bk-tree/)

---

# Sharding your search index Pt. 1

## TLDR: think about it

### What is a shard?

- A shard is an instance of the search lib (Lucene, FAISS, etc.)
- Shards are made up of segments
- Segments are indices (IDF or HNSW)
- Small segments should be merged off-peak hours

---

# Sharding your search index Pt. 2

### How to optimize?

- Usually best to think about sharding before HNSW params
- Always have `shards > 2*nodes` such that you can horizontally scale CPU if needed
- Replicate shards for more read throughput
- Different db's map their thread pools to shards differently and you should work with your vendor to address it

---

# Questions?

![bg right:50%](https://cdn.trieve.ai/scaling-search-presentation/cubism-questions.jpeg)

Thank you!
