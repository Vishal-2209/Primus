---
created: 2026-08-30
purpose: Top SQL queries for Infosys interview with solutions
---

# SQL Queries Practice - Infosys Campus Placement

> Track your progress by checking off queries as you practice them.

## Progress Tracker

| # | Status | Problem | LeetCode |
|---|--------|---------|----------|
| 1 | `[ ]` | Second Highest Salary | [LC](https://leetcode.com/problems/second-highest-salary/) |
| 2 | `[ ]` | Nth Highest Salary | [LC](https://leetcode.com/problems/nth-highest-salary/) |
| 3 | `[ ]` | Delete Duplicate Emails | [LC](https://leetcode.com/problems/delete-duplicate-emails/) |
| 4 | `[ ]` | Game Play Analysis I | [LC](https://leetcode.com/problems/game-play-analysis-i/) |
| 5 | `[ ]` | Sales Person (NOT EXISTS) | [LC](https://leetcode.com/problems/sales-person/) |
| 6 | `[ ]` | Department Highest Salary | [LC](https://leetcode.com/problems/department-highest-salary/) |
| 7 | `[ ]` | Consecutive Numbers | [LC](https://leetcode.com/problems/consecutive-numbers/) |
| 8 | `[ ]` | Rank Scores | [LC](https://leetcode.com/problems/rank-scores/) |
| 9 | `[ ]` | Employees Earning More Than Manager | [LC](https://leetcode.com/problems/employees-earning-more-than-their-managers/) |
| 10 | `[ ]` | Customers Who Never Order | [LC](https://leetcode.com/problems/customers-who-never-order/) |
| 11 | `[ ]` | Exchange Seats | [LC](https://leetcode.com/problems/exchange-seats/) |
| 12 | `[ ]` | Department Top Three Salaries | [LC](https://leetcode.com/problems/department-top-three-salaries/) |
| 13 | `[ ]` | Rising Temperature | [LC](https://leetcode.com/problems/rising-temperature/) |
| 14 | `[ ]` | Immediate Food Delivery | [LC](https://leetcode.com/problems/immediate-food-delivery-i/) |
| 15 | `[ ]` | Friend Requests II | [LC](https://leetcode.com/problems/friend-requests-ii-who-has-the-most-friends/) |

---

## Must-Know Queries (Very High Probability)

### 1. Second Highest Salary
**Problem**: Find the second highest salary from Employee table.

```sql
-- Method 1: Subquery
SELECT MAX(salary) AS SecondHighestSalary
FROM Employee
WHERE salary < (SELECT MAX(salary) FROM Employee);

-- Method 2: LIMIT/OFFSET
SELECT DISTINCT salary AS SecondHighestSalary
FROM Employee
ORDER BY salary DESC
LIMIT 1 OFFSET 1;

-- Method 3: DENSE_RANK (handles ties correctly)
SELECT MAX(salary) AS SecondHighestSalary
FROM (
    SELECT salary, DENSE_RANK() OVER (ORDER BY salary DESC) AS rnk
    FROM Employee
) ranked
WHERE rnk = 2;

-- Method 4: No second highest (edge case)
SELECT IFNULL(
    (SELECT DISTINCT salary FROM Employee ORDER BY salary DESC LIMIT 1 OFFSET 1),
    NULL
) AS SecondHighestSalary;
```

**LeetCode**: [#176 Second Highest Salary](https://leetcode.com/problems/second-highest-salary/)

---

### 2. Nth Highest Salary
**Problem**: Find the Nth highest salary.

```sql
CREATE FUNCTION getNthHighestSalary(N INT) RETURNS INT
BEGIN
    RETURN (
        SELECT DISTINCT salary
        FROM Employee
        ORDER BY salary DESC
        LIMIT 1 OFFSET N-1
    );
END
```

**LeetCode**: [#177 Nth Highest Salary](https://leetcode.com/problems/nth-highest-salary/)

---

### 3. Delete Duplicate Emails
**Problem**: Delete duplicate emails, keep the one with smallest id.

```sql
DELETE p1 FROM Person p1
INNER JOIN Person p2
WHERE p1.email = p2.email
  AND p1.id > p2.id;
```

**LeetCode**: [#196 Delete Duplicate Emails](https://leetcode.com/problems/delete-duplicate-emails/)

---

### 4. Game Play Analysis I
**Problem**: Find first login date for each player.

```sql
SELECT
    player_id,
    MIN(event_date) AS first_login
FROM Activity
GROUP BY player_id;
```

**LeetCode**: [#511 Game Play Analysis I](https://leetcode.com/problems/game-play-analysis-i/)

---

### 5. Sales Person (NOT EXISTS)
**Problem**: Find salespersons with no orders related to company "RED".

```sql
SELECT s.name
FROM SalesPerson s
WHERE NOT EXISTS (
    SELECT 1
    FROM Orders o
    JOIN Company c ON c.com_id = o.com_id
    WHERE o.sales_id = s.sales_id
      AND c.name = 'RED'
);
```

**LeetCode**: [#607 Sales Person](https://leetcode.com/problems/sales-person/)

---

### 6. Department Highest Salary
**Problem**: Find employees who earn the highest salary in each department.

```sql
-- Method 1: DENSE_RANK
SELECT department, name, salary
FROM (
    SELECT
        e.name,
        e.salary,
        d.name AS department,
        DENSE_RANK() OVER (PARTITION BY d.name ORDER BY e.salary DESC) AS rnk
    FROM Employee e
    JOIN Department d ON e.departmentId = d.id
) ranked
WHERE rnk = 1;

-- Method 2: Subquery
SELECT d.name AS Department, e.name AS Employee, e.salary
FROM Employee e
JOIN Department d ON e.departmentId = d.id
WHERE e.salary = (
    SELECT MAX(salary) FROM Employee WHERE departmentId = d.id
);
```

**LeetCode**: [#184 Department Highest Salary](https://leetcode.com/problems/department-highest-salary/)

---

### 7. Consecutive Numbers
**Problem**: Find all numbers that appear at least three times consecutively.

```sql
SELECT DISTINCT l1.Num AS ConsecutiveNums
FROM Logs l1
JOIN Logs l2 ON l1.Id = l2.Id - 1
JOIN Logs l3 ON l1.Id = l3.Id - 2
WHERE l1.Num = l2.Num AND l2.Num = l3.Num;
```

**LeetCode**: [#180 Consecutive Numbers](https://leetcode.com/problems/consecutive-numbers/)

---

### 8. Rank Scores
**Problem**: Rank scores with ties getting same rank, no gaps.

```sql
SELECT
    score,
    DENSE_RANK() OVER (ORDER BY score DESC) AS rank
FROM Scores;
```

**LeetCode**: [#178 Rank Scores](https://leetcode.com/problems/rank-scores/)

---

### 9. Employees Earning More Than Manager
**Problem**: Find employees who earn more than their managers.

```sql
-- Method 1: Self Join
SELECT e.name AS Employee
FROM Employee e
JOIN Employee m ON e.managerId = m.id
WHERE e.salary > m.salary;

-- Method 2: Subquery
SELECT name AS Employee
FROM Employee
WHERE salary > (
    SELECT salary FROM Employee WHERE id = managerId
);
```

**LeetCode**: [#181 Employees Earning More Than Their Managers](https://leetcode.com/problems/employees-earning-more-than-their-managers/)

---

### 10. Customers Who Never Order
**Problem**: Find customers who never placed an order.

```sql
-- Method 1: LEFT JOIN
SELECT c.name AS Customers
FROM Customers c
LEFT JOIN Orders o ON c.id = o.customerId
WHERE o.id IS NULL;

-- Method 2: NOT IN
SELECT name AS Customers
FROM Customers
WHERE id NOT IN (SELECT customerId FROM Orders);

-- Method 3: NOT EXISTS
SELECT c.name AS Customers
FROM Customers c
WHERE NOT EXISTS (
    SELECT 1 FROM Orders o WHERE o.customerId = c.id
);
```

**LeetCode**: [#183 Customers Who Never Order](https://leetcode.com/problems/customers-who-never-order/)

---

## Medium Difficulty Queries

### 11. Exchange Seats
**Problem**: Swap consecutive seat ids, odd students swap with next, last odd stays.

```sql
SELECT
    CASE
        WHEN id % 2 = 1 AND id < (SELECT MAX(id) FROM Seat) THEN id + 1
        WHEN id % 2 = 0 THEN id - 1
        ELSE id
    END AS id,
    student
FROM Seat
ORDER BY id;
```

**LeetCode**: [#626 Exchange Seats](https://leetcode.com/problems/exchange-seats/)

---

### 12. Department Top Three Salaries
**Problem**: Find top 3 highest salaries in each department.

```sql
SELECT department, employee, salary
FROM (
    SELECT
        d.name AS department,
        e.name AS employee,
        e.salary,
        DENSE_RANK() OVER (PARTITION BY d.name ORDER BY e.salary DESC) AS rnk
    FROM Employee e
    JOIN Department d ON e.departmentId = d.id
) ranked
WHERE rnk <= 3;
```

**LeetCode**: [#185 Department Top Three Salaries](https://leetcode.com/problems/department-top-three-salaries/)

---

### 13. Rising Temperature
**Problem**: Find ids with higher temperature than previous day.

```sql
SELECT w1.id
FROM Weather w1
JOIN Weather w2
  ON DATEDIFF(w1.recordDate, w2.recordDate) = 1
WHERE w1.temperature > w2.temperature;
```

**LeetCode**: [#197 Rising Temperature](https://leetcode.com/problems/rising-temperature/)

---

### 14. Immediate Food Delivery
**Problem**: Find percentage of immediate orders (order_date = customer_pref_delivery_date).

```sql
SELECT
    ROUND(
        100.0 * SUM(CASE WHEN order_date = customer_pref_delivery_date THEN 1 ELSE 0 END)
        / COUNT(*),
        2
    ) AS immediate_percentage
FROM Delivery;
```

**LeetCode**: [#1173 Immediate Food Delivery I](https://leetcode.com/problems/immediate-food-delivery-i/)

---

### 15. Friend Requests II
**Problem**: Find the person with the most friends (acceptance rate).

```sql
SELECT
    id,
    COUNT(*) AS num
FROM (
    SELECT requester_id AS id FROM RequestAccepted
    UNION ALL
    SELECT accepter_id AS id FROM RequestAccepted
) all_ids
GROUP BY id
ORDER BY num DESC
LIMIT 1;
```

**LeetCode**: [#602 Friend Requests II: Who Has the Most Friends](https://leetcode.com/problems/friend-requests-ii-who-has-the-most-friends/)

---

## Infosys-Specific Practice Tips

### Query Writing Approach
1. **Understand the requirement** - read carefully
2. **Identify tables** needed
3. **Determine join type** (INNER, LEFT, etc.)
4. **Write WHERE conditions**
5. **Handle grouping** (GROUP BY, HAVING)
6. **Add sorting** (ORDER BY)
7. **Test edge cases** (NULLs, empty results)

### Common Mistakes to Avoid
- Forgetting GROUP BY when using aggregate functions
- Using WHERE instead of HAVING for aggregate conditions
- Not handling NULL values
- Incorrect JOIN conditions
- Forgetting DISTINCT when needed

---

> **For Infosys**: Practice writing these queries by hand on paper. They often ask you to write SQL on a whiteboard or paper during the interview. Speed and accuracy both matter.
