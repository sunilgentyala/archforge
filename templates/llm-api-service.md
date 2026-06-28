# LLM API Service

**Scale target:** 10 million requests per day, P99 latency under 2 seconds, streaming responses

---

## Functional Requirements

- Accept prompts from authenticated clients via REST
- Stream tokens back to clients using Server-Sent Events
- Route requests to different model tiers based on complexity
- Cache identical or near-identical prompt results
- Track per-client token usage for billing

## Non-Functional Requirements

- P50 time-to-first-token: under 300ms
- P99 latency: under 2 seconds
- Availability: 99.9% (about 9 hours of downtime per year)
- Cost efficiency: keep GPU utilization above 70%

---

## Scale Estimates

Daily active users: 100,000
Requests per user per day: 100
Total requests per day: 10 million
Average RPS: 116
Peak RPS (3x): 350
Average prompt tokens: 500
Average completion tokens: 300
Total tokens per day: 8 billion

At $0.25 per million output tokens for a large model, naive routing costs $2,000 per day. Model tiering with a complexity classifier brings this to roughly $400-600 per day.

---

## Component List

### API Gateway (Kong / AWS API GW)
Handles authentication, rate limiting per client, SSL termination, and request logging before anything touches your LLM stack. Rate limits: free tier at 10 RPM, paid tier at 100 RPM.

### Complexity Classifier
A small (3B parameter) model or a rules-based classifier that decides whether a prompt needs the large model or the fast model. Simple factual questions, short completions, and structured extraction go to the fast tier. Reasoning, long context, and creative tasks go to the powerful tier.

Routes approximately 70% of traffic to the fast model, cutting GPU cost by 60-80%.

### Prompt Cache (Redis)
Exact-match cache keyed on a SHA-256 hash of the model + prompt + parameters. Cache hit returns the full completion instantly. Target hit rate: 30-40% for common API patterns.

TTL: 5 minutes for most prompts, 24 hours for deterministic extraction prompts with temperature=0.

### Fast Model Tier (vLLM on A10G)
Smaller models (Haiku, Gemini Flash, Mistral 7B) running on A10G or L4 GPUs with continuous batching. Target: 50ms TTFT at 100 concurrent requests.

Auto-scale based on GPU KV-cache utilization, not CPU or memory.

### Powerful Model Tier (vLLM on A100)
Larger models (Opus, GPT-4o class) for complex tasks. More expensive, so only used when the classifier says it is necessary. Scale conservatively: these GPUs are expensive.

### Response Streamer
A thin proxy that receives the token stream from vLLM and forwards it to the client as Server-Sent Events. Handles client disconnects gracefully by stopping inference early. Buffers the full response in Redis for idempotent replay.

### Usage Metering
Counts input and output tokens per request, writes to a time-series DB (ClickHouse or TimescaleDB) for billing aggregation. Real-time usage dashboard per client. Hard-stop enforcement when monthly quota is exceeded.

---

## Data Model

```
Request
  id           UUID
  client_id    UUID
  model_tier   ENUM(fast, powerful)
  prompt_hash  SHA256
  input_tokens INT
  output_tokens INT
  latency_ms   INT
  created_at   TIMESTAMP

CacheEntry
  prompt_hash  SHA256  (primary key)
  model_id     VARCHAR
  completion   TEXT
  expires_at   TIMESTAMP
```

---

## Key Trade-offs

**Model routing vs always using the best model**
Routing reduces cost by 4-5x. The downside is that the classifier makes mistakes. A miscategorized prompt goes to the wrong tier. In practice, false negatives (routing a complex prompt to the fast model) are more damaging than false positives. Tune the classifier to be conservative: when in doubt, escalate to the powerful tier.

**Exact-match cache vs semantic cache**
Exact-match is simple, has no false positives, and adds under 1ms per request. Semantic cache (embed the prompt, find similar cached completions) can hit on paraphrased queries but risks serving wrong answers. Start with exact-match. Consider semantic cache only for high-traffic deterministic endpoints.

**Streaming vs buffered responses**
Streaming reduces perceived latency dramatically: users see output in 300ms instead of waiting 2 seconds for the full response. Complexity cost: you need a streaming proxy, the client must handle SSE, and you cannot know the full output token count until the stream ends (which affects billing). Worth it for user-facing applications. Use buffered responses for API-to-API calls where the full output is needed before processing.

---

## Capacity Estimate

Using the ArchForge Capacity Calculator with:
- DAU: 100K (peak 200K)
- Requests per user per day: 100
- Average response size: 300KB (streamed tokens)
- Storage per user per day: 500KB (prompt/completion logs)
- Retention: 2 years

Results:
- Average RPS: 116, Peak RPS: 350
- Daily log storage: 47GB
- Total log storage over 2 years: 34TB
- Peak streaming bandwidth: 98 MB/s

Infrastructure sizing:
- API Gateway: 2 nodes, 4 vCPU each (active-active)
- Classifier: 1 A10G GPU (shared, low utilization)
- Fast model tier: 2-4 A10G GPUs (auto-scale 1-8)
- Powerful model tier: 1-2 A100 GPUs (auto-scale 0-4)
- Redis: 16GB cluster (prompt cache + session state)
- ClickHouse: 3-node cluster (usage metering)

---

## Alternative Approaches

**Use hosted APIs instead of self-hosted models**
Anthropic, OpenAI, and Google APIs eliminate GPU management entirely. Pay-per-token pricing is higher per token but removes the operational overhead of running GPU infrastructure. Correct for most early-stage products. Switch to self-hosted when you can prove you will hit the break-even point (roughly 500M tokens per month for Llama 3 vs Claude Haiku).

**No complexity classifier at the start**
Route everything to one model tier. Simpler to operate, predictable quality. Switch to tiered routing when GPU costs become a meaningful budget line.

**Use OpenAI-compatible API servers**
vLLM, LiteLLM, and Ollama expose an OpenAI-compatible REST API so your gateway, clients, and tooling do not need to know which model is running behind them. This makes swapping models trivially easy.
