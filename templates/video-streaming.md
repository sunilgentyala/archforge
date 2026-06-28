# Video Streaming Platform

**Scale target:** 2 billion DAU, 500 hours of video uploaded every minute (YouTube-scale reference design)

---

## Functional Requirements

- Upload videos (up to 10GB, any format)
- Transcode to multiple resolutions and bitrates
- Stream videos with adaptive bitrate
- Search videos by title, description, and transcript
- Recommendations based on watch history

## Non-Functional Requirements

- Upload processing: video available for streaming within 5 minutes of upload
- Streaming start time: under 2 seconds globally
- Availability: 99.99%
- Support 4K, 1080p, 720p, 480p, 360p, 240p quality levels

---

## Scale Estimates

Uploads: 500 hours per minute = 30,000 hours per hour
Each hour of video produces 6 quality variants, average 5GB each: 30 GB per uploaded hour
Daily upload storage: ~220 TB
Total platform storage (10 years): ~800 PB

Streaming: 2 billion users watching an average of 30 minutes per day
Video bitrate: 2 Mbps average across quality levels
Total streaming bandwidth: ~833 Tbps peak

This scale requires a CDN to be feasible.

---

## Component List

### Upload Service
Accepts chunked multipart uploads from clients. Stores raw video in S3 (or GCS). Publishes a transcoding job to a Kafka queue on successful upload. Generates a unique video ID using Snowflake.

Chunked upload allows resume after network interruption. Chunk size: 5-10MB per part.

### Transcoding Pipeline
Workers pull transcoding jobs from Kafka. Each job spins up FFmpeg to transcode the raw video into 6 quality variants using H.264 or H.265. Output goes back to S3 in HLS format (segments + manifest files).

Workers run on spot/preemptible GPU instances to reduce cost by 60-70% vs on-demand. Each 1-hour video takes approximately 10 minutes to transcode at 1080p.

### Metadata Service
Stores video metadata (title, description, upload time, channel, duration, view count) in a PostgreSQL database. Read-heavy, so uses read replicas for search and recommendation queries.

Video status state machine: UPLOADING -> PROCESSING -> READY -> REMOVED.

### HLS Manifest and Segment Storage (S3 + CDN)
Each video is stored as an HLS package: a master manifest listing quality variants, per-quality manifests, and 6-second video segments. Stored in S3 and served via CDN (Cloudflare, Akamai, or a custom CDN at YouTube scale).

Adaptive bitrate: the player requests the master manifest, measures available bandwidth, and selects the appropriate quality variant automatically. Switches quality mid-stream based on bandwidth changes.

### Search Index (Elasticsearch)
Full-text search across title, description, and auto-generated transcripts. Also indexes tags, channel metadata, and engagement signals (view count, like rate). Transcript indexing enables searching for spoken content within videos.

Updated asynchronously as transcription jobs complete.

### Recommendation Engine
Two-stage recommendation: candidate generation using collaborative filtering (who watches similar videos) and ranking using a trained neural network that considers watch time, click-through rate, and freshness.

Training data lives in BigQuery. Model trained daily in batch. Served via a feature store with pre-computed user embeddings.

### CDN (Multi-PoP)
The CDN is the most important component for streaming performance. At 2B DAU, you need 200-500 CDN edge nodes globally to hit sub-2-second start times. CDN handles 95%+ of all video traffic.

Cache eviction: cache the top 20% of videos (by views) indefinitely. Less popular videos are re-fetched from S3 on CDN miss.

---

## Data Model

```
Video
  video_id       BIGINT      PRIMARY KEY (Snowflake)
  channel_id     UUID
  title          VARCHAR(500)
  description    TEXT
  status         ENUM(uploading, processing, ready, removed)
  duration_sec   INT
  raw_s3_key     VARCHAR
  hls_s3_prefix  VARCHAR
  view_count     BIGINT
  created_at     TIMESTAMP

VideoSegment (HLS manifest metadata)
  video_id       BIGINT
  quality        ENUM(240p, 360p, 480p, 720p, 1080p, 4k)
  duration_sec   FLOAT
  file_size_bytes BIGINT
  s3_key         VARCHAR
```

---

## Key Trade-offs

**HLS vs DASH**
HLS is Apple's format and has universal browser support through the video element. DASH is the open standard with slightly better compression and segment flexibility. Most platforms default to HLS for compatibility. DASH is preferred for DRM-heavy content.

**Push vs on-demand transcoding**
Transcode all quality levels immediately after upload (push/eager). Alternatively, transcode on-demand: only produce the qualities that users actually request. Push is simpler and ensures instant playback at all qualities. On-demand saves storage for long-tail videos no one watches. YouTube uses eager transcoding for popular videos and lazy for unpopular ones.

**Single vs multi-CDN**
Running two CDN providers (Cloudflare + Akamai, or a cloud CDN + a specialist) adds cost but provides failover if one CDN has an outage. At scale, this is worth it. Below 10M DAU, a single CDN is fine.

---

## Capacity Estimate

Using the ArchForge Capacity Calculator with:
- DAU: 2B
- Requests (stream initiations) per user per day: 5
- Average segment size: 2MB (at 2 Mbps for 8 seconds)
- Storage per user per day: 10MB (new uploads distributed across users)
- Retention: forever (videos are not deleted by default)

Results:
- Average RPS: 115,741 (stream initiations)
- Peak RPS: 347,222
- Daily raw video upload storage: ~190TB
- Peak streaming bandwidth: ~833 Tbps (handled by CDN, not your origin)
- Origin bandwidth (5% CDN miss): ~41 Tbps

The CDN absorbs the vast majority of bandwidth. Your origin only serves cache misses and fresh uploads.
