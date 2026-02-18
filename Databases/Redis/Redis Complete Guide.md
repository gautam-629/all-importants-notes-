
## 1. What is Redis?

**Redis** stands for **REmote DIctionary Server**. It is:

- An **in-memory database** (data is stored in RAM)
- **Extremely fast**
- Often used as:
  - Cache
  - Session store
  - Message broker
  - Real-time data store

> 👉 **Think of Redis as a super-fast key–value store.**

---

## 2. How Redis is Different from SQL Databases

| Feature        | Redis                    | MySQL / PostgreSQL  |
|----------------|--------------------------|---------------------|
| **Storage**    | In-memory (RAM)          | Disk                |
| **Speed**      | Very fast (microseconds) | Slower              |
| **Schema**     | Schema-less              | Fixed schema        |
| **Query Language** | Commands             | SQL                 |
| **Use case**   | Cache, real-time         | Persistent data     |

**Redis does not replace SQL databases — it complements them.**

---

## 3. Redis Data Types (Very Important)

Redis is powerful because it supports multiple data types, not just strings.

### 1️⃣ String
```redis
SET name "Alice"
GET name
```

**Used for:**
- Caching values
- Tokens
- Counters

---

### 2️⃣ List (Ordered collection)
```redis
LPUSH tasks "task1"
LPUSH tasks "task2"
LRANGE tasks 0 -1
```

**Used for:**
- Queues
- Job processing

---

### 3️⃣ Set (Unique values, unordered)
```redis
SADD users "alice"
SADD users "bob"
SMEMBERS users
```

**Used for:**
- Unique items
- Tags
- Permissions

---

### 4️⃣ Sorted Set (Set + score)
```redis
ZADD leaderboard 100 "Alice"
ZADD leaderboard 200 "Bob"
ZRANGE leaderboard 0 -1 WITHSCORES
```

**Used for:**
- Leaderboards
- Rankings
- Priority queues

---

### 5️⃣ Hash (Like a JSON object)
```redis
HSET user:1 name "Alice" age 25
HGETALL user:1
```

**Used for:**
- User profiles
- Objects

---

## 4. Why Redis Is So Fast ⚡

- Data stored in **RAM**
- **Single-threaded** (no locking)
- Optimized data structures
- Minimal disk I/O

---

## 5. Persistence (Important!)

Even though Redis is in-memory, it can save data to disk:

### Options:

- **RDB** – Snapshotting
- **AOF** – Append-only file
- **RDB + AOF** – Best reliability

**So Redis won't lose data if configured properly.**

---

## 6. Common Redis Use Cases

### ✅ Caching
```redis
SET user:1 "Alice" EX 60  # Cache for 60 seconds
```

### ✅ Sessions

- Store login sessions
- Faster than database

### ✅ Rate Limiting

- API request limits

### ✅ Pub/Sub (Messaging)
```redis
SUBSCRIBE notifications
PUBLISH notifications "Hello!"
```

### ✅ Real-time Analytics

- Counters
- Live dashboards

---

## 7. When NOT to Use Redis

❌ Large datasets that don't fit in RAM  
❌ Complex joins & reporting  
❌ Strong ACID transactions only

---

## 8. Redis Architecture (Simple)
```
App → Redis → Database
       ↑
     Cache
```

**Redis sits between your app and database.**

---

## 9. How Redis Is Used in Real Projects

### Example:

1. User logs in
2. Session stored in Redis
3. Database used only when needed
4. **Result:** Faster response & lower DB load

---
