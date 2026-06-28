# URL Shortener

**Scale target:** 10 billion shortened URLs stored, 100K redirects per second

---

## Functional Requirements

- Generate a short alias for any long URL
- Redirect short URLs to the original URL
- Custom aliases (optional)
- Analytics: click count, referrer, geography (optional)

## Non-Functional Requirements

- Redirect latency: under 10ms at P99 with cache
- Write latency: under 100ms
- Availability: 99.99%
- Short codes must be globally unique

---

## Scale Estimates

Writes: 100 new URLs per second
Reads: 100,000 redirects per second (1000:1 read-write ratio)
Storage per URL: ~100 bytes (short code + long URL + metadata)
10 billion URLs: 1 TB of raw storage
Cache: top 20% of URLs account for 80% of traffic

---

## Component List

### Write API
Accepts the long URL, generates a 7-character base62 ID, stores it, returns the short URL. Simple stateless service behind the load balancer.

ID generation: use a distributed counter (Redis INCR) or a Snowflake-style ID generator (time + machine ID + sequence), then encode to base62. 7 characters of base62 gives 62^7 = 3.5 trillion unique IDs.

### Read / Redirect API
Accepts the short code, looks up in Redis first, falls back to Cassandra, returns a 301 or 302 redirect. If the key is not found, return 404.

Use 301 (permanent) for standard URLs. Use 302 (temporary) if you need analytics because 301 responses get cached by browsers and CDNs, bypassing your analytics pipeline.

### Redis Cache
Cache the top hot short codes. Target cache hit rate of 99%. Memory estimate: 10 million hot URLs at 200 bytes each = 2GB, well within a single Redis node.

TTL: set to 24h with lazy refresh on access. No need for eager invalidation since original URLs rarely change.

### Cassandra Cluster
Stores all URL mappings. Partition key is the short code. Cassandra handles 10 billion rows with no special configuration, and its hash-based partitioning means no hotspot for popular short codes.

Replication factor 3 across two data centers for availability.

### Analytics Pipeline (optional)
On every redirect, emit an event to Kafka (short code, timestamp, IP, referrer, user agent). A Flink or Spark Streaming consumer aggregates clicks per URL per hour into ClickHouse. Dashboards query ClickHouse.

The analytics path is async so it does not add latency to the redirect hot path.

---

## Data Model

```
URLMapping
  short_code   VARCHAR(10)   PRIMARY KEY
  long_url     TEXT
  created_at   TIMESTAMP
  user_id      UUID (nullable, for custom aliases)
  expires_at   TIMESTAMP (nullable)
```

---

## Key Trade-offs

**301 vs 302 redirect**
301 is permanent: browsers and CDNs cache it forever, so your server never sees repeat visits from the same client. Zero load on repeat visits, but you lose all analytics data after the first visit. 302 is temporary: every redirect hits your server, so analytics work perfectly, but you handle much more traffic. Pick 301 for link shorteners where cost matters, 302 for marketing campaigns where analytics matter.

**Base62 vs MD5 hash**
Base62 encoding of an auto-incrementing ID is sequential, predictable, and collision-free. MD5 hashing of the long URL is content-addressable (same URL gives same short code), but you need to handle hash collisions. Base62 is simpler and the right default.

**Custom aliases**
Custom aliases require a second index lookup (human-readable alias to short code) and a uniqueness check. Store them in the same table with a secondary index, or in a separate key-value store.

---

## Capacity Estimate

Using the ArchForge Capacity Calculator with:
- DAU: 1M unique users
- Reads per user per day: 100 redirects
- Response size: 1KB (including redirect response headers)
- Storage: 100 bytes per URL, 1M new URLs per day
- Retention: 10 years

Results:
- Average RPS: 1,157, Peak RPS: 3,472
- Daily storage: 95MB
- Total storage over 10 years: 350GB (URLs only, excluding logs)
- Peak bandwidth: 3.4 MB/s
