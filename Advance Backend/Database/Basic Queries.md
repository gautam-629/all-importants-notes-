# 📚 Table of Contents — SQL & Database Notes

---
## Part 1 — SQL Language & Commands

```
1.  Types of Database Language
    ├── 1.1  DDL – Data Definition Language
    │         CREATE · ALTER · DROP · TRUNCATE · RENAME
    ├── 1.2  DML – Data Manipulation Language
    │         INSERT · UPDATE · DELETE · SELECT
    ├── 1.3  DCL – Data Control Language
    │         GRANT · REVOKE
    └── 1.4  TCL – Transaction Control Language
              COMMIT · ROLLBACK · SAVEPOINT
```

---
## Part 2 — Data Types & Operators

```
2.  MySQL Data Types
    ├── Numeric     INT · BIGINT · FLOAT · DOUBLE · DECIMAL
    ├── String      CHAR · VARCHAR · TEXT
    ├── Date        DATE · TIME · DATETIME · TIMESTAMP
    └── Boolean     BOOLEAN · TINYINT(1)

3.  SQL Comparison Operators
    ├── Basic       =  <>  !=  <  >  <=  >=
    ├── NULL Checks IS NULL · IS NOT NULL
    ├── Range & Set BETWEEN · NOT BETWEEN · IN · NOT IN
    └── Pattern     LIKE · NOT LIKE · ILIKE

4.  Logical Operators
    AND · OR · NOT
```

---
## Part 3 — Querying Data

```
5.  WHERE Clause
    └── Filter rows based on conditions

6.  ORDER BY Clause
    ├── ASC  – Ascending order
    └── DESC – Descending order

7.  ILIKE Operator Examples
    ├── Starts With   name ILIKE 'b%'
    ├── Ends With     name ILIKE '%d'
    └── Contains      name ILIKE '%no%'
```

---
## Part 4 — Database & Table Management

```
8.  Database Commands
    ├── CREATE DATABASE
    ├── SHOW DATABASES
    ├── DROP DATABASE
    └── USE

9.  Table Commands
    └── SHOW TABLES

10. Create Tables (Users & Posts)
    ├── users  – id · name · email · created_at
    └── posts  – id · title · content · user_id · created_at

11. Primary Key
    └── AUTO_INCREMENT · NOT NULL · Unique per table

12. Describe Table
    └── DESC table_name
```

---
## Part 5 — CRUD Operations

```
13. Insert Data
    ├── Single insert into users
    └── Single insert into posts

14. Display Data
    └── SELECT * FROM table

15. Bulk Insert
    └── Multiple rows in one INSERT statement

16. CRUD Operations
    ├── CREATE  – INSERT INTO
    ├── READ    – SELECT *
    ├── UPDATE  – UPDATE ... SET ... WHERE
    └── DELETE  – DELETE FROM ... WHERE
```

---
## Part 6 — Advanced Queries

```
17. Join Users and Posts
    └── JOIN posts ON users.id = posts.user_id

18. WHERE + ORDER BY Combined
    └── Filter with LIKE + sort with ORDER BY DESC

19. Full Example Flow
    └── CREATE → USE → SHOW TABLES → SELECT
```

---
## Part 7 — Pagination

```
20. Pagination (OFFSET & LIMIT)
    ├── Syntax    LIMIT n OFFSET n
    ├── Example   LIMIT 5 OFFSET 10
    └── Benefits
          · Reduces DB load
          · Improves response time
          · Useful for APIs & large datasets
```

---
## Part 8 — Database Design

```
21. Database Design (Users, Posts, Comments, Likes)
    ├── Users Table
    ├── Posts Table
    ├── Comments Table
    ├── Bad Likes Design ❌
    │     · NULL columns (post_id or comment_id wasted)
    │     · Not scalable
    ├── Better Scalable Likes Design ✅
    │     · likeable_id + likeable_type ENUM
    │     · No NULL confusion
    ├── Extending for Reels
    │     ALTER TABLE → MODIFY ENUM
    └── Importance of ENUM
          · Restricts to predefined values
          · Prevents invalid data
```

---
## Part 9 — Constraints

```
22. Constraints in Databases
    ├── Advantages
    │     · Prevents invalid data
    │     · Maintains consistency
    │     · Enforces relationships
    ├── Types of Constraints
    │     ├── NOT NULL    – column cannot be empty
    │     ├── UNIQUE      – no duplicate values
    │     └── PRIMARY KEY – unique row identifier
    └── Foreign Key Constraint
          ├── Problem Without FK ❌
          │     · Orphan records possible
          │     · Data inconsistency
          └── Solution With FK ✅
                · user_id must exist in users
                · post_id must exist in posts
```

# 1. Types of Database Language
SQL commands are divided into **4 types**:
## 1.1 DDL – Data Definition Language
Used to define database structure.
Commands:
- `CREATE` – create database/table
- `ALTER` – modify table
- `DROP` – delete database/table
- `TRUNCATE` – delete all records
- `RENAME` – rename table
Example:
```sql
CREATE TABLE users (
  id INT,
  name VARCHAR(50)
);
```
---
## 1.2 DML – Data Manipulation Language
Used to manipulate data.
Commands:
- `INSERT`
- `UPDATE`
- `DELETE`
- `SELECT`
Example:
```sql
INSERT INTO users (id,name) VALUES (1,'Binod');
```
---
## 1.3 DCL – Data Control Language
Used for permissions.
Commands:
- `GRANT`
- `REVOKE`
Example:
```sql
GRANT SELECT ON mydb.users TO 'user'@'localhost';
```
---
## 1.4 TCL – Transaction Control Language
Used to manage transactions.
Commands:
- `COMMIT`
- `ROLLBACK`
- `SAVEPOINT`
Example:

```sql
START TRANSACTION;
UPDATE users SET name='Ram' WHERE id=1;
ROLLBACK;
```

---
# 2. MySQL Data Types
## Numeric Types
- `INT`
- `BIGINT`
- `FLOAT`
- `DOUBLE`
- `DECIMAL`
## String Types
- `CHAR(n)`
- `VARCHAR(n)`
- `TEXT`
## Date Types
- `DATE`
- `TIME`
- `DATETIME`
- `TIMESTAMP`
## Boolean
- `BOOLEAN`
- `TINYINT(1)`
---
# 3. SQL Comparison Operators
## 🔢 Basic Comparison
- `=` Equal to
- `<>` Not equal to
- `!=` Not equal to
- `<` Less than
- `>` Greater than
- `<=` Less than equal
- `>=` Greater than equal
---
## 🧩 NULL Checks
- `IS NULL`
- `IS NOT NULL`
Example

```sql
SELECT * FROM users WHERE email IS NULL;
```
---
## 🎯 Range & Set
- `BETWEEN`
- `NOT BETWEEN`
- `IN`
- `NOT IN`
Example
```sql
SELECT * FROM users
WHERE id BETWEEN 1 AND 5;
```
---
## 🔍 Pattern Matching
- `LIKE`
- `NOT LIKE`
- `ILIKE` (PostgreSQL only)
Wildcards:
- `%` → any characters
- `_` → single character
Example

```sql
SELECT * FROM users
WHERE name LIKE 'B%';
```
---
# 4. Logical Operators
- `AND`
- `OR`
- `NOT`
Example
```sql
SELECT * FROM users
WHERE age > 18 AND city='Kathmandu';
```
---
# 5. WHERE Clause
Used to filter data.
```sql
SELECT * FROM users
WHERE age > 20;
```
---
# 6. ORDER BY Clause
Used to sort results.
Ascending:
```sql
SELECT * FROM users
ORDER BY name ASC;
```
Descending:
```sql
SELECT * FROM users
ORDER BY id DESC;
```
---
# 7. ILIKE Operator Examples (3 Types)
## Starts With
```sql
SELECT * FROM users
WHERE name ILIKE 'b%';
```
## Ends With
```sql
SELECT * FROM users
WHERE name ILIKE '%d';
```
## Contains
```sql
SELECT * FROM users
WHERE name ILIKE '%no%';
```
---
# 8. Database Commands
## Create Database
```sql
CREATE DATABASE social_app;
```
## Show Database
```sql
SHOW DATABASES;
```
## Delete Database
```sql
DROP DATABASE social_app;
```
## Select Database

```sql
USE social_app;
```

---

# 9. Table Commands
## Show Tables
```sql
SHOW TABLES;
```
---
# 10. Create Tables (Users & Posts)
```sql
CREATE TABLE users (
  id INT PRIMARY KEY AUTO_INCREMENT,
  name VARCHAR(50),
  email VARCHAR(100) UNIQUE,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```
```sql
CREATE TABLE posts (
  id INT PRIMARY KEY AUTO_INCREMENT,
  title VARCHAR(100),
  content TEXT,
  user_id INT,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (user_id) REFERENCES users(id)
);
```
---
# 11. Primary Key
- Uniquely identifies each row
- Cannot be NULL
- Only one per table
Example
```sql
id INT PRIMARY KEY AUTO_INCREMENT
```
---
# 12. Describe Table
```sql
DESC users;
```
---
# 13. Insert Data
## Insert into users
```sql
INSERT INTO users (name,email)
VALUES ('Binod','binod@gmail.com');
```
## Insert into posts
```sql
INSERT INTO posts (title,content,user_id)
VALUES ('First Post','Hello world',1);
```
---
# 14. Display Data
```sql
SELECT * FROM users;```
```sql
SELECT * FROM posts;
```
---
# 15. Bulk Insert

```sql
INSERT INTO users (name,email) VALUES
('Ram','ram@gmail.com'),
('Hari','hari@gmail.com'),
('Sita','sita@gmail.com');
```
---
# 16. CRUD Operations
# CREATE
```sql
INSERT INTO users (name,email)
VALUES ('Gita','gita@gmail.com');
```
---
# READ
```sql
SELECT * FROM users;
```
```sql
SELECT * FROM posts;
```
---
# UPDATE
```sql
UPDATE users
SET name='Binod Gautam'
WHERE id=1;
```
---
# DELETE
```sql
DELETE FROM users
WHERE id=3;
```
---
# 17. Join Users and Posts
```sql
SELECT users.name, posts.title
FROM users
JOIN posts ON users.id = posts.user_id;
```
---
# 18. WHERE + ORDER BY Example
```sql
SELECT * FROM users
WHERE name LIKE 'B%'
ORDER BY id DESC;
```
---
# 19. Full Example Flow
```sql
CREATE DATABASE social_app;
USE social_app;
SHOW TABLES;
SELECT * FROM users;
```

---
## 1. Pagination (OFFSET & LIMIT)
Pagination is used to fetch data in chunks instead of loading everything at once. This improves performance and user experience.
### 🔹 Syntax
```sql
SELECT * FROM table_name
LIMIT 10 OFFSET 0;
```
### 🔹 Example
```sql
SELECT * FROM posts
LIMIT 5 OFFSET 10;
```
👉 This skips the first 10 rows and returns the next 5 rows.
### 🔹 Why use pagination?
- Reduces load on database
- Improves response time
- Useful for APIs and large datasets
---
## 2. Database Design (Users, Posts, Comments, Likes)
### 🔹 Tables Structure
#### Users Table
```sql
CREATE TABLE users (
    id INT PRIMARY KEY,
    name VARCHAR(100)
);
```
#### Posts Table
```sql
CREATE TABLE posts (
    id INT PRIMARY KEY,
    user_id INT,
    content TEXT,
    FOREIGN KEY (user_id) REFERENCES users(id)
);
```
#### Comments Table
```sql
CREATE TABLE comments (
    id INT PRIMARY KEY,
    user_id INT,
    post_id INT,
    comment TEXT,
    FOREIGN KEY (user_id) REFERENCES users(id),
    FOREIGN KEY (post_id) REFERENCES posts(id)
);
```
---
## 🔹 Bad Design for Likes ❌
```sql
CREATE TABLE likes (
    user_id INT,
    post_id INT,
    comment_id INT
);
```
### ❌ Problems:
- If liking a post → `comment_id` is NULL
- If liking a comment → `post_id` is NULL
- Wasted space & confusion
- Not scalable (what if reels are added?)
---
## 🔹 Better Scalable Design ✅
```sql
CREATE TABLE likes (
    user_id INT,
    likeable_id INT,
    likeable_type ENUM('post','comment')
);
```
### 🔹 Advantages:
- Supports multiple entity types
- Clean structure (no NULL confusion)
- Easily extendable
---
## 🔹 Extending for Reels
```sql
ALTER TABLE likes
MODIFY likeable_type ENUM('post','comment','reel');
```
👉 Now likes can support reels without changing table structure.
---
## 🔹 Importance of ENUM
- Restricts values to predefined set
- Prevents invalid data
- Improves data consistency
Example:
```sql
likeable_type ENUM('post','comment','reel')
```
---
## 3. Constraints in Databases
Constraints are rules applied on table columns to maintain data integrity.
### 🔹 Advantages
- Prevents invalid data
- Maintains consistency
- Enforces relationships
---
## 🔹 Types of Constraints

### 1. NOT NULL
```sql
name VARCHAR(100) NOT NULL
```
👉 Ensures column cannot be empty
### 2. UNIQUE

```sql
email VARCHAR(100) UNIQUE
```
👉 No duplicate values allowed
### 3. PRIMARY KEY

```sql
id INT PRIMARY KEY
```
👉 Unique identifier for each row

---
## 🔹 Foreign Key Constraint
Ensures that a value exists in another table.
### 🔹 Problem Without Foreign Key ❌
You can insert invalid data:
```sql
INSERT INTO comments (user_id, post_id)
VALUES (999, 888);
```
👉 User or Post may not exist → Data inconsistency
### 🔹 Solution Using Foreign Key ✅
```sql
CREATE TABLE comments (
    id INT PRIMARY KEY,
    user_id INT,
    post_id INT,
    comment TEXT,
    FOREIGN KEY (user_id) REFERENCES users(id),
    FOREIGN KEY (post_id) REFERENCES posts(id)
);
```
👉 Now database ensures:
- user_id must exist in users table
- post_id must exist in posts table
---
## 🔹 Real-World Benefit
- Prevents orphan records
- Keeps relationships valid
- Automatically enforces rules at DB level
---
