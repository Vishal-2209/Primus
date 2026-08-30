## Introduction

Window functions compute aggregate values or perform ranking operations across sets of rows, without collapsing the result set like GROUP BY does. They operate on a "window" of rows defined by you, returning a single value for each row while keeping the original row visible in results.

**Key difference from GROUP BY:** Window functions preserve row identity while applying aggregate logic. GROUP BY reduces rows; window functions don't.

---

## 1. Basic Syntax

```sql
SELECT
    column1,
    column2,
    FUNCTION_NAME() OVER (window_specification) AS window_result
FROM table_name;
```

The OVER clause defines the window. At minimum, it can be empty:

```sql
SELECT salary, AVG(salary) OVER () AS avg_salary FROM employees;
```

This creates a window containing all rows.

---

## 2. Window Specification Components

### A. PARTITION BY

Divides rows into logical groups, applying the window function to each partition separately.

```sql
SELECT
    employee_id,
    department,
    salary,
    AVG(salary) OVER (PARTITION BY department) AS dept_avg
FROM employees;
```

Result: Each row shows the average salary for its department, not the global average.

**Multiple partitions:**

```sql
SELECT
    employee_id,
    department,
    job_title,
    salary,
    AVG(salary) OVER (PARTITION BY department, job_title) AS role_salary_avg
FROM employees;
```

### B. ORDER BY in Window Functions

Creates an order within each partition. Critical for ranking and sequential operations.

```sql
SELECT
    employee_id,
    department,
    salary,
    RANK() OVER (PARTITION BY department ORDER BY salary DESC) AS salary_rank
FROM employees;
```

Result: Ranks employees within each department by salary, highest first.

**Without PARTITION BY (global order):**

```sql
SELECT
    employee_id,
    salary,
    RANK() OVER (ORDER BY salary DESC) AS global_rank
FROM employees;
```

### C. Frame Specification

Defines a range of rows within a partition for aggregate operations.

```sql
SELECT
    sales_date,
    daily_sales,
    SUM(daily_sales) OVER (
        ORDER BY sales_date 
        ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
    ) AS three_day_moving_sum
FROM sales;
```

**Frame keywords:**

- ROWS: Physical row count
- RANGE: Value-based range (MySQL 8.0.14+)
- UNBOUNDED PRECEDING: From start of partition
- N PRECEDING: N rows before current row
- CURRENT ROW: Current row only
- N FOLLOWING: N rows after current row
- UNBOUNDED FOLLOWING: To end of partition

---

## 3. Window Function Types

### Type 1: Ranking Functions

These assign ranks or sequential numbers to rows.

#### ROW_NUMBER()

Assigns unique sequential integers, even for ties.

```sql
SELECT
    employee_id,
    department,
    salary,
    ROW_NUMBER() OVER (PARTITION BY department ORDER BY salary DESC) AS row_num
FROM employees;
```

**Output:**

```
emp_id | dept     | salary | row_num
1      | Sales    | 50000  | 1
2      | Sales    | 45000  | 2
3      | Sales    | 45000  | 3  <- Different row number despite same salary
4      | HR       | 40000  | 1
```

#### RANK()

Assigns ranks with gaps for ties. If two rows tie for 1st, the next gets rank 3.

```sql
SELECT
    employee_id,
    department,
    salary,
    RANK() OVER (PARTITION BY department ORDER BY salary DESC) AS rank
FROM employees;
```

**Output:**

```
emp_id | dept     | salary | rank
1      | Sales    | 50000  | 1
2      | Sales    | 45000  | 2
3      | Sales    | 45000  | 2  <- Same rank for same salary
4      | Sales    | 40000  | 4  <- Rank skips to 4
5      | HR       | 40000  | 1
```

#### DENSE_RANK()

Like RANK() but without gaps. No rank is skipped.

```sql
SELECT
    employee_id,
    department,
    salary,
    DENSE_RANK() OVER (PARTITION BY department ORDER BY salary DESC) AS dense_rank
FROM employees;
```

**Output:**

```
emp_id | dept     | salary | dense_rank
1      | Sales    | 50000  | 1
2      | Sales    | 45000  | 2
3      | Sales    | 45000  | 2  <- Same dense_rank
4      | Sales    | 40000  | 3  <- Dense_rank continues 3, no gap
5      | HR       | 40000  | 1
```

#### PERCENT_RANK()

Relative position as percentage: (rank - 1) / (total_rows - 1)

```sql
SELECT
    employee_id,
    salary,
    PERCENT_RANK() OVER (ORDER BY salary) AS pct_rank
FROM employees;
```

Result: 0.0 for first row, 1.0 for last row, proportional values for middle rows.

#### NTILE(n)

Divides rows into n roughly equal groups (buckets).

```sql
SELECT
    employee_id,
    salary,
    NTILE(4) OVER (ORDER BY salary) AS salary_quartile
FROM employees;
```

Result: Assigns 1, 2, 3, or 4 based on which quartile the salary falls into.

---

### Type 2: Aggregate Functions (with OVER)

Standard aggregate functions become window functions when used with OVER.

#### SUM()

```sql
SELECT
    order_id,
    order_date,
    amount,
    SUM(amount) OVER (ORDER BY order_date) AS cumulative_total
FROM orders;
```

Result: Running cumulative sum in chronological order.

**With frame:**

```sql
SELECT
    order_id,
    amount,
    SUM(amount) OVER (
        ORDER BY order_id 
        ROWS BETWEEN 1 PRECEDING AND 1 FOLLOWING
    ) AS three_row_sum
FROM orders;
```

This sums the previous row, current row, and next row.

#### AVG()

```sql
SELECT
    sales_date,
    daily_sales,
    AVG(daily_sales) OVER (
        ORDER BY sales_date 
        ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
    ) AS seven_day_moving_avg
FROM sales;
```

Result: 7-day moving average for each date.

#### COUNT()

```sql
SELECT
    employee_id,
    department,
    hire_date,
    COUNT(*) OVER (PARTITION BY department ORDER BY hire_date) AS cumulative_hires
FROM employees;
```

Result: Running count of cumulative hires by department in hire order.

#### MIN() / MAX()

```sql
SELECT
    employee_id,
    salary,
    MIN(salary) OVER (ORDER BY employee_id) AS running_min_salary,
    MAX(salary) OVER (ORDER BY employee_id) AS running_max_salary
FROM employees;
```

Result: Running minimum and maximum values up to current row.

---

### Type 3: Offset Functions

Access values from other rows without self-joins.

#### LAG()

Accesses the previous row's value.

```sql
SELECT
    employee_id,
    hire_date,
    salary,
    LAG(salary, 1) OVER (ORDER BY hire_date) AS previous_salary,
    salary - LAG(salary, 1) OVER (ORDER BY hire_date) AS salary_increase
FROM employees;
```

Result: Shows each employee's salary increase compared to the employee hired before them.

**Syntax:** `LAG(column, offset, default_value) OVER (ORDER BY ...)`

- offset: How many rows back (default 1)
- default_value: Value if no previous row exists (default NULL)

```sql
SELECT
    sales_date,
    sales,
    LAG(sales, 1, 0) OVER (ORDER BY sales_date) AS previous_day_sales,
    sales - LAG(sales, 1, 0) OVER (ORDER BY sales_date) AS daily_change
FROM daily_sales;
```

#### LEAD()

Accesses the next row's value. Opposite of LAG.

```sql
SELECT
    employee_id,
    hire_date,
    salary,
    LEAD(salary, 1) OVER (ORDER BY hire_date) AS next_employee_salary
FROM employees;
```

Result: Shows the next hired employee's salary.

#### FIRST_VALUE()

Returns the first value in the window frame.

```sql
SELECT
    employee_id,
    department,
    salary,
    FIRST_VALUE(salary) OVER (
        PARTITION BY department 
        ORDER BY salary DESC
    ) AS highest_salary_in_dept
FROM employees;
```

Result: Every row shows the highest salary in its department.

#### LAST_VALUE()

Returns the last value in the window frame. Requires explicit frame specification!

```sql
SELECT
    employee_id,
    department,
    salary,
    LAST_VALUE(salary) OVER (
        PARTITION BY department 
        ORDER BY salary DESC
        ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
    ) AS lowest_salary_in_dept
FROM employees;
```

**Critical:** Without ROWS BETWEEN, the default frame is "UNBOUNDED PRECEDING to CURRENT ROW", making LAST_VALUE return only the current row's value. Always specify the frame for LAST_VALUE.

#### NTH_VALUE()

Returns the Nth value in the frame.

```sql
SELECT
    employee_id,
    department,
    salary,
    NTH_VALUE(salary, 2) OVER (
        PARTITION BY department 
        ORDER BY salary DESC
        ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
    ) AS second_highest_salary
FROM employees;
```

---

## 4. Practical Cases and Examples

### Case 1: Employee Salary Analysis

```sql
SELECT
    employee_id,
    name,
    department,
    salary,
    ROUND(AVG(salary) OVER (PARTITION BY department), 2) AS dept_avg_salary,
    ROUND(salary - AVG(salary) OVER (PARTITION BY department), 2) AS variance_from_avg,
    RANK() OVER (PARTITION BY department ORDER BY salary DESC) AS dept_rank,
    RANK() OVER (ORDER BY salary DESC) AS company_rank
FROM employees;
```

### Case 2: Running Totals with Reset

```sql
-- Track cumulative sales per salesperson, resetting monthly
SELECT
    salesperson_id,
    sale_date,
    amount,
    MONTH(sale_date) AS sale_month,
    SUM(amount) OVER (
        PARTITION BY salesperson_id, YEAR(sale_date), MONTH(sale_date)
        ORDER BY sale_date
    ) AS monthly_cumulative_sales
FROM sales;
```

### Case 3: Year-over-Year Comparison

```sql
SELECT
    product_id,
    sale_date,
    sales_amount,
    LAG(sales_amount) OVER (
        PARTITION BY product_id 
        ORDER BY YEAR(sale_date), MONTH(sale_date)
    ) AS previous_month_sales,
    ROUND(
        ((sales_amount - LAG(sales_amount) OVER (
            PARTITION BY product_id 
            ORDER BY YEAR(sale_date), MONTH(sale_date)
        )) / LAG(sales_amount) OVER (
            PARTITION BY product_id 
            ORDER BY YEAR(sale_date), MONTH(sale_date)
        ) * 100),
        2
    ) AS month_over_month_growth_pct
FROM monthly_sales;
```

### Case 4: Top N Records per Group

```sql
-- Get top 3 earners per department
SELECT *
FROM (
    SELECT
        employee_id,
        name,
        department,
        salary,
        ROW_NUMBER() OVER (PARTITION BY department ORDER BY salary DESC) AS dept_rank
    FROM employees
) ranked
WHERE dept_rank <= 3;
```

### Case 5: Moving Average (Smoothing Data)

```sql
-- 30-day moving average of stock prices
SELECT
    stock_date,
    closing_price,
    ROUND(
        AVG(closing_price) OVER (
            ORDER BY stock_date 
            ROWS BETWEEN 29 PRECEDING AND CURRENT ROW
        ),
        2
    ) AS moving_avg_30day
FROM stock_prices;
```

### Case 6: Percentile Analysis

```sql
SELECT
    customer_id,
    order_amount,
    PERCENT_RANK() OVER (ORDER BY order_amount) AS percentile_rank,
    NTILE(4) OVER (ORDER BY order_amount) AS quartile
FROM orders;
```

Result: Customers in 4th quartile have order_amount in top 25%.

### Case 7: Streak Detection

```sql
-- Identify winning streaks in game results
SELECT
    player_id,
    game_date,
    result,
    ROW_NUMBER() OVER (
        PARTITION BY player_id, result 
        ORDER BY game_date
    ) - ROW_NUMBER() OVER (
        PARTITION BY player_id 
        ORDER BY game_date
    ) AS streak_group,
    COUNT(*) OVER (
        PARTITION BY player_id,
        ROW_NUMBER() OVER (
            PARTITION BY player_id, result 
            ORDER BY game_date
        ) - ROW_NUMBER() OVER (
            PARTITION BY player_id 
            ORDER BY game_date
        )
    ) AS streak_length
FROM game_results
ORDER BY player_id, game_date;
```

### Case 8: Pagination with Ranking

```sql
-- Get pages of results, 10 per page
SELECT * FROM (
    SELECT
        product_id,
        name,
        price,
        ROW_NUMBER() OVER (ORDER BY price DESC) AS row_num
    FROM products
) p
WHERE ROW_NUMBER() BETWEEN 21 AND 30;  -- Page 3
```

### Case 9: Cumulative Distribution

```sql
-- Find at what salary level an employee falls percentile-wise
SELECT
    employee_id,
    salary,
    SUM(COUNT(*)) OVER (
        ORDER BY salary 
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) as cumulative_employee_count,
    ROUND(
        100.0 * SUM(COUNT(*)) OVER (
            ORDER BY salary 
            ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
        ) / COUNT(*) OVER (),
        2
    ) AS percentile
FROM employees
GROUP BY salary;
```

### Case 10: Before-After Comparison

```sql
-- Compare metrics before and after a campaign
SELECT
    customer_id,
    campaign_date,
    LEAD(purchase_count) OVER (
        PARTITION BY customer_id 
        ORDER BY campaign_date
    ) - purchase_count AS purchase_increase,
    LEAD(purchase_amount) OVER (
        PARTITION BY customer_id 
        ORDER BY campaign_date
    ) - purchase_amount AS revenue_increase
FROM customer_metrics
WHERE campaign_date IS NOT NULL;
```

### Case 11: Identifying Gaps and Islands

```sql
-- Find consecutive date gaps in orders
SELECT
    order_id,
    order_date,
    DATEDIFF(
        order_date,
        LAG(order_date) OVER (ORDER BY order_date)
    ) AS days_since_previous_order
FROM orders
HAVING days_since_previous_order > 7;
```

### Case 12: Ranking with Ties (Same Percentile)

```sql
-- Show all employees in top 10% by salary
SELECT
    employee_id,
    salary,
    PERCENT_RANK() OVER (ORDER BY salary DESC) AS pct_rank
FROM employees
WHERE PERCENT_RANK() OVER (ORDER BY salary DESC) <= 0.10;
```

---

## 5. Frame Specification Details

### Default Frame Behavior

Without explicit frame specification, the default depends on ORDER BY:

- With ORDER BY: `ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`
- Without ORDER BY: `ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING`

This matters for aggregate functions like SUM and AVG!

```sql
-- These produce different results:
SELECT
    sales_date,
    daily_sales,
    SUM(daily_sales) OVER (ORDER BY sales_date) AS running_sum,
    SUM(daily_sales) OVER (ORDER BY sales_date ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING) AS total_sum
FROM sales;
```

First column: Running total up to current row. Second column: All-time total for every row.

### RANGE vs ROWS

ROWS counts physical rows; RANGE matches based on values.

```sql
-- ROWS frame
SELECT
    salary,
    COUNT(*) OVER (
        ORDER BY salary 
        ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
    ) AS row_count_in_frame
FROM employees;

-- RANGE frame (MySQL 8.0.14+)
SELECT
    salary,
    COUNT(*) OVER (
        ORDER BY salary 
        RANGE BETWEEN 1000 PRECEDING AND 1000 FOLLOWING
    ) AS employees_within_1000_of_salary
FROM employees;
```

---

## 6. Common Mistakes and Solutions

### Mistake 1: Forgetting PARTITION BY in Multiple Partitions

```sql
-- Wrong: Ignores department
SELECT
    employee_id,
    department,
    salary,
    RANK() OVER (ORDER BY salary DESC) AS rank
FROM employees;

-- Correct:
SELECT
    employee_id,
    department,
    salary,
    RANK() OVER (PARTITION BY department ORDER BY salary DESC) AS rank
FROM employees;
```

### Mistake 2: LAST_VALUE Without Full Frame

```sql
-- Wrong: Returns current row value due to default frame
SELECT
    salary,
    LAST_VALUE(salary) OVER (ORDER BY salary)
FROM employees;

-- Correct:
SELECT
    salary,
    LAST_VALUE(salary) OVER (ORDER BY salary ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING)
FROM employees;
```

### Mistake 3: Using Window Function in WHERE

```sql
-- Wrong: Window functions can't be in WHERE
SELECT *
FROM employees
WHERE RANK() OVER (ORDER BY salary DESC) <= 3;

-- Correct: Use subquery or CTE
SELECT * FROM (
    SELECT
        employee_id,
        salary,
        RANK() OVER (ORDER BY salary DESC) AS rank
    FROM employees
) ranked
WHERE rank <= 3;
```

### Mistake 4: Multiple ORDER BY Columns Without Understanding

```sql
SELECT
    employee_id,
    department,
    hire_date,
    salary,
    ROW_NUMBER() OVER (PARTITION BY department ORDER BY salary DESC, hire_date ASC) AS row_num
FROM employees;
```

This ranks by salary (highest first), then by hire_date (earliest first) for salary ties.

---

## 7. Performance Considerations

### Index Strategy

```sql
-- Create indexes for window function performance
CREATE INDEX idx_dept_salary ON employees(department, salary);
CREATE INDEX idx_date_amount ON sales(sale_date, amount);
```

### Avoiding Expensive Operations

```sql
-- Inefficient: Multiple window function calls
SELECT
    employee_id,
    AVG(salary) OVER (PARTITION BY department) AS dept_avg,
    SUM(bonus) OVER (PARTITION BY department) AS dept_bonus_total
FROM employees;

-- Better: Compute once in CTE if querying multiple times
WITH dept_stats AS (
    SELECT
        employee_id,
        department,
        AVG(salary) OVER (PARTITION BY department) AS dept_avg,
        SUM(bonus) OVER (PARTITION BY department) AS dept_bonus_total
    FROM employees
)
SELECT * FROM dept_stats;
```

### Frame Specification Impact

```sql
-- UNBOUNDED FOLLOWING can be expensive
SELECT
    date,
    value,
    SUM(value) OVER (
        ORDER BY date 
        ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
    ) AS total
FROM metrics;

-- Better when possible: Use PARTITION BY to limit scope
SELECT
    date,
    department,
    value,
    SUM(value) OVER (
        PARTITION BY department 
        ORDER BY date 
        ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
    ) AS dept_total
FROM metrics;
```

---

## 8. Version Compatibility

MySQL window functions were introduced in MySQL 8.0. Not available in:

- MySQL 5.7 and earlier
- MariaDB < 10.2 (though MariaDB support is partial)

Check version:

```sql
SELECT VERSION();
```

---

## 9. Quick Reference Table

|Function|Purpose|Example|
|---|---|---|
|ROW_NUMBER()|Sequential numbering|Pagination, deduplication|
|RANK()|Ranking with gaps|Sports standings|
|DENSE_RANK()|Ranking without gaps|Top N per group|
|PERCENT_RANK()|Percentile position|Performance tiers|
|NTILE(n)|Bucket assignment|Quartiles, deciles|
|SUM()|Running total|Cumulative sales|
|AVG()|Moving average|Trend smoothing|
|COUNT()|Running count|Progress tracking|
|LAG()|Previous row value|Day-over-day change|
|LEAD()|Next row value|Forecasting|
|FIRST_VALUE()|First in frame|Baseline comparison|
|LAST_VALUE()|Last in frame|Range bounds|
|NTH_VALUE()|Nth in frame|Specific position|

---

## Summary

Window functions are powerful for analytics without collapsing data. Master these patterns:

1. Always specify PARTITION BY if you need per-group analysis
2. Always specify ORDER BY for ranking and offset functions
3. Explicitly specify frame for aggregate functions when needed
4. Use CTEs to avoid redundant window function calls
5. Remember LAST_VALUE needs explicit frame specification
6. Use subqueries to filter window function results
7. Index columns used in PARTITION BY and ORDER BY

Window functions transform complex multi-step queries into elegant, readable SQL.