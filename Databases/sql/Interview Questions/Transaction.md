### 🔄 What is a Transaction?
A **transaction** is a sequence of one or more database operations executed as a single unit of work.
It ensures data consistency using **ACID properties**:
- **Atomicity** – All operations succeed or all fail  
    **Example:** In a money transfer, if deducting money from Account A succeeds but adding to Account B fails, the entire transaction is rolled back so no money is lost.
 **Consistency** – Database remains valid  
**Example:** If a rule says account balance cannot be negative, the database prevents committing a transaction that would result in a negative balance.

**Isolation** – Concurrent transactions don’t interfere  
**Example:** If two users try to withdraw money from the same account at the same time, isolation ensures the balance is calculated correctly and prevents incorrect double deductions.

**Durability** – Committed data is permanently saved  
**Example:** Once a transaction is committed and the system crashes immediately after, the committed changes will still exist when the database restarts.

---
### 🐘 PostgreSQL Transaction Example
### Example: Money Transfer Between Accounts
Without a transaction, money could be deducted but not added if an error occurs.
```sql
BEGIN;

UPDATE accounts
SET balance = balance - 100
WHERE id = 1;

UPDATE accounts
SET balance = balance + 100
WHERE id = 2;

COMMIT;
```
If something fails, use:
```sql
ROLLBACK;
```
✔ Ensures both updates succeed together  
✔ Prevents partial data changes
# ❌ When NOT to Use Transactions
- Simple read-only queries
- Single independent insert/update
- Long-running operations (can cause locks)
---
# 🧠 Interview Summary
> A transaction ensures multiple database operations execute safely as a single unit using ACID principles. I use it when operations must either fully succeed or fully fail to maintain data integrity.