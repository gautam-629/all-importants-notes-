
SQL functions vary depending on the database system (MySQL, PostgreSQL, SQL Server, Oracle, SQLite). Below is a comprehensive categorized list of the most common SQL functions with their concepts, use cases, and syntax.
# SQL Comparison Operators
## 🔢 Basic Comparison
- `=` : Equal to  
- `<>` or `!=` : Not equal to  
- `<` : Less than  
- `>` : Greater than  
- `<=` : Less than or equal to  
- `>=` : Greater than or equal to  

---
## 🧩 NULL Checks
- `IS NULL` : Value is null  
- `IS NOT NULL` : Value is not null  

---
## 🎯 Range & Set
- `BETWEEN value1 AND value2` : Within the range (inclusive)  
- `NOT BETWEEN value1 AND value2` : Outside the range  
- `IN (val1, val2, …)` : Matches any value in the list  
- `NOT IN (val1, val2, …)` : Does not match values in the list  

---
## 🔍 Pattern Matching
- `LIKE 'pattern'` : Pattern match using `%` (any length) and `_` (single char)  
- `NOT LIKE 'pattern'` : Pattern does not match  
- `ILIKE 'pattern'` (PostgreSQL) : Case-insensitive LIKE  

---
## 📊 Logical Operators
- `AND` : Both conditions must be true  
- `OR` : At least one condition is true  
- `NOT` : Negates a condition  
---
## 📈 Advanced
- `EXISTS (subquery)` : True if subquery returns rows  
- `NOT EXISTS (subquery)` : True if subquery returns no rows  
- `ANY` / `SOME` : Compare value to **any** in a list/subquery  
- `ALL` : Compare value to **all** in a list/subquery  

## 1. Aggregate Functions

**Concept:** Aggregate functions perform calculations on multiple rows and return a single summarized value.

**Use Cases:**

- Count registered users in a system
- Calculate total revenue from orders
- Find average product ratings
- Determine highest and lowest salaries
- Concatenate customer names into a single list

**Functions:**

- `COUNT()` → Counts rows
- `SUM()` → Returns sum of values
- `AVG()` → Returns average value
- `MIN()` → Returns minimum value
- `MAX()` → Returns maximum value
- `GROUP_CONCAT()` _(MySQL)_ / `STRING_AGG()` _(PostgreSQL/SQL Server)_ → Concatenates values

_Note: Often used with `GROUP BY` clause_
## 2. String/Text Functions

**Concept:** String functions allow manipulation, formatting, and extraction of text data.
**Use Cases:**
- Combine first and last names into full names
- Extract domain names from email addresses
- Format text for consistency (uppercase/lowercase)
- Replace unwanted characters (e.g., "st." to "street")
- Remove extra spaces from user inputs
- Pad invoice numbers with leading zeros
**Functions:**
- `CONCAT()` → Concatenate strings
- `SUBSTRING()` / `SUBSTR()` → Extract substring
- `LEFT()`, `RIGHT()` → Get characters from left/right
- `TRIM()`, `LTRIM()`, `RTRIM()` → Remove spaces
- `UPPER()` / `LOWER()` → Change case
- `REPLACE()` → Replace substring
- `CHAR_LENGTH()` / `LENGTH()` → String length
- `INSTR()` / `POSITION()` → Find substring position
- `LPAD()`, `RPAD()` → Pad strings
## 3. Mathematical/Arithmetic Functions

**Concept:** Math functions perform numeric calculations and transformations.
**Use Cases:**
- Calculate discounts or taxes on prices
- Round currency values for billing
- Generate random numbers for data sampling
- Compute growth rates using exponentials
- Find remainder values in modular arithmetic
- Ensure positive numbers using absolute value
**Functions:**
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

## 4. Date and Time Functions

**Concept:** Date/time functions handle operations on dates, times, and intervals.
**Use Cases:**
- Record current date/time for transaction logs
- Extract specific components (year/month/day) from timestamps
- Calculate shipping duration in days
- Add/subtract days for setting due dates
- Schedule reports by extracting date parts
- Track user login activity with timestamps
**Functions:**
- `NOW()` → Current datetime
- `CURRENT_DATE` / `CURDATE()` → Current date
- `CURRENT_TIME` / `CURTIME()` → Current time
- `DATE()` → Extract date
- `TIME()` → Extract time
- `YEAR()`, `MONTH()`, `DAY()` → Extract components
- `HOUR()`, `MINUTE()`, `SECOND()` → Extract time components
- `DATEDIFF()` → Difference between dates
- `DATE_ADD()` / `DATE_SUB()` → Add/subtract intervals
- `EXTRACT()` → Get part of a date

## 5. Conversion & Casting Functions
**Concept:** Conversion functions change data from one type to another (e.g., string to number, date to string).
**Use Cases:**
- Convert string-formatted numbers for calculations
- Format dates for reporting (e.g., "2025-09-02" → "September 2, 2025")
- Change numeric values to text for invoices
- Ensure compatibility when combining different data types
**Functions:**
- `CAST(expression AS type)` → Standard casting
- `CONVERT(expression, type)` → Alternative casting
- `FORMAT()` _(SQL Server, MySQL)_ → Format values
- `TO_CHAR()` _(Oracle, PostgreSQL)_ → Convert to character
- `TO_DATE()` → Convert to date
- `TO_NUMBER()` → Convert to number
## 6. Conditional Functions
**Concept:** Conditional functions provide logic-based output depending on given conditions.
**Use Cases:**
- Apply discounts only above purchase thresholds
- Display "Active" or "Inactive" based on user status
- Replace NULL values with default text like "N/A"
- Provide fallback values when primary data is missing
- Handle exceptions in financial reporting
**Functions:**
- `CASE WHEN ... THEN ... ELSE ... END` → Complex conditional logic
- `IF(condition, value_if_true, value_if_false)` _(MySQL)_ → Simple conditional
- `NULLIF(a, b)` → Returns NULL if `a = b`
- `COALESCE(a, b, c...)` → First non-null value
- `ISNULL()` _(SQL Server, MySQL)_ → Handle NULL values
## 7. Window/Analytic Functions
**Concept:** Advanced functions used with `OVER()` clause for complex analytical queries.
**Functions:**
- `ROW_NUMBER()` → Unique row number
- `RANK()` → Ranking with gaps
- `DENSE_RANK()` → Ranking without gaps
- `NTILE(n)` → Distribute rows into `n` groups
- `LEAD()` / `LAG()` → Access next/previous row value
- `FIRST_VALUE()` / `LAST_VALUE()` → First/last values in window
## 8. JSON Functions
**Concept:** Functions for working with JSON data _(Available in modern SQL engines only)_
**Functions:**
- `JSON_EXTRACT()` _(MySQL)_ → Extract JSON values
- `JSON_VALUE()` _(SQL Server)_ → Extract scalar JSON value
- `->`, `->>` _(PostgreSQL, MySQL)_ → JSON operators
- `JSON_ARRAY()`, `JSON_OBJECT()` → Create JSON structures
- `JSON_AGG()` → Aggregate into JSON array

## 9. System/Metadata Functions
**Concept:** Functions that provide information about the database system and environment.
**Functions:**
- `DATABASE()` → Current database name
- `USER()` / `CURRENT_USER()` → Current user
- `VERSION()` → Database version
- `@@VARIABLES` _(MySQL, SQL Server)_ → System variables
## Database-Specific Extensions
Different database systems provide additional specialized functions:
**Oracle:**
- `NVL()` → Handle NULL values
- Advanced date functions
- Hierarchical queries support
**PostgreSQL:**
- Regular expression functions
- Array functions
- Advanced JSON support
**MySQL:**
- `ELT()` → Return string at index
- `FIELD()` → Find position in list
- Full-text search functions
**SQLite:**
- Minimal but supports core functions
- `ABS()`, `LENGTH()`, `SUBSTR()`, etc.
- Lightweight mathematical functions

---

> **Note:** This reference covers the most standard SQL functions across different database systems. Always consult your specific database documentation for complete function lists and syntax variations.