A **transaction** is a sequence of one or more database operations executed as a single, indivisible unit of work. If any operation within the transaction fails, the entire transaction is rolled back — leaving the database in its original state.

---
## ⚡ ACID Properties

### 1. Atomicity — _All or Nothing_

Every operation in a transaction either **fully succeeds** or **fully fails**. There is no partial completion.

> **Example:** In a bank transfer, if $100 is deducted from Account A but the credit to Account B fails due to an error, atomicity ensures the deduction is automatically reversed — no money is lost or created.

---

### 2. Consistency — _Rules Are Always Enforced_

A transaction can only bring the database from one **valid state** to another. Business rules and constraints are never violated.

> **Example:** If a rule states that an account balance cannot go negative, the database will reject any transaction that would result in a negative balance — even if the debit operation succeeds on its own.

---

### 3. Isolation — _Concurrent Transactions Don't Interfere_

Each transaction executes as if it is the **only transaction** running, even when multiple transactions occur simultaneously.

> **Example:** If two users simultaneously attempt to withdraw $500 from an account with a $600 balance, isolation ensures only one withdrawal succeeds — preventing both from succeeding and causing an overdraft.

---

### 4. Durability — _Committed Data Is Permanent_

Once a transaction is **committed**, its changes are permanently saved — even in the event of a system crash, power failure, or restart.

> **Example:** After a successful payment is committed, a sudden server crash will not lose that transaction. When the database restarts, the payment record is still intact.

---

## 🐘 PostgreSQL Transaction Examples

### Basic Example — Manual Transaction Block

```sql
-- Start the transaction
BEGIN;

-- Step 1: Deduct $100 from Account 1
UPDATE accounts
SET balance = balance - 100
WHERE id = 1;

-- Step 2: Credit $100 to Account 2
UPDATE accounts
SET balance = balance + 100
WHERE id = 2;

-- Commit if both steps succeeded
COMMIT;

-- Rollback if something went wrong
-- ROLLBACK;
```

> ✅ Both updates succeed together  
> ✅ If either fails, `ROLLBACK` undoes all changes  
> ✅ No partial data modifications

---

### Advanced Example — Transaction Using a Stored Procedure

Wrapping transaction logic inside a **stored procedure** is best practice for reusability, maintainability, and encapsulation.

#### Step 1: Create the Accounts Table

```sql
CREATE TABLE accounts (
    id      SERIAL PRIMARY KEY,
    name    VARCHAR(100) NOT NULL,
    balance NUMERIC(15, 2) NOT NULL CHECK (balance >= 0)
);

-- Seed data
INSERT INTO accounts (name, balance) VALUES
    ('Alice', 1000.00),
    ('Bob',    500.00);
```

#### Step 2: Create the Transfer Procedure

```sql
CREATE OR REPLACE PROCEDURE transfer_funds(
    sender_id   INT,
    receiver_id INT,
    amount      NUMERIC
)
LANGUAGE plpgsql
AS $$
BEGIN
    -- Guard: amount must be positive
    IF amount <= 0 THEN
        RAISE EXCEPTION 'Transfer amount must be greater than zero. Got: %', amount;
    END IF;

    -- Guard: sender must have sufficient balance
    IF (SELECT balance FROM accounts WHERE id = sender_id) < amount THEN
        RAISE EXCEPTION 'Insufficient funds in account ID %', sender_id;
    END IF;

    -- Deduct from sender
    UPDATE accounts
    SET balance = balance - amount
    WHERE id = sender_id;

    -- Credit to receiver
    UPDATE accounts
    SET balance = balance + amount
    WHERE id = receiver_id;

    RAISE NOTICE 'Transfer of % from account % to account % completed successfully.',
        amount, sender_id, receiver_id;
END;
$$;
```

#### Step 3: Call the Procedure

```sql
-- Execute inside a transaction block for full control
BEGIN;

CALL transfer_funds(
    sender_id   => 1,   -- Alice
    receiver_id => 2,   -- Bob
    amount      => 200.00
);

COMMIT;
```

#### Step 4: Verify the Results

```sql
SELECT id, name, balance FROM accounts ORDER BY id;
```

|id|name|balance|
|---|---|---|
|1|Alice|800.00|
|2|Bob|700.00|

---

### Error Handling — What Happens on Failure

```sql
BEGIN;

CALL transfer_funds(
    sender_id   => 1,    -- Alice (balance: 800.00)
    receiver_id => 2,    -- Bob
    amount      => 9999.00  -- Exceeds Alice's balance
);

-- Output:
-- ERROR: Insufficient funds in account ID 1
-- ROLLBACK automatically triggered; no changes are saved

COMMIT; -- Never reached due to the exception above
```

> PostgreSQL automatically rolls back the transaction when an unhandled exception is raised inside a procedure.

---

### Savepoints — Partial Rollbacks Within a Transaction

```sql
BEGIN;

UPDATE accounts SET balance = balance - 50 WHERE id = 1;

SAVEPOINT before_bob_update;

UPDATE accounts SET balance = balance + 50 WHERE id = 2;

-- Oops, need to undo just the Bob update
ROLLBACK TO SAVEPOINT before_bob_update;

-- Alice's deduction is still in effect; now credit a different account
UPDATE accounts SET balance = balance + 50 WHERE id = 3;

COMMIT;
```

---

## ❌ When NOT to Use Transactions

|Scenario|Reason to Avoid|
|---|---|
|Simple `SELECT` queries|Read-only; no data modification occurs|
|Single, independent `INSERT` / `UPDATE`|Auto-committed; transaction overhead unnecessary|
|Long-running batch operations|Holds locks too long; can block other users|
|Reporting or analytics queries|Read consistency is handled by isolation level, not explicit transactions|

---

## 🧠 Interview Summary

> A **transaction** guarantees that a group of database operations executes safely as a single atomic unit, following **ACID principles** — Atomicity, Consistency, Isolation, and Durability.
> 
> I use transactions whenever multiple operations must either **fully succeed or fully fail together** to preserve data integrity — such as financial transfers, order processing, or inventory updates.
> 
> In PostgreSQL, I prefer encapsulating transaction logic inside **stored procedures** (`CALL transfer_funds(...)`) for reusability and cleaner application code. I also use **savepoints** when I need granular rollback control within a larger transaction.