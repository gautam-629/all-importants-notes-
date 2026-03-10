## 1. B-Tree Index

### What is B-Tree?
-   A **self-balancing tree** data structure.
-   Stores data in **sorted order**.
-   Used as the **default index in PostgreSQL**.
### Time Complexity:
-   Search: O(log n)
-   Insert: O(log n)
-   Delete: O(log n)
### Best For:

-   Range queries (`BETWEEN`, `<`, `>`)
-   `ORDER BY`
-   Prefix searches
-   Equality search (`=`)
### Example:

``` sql
CREATE INDEX idx_amount ON orders(amount);
```

------------------------------------------------------------------------
## 2. Hash Index
### What is Hash Index?
-   Uses a **hash function** to store data.
-   Provides direct access using hash buckets.
-   Best for exact match queries.
### Time Complexity:
-   Search: O(1) (Exact match)
-   Insert: O(1)
-   Delete: O(1)
### Limitations:
-   ❌ No range queries
-   ❌ No sorting support
-   Works only with `=` operator
### Example:

``` sql
CREATE INDEX idx_user_id
ON users USING HASH (user_id);
```

------------------------------------------------------------------------
## 3. Key Differences
  Feature                 B-Tree   Hash
----------------------- -------- ------------
  Data Order              Sorted   Not Sorted
  Range Query             ✅ Yes   ❌ No
  Exact Match             ✅ Yes   ✅ Yes
  ORDER BY                ✅ Yes   ❌ No
  Default in PostgreSQL   ✅ Yes   ❌ No
-----------------------------------------------------------------------
### Interview One-Liner:
> B-Tree supports range and sorted queries with O(log n), while Hash
> index is faster for exact matches with O(1) but does not support range
> queries.