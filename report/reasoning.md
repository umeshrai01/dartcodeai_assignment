# Reasoning Notes

Briefly document key decisions and tradeoffs for:
- embeddings endpoint selection,
- cosine similarity implementation,
- quota logic,
- analytics delta handling.

## Embeddings Endpoint Selection
I chose HuggingFace's `sentence-transformers/all-MiniLM-L6-v2` because it's a 
free, lightweight model that produces 384-dimensional vectors optimized for 
semantic similarity tasks. It has low inference latency compared to larger 
models, making it practical for a CLI tool without incurring API costs.

## Cosine Similarity Implementation
Implemented cosine similarity from scratch using the formula: dot(a,b) / (||a|| 
* ||b||). A zero-vector guard is included to avoid division-by-zero — if either 
vector has zero magnitude, we return 0.0 instead of NaN. This is more robust 
than relying on a library and keeps dependencies minimal.


## Quota Logic
The [canProcess] function enforces a two-level check: individual field caps
(512 tokens each for prompt and completion) prevent any single request from 
being disproportionately large, while the combined check (`prompt + completion 
+ currentUsage > maxTokens`) enforces the global budget. This prevents both 
per-request abuse and overall quota exhaustion.

## Analytics Delta Handling
Negative deltas (curr < prev) indicate counter resets or data anomalies, so we 
safely return 0 instead of a misleading negative value. Large spikes are 
clamped at 100 MB to prevent outlier data from corrupting aggregate analytics. 
Equal values trivially return 0. This makes the delta function safe for use in 
automated pipelines where anomalies could cascade.