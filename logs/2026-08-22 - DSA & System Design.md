## SQL Practice

Today I solved some SQL problems on LeetCode along with an attempt to solve one system design question. Also, now I am going to cope up with leetcode along with the concepts of FastAPI, Django, Flask, AI/ML, etc.

### 196. Delete Duplicate Emails

**Difficulty:** Easy  
**Concepts:** Self Join, `DELETE`, Duplicate Removal

The goal was to delete duplicate emails while keeping the row with the **smallest `id`**.

```sql
DELETE p1
FROM Person p1
JOIN Person p2
ON p1.email = p2.email
AND p1.id > p2.id;
```

The key idea is to perform a **self join**. For two rows with the same email, the row having the larger `id` is considered the duplicate and is deleted.

**Pattern:**

```sql
DELETE p1
FROM Table p1
JOIN Table p2
ON p1.<duplicate_column> = p2.<duplicate_column>
AND p1.id > p2.id;
```

This preserves the record with the smallest `id`.

---

### 511. Game Play Analysis I

**Difficulty:** Easy  
**Concepts:** `GROUP BY`, `MIN()`, Aggregate Functions

The goal was to find the **first login date for each player**.

```sql
SELECT
    player_id,
    MIN(event_date) AS first_login
FROM Activity
GROUP BY player_id;
```

The solution groups records by `player_id` and uses `MIN(event_date)` to retrieve the earliest login date for each player.

**General pattern:**

```sql
SELECT
    entity_id,
    MIN(date_column)
FROM table
GROUP BY entity_id;
```

This pattern is useful whenever we need the **earliest value for each entity**. Similarly, `MAX()` can be used when the latest value is required.

---
## System Design

Today I started **System Design** with the classic **LRU Cache** problem.

### Q1. LRU Cache

**Difficulty:** Medium  
**Concepts:** HashMap, Doubly Linked List, Cache Eviction, `O(1)` Operations

The objective is to design a cache following the **Least Recently Used (LRU)** policy.
The cache supports:

```text
get(key)
put(key, value)
```

Both operations must run in **O(1) average time**. The main challenge is maintaining both fast lookup and usage order. A simple list can maintain the order, but searching for a key or moving an element can take `O(n)`. A HashMap provides `O(1)` lookup, but does not maintain ordering.

Therefore, the standard solution combines:

```text
HashMap + Doubly Linked List
```

The **HashMap** stores:

```text
key → node
```

This allows the cache to locate any key in `O(1)` average time. The **Doubly Linked List** maintains the usage order:

```text
LRU ←→ ... ←→ ... ←→ MRU
```

where:

- `LRU` = Least Recently Used
- `MRU` = Most Recently Used

When a key is accessed or inserted, it is moved to the **MRU** position. When the cache exceeds its capacity, the node at the **LRU** position is removed.

#### `get(key)`

If the key does not exist:

```text
return -1
```

Otherwise:

1. Find the node through the HashMap.
    
2. Remove it from its current position.
    
3. Move it to the MRU position.
    
4. Return its value.
    

**Complexity:** `O(1)`

#### `put(key, value)`

If the key already exists:

1. Update its value.
    
2. Move the node to MRU.
    

If the key does not exist:

1. Create a new node.
    
2. Add it to the HashMap.
    
3. Add it to the MRU position.
    
4. If capacity is exceeded, remove the LRU node and its HashMap entry.
    

**Complexity:** `O(1)`

#### Core Insight

The important takeaway from LRU Cache is how two data structures can be combined to satisfy strict performance requirements:

```text
HashMap
    ↓
O(1) key lookup

Doubly Linked List
    ↓
O(1) insertion, deletion and reordering

Together
    ↓
O(1) average get() and put()
```

This was my starting point for **System Design**, focusing first on understanding the underlying data-structure design before moving toward larger system-level problems.