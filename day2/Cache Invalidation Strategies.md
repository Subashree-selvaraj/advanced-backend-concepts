# Day 02 – Cache Invalidation Strategies 🚀

## Overview

Today I learned one of the most critical and challenging concepts in backend engineering — **Cache Invalidation**.

Caching improves performance, but without proper invalidation, systems can return **stale or incorrect data**, leading to serious bugs in production systems.

This concept is widely used in companies like Amazon and Netflix.

---

# 🔥 Problem Statement

Cache stores data temporarily to reduce database load.

But when database data changes:

```bash
Cache → OLD DATA ❌  
Database → NEW DATA ✅
```

This creates **stale data problem**.

---

# 🧠 What is Cache Invalidation?

Cache invalidation is the process of:

> Ensuring cached data stays consistent with the database.

---

# ⚠️ Issues Without Invalidation

* Users see outdated data
* System inconsistency
* Hard-to-debug bugs
* Wrong business logic execution

---

# 🧩 Strategies Learned

## 1. TTL (Time-To-Live)

Cache expires automatically after fixed time.

✔ Simple
❌ Not always accurate

---

## 2. Write-Through

Update DB and cache together.

✔ Strong consistency
❌ More complex writes

---

## 3. Write-Back (Write-Behind)

Update cache first, DB later.

✔ Fast writes
❌ Risk of data loss

---

## 4. Cache-Aside (Used in this project)

Flow:

* Read → Cache → DB if needed
* Write → DB → manually manage cache

✔ Most commonly used in industry

---

## 5. Explicit Invalidation (Main Focus)

Flow:

```bash
Update DB → Delete Cache → Next request rebuilds cache
```

✔ Safe
✔ Simple
✔ Widely used

---

# 🧠 Key Insight

### Why delete cache instead of updating it?

Updating cache requires two operations:

* Update DB
* Update Redis

If one fails → system becomes inconsistent ❌

Deleting cache:

* Only one critical operation
* Next request rebuilds correct data ✅

👉 Hence, **delete cache is safer**

---

# 🏗️ Project Implementation

## APIs Built

### GET Product

```bash
GET /products/:id
```

Flow:

* Check Redis
* If hit → return cached data
* If miss → fetch DB → store in Redis → return

---

### PUT Product (Cache Invalidation)

```bash
PUT /products/:id
```

Flow:

1. Validate input
2. Update database
3. Delete Redis cache
4. Rebuild cache (optimization)
5. Return updated data

---

# 💻 Code Implementation

## GET API (Cache-Aside Pattern)

```javascript
app.get("/products/:id", async (req, res) => {
  const id = req.params.id;
  const key = `product:${id}`;

  try {
    const cachedData = await client.get(key);

    if (cachedData) {
      console.log("Cache Hit");
      return res.json({
        source: "cache",
        data: JSON.parse(cachedData)
      });
    }

    console.log("Cache Miss");

    const product = mockDB[id];

    if (!product) {
      return res.status(404).json({ message: "Product not found" });
    }

    await client.setEx(key, 60, JSON.stringify(product));

    res.json({
      source: "db",
      data: product
    });

  } catch (err) {
    console.error(err);
    res.status(500).json({ message: "Server error" });
  }
});
```

---

## PUT API (Invalidation + Rebuild Optimization)

```javascript
app.put("/products/:id", async (req, res) => {
  try {
    const id = req.params.id;
    const { price } = req.body;

    if (price === undefined) {
      return res.status(400).json({ message: "Price is required" });
    }

    if (!mockDB[id]) {
      return res.status(404).json({ message: "Product not found" });
    }

    // Update DB
    mockDB[id].price = price;

    const key = `product:${id}`;

    // Invalidate cache
    await client.del(key);

    // 🔥 Rebuild cache (avoid cache stampede)
    await client.setEx(key, 60, JSON.stringify(mockDB[id]));

    console.log("Cache invalidated and refreshed");

    res.json({
      message: "Product updated successfully",
      data: mockDB[id]
    });

  } catch (err) {
    console.error(err);
    res.status(500).json({ message: "Server error" });
  }
});
```

---

# 🧠 Advanced Concept: Cache Stampede

### Problem:

After cache deletion:

```bash
Many users → hit API → all go to DB 😱
```

This overloads database.

---

### Solution:

Rebuild cache immediately after update.

```bash
Update DB → Delete Cache → Set New Cache
```

---

# 📊 Final System Flow

```bash
GET:
Client → Redis → (hit/miss) → DB → Redis → Response

PUT:
Client → DB Update → Cache Delete → Cache Rebuild → Response
```

---

# 🧠 Key Learnings

* Cache invalidation is critical for data consistency
* DB is always source of truth
* Cache is only a temporary layer
* Delete cache is safer than updating cache
* Cache stampede can crash systems
* Rebuilding cache improves stability
* Error handling is essential in backend systems

---

# ⚠️ Challenges Faced

* Understanding stale data issues
* Designing correct invalidation flow
* Writing safe backend APIs
* Handling edge cases (invalid input, missing product)

---

# 🚀 Next Step

## Day 03 – Rate Limiting

Learn how to:

* Prevent API abuse
* Control traffic
* Protect backend systems

---

# 💡 Final Thought

Caching is easy.
Invalidating cache correctly is what makes you a backend engineer.
