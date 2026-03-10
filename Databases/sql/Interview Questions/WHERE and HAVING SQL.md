### 1. WHERE Clause
-   Filters **individual rows**
-   Executes **before GROUP BY**
-   Cannot use aggregate functions (SUM, COUNT, AVG)
-   Improves performance by reducing dataset early
### Example:

``` sql
SELECT *
FROM orders
WHERE amount > 5000;
```
------------------------------------------------------------------------
## 2. HAVING Clause
-   Filters **grouped results**
-   Executes **after GROUP BY**
-   Used with aggregate functions
### Example:
``` sql
SELECT customer_id, COUNT(order_id)
FROM orders
GROUP BY customer_id
HAVING COUNT(order_id) > 2;
```
------------------------------------------------------------------------
## 3. Key Differences
  Feature               WHERE                    HAVING
--------------------- ------------------------ ----------------
  Filters               Rows                     Groups
  Used With             SELECT, UPDATE, DELETE   GROUP BY
  Aggregate Functions   ❌ No                    ✅ Yes
  Execution Time        Before grouping          After grouping
------------------------------------------------------------------------
## 4. Combined Example
``` sql
SELECT c.customer_name, SUM(o.amount) AS total_spent
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id
WHERE c.city = 'Delhi'
GROUP BY c.customer_name
HAVING SUM(o.amount) > 10000;
```

------------------------------------------------------------------------
### Interview One-Liner:
> WHERE filters rows before grouping, HAVING filters groups after
> aggregation.