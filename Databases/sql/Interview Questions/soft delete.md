### 📌 What is Soft Delete?

Soft delete means marking a record as deleted instead of permanently removing it from the database.

The record remains in the table but is excluded from normal queries.

---
### 🆚 Soft Delete vs Hard Delete
- **Hard Delete** → Permanently removes row using `DELETE`
- **Soft Delete** → Updates a column like `deleted_at`
---
## 🐘 PostgreSQL Implementation (Recommended Approach)
### 1️⃣ Add Column
```sql
ALTER TABLE users
ADD COLUMN deleted_at TIMESTAMP NULL;
```
### 2️⃣ Soft Delete Record
```sql
UPDATE users
SET deleted_at = NOW()
WHERE id = 1;
```
### 3️⃣ Fetch Active Records Only

```sql
SELECT * FROM users
WHERE deleted_at IS NULL;
```
### 4️⃣ Restore Record

```sql
UPDATE users
SET deleted_at = NULL
WHERE id = 1;
```

---
## 🔎 Alternative: Boolean Flag

```sql
ALTER TABLE users ADD COLUMN is_deleted BOOLEAN DEFAULT FALSE;
```
Soft delete:

```sql
UPDATE users SET is_deleted = TRUE WHERE id = 1;
```

---
## ⚠️ Best Practices
- Prefer `deleted_at` over boolean flag
- Add index on `deleted_at`
- Always filter `WHERE deleted_at IS NULL`
- Be careful with unique constraints
- Handle cascading soft deletes properly
---
## 🧠 Interview Summary

> Soft delete marks data as deleted using a flag or timestamp instead of removing it. It helps with recovery, auditing, and maintaining data integrity.