<div align="center">
  <img src="docs/assets/logo.svg" width="100" height="100" alt="ArchForge logo"/>

  <h1>ArchForge</h1>

  <p><strong>The Modern System Design Workbench</strong></p>

  <p>
    <a href="https://sunilgentyala.github.io/archforge/">Live Site</a>
    &nbsp;&middot;&nbsp;
    <a href="#pattern-library">60+ Patterns</a>
    &nbsp;&middot;&nbsp;
    <a href="#capacity-planner">Capacity Planner</a>
    &nbsp;&middot;&nbsp;
    <a href="#architecture-templates">Templates</a>
    &nbsp;&middot;&nbsp;
    <a href="CONTRIBUTING.md">Contribute</a>
  </p>

  <p>
    <img alt="Patterns" src="https://img.shields.io/badge/patterns-60%2B-6366f1?style=flat-square"/>
    <img alt="AI Patterns" src="https://img.shields.io/badge/AI%2FML%20patterns-7-8b5cf6?style=flat-square"/>
    <img alt="License" src="https://img.shields.io/badge/license-MIT-06b6d4?style=flat-square"/>
    <img alt="Free" src="https://img.shields.io/badge/price-free%20forever-10b981?style=flat-square"/>
    <img alt="No Build Step" src="https://img.shields.io/badge/build%20step-none-f59e0b?style=flat-square"/>
  </p>
</div>

---

## What is ArchForge?

Most system design resources fall into one of two buckets: a static wall of markdown that you read once and forget, or a paywalled course that recycles the same eight example problems. The most popular free reference covers roughly 20 patterns, was last meaningfully updated around 2017, and predates the entire AI/ML infrastructure space.

ArchForge was built to fill those gaps. It is a browser-based workbench with no install, no account, and no cost:

- **60+ interactive pattern cards**, each with a trade-off breakdown and complexity rating
- **A live capacity planner** that calculates RPS, storage, bandwidth, and cache size from sliders
- **7 AI and LLM architecture patterns** covering things like RAG pipelines, LLM serving, multi-agent orchestration, and model routing
- **6 fully worked system design templates** for interviews and production planning
- **Modern patterns** that simply did not exist when the primer was written: vector databases, service meshes, cell-based architecture, edge computing, serverless

Everything is open source and will stay that way.

---

## Live Site

> **[sunilgentyala.github.io/archforge](https://sunilgentyala.github.io/archforge/)**
>
> No signup. No paywall. No ads. Works in any browser.

---

## Where ArchForge Fills the Gaps

| Feature | ArchForge | System Design Primer | ByteByteGo | Grokking |
|---|:---:|:---:|:---:|:---:|
| AI / LLM Architecture Patterns | 7 patterns | None | Partial | None |
| Interactive Capacity Calculator | Built-in | Static tables | No | No |
| Total Patterns Covered | 60+ | ~20 | Many | ~15 |
| Edge / Serverless Patterns | Yes | No | Partial | No |
| Vector Database Pattern | Yes | No | Partial | No |
| Architecture Templates | 6 detailed | 8 solutions | Yes | Yes |
| Free and Open Source | Yes | Yes | Freemium | Paid |

---

## Pattern Library

60+ patterns across 8 categories. Each card shows the tagline and complexity rating. Click any card to expand the full detail: what it is, when to reach for it, what you gain, and what you give up.

| Category | Count | Patterns |
|---|:---:|---|
| Infrastructure | 10 | Load Balancer, CDN, API Gateway, Service Mesh, Reverse Proxy, DNS Load Balancing, Edge Computing, Container Orchestration, Serverless Functions, Service Discovery |
| Data Storage | 12 | Master-Replica Replication, Multi-Master Replication, Horizontal Sharding, Database Federation, Connection Pooling, Time-Series Storage, Graph Database, Vector Database, Data Lake, Data Warehouse, Write-Ahead Log, MVCC |
| Caching | 8 | Cache-Aside, Write-Through, Write-Behind, Read-Through, Cache Invalidation Strategies, Distributed Cache, In-Process Cache, CDN Cache Headers |
| Scalability | 8 | Horizontal Scaling, Vertical Scaling, Auto-Scaling Policies, Queue-Based Load Leveling, Back-Pressure, Rate Limiting, Bulkhead Isolation, Cell-Based Architecture |
| Reliability | 6 | Circuit Breaker, Retry with Exponential Backoff, Saga Pattern, Idempotency Design, Blue-Green Deployment, Canary Release |
| API and Messaging | 8 | REST, GraphQL, gRPC, WebSocket, Server-Sent Events, Message Queue, Event Streaming, Webhook Design |
| AI / ML Systems | 7 | LLM Serving Architecture, RAG Pipeline, Feature Store, ML Training Pipeline, Embedding Search, Multi-Agent Orchestration, Model A/B Testing |
| Security | 5 | Zero Trust Architecture, OAuth 2.0 + PKCE, JWT Authentication, Secrets Management, API Rate Limiting |

---

## Capacity Planner

Adjust five sliders and get six live estimates instantly. No spreadsheet, no mental math.

**Inputs**
- Daily active users
- Average requests per user per day
- Average request and response size
- Storage per user per day
- Data retention period in years

**Outputs**
- Average RPS and 3x peak RPS
- Daily storage growth
- Total storage over the retention window
- Peak bandwidth
- Cache size based on the 10% hot data heuristic

---

## Architecture Templates

Six complete system designs, each with a component list, scale targets, data model sketch, and the specific trade-offs worth mentioning out loud in an interview.

| Template | Difficulty | Scale Target |
|---|:---:|---|
| URL Shortener | Beginner | 10B URLs, 100K redirects/s |
| Social Media Feed | Intermediate | 500M DAU, push/pull hybrid fan-out |
| Video Streaming | Advanced | 2B DAU, adaptive bitrate HLS |
| Chat System | Intermediate | 2B DAU, 50B messages/day |
| Ride Sharing | Advanced | 10M live trips, 4Hz driver location |
| LLM API Service | Advanced | 10M requests/day, prompt caching, model tiering |

Template markdown files live in [`/templates`](templates/) and can be read directly without opening the browser app.

---

## Running Locally

There is no build step. The entire app is a single HTML file.

```bash
# Clone the repo (useful if you found this via the live site or a link elsewhere)
git clone https://github.com/sunilgentyala/archforge.git
cd archforge
```

Then open the app in one of two ways:

```bash
# Option 1 -- open the file directly in your default browser
# macOS
open docs/index.html
# Windows
start docs/index.html
# Linux
xdg-open docs/index.html

# Option 2 -- serve locally (recommended, ensures relative paths work correctly)
python -m http.server 8080 --directory docs
# Then visit http://localhost:8080
```

---

## Repository Layout

```
archforge/
├── docs/
│   ├── index.html          # The full interactive app (one file, no dependencies)
│   ├── assets/
│   │   └── logo.svg        # SVG brand mark
│   └── og-image.svg        # Social sharing thumbnail (1200x630)
├── patterns/
│   └── README.md           # Pattern authoring guide and schema reference
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

Adding a pattern takes roughly 15 minutes. You write one JavaScript object, verify it looks right in a browser, and open a PR. No build tools, no npm install.

See [CONTRIBUTING.md](CONTRIBUTING.md) for the full pattern schema, complexity scale, writing style guide, and review checklist.

Areas where contributions are especially welcome:

- Cloud provider-specific patterns (AWS, Azure, GCP)
- FinTech and HealthTech system designs
- Database engine internals
- Additional AI and ML production patterns

---

## License

MIT. Use it, fork it, teach with it, ship with it.

---

<div align="center">
  <p>Built by <a href="https://github.com/sunilgentyala">Sunil Gentyala</a></p>
  <p><sub>Something missing? Open an issue. Something wrong? Open a PR.</sub></p>
</div>
