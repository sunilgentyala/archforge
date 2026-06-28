# Contributing to ArchForge

First off: thank you. ArchForge exists to be the most useful free system design reference on the internet, and that only happens if engineers who know things write them down.

This guide covers how to add a new pattern, fix an existing one, or add an architecture template.

---

## The Fastest Way to Contribute

1. Fork the repo
2. Open `docs/index.html` in your editor
3. Find the `PATTERNS` array around line 280
4. Add your pattern entry following the schema below
5. Open `docs/index.html` in a browser and verify it looks right
6. Open a pull request

No build step. No npm install. One file, done.

---

## Pattern Schema

Each pattern in the `PATTERNS` array looks like this:

```js
{
  id: 'my-pattern',           // kebab-case, unique across all patterns
  cat: 'infra',               // one of: infra, data, cache, scale, reliable, api, ai, security
  icon: '🔧',                 // single emoji
  name: 'My Pattern',         // Title Case, 2-4 words
  tag: 'One-sentence tagline that fits on a card', // under 80 chars
  desc: 'Full description...',// 2-3 sentences, plain language, no jargon without definition
  when: [                     // 3 bullet points: concrete scenarios, not vague generalities
    'First concrete situation',
    'Second concrete situation',
    'Third concrete situation',
  ],
  pros: [                     // 3 bullet points: specific benefits with some quantification if possible
    'What you gain specifically',
    'Another concrete benefit',
    'Third benefit',
  ],
  cons: [                     // 3 bullet points: real trade-offs, not just warnings
    'What you give up specifically',
    'Another real cost',
    'Third constraint',
  ],
  complexity: 3,              // integer 1-5: 1=anyone can implement, 5=distributed systems PhD territory
}
```

### Category Guide

| Category | What belongs here |
|---|---|
| `infra` | Infrastructure components: proxies, gateways, orchestrators, DNS |
| `data` | Storage engines, replication, partitioning, data formats |
| `cache` | All caching strategies, eviction policies, cache topologies |
| `scale` | Patterns that increase throughput or capacity under load |
| `reliable` | Patterns that improve fault tolerance, recovery, or deployment safety |
| `api` | Communication protocols, messaging, API design styles |
| `ai` | Machine learning infrastructure, LLM serving, training pipelines |
| `security` | Auth, authorization, secrets, input validation, audit |

### Writing Style

- Write for a senior engineer who does not know this specific pattern yet.
- No em dashes. No corporate speak. No passive voice where active works.
- Descriptions should say what the thing does, not what it is called.
- Trade-offs should be honest. If a pattern has a serious downside, say so plainly.
- Complexity ratings: 1 is a cache-aside pattern, 5 is a Saga implementation across three microservices.

---

## Adding an Architecture Template

Templates live in `templates/` as Markdown files. Copy the structure from an existing one like `url-shortener.md`. Each template covers:

1. System requirements (functional and non-functional)
2. Scale targets (DAU, RPS, storage)
3. Component list with rationale for each choice
4. Data model sketch
5. Key trade-offs and alternative approaches
6. Capacity estimate using the ArchForge calculator

---

## What Makes a Good PR

- One pattern or template per PR (keeps review focused)
- The `id` is unique and kebab-case
- All three `when`, `pros`, and `cons` arrays have at least 3 entries each
- No broken links or dead references
- Tested locally in a browser before opening the PR

---

## What Not to Send

- Vendor marketing (patterns that happen to require Product X are fine; ads for Product X are not)
- Patterns that are just renamed versions of existing ones
- Templates without a capacity estimate section
- Large refactors without a prior issue discussion

---

## Questions

Open an issue on GitHub. Response time is typically within a few days.
