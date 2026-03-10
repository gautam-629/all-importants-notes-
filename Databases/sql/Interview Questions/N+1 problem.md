#### N+1 Problem – TypeORM (Node.js)
### 📌 What is N+1 Problem?
The N+1 problem occurs when an application makes:
- 1 query to fetch parent records
- N additional queries to fetch related child records
👉 This causes performance degradation.
---
## 🔎 Example Scenario
Entities:
- User → has many Posts
## 🐘 PostgreSQL SQL Example
Fetch users with posts using JOIN (Recommended).
```sql
SELECT users.id, users.name, posts.id AS post_id, posts.title
FROM users
LEFT JOIN posts ON posts.user_id = users.id;
```
✔ Retrieves all data in **one query**  
✔ Prevents N+1 problem
---
### ❌ N+1 Problem Example (Bad Practice – TypeORM)

```ts
const users = await userRepository.find();

for (const user of users) {
  console.log(await postRepository.find({ where: { user: { id: user.id } } }));
}
```
👉 This generates:
- 1 query to fetch users
- N queries to fetch posts
---
### ✅ Solution – Use Eager Loading (Recommended)
### Using `relations`
```ts
const users = await userRepository.find({
  relations: ['posts'],
});
```
✔ Fetches users and posts in **1 query (or optimized joins)**
---
### ✅ Better Solution – Query Builder (Best Control)

```ts
const users = await userRepository
  .createQueryBuilder('user')
  .leftJoinAndSelect('user.posts', 'post')
  .getMany();
```

✔ Prevents N+1 problem  
✔ More performance control
---
### 🧠 Summary
- Avoid fetching related data inside loops.
- Use JOINs, eager loading, or QueryBuilder.
- Monitor SQL queries in development.
---
### 🚀 Best Practice
✅ Use `leftJoinAndSelect` when possible  
✅ Avoid repository calls inside loops  
✅ Enable SQL logging while debugging

- [ ] **How do you optimize slow queries?**
### 🚀 How to Optimize Slow Queries
### 1️⃣ Identify the Problem
- Enable query logging
- Use `EXPLAIN ANALYZE
- Look for: Seq Scan, large row count, expensive joins
---
## 2️⃣ Add Proper Indexes
- Index columns used in:
    - `WHERE`
    - `JOIN`
    - `ORDER BY`
- Avoid over-indexing
---
## 3️⃣ Avoid `SELECT *`
- Fetch only required columns
- Reduces I/O and memory usage
---
## 4️⃣ Optimize JOINs
- Index foreign keys
- Use correct join type (INNER vs LEFT)
- Remove unnecessary joins
---
## 5️⃣ Fix N+1 Problems
- Use JOINs
- Use eager loading
- Avoid queries inside loop
---
## 6️⃣ Use Pagination
- Apply `LIMIT` and `OFFSET`
- Avoid loading large datasets at once
---
## 7️⃣ Use Caching
- Redis / in-memory cache
---
## 8️⃣ Schema & DB Tuning
- Proper data types
- Normalize correctly