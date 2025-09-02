# SQL Functions Reference

SQL functions vary depending on the database (MySQL, PostgreSQL, SQL Server, Oracle, SQLite).  
Below is a categorized list of the most common SQL functions.

---
## 1. Aggregate Functions
Used to summarize data (often with `GROUP BY`):
- `COUNT()` → Counts rows  
- `SUM()` → Returns sum of values  
- `AVG()` → Returns average value  
- `MIN()` → Returns minimum value  
- `MAX()` → Returns maximum value  
- `GROUP_CONCAT()` *(MySQL)* / `STRING_AGG()` *(Postgres/SQL Server)* → Concatenates values  

---

## 2. String/Text Functions
Manipulate text strings:

- `CONCAT()` → Concatenate strings  
- `SUBSTRING()` / `SUBSTR()` → Extract substring  
- `LEFT()`, `RIGHT()` → Get characters from left/right  
- `TRIM()`, `LTRIM()`, `RTRIM()` → Remove spaces  
- `UPPER()` / `LOWER()` → Change case  
- `REPLACE()` → Replace substring  
- `CHAR_LENGTH()` / `LENGTH()` → String length  
- `INSTR()` / `POSITION()` → Find substring position  
- `LPAD()`, `RPAD()` → Pad strings  

---

## 3. Mathematical/Arithmetic Functions
Work with numeric data:

- `ABS()` → Absolute value  
- `CEIL()` / `CEILING()` → Round up  
- `FLOOR()` → Round down  
- `ROUND()` → Round to nearest  
- `POWER(x, y)` → Exponentiation  
- `SQRT()` → Square root  
- `MOD()` → Remainder  
- `RAND()` → Random number  
- `EXP()` → Exponential  
- `LOG()` / `LN()` → Logarithm  

---
## 4. Date and Time Functions
Work with dates and times:
- `NOW()` → Current datetime  
- `CURRENT_DATE` / `CURDATE()` → Current date  
- `CURRENT_TIME` / `CURTIME()` → Current time  
- `DATE()` → Extract date  
- `TIME()` → Extract time  
- `YEAR()`, `MONTH()`, `DAY()` → Extract components  
- `HOUR()`, `MINUTE()`, `SECOND()`  
- `DATEDIFF()` → Difference between dates  
- `DATE_ADD()` / `DATE_SUB()` → Add/Subtract intervals  
- `EXTRACT()` → Get part of a date  

---
## 5. Conversion & Casting Functions
Convert between data types:

- `CAST(expression AS type)`  
- `CONVERT(expression, type)`  
- `FORMAT()` *(SQL Server, MySQL)*  
- `TO_CHAR()` *(Oracle, Postgres)*  
- `TO_DATE()`  
- `TO_NUMBER()`  

---
## 6. Conditional Functions
Used for conditional logic:

- `CASE WHEN ... THEN ... ELSE ... END`  
- `IF(condition, value_if_true, value_if_false)` *(MySQL)*  
- `NULLIF(a, b)` → Returns NULL if `a = b`  
- `COALESCE(a, b, c...)` → First non-null value  
- `ISNULL()` *(SQL Server, MySQL)*  

---
## 7. Window/Analytic Functions
Used with `OVER()` for advanced queries:

- `ROW_NUMBER()` → Unique row number  
- `RANK()` → Ranking with gaps  
- `DENSE_RANK()` → Ranking without gaps  
- `NTILE(n)` → Distribute rows into `n` groups  
- `LEAD()` / `LAG()` → Access next/previous row value  
- `FIRST_VALUE()` / `LAST_VALUE()`  

---
## 8. JSON Functions
*(Modern SQL engines only)*
- `JSON_EXTRACT()` *(MySQL)*  
- `JSON_VALUE()` *(SQL Server)*  
- `->`, `->>` *(Postgres, MySQL)*  
- `JSON_ARRAY()`, `JSON_OBJECT()`  
- `JSON_AGG()`  

---
## 9. System / Metadata Functions
- `DATABASE()` → Current database  
- `USER()` / `CURRENT_USER()`  
- `VERSION()`  
- `@@VARIABLES` *(MySQL, SQL Server)*  

---

✅ This list covers most **standard SQL functions**.  
Different databases also add their own:

- Oracle → `NVL()`  
- PostgreSQL → Regex functions, array functions  
- MySQL → `ELT()`, `FIELD()`  
- SQLite → Minimal but supports `ABS()`, `LENGTH()`, `SUBSTR()`, etc.
