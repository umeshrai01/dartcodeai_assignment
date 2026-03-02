# Benchmarks

Fill this table after running 10 calls.

| Run | Latency (ms) |
|-----|--------------|
| 1   |    717       |
| 2   |    657       |
| 3   |    671       |
| 4   |    677       |
| 5   |    678       |
| 6   |    651       |
| 7   |    704       |
| 8   |    817       |
| 9   |    715       |
| 10  |    794       |

**Min:** 651 ms  
**Max:** 817 ms  
**Average:** 710 ms  
**P95:** 794 ms  

## RCA (3–5 sentences)
Explain likely causes of variance (e.g., model queue depth, batching, cold start, network jitter, token generation variance, etc.).
--> The varience in latency is mostly due to network jitter on the round trip 
to HuggingFace's inference API, as each run makes two sequential HTTP calls. 
Occasional higher latencies likely reflect transient server-side queue depth or 
connection setup overhead on the HuggingFace side. Since the model (all-MiniLM-L6-v2) is lightweight and likely already warm, cold-start effects are minimal. The consistent similarity scores (0.534091) confirm deterministic model output, meaning all variance is network related. Overall, the 25% spread is typical for hosted inference endpoints without dedicated provisioned capacity.