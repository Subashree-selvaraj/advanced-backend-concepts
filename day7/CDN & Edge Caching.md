# Day 07 – CDN & Edge Caching (Production Deep Dive 🚀)

## Overview

Today I explored how modern products deliver static and media content globally with low latency using **CDNs** and **edge caching**.

A CDN is not just a performance add-on. In production, it is a **critical traffic-distribution layer** that protects origin systems, improves user experience, and helps applications scale to global demand.

---

## Why CDN Is Required in Real Systems

If your backend is hosted in India and users access your app from North America or Europe, every uncached static request has to cross continents.

Without CDN:

```text
User (New York)
   ↓
Internet backbone (long-distance routing)
   ↓
Origin (Bangalore)
   ↓
Response back to user
```

This introduces:

- High round-trip latency
- Slower page loads
- Higher origin bandwidth and compute load
- Poor experience during traffic spikes

---

## What Is a CDN?

A **Content Delivery Network (CDN)** is a globally distributed network of edge servers that caches and serves content from locations closer to users.

With CDN:

```text
User
 ↓
Nearest Edge POP (Point of Presence)
 ↓
Response
```

If the content is cached at the edge, the origin is skipped completely.

---

## Where CDN Sits in Production Architecture

```text
Client
  ↓
CDN / Edge Network
  ↓
WAF / DDoS Protection (often integrated)
  ↓
Load Balancer
  ↓
NGINX / API Gateway
  ↓
Backend Services
  ↓
Cache + Database
```

### Practical insight

In many systems, **most static traffic is terminated at CDN**, so only dynamic/API traffic reaches origin layers.

---

## What Should Be Cached at Edge

### Ideal candidates

- Images (`.jpg`, `.png`, `.webp`, `.svg`)
- CSS/JavaScript bundles
- Fonts
- Public PDFs and documents
- Public video segments/chunks
- Versioned static assets from build pipelines

### Avoid caching blindly

- Personalized dashboard responses
- Sensitive user-specific API responses
- Non-idempotent endpoints

---

## Edge Caching Explained

Edge caching means storing content in servers geographically close to users.

Example:

```text
User (Germany)
   ↓
Frankfurt Edge
   ↓
Response
```

instead of:

```text
User (Germany)
   ↓
Origin (India)
   ↓
Response
```

### Why this matters

- Lower latency (fewer network hops)
- Better Time To First Byte (TTFB)
- Improved Core Web Vitals
- Better resilience during regional network congestion

---

## Cache Hit vs Cache Miss (Operational View)

### Cache Hit

Requested object is already present at edge.

```text
Client → Edge Cache → Response
```

Result:

- Fastest response path
- Zero origin compute for that request
- Lower egress cost from origin

### Cache Miss

Object not present at edge.

```text
Client → Edge
         ↓
       Origin fetch
         ↓
       Store at edge
         ↓
       Response
```

Result:

- First user pays origin latency
- Next users in that region usually get cache hits

---

## Critical HTTP Headers for CDN Behavior

CDNs respect HTTP caching directives from origin responses.

### 1) `Cache-Control`

```http
Cache-Control: public, max-age=3600
```

Meaning: edge/browser can cache for 1 hour.

Useful directives:

- `public` — cacheable by shared caches
- `private` — browser cache only (not shared CDN)
- `max-age` — freshness lifetime
- `s-maxage` — shared cache lifetime (CDN-focused)
- `stale-while-revalidate` — serve stale while refreshing in background
- `immutable` — resource will not change during freshness window

### 2) `ETag` / `Last-Modified`

Used for conditional revalidation to reduce payload transfer.

---

## Cache Invalidation Strategies

## Problem

`logo.png` is updated at origin, but edge still serves old version.

### Strategy A: Versioned filenames (best practice)

```text
app.9f3a1.js
styles.71c2d.css
```

When content changes, filename changes. Old cache naturally becomes irrelevant.

### Strategy B: Explicit purge/invalidation

Examples:

```text
Purge /logo.png
Purge /assets/*
```

Use this for urgent updates when URL cannot be changed.

### Strategy C: Short TTL for frequently changing assets

Use lower `max-age` for content that changes often.

---

## Real-World Architecture Patterns

### Frontend SPA delivery

```text
React/Vue Build
   ↓
Object Storage (S3/GCS/Azure Blob)
   ↓
CDN (CloudFront/Cloudflare/Akamai/Fastly)
   ↓
Global Users
```

### Media-heavy e-commerce

```text
Product image upload
   ↓
Object storage + image processing
   ↓
CDN edge cache
   ↓
Users across regions
```

### Streaming platforms

```text
Origin media storage
   ↓
CDN edge nodes (segment caching)
   ↓
Adaptive playback clients
```

---

## CDN + Backend Protection Benefits

CDN adoption improves more than speed:

- Offloads origin from repetitive static requests
- Absorbs traffic spikes and flash sales better
- Reduces infrastructure cost by lowering origin bandwidth/CPU
- Improves availability with multi-region edge footprint
- Adds security layers (WAF, bot mitigation, DDoS filtering)

---

## Metrics to Track After CDN Rollout

- Cache hit ratio
- Origin request rate
- TTFB by geography
- 95th/99th percentile latency
- Origin egress bandwidth
- Error rate by edge region

A good rollout shows **higher cache hit ratio + lower origin load + lower latency**.

---

## Common Mistakes in Production

- Caching dynamic personalized responses unintentionally
- Missing cache headers for static assets
- Using same filename for changed assets without purge
- Setting very long TTL for frequently changing files
- Ignoring regional performance metrics after launch

---

## Hands-On Practice (Suggested)

### Goal

Deploy a frontend behind CDN and observe performance behavior.

### Setup

```text
Frontend Build → S3 → CloudFront → Users
```

### Validate

- Compare first load vs repeat load time
- Check CDN response headers (`Age`, `X-Cache`, provider-specific headers)
- Test from multiple geographic regions

---

## Interview-Focused Answers

### What is a CDN?

A globally distributed edge network that caches and serves content from locations closest to users, reducing latency and origin load.

### What is edge caching?

Storing content at geographically distributed edge locations so users fetch data from nearby servers instead of distant origin systems.

### Cache vs CDN?

- **Cache**: temporary storage mechanism
- **CDN**: global distributed system that applies caching at scale across edge locations

### Why do companies use CDNs?

For lower latency, better scalability, reduced origin cost/load, and stronger edge security posture.

---

## Key Learnings

- CDN is a core architecture layer for global-scale delivery
- Edge cache hits provide the biggest latency and cost gains
- Correct cache-control strategy is essential for performance and freshness
- Versioned assets are the safest invalidation pattern
- CDN decisions must be measured with real production metrics

---

## Final Reflection

Modern applications should not serve static assets directly from backend servers at global scale.

A strong production pattern is:

```text
Internet → CDN Edge → Origin Stack
```

Mastering CDN behavior, cache policies, and invalidation strategy is essential for designing high-performance and scalable backend systems.
