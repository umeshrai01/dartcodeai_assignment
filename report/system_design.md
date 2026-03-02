# System Design — Scaling to 10k req/min

Write ~10–15 sentences covering:
- batching strategy,
- embedding caching,
- backpressure + rejection policies,
- retry & exponential backoff,
- observability (metrics/logs/traces),
- degraded/fallback behavior,
- capacity planning notes.

# System Design

To scale DartCodeAI to 10,000 requests per minute, I would deploy the service behind a load balancer with multiple stateless worker instances, each running the embedding CLI as an HTTP microservice. Incoming embedding requests would be grouped using a **batching layer** that collects requests over a short window (e.g., 50ms) and sends them as a single batch to the model endpoint, significantly reducing per-request overhead and improving GPU utilization.

An **embedding cache** (Redis or Memcached) keyed on a hash of the input text would eliminate redundant computations — if the same document has been embedded before, we return the cached vector instantly. Based on typical workloads, this alone could reduce effective load by 30–50%.

For **backpressure and retries**, I would use a token-bucket rate limiter at the gateway to reject excess traffic with HTTP 429 responses, preventing overload from cascading to the model backend. Failed requests would be retried with exponential backoff and jitter (e.g., base 200ms, max 3 retries) to avoid thundering herd effects on recovery.

**Observability** is critical at this scale: I would instrument the service with OpenTelemetry to emit latency histograms (p50/p95/p99), error rate counters, and distributed traces across the gateway, cache, and model layers. Structured JSON logs with request IDs would enable fast debugging. Alerts on p95 latency breaches and error rate spikes would trigger on-call workflows.

For **degraded-mode behavior**, if the upstream model endpoint becomes unavailable or exceeds latency SLOs, a circuit breaker would trip and the service would return cached embeddings where available or a graceful error response with a retry-after header. This ensures partial availability rather than total failure. Capacity planning would target 2x headroom over peak expected traffic to absorb bursts without degradation.
