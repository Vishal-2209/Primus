## 607. Sales Person

**Question:** Find the names of salespersons who did not have any orders related to the company **"RED"**.

**Answer:**
```sql
SELECT s.name
FROM SalesPerson s
WHERE NOT EXISTS (
    SELECT 1
    FROM Orders o
    JOIN Company c
        ON c.com_id = o.com_id
    WHERE o.sales_id = s.sales_id
      AND c.name = 'RED'
);
```

**Explanation:**  
For each salesperson, `NOT EXISTS` checks whether they have an order associated with the company `RED`. If such an order exists, the salesperson is excluded; otherwise, their name is returned. `Orders` is joined with `Company` to identify the company by name.

**Key Concept:** `NOT EXISTS` is useful for finding records that **do not have a related record satisfying a condition**.