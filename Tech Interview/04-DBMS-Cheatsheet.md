---
created: 2026-08-30
purpose: DBMS concepts for Infosys technical interview
---
# DBMS Cheatsheet - Infosys Interview

## 1. ACID Properties

| Property | Description | Example |
|----------|-------------|---------|
| **Atomicity** | All operations succeed or all fail | Bank transfer: debit + credit both happen or neither |
| **Consistency** | Database moves from one valid state to another | Total balance remains same after transfer |
| **Isolation** | Concurrent transactions don't interfere | Two transfers on same account don't corrupt data |
| **Durability** | Committed data survives crashes | After commit, data persists even if power fails |

**Interview Answer**: "ACID ensures database reliability. In LawPrix, when a case is created and assigned to a lawyer, both operations succeed together (atomicity) - you can't have a case without an assignment."

---

## 2. Normalization

### Why Normalize?
- Reduce data redundancy
- Improve data integrity
- Eliminate update anomalies

### Normal Forms

| NF | Rule | Example Problem | Solution |
|----|------|-----------------|----------|
| **1NF** | Each cell has atomic value, no repeating groups | Address field has "Street, City, State" | Split into separate columns |
| **2NF** | 1NF + No partial dependency (all non-key depend on whole PK) | StudentID+CourseID -> StudentName (StudentName depends only on StudentID) | Move StudentName to Student table |
| **3NF** | 2NF + No transitive dependency (non-key -> non-key) | StudentID -> DeptID -> DeptName | Move DeptName to Department table |
| **BCNF** | Every determinant is a candidate key | More strict version of 3NF | Split table further |

### Denormalization
Sometimes add redundancy intentionally for read performance (common in real-world systems).

**Your Context**: "LawPrix uses a normalized schema for data integrity, but I denormalize frequently accessed case-lawyer combinations for faster dashboard queries."

---

## 3. Joins

### Types of Joins

```sql
-- INNER JOIN: Only matching rows from both tables
SELECT * FROM orders o
INNER JOIN customers c ON o.customer_id = c.id;

-- LEFT JOIN: All from left + matching from right
SELECT * FROM customers c
LEFT JOIN orders o ON c.id = o.customer_id;

-- RIGHT JOIN: All from right + matching from left
SELECT * FROM orders o
RIGHT JOIN customers c ON o.customer_id = c.id;

-- FULL OUTER JOIN: All from both tables
SELECT * FROM customers c
FULL OUTER JOIN orders o ON c.id = o.customer_id;

-- CROSS JOIN: Cartesian product (every combination)
SELECT * FROM colors CROSS JOIN sizes;

-- SELF JOIN: Table joined with itself
SELECT e.name, m.name as manager
FROM employees e
INNER JOIN employees m ON e.manager_id = m.id;
```

### JOIN Visual Guide
```
INNER JOIN:     A ∩ B (intersection)
LEFT JOIN:      A + (A ∩ B)
RIGHT JOIN:     B + (A ∩ B)
FULL OUTER:     A + B (union)
```

---

## 4. Window Functions

### Syntax
```sql
FUNCTION() OVER (
    PARTITION BY column  -- Optional: group by
    ORDER BY column      -- Optional: ordering within partition
    ROWS/RANGE ...       -- Optional: frame specification
)
```

### Ranking Functions
```sql
-- ROW_NUMBER(): Unique sequential (no ties)
SELECT *, ROW_NUMBER() OVER (ORDER BY salary DESC) as rank FROM employees;

-- RANK(): Ranking with gaps for ties
SELECT *, RANK() OVER (ORDER BY salary DESC) as rank FROM employees;
-- If two people are rank 2, next person is rank 4 (skips 3)

-- DENSE_RANK(): Ranking without gaps
SELECT *, DENSE_RANK() OVER (ORDER BY salary DESC) as rank FROM employees;
-- If two people are rank 2, next person is rank 3 (no skip)
```

### Aggregate Window Functions
```sql
-- Running total
SELECT amount, SUM(amount) OVER (ORDER BY date) as running_total FROM orders;

-- Moving average
SELECT amount, AVG(amount) OVER (
    ORDER BY date ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
) as moving_avg FROM orders;

-- Partition-wise aggregate
SELECT *, AVG(salary) OVER (PARTITION BY department) as dept_avg FROM employees;
```

### LAG/LEAD
```sql
-- Previous row value
SELECT date, revenue, LAG(revenue) OVER (ORDER BY date) as prev_day FROM sales;

-- Next row value
SELECT date, revenue, LEAD(revenue) OVER (ORDER BY date) as next_day FROM sales;
```

**LeetCode**: [#178 Rank Scores](https://leetcode.com/problems/rank-scores/), [#180 Consecutive Numbers](https://leetcode.com/problems/consecutive-numbers/), [#197 Rising Temperature](https://leetcode.com/problems/rising-temperature/)

---

## 5. SQL Query Execution Order

```
1. FROM        -- Determine tables
2. JOIN        -- Combine tables
3. WHERE       -- Filter rows (before grouping)
4. GROUP BY    -- Group rows
5. HAVING      -- Filter groups (after grouping)
6. SELECT      -- Choose columns
7. DISTINCT    -- Remove duplicates
8. ORDER BY    -- Sort results
9. LIMIT/OFFSET -- Limit results
```

### Key Insight
```sql
-- This FAILS because WHERE is evaluated before SELECT
SELECT name FROM employees WHERE salary > AVG(salary);

-- This WORKS using HAVING or subquery
SELECT department, AVG(salary) as avg_sal
FROM employees
GROUP BY department
HAVING AVG(salary) > 50000;
```

---

## 6. Indexes

### What is an Index?
Data structure (usually B-Tree) that speeds up data retrieval at the cost of extra storage and write overhead.

```sql
-- Create index
CREATE INDEX idx_email ON users(email);

-- Composite index
CREATE INDEX idx_name_email ON users(name, email);

-- Unique index
CREATE UNIQUE INDEX idx_unique_email ON users(email);
```

### When to Use Indexes
- WHERE clause columns
- JOIN columns
- ORDER BY columns
- Columns with high cardinality (many unique values)

### When NOT to Use Indexes
- Small tables
- Columns with low cardinality (e.g., boolean)
- Frequently updated columns (index maintenance cost)
- Tables with heavy write operations

### B-Tree vs Hash Index
| Aspect | B-Tree | Hash |
|--------|--------|------|
| Range Queries | Yes | No |
| Equality Queries | Yes | Yes (faster) |
| Ordering | Yes | No |
| Default | Most databases | Memory tables |

---

## 7. Common SQL Queries

### Second Highest Salary
```sql
-- Method 1: Subquery
SELECT MAX(salary) FROM employees
WHERE salary < (SELECT MAX(salary) FROM employees);

-- Method 2: LIMIT/OFFSET
SELECT DISTINCT salary FROM employees
ORDER BY salary DESC
LIMIT 1 OFFSET 1;

-- Method 3: DENSE_RANK
SELECT salary FROM (
    SELECT salary, DENSE_RANK() OVER (ORDER BY salary DESC) as rank
    FROM employees
) ranked WHERE rank = 2;
```

### Nth Highest Salary
```sql
SELECT DISTINCT salary FROM employees
ORDER BY salary DESC
LIMIT 1 OFFSET N-1;
```

### Employees Earning More Than Manager
```sql
SELECT e.name FROM employees e
INNER JOIN employees m ON e.manager_id = m.id
WHERE e.salary > m.salary;
```

### Delete Duplicate Emails
```sql
DELETE p1 FROM Person p1
INNER JOIN Person p2
WHERE p1.email = p2.email AND p1.id > p2.id;
```

### Department Highest Salary
```sql
SELECT department, name, salary FROM (
    SELECT *, DENSE_RANK() OVER (PARTITION BY department ORDER BY salary DESC) as rank
    FROM employees
) ranked WHERE rank = 1;
```

---

## 8. Transactions & Isolation Levels

### Isolation Levels
| Level | Dirty Read | Non-Repeatable | Phantom |
|-------|-----------|----------------|---------|
| Read Uncommitted | Yes | Yes | Yes |
| Read Committed | No | Yes | Yes |
| Repeatable Read | No | No | Yes |
| Serializable | No | No | No |

### Concurrency Problems
- **Dirty Read**: Reading uncommitted data
- **Non-Repeatable Read**: Same query returns different results
- **Phantom Read**: New rows appear between queries

---

## 9. Performance Optimization

### EXPLAIN Query
```sql
EXPLAIN SELECT * FROM employees WHERE department = 'Engineering';
```
Shows: scan type (full table vs index), rows examined, join type.

### Optimization Tips
1. **Use indexes** on WHERE, JOIN, ORDER BY columns
2. **Avoid SELECT *** - fetch only needed columns
3. **Use LIMIT** for large result sets
4. **Avoid subqueries** when JOINs work (sometimes)
5. **Use appropriate data types** (INT vs VARCHAR for IDs)
6. **Batch inserts** instead of individual inserts
7. **Use connection pooling** for frequently accessed data

---

## 10. SQL Interview Quick Answers

**Q: What is the difference between WHERE and HAVING?**
A: WHERE filters rows before GROUP BY. HAVING filters groups after GROUP BY. You can't use aggregate functions in WHERE, but you can in HAVING.

**Q: What is a primary key vs foreign key?**
A: Primary key uniquely identifies each row in a table (cannot be NULL). Foreign key is a column that references the primary key of another table (establishes relationship).

**Q: What is a subquery?**
A: A query nested inside another query. Can be in SELECT, FROM, WHERE, or HAVING. Correlated subquery references outer query.

**Q: UNION vs UNION ALL?**
A: UNION removes duplicates (slower). UNION ALL keeps all rows including duplicates (faster). Use UNION ALL when duplicates don't matter or are impossible.

**Q: What is a view?**
A: A virtual table based on a SQL query. Doesn't store data physically. Useful for simplifying complex queries and security (hiding columns).

**Q: Clustered vs Non-clustered index?**
A: Clustered index determines physical order of data (one per table). Non-clustered index creates separate structure with pointers (multiple allowed).

---

> **For Infosys**: They heavily test SQL queries (joins, window functions, subqueries). Practice writing queries by hand. The "Second Highest Salary" question is almost guaranteed.
