
> A complete guide using `users`, `posts`, `comments`, and `likes` tables
---
## 📌 Table of Contents

1. [What is a JOIN?](https://claude.ai/chat/d74e8d3d-673f-41e3-9619-3ee9c04c30f0#what-is-a-join)
2. [Default JOIN — INNER JOIN](https://claude.ai/chat/d74e8d3d-673f-41e3-9619-3ee9c04c30f0#default-join--inner-join)
3. [INNER JOIN](https://claude.ai/chat/d74e8d3d-673f-41e3-9619-3ee9c04c30f0#-inner-join)
4. [LEFT JOIN](https://claude.ai/chat/d74e8d3d-673f-41e3-9619-3ee9c04c30f0#-left-join)
5. [RIGHT JOIN](https://claude.ai/chat/d74e8d3d-673f-41e3-9619-3ee9c04c30f0#-right-join)
6. [FULL JOIN](https://claude.ai/chat/d74e8d3d-673f-41e3-9619-3ee9c04c30f0#-full-join)
7. [Table Relationships](https://claude.ai/chat/d74e8d3d-673f-41e3-9619-3ee9c04c30f0#-table-relationships)
    - [One-to-One](https://claude.ai/chat/d74e8d3d-673f-41e3-9619-3ee9c04c30f0#one-to-one)
    - [One-to-Many](https://claude.ai/chat/d74e8d3d-673f-41e3-9619-3ee9c04c30f0#one-to-many)
    - [Many-to-Many](https://claude.ai/chat/d74e8d3d-673f-41e3-9619-3ee9c04c30f0#many-to-many)
8. [The Need for a Junction Table](https://claude.ai/chat/d74e8d3d-673f-41e3-9619-3ee9c04c30f0#-the-need-for-a-junction-table)
9. Design leet Code Database

---

## 🧩 Our Schema

```sql
CREATE TABLE users (
    id   INT PRIMARY KEY,
    name VARCHAR(100)
);

CREATE TABLE posts (
    id      INT PRIMARY KEY,
    user_id INT,
    content TEXT,
    FOREIGN KEY (user_id) REFERENCES users(id)
);

CREATE TABLE comments (
    id      INT PRIMARY KEY,
    user_id INT,
    post_id INT,
    comment TEXT,
    FOREIGN KEY (user_id) REFERENCES users(id),
    FOREIGN KEY (post_id) REFERENCES posts(id)
);

CREATE TABLE likes (
    user_id      INT,
    likeable_id  INT,
    likeable_type ENUM('post', 'comment')
);
```

---

## What is a JOIN?

A **JOIN** is a SQL clause used to **combine rows from two or more tables** based on a related column between them.

> Without JOINs, you can only query one table at a time. JOINs let you _relate_ data spread across multiple tables.
### Why do we need JOINs?

- Our `posts` table stores `user_id` but not the user's name.
- To get both the **post content** AND the **author's name**, we need to JOIN `posts` with `users`.

---
## Default JOIN — INNER JOIN

> ⚠️ **When you write `JOIN` without any keyword, SQL defaults to `INNER JOIN`.**

```sql
-- These two queries are IDENTICAL:
SELECT users.name, posts.content
FROM users
JOIN posts ON users.id = posts.user_id;

SELECT users.name, posts.content
FROM users
INNER JOIN posts ON users.id = posts.user_id;
```

---
## 🔵 INNER JOIN

### Definition
Returns **only the rows where there is a match** in BOTH tables.
### Figure

```
   users                posts
┌────┬───────┐      ┌────┬─────────┬──────────────────┐
│ id │ name  │      │ id │ user_id │ content          │
├────┼───────┤      ├────┼─────────┼──────────────────┤
│  1 │ Alice │      │ 1  │    1    │ "Hello World"    │
│  2 │ Bob   │      │ 2  │    1    │ "SQL is fun"     │
│  3 │ Carol │      │ 3  │    2    │ "I love joins"   │
└────┴───────┘      └────┴─────────┴──────────────────┘
         ↑                    ↑
         └────── MATCH ───────┘
         Only rows where users.id = posts.user_id

         ✅ Alice  → "Hello World", "SQL is fun"
         ✅ Bob    → "I love joins"
         ❌ Carol  → No posts → EXCLUDED
```
### Venn Diagram

```
     users         posts
   ┌───────────────────┐
   │       │███│       │
   │       │███│       │
   └───────────────────┘
            ↑
       Only matching
         rows here
```
### Example Query

```sql
-- Get all users who have made posts, with post content
SELECT
    users.name   AS author,
    posts.content AS post_content
FROM users
INNER JOIN posts ON users.id = posts.user_id;
```
### Result

|author|post_content|
|---|---|
|Alice|Hello World|
|Alice|SQL is fun|
|Bob|I love joins|

> Carol is **not included** because she has no posts.

---
## 🟡 LEFT JOIN

### Definition

Returns **all rows from the LEFT table**, and matching rows from the right table. If there's no match, the right side columns are `NULL`.
### Figure

```
   users (LEFT)          posts (RIGHT)
┌────┬───────┐         ┌────┬─────────┬──────────────┐
│ id │ name  │         │ id │ user_id │ content      │
├────┼───────┤         ├────┼─────────┼──────────────┤
│  1 │ Alice │ ──────► │ 1  │    1    │ "Hello World"│
│  2 │ Bob   │ ──────► │ 3  │    2    │ "I love joins│
│  3 │ Carol │ ──────► │    │  NULL   │    NULL      │
└────┴───────┘         └────┴─────────┴──────────────┘

✅ Alice  → matched
✅ Bob    → matched
✅ Carol  → included with NULL (no post)
```
### Venn Diagram

```
     users         posts
   ┌───────────────────┐
   │███████│███│       │
   │███████│███│       │
   └───────────────────┘
       ↑
  ALL of left
  + matching right
```

### Example Query

```sql
-- Get ALL users and their posts (even users with no posts)
SELECT
    users.name        AS author,
    posts.content     AS post_content
FROM users
LEFT JOIN posts ON users.id = posts.user_id;
```
### Result

|author|post_content|
|---|---|
|Alice|Hello World|
|Alice|SQL is fun|
|Bob|I love joins|
|Carol|NULL|

> Carol appears with `NULL` — she exists in `users` but has no posts.

---
## 🟠 RIGHT JOIN

### Definition

Returns **all rows from the RIGHT table**, and matching rows from the left table. If there's no match, the left side columns are `NULL`.

### Figure

```
   users (LEFT)          posts (RIGHT)
┌────┬───────┐         ┌────┬─────────┬──────────────┐
│ id │ name  │         │ id │ user_id │ content      │
├────┼───────┤         ├────┼─────────┼──────────────┤
│  1 │ Alice │ ◄──────  │ 1  │    1    │ "Hello World"│
│  2 │ Bob   │ ◄──────  │ 3  │    2    │ "I love joins│
│  NULL│NULL │ ◄──────  │ 4  │   99   │ "Orphan post"│
└────┴───────┘         └────┴─────────┴──────────────┘

✅ Post 1 → matched with Alice
✅ Post 3 → matched with Bob
✅ Post 4 → included with NULL (user 99 doesn't exist)
```

### Venn Diagram

```
     users         posts
   ┌───────────────────┐
   │       │███│███████│
   │       │███│███████│
   └───────────────────┘
                ↑
          ALL of right
          + matching left
```

### Example Query

```sql
-- Get ALL posts and their authors (even posts with no matching user)
SELECT
    users.name        AS author,
    posts.content     AS post_content
FROM users
RIGHT JOIN posts ON users.id = posts.user_id;
```

### Result

|author|post_content|
|---|---|
|Alice|Hello World|
|Alice|SQL is fun|
|Bob|I love joins|
|NULL|Orphan post|

> "Orphan post" appears with `NULL` author — post has no matching user.

---

## 🔴 FULL JOIN (FULL OUTER JOIN)

### Definition

Returns **ALL rows from BOTH tables**. Where there's no match on either side, `NULL` fills the gaps.

### Figure

```
   users (LEFT)          posts (RIGHT)
┌────┬───────┐         ┌────┬─────────┬──────────────┐
│  1 │ Alice │ ◄──────► │ 1  │    1   │ "Hello World"│
│  2 │ Bob   │ ◄──────► │ 3  │    2   │ "I love joins│
│  3 │ Carol │ ──────── │    │  NULL  │    NULL      │  ← no post
│ NULL│ NULL │ ◄──────  │ 4  │   99   │ "Orphan post"│  ← no user
└────┴───────┘         └────┴─────────┴──────────────┘

✅ Alice  → matched
✅ Bob    → matched
✅ Carol  → left-only (NULL on right)
✅ Post 4 → right-only (NULL on left)
```

### Venn Diagram

```
     users         posts
   ┌───────────────────┐
   │███████│███│███████│
   │███████│███│███████│
   └───────────────────┘
       ↑           ↑
  Everything from both sides
```

### Example Query

```sql
-- Get ALL users and ALL posts, matched where possible
SELECT
    users.name        AS author,
    posts.content     AS post_content
FROM users
FULL OUTER JOIN posts ON users.id = posts.user_id;
```

### Result

|author|post_content|
|---|---|
|Alice|Hello World|
|Alice|SQL is fun|
|Bob|I love joins|
|Carol|NULL|
|NULL|Orphan post|

---

## 📊 JOIN Summary Comparison

```
┌─────────────────┬──────────────────────────────────────────┐
│   JOIN Type     │          Rows Returned                   │
├─────────────────┼──────────────────────────────────────────┤
│  INNER JOIN     │ Only matched rows from BOTH tables       │
│  LEFT JOIN      │ ALL from left + matched from right       │
│  RIGHT JOIN     │ ALL from right + matched from left       │
│  FULL JOIN      │ ALL rows from BOTH tables                │
└─────────────────┴──────────────────────────────────────────┘
```

---

## 🔗 Table Relationships

Relationships describe **how tables relate to each other** in terms of data cardinality.

---

### One-to-One

> One record in Table A corresponds to **exactly one** record in Table B.

**Example:** One user has one profile.

```
users                    user_profiles
┌────┬───────┐           ┌────┬─────────┬──────────┐
│ id │ name  │           │ id │ user_id │ bio      │
├────┼───────┤           ├────┼─────────┼──────────┤
│  1 │ Alice │ ────────► │  1 │    1    │ "Dev..." │
│  2 │ Bob   │ ────────► │  2 │    2    │ "Des..." │
└────┴───────┘           └────┴─────────┴──────────┘
   1 user  ────────────────► 1 profile
```

**Cardinality Notation:** `1 ──── 1`

---

### One-to-Many

> One record in Table A corresponds to **many** records in Table B.

**Example:** One user can write many posts.

```
users                    posts
┌────┬───────┐           ┌────┬─────────┬───────────────┐
│ id │ name  │           │ id │ user_id │ content       │
├────┼───────┤           ├────┼─────────┼───────────────┤
│  1 │ Alice │ ────┬───► │  1 │    1    │ "Hello World" │
│    │       │     └───► │  2 │    1    │ "SQL is fun"  │
│  2 │ Bob   │ ────────► │  3 │    2    │ "I love joins"│
└────┴───────┘           └────┴─────────┴───────────────┘
  1 user  ────────────────► many posts
```

**Cardinality Notation:** `1 ──── ∞`

**In our schema:**

- `users` → `posts` (one user, many posts)
- `users` → `comments` (one user, many comments)
- `posts` → `comments` (one post, many comments)

```sql
-- Example: Get all comments on a specific post with author names
SELECT
    users.name    AS commenter,
    comments.comment,
    posts.content AS on_post
FROM comments
INNER JOIN users ON comments.user_id = users.id
INNER JOIN posts ON comments.post_id = posts.id
WHERE posts.id = 1;
```

---

### Many-to-Many

> Many records in Table A correspond to **many** records in Table B.

**Example:** Many users can like many posts/comments.

```
users                              posts
┌────┬───────┐                ┌────┬──────────────┐
│ id │ name  │                │ id │ content      │
├────┼───────┤                ├────┼──────────────┤
│  1 │ Alice │──┐          ┌──│ 1  │ "Hello World"│
│  2 │ Bob   │──┼──────────┼──│ 2  │ "SQL is fun" │
│  3 │ Carol │──┘          └──│ 3  │ "I love joins│
└────┴───────┘                └────┴──────────────┘
    ↑  multiple users can like multiple posts  ↑
```

**Cardinality Notation:** `∞ ──── ∞`

---

## 🔀 The Need for a Junction Table

### Problem: You Can't Store Many-to-Many Directly

If you tried to store likes in `users` or `posts` directly:

```
-- ❌ WRONG approach — impossible to represent cleanly
users
┌────┬───────┬──────────────────────────────┐
│ id │ name  │ liked_post_ids               │
├────┼───────┼──────────────────────────────┤
│  1 │ Alice │ 1, 2, 3  ← NOT valid SQL!    │
└────┴───────┴──────────────────────────────┘
```

> SQL tables cannot store arrays of values in a single cell cleanly — this violates **First Normal Form (1NF)**.

### Solution: A Junction (Bridge) Table

A **junction table** (also called a pivot or bridge table) sits **between** the two tables and holds the relationship.

**Our `likes` table IS the junction table:**

```sql
CREATE TABLE likes (
    user_id       INT,
    likeable_id   INT,
    likeable_type ENUM('post', 'comment')  -- polymorphic!
);
```

### How It Works

```
users              likes                   posts / comments
┌────┬───────┐   ┌─────────┬────────────┬───────────────┐
│ id │ name  │   │ user_id │likeable_id │likeable_type  │
├────┼───────┤   ├─────────┼────────────┼───────────────┤
│  1 │ Alice │◄──│    1    │     1      │    post       │
│  1 │ Alice │◄──│    1    │     2      │    post       │
│  2 │ Bob   │◄──│    2    │     1      │    post       │
│  2 │ Bob   │◄──│    2    │     5      │   comment     │
│  3 │ Carol │◄──│    3    │     5      │   comment     │
└────┴───────┘   └─────────┴────────────┴───────────────┘
                         ↑
              Junction Table — resolves
              the many-to-many into
              two one-to-many relationships
```

```
users ──────1───────► likes ◄───────1────── posts
            ∞                  ∞
```

### Why is the Junction Table Necessary?

|Reason|Explanation|
|---|---|
|**Avoids data duplication**|No need to repeat user or post data|
|**Maintains referential integrity**|FK constraints ensure no orphan records|
|**Enables queries in both directions**|"Who liked this post?" and "What did Alice like?"|
|**Scalability**|Millions of likes don't bloat the users or posts table|
|**Flexibility**|`likeable_type` makes it polymorphic — works for both posts AND comments|

### Example Queries Using the Junction Table

```sql
-- Who liked post #1?
SELECT users.name
FROM likes
INNER JOIN users ON likes.user_id = users.id
WHERE likes.likeable_id = 1
  AND likes.likeable_type = 'post';

-- What has Alice (user_id = 1) liked?
SELECT likeable_id, likeable_type
FROM likes
WHERE user_id = 1;

-- How many likes does each post have?
SELECT
    posts.content,
    COUNT(likes.user_id) AS total_likes
FROM posts
LEFT JOIN likes
    ON likes.likeable_id = posts.id
    AND likes.likeable_type = 'post'
GROUP BY posts.id, posts.content;
```

---

## 🗺️ Full Schema Relationship Map

```
┌──────────────────────────────────────────────────────────────────┐
│                         SCHEMA OVERVIEW                          │
│                                                                  │
│   ┌──────────┐    1        ∞   ┌──────────┐                     │
│   │  users   │───────────────► │  posts   │                     │
│   │──────────│                 │──────────│                     │
│   │ id  (PK) │                 │ id  (PK) │                     │
│   │ name     │    1        ∞   │ user_id  │◄── FK               │
│   └──────────┘───────────────► │ content  │                     │
│         │                      └──────────┘                     │
│         │  1                        │  1                        │
│         │                           │                           │
│         ▼  ∞                        ▼  ∞                        │
│   ┌──────────┐                 ┌──────────┐                     │
│   │ comments │◄────────────────│ comments │                     │
│   │──────────│  post_id FK     │──────────│                     │
│   │ id  (PK) │                 │ (same    │                     │
│   │ user_id  │◄── FK           │  table)  │                     │
│   │ post_id  │◄── FK           └──────────┘                     │
│   │ comment  │                                                   │
│   └──────────┘                                                   │
│         │                                                        │
│         │  ∞      JUNCTION      ∞                               │
│         └──────► ┌──────────┐ ◄──────────────────────┐          │
│                  │  likes   │                         │          │
│   users ────────►│──────────│◄──── posts / comments   │          │
│                  │ user_id  │                                    │
│                  │ like_id  │                                    │
│                  │ like_type│                                    │
│                  └──────────┘                                    │
└──────────────────────────────────────────────────────────────────┘
```

---

## 🎯 Quick Reference Cheatsheet

```
┌──────────────┬──────────────┬───────────────────────────────────┐
│ Relationship │   Tables     │ How to Identify                   │
├──────────────┼──────────────┼───────────────────────────────────┤
│ One-to-One   │ users ─ profile │ FK in child, UNIQUE constraint │
│ One-to-Many  │ users ─ posts   │ FK in "many" table             │
│ Many-to-Many │ users ─ posts   │ Junction table with 2 FKs      │
└──────────────┴──────────────┴───────────────────────────────────┘

┌──────────────┬────────────────────────────────────────────────┐
│  JOIN Type   │ Use When...                                    │
├──────────────┼────────────────────────────────────────────────┤
│ INNER JOIN   │ You only want records that exist in both tables│
│ LEFT JOIN    │ You want ALL left records, matches optional    │
│ RIGHT JOIN   │ You want ALL right records, matches optional   │
│ FULL JOIN    │ You want everything, matched or not            │
└──────────────┴────────────────────────────────────────────────┘
```

---
## Design Leet Code Database
```sql
CREATE TABLE users (
    id INT PRIMARY KEY AUTO_INCREMENT,
    username VARCHAR(100) UNIQUE,
    email VARCHAR(150) UNIQUE,
    password_hash TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
CREATE TABLE problems (
    id INT PRIMARY KEY AUTO_INCREMENT,
    title VARCHAR(255),
    description TEXT,
    difficulty ENUM('Easy', 'Medium', 'Hard'),
    created_by INT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (created_by) REFERENCES users(id)
);
CREATE TABLE tags (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100) UNIQUE
);
CREATE TABLE problem_tags (
    problem_id INT,
    tag_id INT,
    PRIMARY KEY (problem_id, tag_id),
    FOREIGN KEY (problem_id) REFERENCES problems(id),
    FOREIGN KEY (tag_id) REFERENCES tags(id)
);
CREATE TABLE submissions (
    id INT PRIMARY KEY AUTO_INCREMENT,
    user_id INT,
    problem_id INT,
    code TEXT,
    language VARCHAR(50),
    status ENUM('Accepted', 'Wrong Answer', 'TLE', 'Runtime Error'),
    execution_time FLOAT,
    memory_used FLOAT,
    submitted_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(id),
    FOREIGN KEY (problem_id) REFERENCES problems(id)
);
```
 🫤 confusion about testcase