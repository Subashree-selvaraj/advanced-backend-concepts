# Day 02 – Cache Invalidation Strategies

## Overview

Today I learned how to maintain consistency between cache and database using cache invalidation strategies.

Caching improves performance, but incorrect invalidation leads to stale data and system inconsistencies.

---

## Problem

Cached data becomes outdated when database updates.

Example:

Cache:

```
product:101 → 90000
```

Database:

```
product:101 → 85000
```

Result → Wrong data served ❌

---

## What is Cache Invalidation?

Ensuring cache reflects latest database state.

---

## Strategies Learned

### 1. TTL (Time-Based Expiry)

Cache expires automatically after fixed time.

### 2. Write-Through

Update DB and cache together.

### 3. Write-Back

Update cache first, DB later.

### 4. Cache-Aside

Application manages cache manually.

### 5. Explicit Invalidation (Used Today)

Delete cache when data changes.

---

## Implementation

### Update Flow

1. Update database
2. Delete cache key
3. Next request rebuilds cache

---

## API Endpoints

### GET Product

```
GET /products/:id
```

### Update Product

```
PUT /products/:id
Body: { "price": 85000 }
```

---

## Key Learnings

* Cache invalidation is critical for consistency
* TTL alone is not enough
* Explicit invalidation is widely used
* Incorrect caching leads to serious bugs

---

## Real-World Use Cases

* E-commerce price updates
* User profile updates
* Inventory systems

---

## Challenges Faced

* Understanding stale data issues
* Managing cache manually
* Designing correct invalidation logic

---

## Next Step

Day 03 – Rate Limiting

---

## Key Insight

Caching is easy.
Maintaining correct cache is hard.
