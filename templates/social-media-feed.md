# Social Media Feed

**Scale target:** 500 million DAU, 200K posts per second, feeds served under 200ms globally

---

## Functional Requirements

- Users can post text, images, and videos (up to 2 minutes)
- Users see a ranked feed of posts from accounts they follow
- Likes, comments, and reposts
- Real-time notifications for interactions
- Search across posts and user profiles

## Non-Functional Requirements

- Feed generation latency: under 200ms at P99
- Post ingestion: 200K posts per second
- Notification delivery: under 5 seconds
- Availability: 99.99%

---

## Scale Estimates

DAU: 500 million
Posts per user per day: 0.02 average (1 post per 50 users per day)
Total posts per day: 10 million
Average post size: 500 bytes
Followers per user: average 200, celebrity accounts 50 million

---

## Component List

### Post Service
Accepts new posts, validates content, assigns a post ID (Snowflake), stores in the post database, and publishes a PostCreated event to Kafka.

Media (images, videos) is uploaded separately to a dedicated media service that stores originals in S3 and returns URLs attached to the post.

### Fan-out Service
Reads PostCreated events from Kafka. For normal users (under 1 million followers), writes the post ID to the timeline of every follower. This is the push/write-fan-out model.

For celebrity accounts (10M+ followers), skip the fan-out on write. Instead, fans pull celebrity posts at read time and merge them into the assembled feed. This is the pull/read-fan-out model.

The hybrid is often called the "celebrity problem" solution. Instagram and Twitter both use this approach.

### Timeline Service
Reads a user's pre-computed timeline from Redis (a sorted set of post IDs by timestamp). For most users this is purely a Redis read. For users following celebrities, merge the Redis timeline with a real-time celebrity post fetch.

Return post IDs to the feed service for hydration.

### Feed Hydration Service
Takes a list of post IDs and fetches the full post content (text, media URLs, author info, engagement counts) from the post cache. Returns the assembled feed. Caches popular posts in Redis to avoid repeated DB lookups.

### Post Database (Cassandra)
Stores all posts partitioned by post ID. Wide rows allow fetching a user's posts chronologically. Hot posts are cached in Redis separately.

### Graph Database / Adjacency List (Cassandra)
Stores follower-following relationships. Partitioned by user ID. Used by fan-out service to find all followers of a posting user.

At 500M users with 200 followers each, this is 100 billion edges. Cassandra handles this well with wide rows.

### Notification Service
Subscribes to interaction events (likes, comments, mentions) from Kafka. Delivers in-app notifications via WebSocket for online users and push notifications (APNs/FCM) for offline users. Batches low-priority notifications to reduce noise.

### Search Index (Elasticsearch)
Full-text search across post content, hashtags, and user profiles. Updated asynchronously from a Kafka consumer. Relevance ranked by recency, engagement, and user relationship.

---

## Data Model

```
Post
  post_id      BIGINT      PRIMARY KEY (Snowflake)
  author_id    UUID
  content      TEXT
  media_urls   LIST<VARCHAR>
  like_count   BIGINT
  comment_count BIGINT
  created_at   TIMESTAMP

Timeline (per user, in Redis)
  user_id      UUID        (key)
  post_ids     SORTED_SET  (score = timestamp, value = post_id)
  max_size     2,000 entries

Follow
  follower_id  UUID
  followee_id  UUID
  created_at   TIMESTAMP
```

---

## Key Trade-offs

**Push fan-out vs pull fan-out**
Push fan-out (write to all follower timelines on post) gives fast feed reads because all data is pre-computed. Write amplification is the cost: a user with 1 million followers triggers 1 million Redis writes per post. This is fine for normal users.

Pull fan-out (read and merge at query time) avoids write amplification for high-follower accounts. Read latency increases slightly because you merge live.

The hybrid approach: push for accounts under 1 million followers, pull for celebrity accounts above that threshold. This covers 99.99% of users with push and handles the celebrity problem with pull.

**Ranked feed vs chronological feed**
Chronological feeds are simpler to implement but produce worse engagement because users miss content posted while they were offline. Ranked feeds require a ML model trained on engagement signals but significantly improve retention. Most platforms default to ranked. Let users opt into chronological as a setting.

**Timeline size limit**
Storing unlimited post IDs per user in Redis is expensive. Cap each timeline at 2,000 entries. For users who scroll past 2,000 posts (rare), fall back to a paginated DB query. The 99th percentile user never hits this limit.

---

## Capacity Estimate

Using the ArchForge Capacity Calculator with:
- DAU: 500M
- Requests (feed loads + interactions) per user per day: 20
- Average response size: 50KB (feed page with thumbnails)
- Storage per user per day: 1KB (post data, excluding media)
- Retention: 7 years

Results:
- Average RPS: 115,741
- Peak RPS: 347,222
- Daily text storage: 476GB
- Total text storage over 7 years: 1.2PB
- Media storage: estimated 10x text = 12PB over 7 years
- Peak bandwidth: 16.5 GB/s

Timeline Redis storage: 500M users x 2,000 entries x 8 bytes = 8TB of Redis. Use Redis Cluster with 10-16 nodes.
