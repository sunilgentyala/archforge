<div align="center">
  <img src="docs/assets/logo.svg" width="90" height="90" alt="ArchForge logo"/>
  <h1>ArchForge</h1>
  <p><strong>The Modern System Design Workbench</strong></p>
  <p>
    <a href="https://sunilgentyala.github.io/archforge/">Live Site</a> &nbsp;&middot;&nbsp;
    <a href="#pattern-library">60+ Patterns</a> &nbsp;&middot;&nbsp;
    <a href="#capacity-planner">Capacity Calculator</a> &nbsp;&middot;&nbsp;
    <a href="#architecture-templates">Templates</a> &nbsp;&middot;&nbsp;
    <a href="CONTRIBUTING.md">Contribute</a>
  </p>
  <p>
    <img alt="Patterns" src="https://img.shields.io/badge/patterns-60%2B-6366f1?style=flat-square"/>
    <img alt="AI Patterns" src="https://img.shields.io/badge/AI%2FML%20patterns-7-8b5cf6?style=flat-square"/>
    <img alt="License" src="https://img.shields.io/badge/license-MIT-06b6d4?style=flat-square"/>
    <img alt="Free Forever" src="https://img.shields.io/badge/price-free%20forever-10b981?style=flat-square"/>
  </p>
</div>

---

## What is ArchForge?

ArchForge started from a frustration that most system design resources are either static walls of text or expensive paywalled courses. The [System Design Primer](https://github.com/donnemartin/system-design-primer) is a fantastic starting point and deserves its 350k stars. But it was written in 2016, has no interactive tools, covers about 20 patterns, and has nothing on AI/ML systems architecture because those patterns barely existed then.

ArchForge fills the gaps:

- **60+ interactive pattern cards** organized by category, with trade-offs and complexity ratings
- **A real-time capacity calculator** so you can estimate storage, bandwidth, and RPS before you design anything
- **7 AI/LLM architecture patterns** covering LLM serving, RAG pipelines, multi-agent systems, and model routing
- **6 fully worked architecture templates** you can use in interviews or production planning
- **Modern topics**: vector databases, service meshes, cell-based architecture, edge computing, serverless
- **Head-to-head comparison tables** so you can explain why you chose one option over another

Everything is open source, runs in a browser with no install, and will stay free.

---

## Live Demo

**[sunilgentyala.github.io/archforge](https://sunilgentyala.github.io/archforge/)**

No signup, no paywall, no ads.

---

## What the Existing Tools Miss

| Feature | ArchForge | System Design Primer | ByteByteGo | Grokking |
|---|---|---|---|---|
| AI / LLM Architecture Patterns | 7 patterns | None | Partial | None |
| Interactive Capacity Calculator | Built-in | Static tables | No | No |
| Total Patterns | 60+ | ~20 | Many | ~15 |
| Edge / Serverless Patterns | Yes | No | Partial | No |
| Vector Database Pattern | Yes | No | Partial | No |
| Architecture Templates | 6 detailed | 8 solutions | Yes | Yes |
| Free and Open Source | Yes | Yes | Freemium | Paid |

---

## Pattern Library

60+ patterns across 8 categories. Each card shows the name, tagline, and complexity rating. Click any card to see the full description, when to use it, what you gain, and what you give up.

### Infrastructure (10)
Load Balancer, CDN, API Gateway, Service Mesh, Reverse Proxy, DNS Load Balancing, Edge Computing, Container Orchestration, Serverless Functions, Service Discovery

### Data Storage (12)
Master-Replica Replication, Multi-Master Replication, Horizontal Sharding, Database Federation, Connection Pooling, Time-Series Storage, Graph Database, Vector Database, Data Lake, Data Warehouse, Write-Ahead Log, MVCC

### Caching (8)
Cache-Aside, Write-Through, Write-Behind, Read-Through, Cache Invalidation Strategies, Distributed Cache, In-Process Memory Cache, CDN Cache Headers

### Scalability (8)
Horizontal Scaling, Vertical Scaling, Auto-Scaling Policies, Queue-Based Load Leveling, Back-Pressure Pattern, Rate Limiting, Bulkhead Isolation, Cell-Based Architecture

### Reliability (6)
Circuit Breaker, Retry with Exponential Backoff, Saga Pattern, Idempotency Design, Blue-Green Deployment, Canary Release

### API and Messaging (8)
REST API Design, GraphQL, gRPC, WebSocket, Server-Sent Events, Message Queue, Event Streaming, Webhook Design

### AI / ML Systems (7)
LLM Serving Architecture, RAG Pipeline, Feature Store, ML Training Pipeline, Embedding Search, Multi-Agent Orchestration, Model A/B Testing

### Security (5)
Zero Trust Architecture, OAuth 2.0 + PKCE, JWT Authentication, Secrets Management, API Rate Limiting and Security

---

## Capacity Planner

The capacity calculator lets you estimate any system's requirements before you draw a single box. Adjust sliders for daily active users, requests per user, request size, storage per user, and data retention. You get live results for:

- Average RPS and 3x peak RPS
- Daily storage growth and total storage over your retention window
- Peak bandwidth in MB/s or GB/s
- Cache size based on the 10% hot data rule of thumb

This replaces the manual back-of-napkin math that every system design interview expects you to do.

---

## Architecture Templates

Six complete system designs with component lists, scale targets, data model notes, and the trade-offs you should discuss during an interview.

1. **URL Shortener** -- beginner, 10B URLs, 100K reads/s
2. **Social Media Feed** -- intermediate, 500M DAU, push/pull hybrid
3. **Video Streaming** -- advanced, 2B DAU, adaptive bitrate HLS
4. **Chat System** -- intermediate, 2B DAU, 50B messages/day
5. **Ride Sharing** -- advanced, real-time geo queries at 4Hz per driver
6. **LLM API Service** -- advanced, prompt caching, model tiering, streaming

---

## Running Locally

There is no build step. Just open the file in a browser.

```bash
git clone https://github.com/sunilgentyala/archforge.git
cd archforge

# Option 1: Open directly
open docs/index.html

# Option 2: Serve locally for accurate relative paths
python -m http.server 8080 --directory docs
# Then visit http://localhost:8080
```

---

## Repository Structure

```
archforge/
├── docs/
│   ├── index.html          # The full interactive web app (single file)
│   ├── assets/
│   │   └── logo.svg        # SVG brand mark
│   └── og-image.svg        # Social sharing thumbnail
├── patterns/
│   └── README.md           # Pattern authoring guide
├── templates/
│   ├── url-shortener.md
│   ├── social-media-feed.md
│   ├── video-streaming.md
│   ├── chat-system.md
│   ├── ride-sharing.md
│   └── llm-api-service.md
├── CONTRIBUTING.md
└── LICENSE
```

---

## Contributing

ArchForge grows by community contribution. Adding a pattern takes about 15 minutes: write the JSON entry, test it locally, and open a PR.

See [CONTRIBUTING.md](CONTRIBUTING.md) for the pattern schema, style guide, and review process.

Ideas especially welcome for:
- Cloud provider-specific patterns (AWS, Azure, GCP)
- FinTech and HealthTech system designs
- Database engine internals
- Additional AI/ML production patterns

---

## License

MIT. Use it, fork it, teach with it, build on it.

---

<div align="center">
  <p>Built by <a href="https://github.com/sunilgentyala">Sunil Gentyala</a></p>
  <p><em>Something missing? Open an issue. Something wrong? Open a PR.</em></p>
</div>
