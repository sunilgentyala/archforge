# Chat System

**Scale target:** 2 billion DAU, 50 billion messages per day (WhatsApp-scale reference design)

---

## Functional Requirements

- One-on-one and group messaging (up to 1,000 members per group)
- Message delivery with read receipts
- Online/offline presence indicators
- Push notifications for offline users
- Message history retrieval

## Non-Functional Requirements

- Message delivery latency: under 100ms for online users
- Push notification latency: under 5 seconds for offline users
- Message storage: 5 years of history per user
- Availability: 99.99% (53 minutes of downtime per year)

---

## Scale Estimates

DAU: 2 billion
Messages per user per day: 25 (average)
Total messages per day: 50 billion
Average message size: 100 bytes
Daily storage: 5TB (text only, much more with media)
Connections per server: 100,000 WebSocket connections
Servers needed for connections: 20,000

---

## Component List

### WebSocket Connection Service
Maintains persistent WebSocket connections with online users. Each server holds 100,000 connections. A consistent hash ring routes users to the same server across reconnects.

Connection state (user ID to server ID mapping) lives in Redis so any service can find where a user is connected.

### Message Service
Stateless service that receives outbound messages from the connection servers, validates them, assigns a monotonically increasing message ID per conversation, and fans them out to recipients.

Message IDs are generated using Snowflake: 41 bits timestamp + 10 bits machine ID + 12 bits sequence. This gives ordering within a conversation without a central counter.

### Presence Service
Tracks which users are online. Users send a heartbeat every 30 seconds. If three heartbeats are missed, the user is marked offline.

Stored in Redis with a TTL of 90 seconds per user key. Presence updates are fanned out only to a user's contact list, not globally.

### Push Notification Service
For offline users, the message service publishes to a queue. The notification service reads from the queue and sends via APNs (Apple) or FCM (Android). Handles retry, deduplication, and batching.

### Cassandra Cluster (Message Storage)
Partition key: conversation ID. Clustering key: message ID (descending). This layout means reading the latest 50 messages for a conversation is a single-partition read, which is extremely fast.

Replication factor 3 across three data centers.

### Redis Cluster
- Online presence: TTL-based keys per user
- Connection routing: user ID to server ID mapping
- Recent message cache: last 30 days of active conversations

### Kafka
Fan-out event bus between message service and presence service, notification service, and analytics pipeline. Decouples the hot path from async operations.

---

## Data Model

```
Message
  conversation_id  UUID     (partition key)
  message_id       BIGINT   (clustering key, descending)
  sender_id        UUID
  content          TEXT
  media_url        TEXT     (nullable)
  sent_at          TIMESTAMP
  read_by          SET<UUID>  (or separate receipts table at scale)

Conversation
  conversation_id  UUID     PRIMARY KEY
  type             ENUM(dm, group)
  members          LIST<UUID>
  created_at       TIMESTAMP
```

---

## Key Trade-offs

**WebSocket vs HTTP long polling vs SSE**
WebSockets are bidirectional and have the lowest overhead per message once connected. Long polling works through corporate firewalls more reliably but creates thundering-herd reconnection storms. SSE is unidirectional. For chat, WebSockets are the right choice. Long polling is an acceptable fallback for hostile network environments.

**Push vs pull for message delivery**
Push (server sends to connected client) is simpler for online users and has lower latency. Pull (client polls for new messages) works better when connections are unreliable. The standard design is push for online users and push notifications via APNs/FCM for offline users, with pull as a fallback.

**Group message fan-out**
For groups under 1,000 members, fan out writes: write one copy of the message to each member's inbox. For very large groups (10K+ members), use a pull model: store one copy and have clients pull. WhatsApp-scale groups use a hybrid based on group size.

**Message deduplication**
Networks are unreliable. Clients may send the same message twice. Assign a client-generated UUID to each message at creation time. The message service is idempotent: if it sees the same client message ID twice, it returns the original response without storing a duplicate.

---

## Capacity Estimate

Using the ArchForge Capacity Calculator with:
- DAU: 2B
- Messages per user per day: 25
- Average message size (with overhead): 200 bytes
- Media storage per user per day: 2KB
- Retention: 5 years

Results:
- Average message RPS: 578,703
- Peak RPS: 1,736,111
- Daily storage (text): ~37TB
- Total storage over 5 years: ~67PB (including media, use tiered object storage)
- Peak bandwidth: ~330 GB/s

At this scale, you shard the message service and Cassandra by conversation ID using consistent hashing. The connection layer uses a dedicated tier separate from the business logic layer.
