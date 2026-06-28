# Pattern Authoring Guide

This directory holds supplementary documentation for patterns. The pattern data itself lives in `docs/index.html` inside the `PATTERNS` JavaScript array.

## Adding a Pattern

1. Open `docs/index.html`
2. Find the `PATTERNS` array
3. Add your entry following the schema in `CONTRIBUTING.md`
4. Reload `docs/index.html` in your browser to verify

## Pattern Categories

| Category ID | Display Name | What it covers |
|---|---|---|
| `infra` | Infrastructure | Load balancers, CDNs, gateways, orchestrators, DNS |
| `data` | Data Storage | Databases, replication, sharding, storage formats |
| `cache` | Caching | All caching strategies, topologies, eviction policies |
| `scale` | Scalability | Patterns that increase throughput or capacity |
| `reliable` | Reliability | Fault tolerance, deployment safety, recovery |
| `api` | API and Messaging | Protocols, messaging, event streaming, webhooks |
| `ai` | AI / ML | LLM serving, ML pipelines, embeddings, agents |
| `security` | Security | Auth, authorization, secrets, input validation |

## Complexity Scale

| Rating | What it means |
|---|---|
| 1 | Any developer can implement this in an afternoon |
| 2 | Straightforward with documentation, some edge cases |
| 3 | Requires careful design, has non-obvious failure modes |
| 4 | Distributed systems expertise needed, hard to test correctly |
| 5 | PhD territory, very hard to get right under production conditions |

## Quality Checklist

Before submitting a pattern PR, verify:

- [ ] `id` is unique and kebab-case
- [ ] `tag` is under 80 characters and describes the pattern, not just names it
- [ ] `desc` is 2-3 sentences, plain English, no jargon without definition
- [ ] `when` has exactly 3 concrete use cases
- [ ] `pros` has exactly 3 benefits with some quantification where possible
- [ ] `cons` has exactly 3 real trade-offs, not just warnings
- [ ] `complexity` rating matches the scale above
- [ ] Tested locally in a browser before opening the PR
