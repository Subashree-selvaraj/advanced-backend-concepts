# Day 01 – Redis Distributed Caching

## Overview

Today I learned **Redis Distributed Caching**, one of the most important performance optimization techniques used in production backend systems.

Caching helps reduce database load, improve response speed, and scale backend systems efficiently.

Companies like Amazon, Netflix, and Uber rely heavily on Redis-based caching.

---

# Problem Statement

Without caching:

Every API request hits the database.

Example:

```sql
SELECT * FROM products WHERE id = 101;
```

If 50,000 users request the same product:

* 50,000 repeated DB queries
* Increased database load
* Slower system performance

Database becomes bottleneck.

---

# Solution: Redis Caching

Instead of querying DB every time:

```bash
Request → Redis Cache → Database only if needed
```

Redis stores frequently accessed data in memory (RAM), making retrieval extremely fast.

---

# What is Redis?

Redis is an in-memory key-value store.

Example:

```bash
Key: product:101
Value: {"id":101,"name":"iPhone","price":90000}
```

Redis is:

* Fast
* Lightweight
* Ideal for caching
* Shared across multiple servers

---

# Why Distributed Cache?

In real production:

```bash
Users
 ↓
Load Balancer
 ↓ ↓ ↓
API1 API2 API3
   ↓
 Shared Redis Cache
   ↓
 Database
```

All backend servers use same Redis cache.

This makes caching centralized and consistent.

---

# Cache Hit vs Cache Miss

## Cache Hit

Data exists in Redis:

```bash
Request → Redis Found → Return Fast
```

No DB query needed.

---

## Cache Miss

Data not in Redis:

```bash
Request → Redis Empty → Query DB → Save in Redis → Return Response
```

---

# TTL (Time To Live)

TTL defines cache expiry time.

Example:

```bash
Expires in 60 seconds
```

Why TTL matters:

* Prevents stale data
* Keeps cache fresh
* Controls memory usage

---

# Cache-Aside Pattern

Industry-standard caching pattern:

1. Check Redis
2. If missing → fetch DB
3. Store in Redis
4. Return response

This is the pattern implemented today.

---

# Project: Cached Product API

### Objective:

Build API that:

* Checks Redis first
* If cached → return fast
* Else fetch mock DB
* Save in Redis
* Return response

---

# Project Structure

```bash
day-01-redis-distributed-caching/
│
├── app.js
├── package.json
├── README.md
└── screenshots/
```

---

# Installation

## Step 1: Initialize project

```bash
npm init -y
```

## Step 2: Install dependencies

```bash
npm install express redis
```

## Step 3: Start Redis server

```bash
redis-server
```

## Step 4: Run app

```bash
node app.js
```

---

# Source Code (app.js)

```javascript
const express = require("express");
const redis = require("redis");

const app = express();
const client = redis.createClient();

client.on("error", (err) => console.log("Redis Client Error", err));

(async () => {
  await client.connect();
})();

const mockDB = {
  101: { id: 101, name: "iPhone", price: 90000 },
  102: { id: 102, name: "Laptop", price: 65000 }
};

app.get("/products/:id", async (req, res) => {
  const id = req.params.id;
  const key = `product:${id}`;

  try {
    // Check Redis cache
    const cachedData = await client.get(key);

    if (cachedData) {
      console.log("Cache Hit");
      return res.json({
        source: "Redis Cache",
        data: JSON.parse(cachedData)
      });
    }

    console.log("Cache Miss");

    // Fetch from mock DB
    const product = mockDB[id];

    if (!product) {
      return res.status(404).json({ message: "Product not found" });
    }

    // Store in Redis with TTL 60 sec
    await client.setEx(key, 60, JSON.stringify(product));

    res.json({
      source: "Database",
      data: product
    });

  } catch (error) {
    console.error(error);
    res.status(500).json({ message: "Server Error" });
  }
});

app.listen(3000, () => {
  console.log("Server running on port 3000");
});
```

---

# Example API Test

### First Request:

```bash
GET /products/101
```

Output:

```json
{
  "source": "Database",
  "data": {
    "id": 101,
    "name": "iPhone",
    "price": 90000
  }
}
```

(Cache Miss)

---

### Second Request:

```bash
GET /products/101
```

Output:

```json
{
  "source": "Redis Cache",
  "data": {
    "id": 101,
    "name": "iPhone",
    "price": 90000
  }
}
```

(Cache Hit)

---

# Real-World Production Use Cases

Redis caching is used for:

* Product catalog caching
* Session storage
* API response caching
* Rate limiting counters
* Leaderboards
* Real-time analytics

---

# Key Learnings Today

### I understood:

* Why caching is essential
* Difference between cache hit and miss
* How Redis reduces DB load
* TTL expiration importance
* Distributed cache architecture

---

# Interview Question

### Q: Why not store everything in cache forever?

Because:

* Memory is limited
* Data becomes stale
* Cache invalidation becomes difficult

---

# Challenges Faced

* Understanding Redis client connection flow
* Learning async Redis operations
* Handling cache miss logic correctly

---

# Next Improvement Tasks

Tomorrow I will enhance this by:

* Adding cache invalidation
* Updating stale cache automatically
* Implementing smarter expiry strategies

---

# Tomorrow Topic:

## Day 02 – Cache Invalidation Strategies

“Cache invalidation is one of the hardest problems in computer science.”
