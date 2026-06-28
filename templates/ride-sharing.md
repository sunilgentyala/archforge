# Ride Sharing Platform

**Scale target:** 50 million DAU, 10 million active trips simultaneously, real-time driver location at 4Hz

---

## Functional Requirements

- Riders request a trip from location A to B
- System matches rider to nearest available driver
- Real-time tracking of driver location during trip
- Dynamic pricing (surge) based on supply and demand
- Trip history and receipts

## Non-Functional Requirements

- Driver match time: under 3 seconds from request to match
- Location update latency: driver location fresh within 500ms
- Map query latency: nearest-driver search under 100ms
- Availability: 99.99%

---

## Scale Estimates

DAU: 50M riders + 5M drivers
Active trips at peak: 10M
Driver location updates: 4 per second per driver = 20M location events per second during peak
Trip events per day: 30M trips
Average trip duration: 20 minutes

---

## Component List

### Location Service
Receives GPS coordinates from driver apps at 4Hz. Stores current position in Redis using GEO commands (geospatial index with O(log N) nearest-neighbor queries). Updates are written directly to Redis without a DB hop.

Publishes location events to Kafka for downstream consumers (trip tracker, analytics, surge pricing).

Redis GEOAGG stores driver positions as lat/lon encoded in sorted sets. Each city or geographic zone gets its own sorted set to keep cardinality manageable.

### Matching Service
When a rider requests a trip, queries Redis GEORADIUS to find available drivers within 5km. Ranks candidates by distance, driver rating, and vehicle type. Sends a match offer to the top candidate. If the driver does not accept within 15 seconds, offers to the next candidate.

Implemented as a stateful service with a short-lived pending request in Redis per rider. Match state machine: REQUESTED -> OFFERED -> ACCEPTED or TIMEOUT.

### Trip Service
Manages the lifecycle of active trips. Stores trip state in Postgres (or CockroachDB for multi-region). State machine: REQUESTED -> DRIVER_ASSIGNED -> PICKUP_IN_PROGRESS -> IN_TRIP -> COMPLETED -> SETTLED.

Emits events to Kafka at each state transition for billing, analytics, and notification services.

### Surge Pricing Service
Reads supply (available drivers per zone) and demand (incoming ride requests per zone) from Kafka. Computes a multiplier per zone every 60 seconds. Publishes surge multiplier to Redis for read by the matching service.

Zone granularity: 1km x 1km hexagonal grid (H3 library from Uber). Finer granularity captures local supply/demand better than city-level pricing.

### Maps Service
Provides ETA estimates, route calculation, and turn-by-turn navigation. At Uber scale, this is a proprietary system. At smaller scale, use the Google Maps Platform or OpenStreetMap with Valhalla or OSRM for routing.

ETA is the most important output: riders and drivers both rely on accurate pickup ETAs for trust.

### Notification Service
Sends real-time trip updates to rider and driver apps. Uses WebSocket for in-trip communication (continuous driver tracking). Falls back to push notifications (APNs/FCM) for trip match and completion events.

### Payment Service
Calculates fare from trip distance, duration, base rate, and surge multiplier. Integrates with Stripe, Braintree, or direct card networks. Holds an authorization at trip start, captures at trip end. Handles refunds and dispute resolution via async workflow.

---

## Data Model

```
Trip
  trip_id       UUID        PRIMARY KEY
  rider_id      UUID
  driver_id     UUID
  status        ENUM(requested, assigned, pickup, in_trip, completed, cancelled)
  origin        POINT
  destination   POINT
  surge_mult    FLOAT
  base_fare     DECIMAL
  final_fare    DECIMAL
  started_at    TIMESTAMP
  ended_at      TIMESTAMP

DriverLocation (Redis, not in DB)
  key: geo:zone:{city_id}
  members: driver_id
  score: geospatial encoded lat/lon
  TTL: 15 seconds (driver removed if no heartbeat)

Driver
  driver_id     UUID        PRIMARY KEY
  status        ENUM(offline, available, on_trip)
  vehicle_type  ENUM(economy, xl, luxury)
  rating        FLOAT
  location_updated_at TIMESTAMP
```

---

## Key Trade-offs

**Redis GEO vs PostGIS for driver location**
Redis GEO is an in-memory geospatial index with O(log N) radius queries in under 1ms. PostGIS is a full relational spatial database with richer query support but higher latency. For live driver locations that update at 4Hz and need sub-100ms radius queries, Redis GEO is the right choice. Use PostGIS for historical trip route storage and analytics.

**Pessimistic vs optimistic driver matching**
Pessimistic locking: lock a driver record when offering a match to prevent double-booking. Safe but slow. Optimistic: offer the match and handle concurrent offers with a conditional update (compare-and-swap on driver status in Redis). Uber uses optimistic matching with conflict resolution: the first ACCEPTED wins, the rest get a cancellation.

**Zone granularity for surge pricing**
City-level surge is simple but inaccurate. A downtown core can have 5x surge while the suburbs have none. 1km hexagonal zones (H3) give much better accuracy. The trade-off is more zones to update and more data to store. At Uber scale, H3 hex grids are the standard approach.

**4Hz vs 1Hz driver updates**
4Hz (four times per second) gives smooth driver animation on the map and accurate ETAs. 1Hz halves the backend write load. The visible difference on a map is significant: 4Hz feels smooth, 1Hz feels choppy. For premium experience, 4Hz is the right call. For cost reduction, drop to 2Hz during non-trip state.

---

## Capacity Estimate

Using the ArchForge Capacity Calculator with:
- DAU: 55M (riders + drivers)
- Requests per user per day: 40 (location updates + trip events + app checks)
- Average request size: 2KB
- Storage per user per day: 5KB (trip history, location logs)
- Retention: 3 years

Results:
- Average RPS: 25,463
- Peak RPS: 76,389
- Daily storage: 257GB
- Total storage over 3 years: 282TB
- Peak bandwidth: 146 MB/s

Location write load:
5M drivers x 4 updates/second = 20M writes/second at peak
This requires Redis Cluster with 20-30 shards, each handling ~700K writes/second.
Partition by city or geographic zone to keep each shard's write rate manageable.
