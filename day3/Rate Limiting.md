# Day 03 – Rate Limiting 🚀

## Overview

Today I learned how to protect backend systems from abuse using **Rate Limiting**.

Rate limiting restricts how many requests a user/client can make within a fixed time window. It is a critical mechanism used in production systems to prevent:

* API abuse
* Brute force attacks
* Server overload

This concept is widely used in systems built by Amazon, Netflix, and almost every scalable backend API.

---

# 🔥 Problem Statement

Without rate limiting:

```bash
Attacker → 1000s of requests/sec → Server crash 😱
```

Issues:

* High CPU usage
* Database overload
* Service downtime
* Poor experience for real users

---

# 🧠 What is Rate Limiting?

> Rate limiting controls the number of requests allowed per user/IP within a defined time window.

---

# 🧩 Core Concepts

### 1. Identifier

Used to track user:

* IP address (used in this project)
* User ID
* API key

---

### 2. Limit

Maximum number of requests allowed.

Example:

```bash
5 requests / 10 seconds
```

---

### 3. Time Window

Defines duration of restriction.

Example:

* 10 seconds
* 1 minute
* 1 hour

---

# ⚡ Why Redis is Used?

Redis is used because:

* Extremely fast (in-memory)
* Supports counters (`INCR`)
* Supports expiration (`TTL`)
* Perfect for request tracking

---

# 🧠 Implementation Approach

### Algorithm Used:

## Fixed Window Counter

---

# 🔁 Flow

```bash
Request comes
 ↓
Check Redis count
 ↓
If count >= limit → block request
Else → increment count
 ↓
Set TTL (if first request)
 ↓
Allow request
```

---

# 💻 Middleware Implementation

```javascript
const rateLimiter = async (req, res, next) => {
  try {
    const ip = req.ip;
    const key = `rate:${ip}`;

    const limit = 5;     // max requests
    const window = 10;   // seconds

    let count = await client.get(key);

    // Block if limit exceeded
    if (count && parseInt(count) >= limit) {
      console.log("Rate limit exceeded");
      return res.status(429).json({
        message: "Too many requests, try again later"
      });
    }

    // First request → set with TTL
    if (!count) {
      await client.setEx(key, window, 1);
    } else {
      // Increment count
      await client.incr(key);
    }

    next();

  } catch (err) {
    console.error("Rate limiter error:", err);

    // Fail-open strategy (allow request if Redis fails)
    next();
  }
};
```

---

# 🔌 Applying Middleware

```javascript
app.get("/products/:id", rateLimiter, async (req, res) => {
  res.send("Product data");
});
```

---

# 🧪 Testing

### Allowed Requests (within limit)

```bash
GET /products/101
```

---

### After Limit Exceeded

```json
{
  "message": "Too many requests, try again later"
}
```

Status Code:

```bash
429 Too Many Requests
```

---

# 🧠 Key Concepts Learned

* Rate limiting protects backend systems from overload
* Redis is ideal for tracking request counts
* TTL automatically resets the time window
* Middleware pattern is used to apply globally
* Fail-open strategy ensures system resilience

---

# ⚠️ Limitations of Fixed Window

Problem:

```bash
User sends 5 requests at end of window + 5 at start of next window
→ 10 requests in short time ❌
```

---

# 🚀 Advanced Algorithms (Interview Topics)

1. Fixed Window (implemented today)
2. Sliding Window
3. Token Bucket ⭐ (most important)
4. Leaky Bucket

---

# 🧠 Real-World Use Cases

* Login attempt limiting
* API usage control
* Payment request throttling
* Preventing bot attacks

---

# ⚠️ Challenges Faced

* Understanding how Redis counters work
* Handling TTL correctly
* Designing middleware logic
* Managing edge cases (limit overflow)

---

# 🚀 Next Step

## Day 04 – API Gateway Pattern

Learn how large systems:

* Route traffic
* Handle authentication centrally
* Manage multiple services

---

# 💡 Final Thought

Without rate limiting, your API is exposed.
With rate limiting, your system becomes resilient.
