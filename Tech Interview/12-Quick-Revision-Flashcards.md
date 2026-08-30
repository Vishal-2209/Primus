---
created: 2026-08-30
purpose: One-liner Q&A for last-minute revision before Infosys interview
---
# Quick Revision Flashcards - Last Minute Prep

> Read this 30 minutes before the interview. One-liner answers for quick recall.

---

## DSA Patterns

| Question | Answer |
|----------|--------|
| Two Sum optimal approach? | HashMap O(n): store seen numbers, check complement |
| Sliding Window when to use? | Subarray/substring with condition, fixed/variable window |
| Binary Search requirement? | Sorted array or monotonic search space |
| BFS vs DFS? | BFS = level-by-level (shortest path). DFS = deep-first (all paths) |
| When to use Greedy? | Local optimal leads to global optimal (activity selection, intervals) |
| When to use DP? | Optimal substructure + overlapping subproblems |
| 0/1 Knapsack pattern? | dp[i][w] = max(take, skip) based on weight constraint |
| LCS pattern? | If chars match: dp[i-1][j-1] + 1. Else: max(dp[i-1][j], dp[i][j-1]) |
| Cycle detection algorithm? | Floyd's tortoise and hare (slow/fast pointers) |
| Topological sort use case? | Task scheduling, dependency resolution in DAG |

---

## OOP & Design Patterns

| Question | Answer |
|----------|--------|
| 4 Pillars of OOP? | Encapsulation, Abstraction, Inheritance, Polymorphism |
| SOLID in one line? | S: One job. O: Open for extension. L: Subtypes substitutable. I: Small interfaces. D: Depend on abstractions. |
| Singleton pattern? | One instance globally (DB connection, config). Use `__new__` in Python |
| Factory pattern? | Create objects without specifying exact class. Delegate to factory method |
| Abstract class vs interface? | Abstract: partial implementation. Interface: pure contract (ABC in Python) |
| Composition vs Inheritance? | HAS-A (flexible) vs IS-A (rigid). Prefer composition |
| Encapsulation vs Abstraction? | Encapsulation: hide internal state. Abstraction: hide complexity |

---

## DBMS

| Question | Answer |
|----------|--------|
| ACID properties? | Atomicity, Consistency, Isolation, Durability |
| 1NF/2NF/3NF? | 1NF: atomic values. 2NF: no partial dependency. 3NF: no transitive dependency |
| Query execution order? | FROM -> JOIN -> WHERE -> GROUP BY -> HAVING -> SELECT -> ORDER BY -> LIMIT |
| WHERE vs HAVING? | WHERE: filter rows (before GROUP BY). HAVING: filter groups (after GROUP BY) |
| INNER vs LEFT JOIN? | INNER: only matching. LEFT: all from left + matching from right |
| Primary key vs foreign key? | PK: unique identifier. FK: references PK of another table |
| Index B-Tree vs Hash? | B-Tree: range queries + equality. Hash: equality only (faster) |
| UNION vs UNION ALL? | UNION: removes duplicates. UNION ALL: keeps all (faster) |
| Window function purpose? | Aggregate across rows without collapsing them |
| RANK vs DENSE_RANK? | RANK: gaps in ranking. DENSE_RANK: no gaps |

---

## SQL Quick Queries

| Question | Answer |
|----------|--------|
| Second highest salary? | `SELECT MAX(salary) FROM Employee WHERE salary < (SELECT MAX(salary) FROM Employee)` |
| Delete duplicates? | `DELETE p1 FROM Person p1 JOIN Person p2 ON p1.email = p2.email AND p1.id > p2.id` |
| Find consecutive numbers? | Join table with itself on Id+1, Id+2 and check same value |
| Rank without gaps? | `DENSE_RANK() OVER (ORDER BY column DESC)` |
| Nth highest salary? | `SELECT DISTINCT salary FROM Employee ORDER BY salary DESC LIMIT 1 OFFSET N-1` |

---

## Operating Systems

| Question | Answer |
|----------|--------|
| Process vs thread? | Process: separate memory. Thread: shared memory within process |
| Deadlock 4 conditions? | Mutual exclusion, hold-and-wait, no preemption, circular wait |
| Virtual memory? | Maps virtual addresses to physical RAM via page tables |
| Mutex vs semaphore? | Mutex: binary, owned. Semaphore: counting, no ownership |
| Context switch? | Saving one process state and loading another (expensive) |
| Round Robin scheduling? | Preemptive, time quantum based, fair |
| Race condition? | Two threads accessing shared data, result depends on timing |

---

## Networking

| Question | Answer |
|----------|--------|
| OSI 7 layers? | Physical, Data Link, Network, Transport, Session, Presentation, Application |
| TCP vs UDP? | TCP: reliable, ordered, connection-oriented. UDP: fast, connectionless |
| HTTP status codes? | 200: OK. 201: Created. 400: Bad Request. 401: Unauthorized. 403: Forbidden. 404: Not Found. 500: Server Error |
| REST principles? | Stateless, uniform interface, client-server, cacheable |
| CORS? | Browser security blocking cross-origin requests. Server must allow via headers |
| HTTPS? | HTTP + TLS encryption. Port 443 |
| What happens typing google.com? | DNS -> TCP handshake -> TLS -> HTTP GET -> HTML render |

---

## AI/ML

| Question | Answer |
|----------|--------|
| Supervised vs unsupervised? | Supervised: labeled data. Unsupervised: find patterns, no labels |
| Classification vs regression? | Classification: discrete classes. Regression: continuous values |
| Overfitting signs? | Training accuracy >> test accuracy. Model memorizes data |
| Underfitting signs? | Both training and test accuracy low. Model too simple |
| Random Forest? | Ensemble of decision trees, majority vote, reduces overfitting |
| Precision vs recall? | Precision: few false positives. Recall: few false negatives |
| What is RAG? | Retrieve relevant docs, pass as context to LLM, ground responses in data |
| Embeddings? | Dense vector representations of text/images. Similar items = similar vectors |
| Cosine similarity? | Measures angle between vectors (ignores magnitude). Good for high-dim data |
| Bias-variance tradeoff? | High bias = underfit. High variance = overfit. Find the sweet spot |

---

## Project Quick Answers

| Question | Answer |
|----------|--------|
| LawPrix in one sentence? | Legal tech SaaS with ML case routing, RAG assessment, 3 portals, 22 Django apps |
| Nexus in one sentence? | Zero-auth resource aggregation, 98% cache hit, 1200+ users, Flask + React |
| PGPulse in one sentence? | Multi-tenant PG management, face recognition attendance, AI meal prediction |
| Why 22 Django apps? | One app per bounded concept, separate concerns, independent testing |
| Why Supabase for LawPrix? | Managed Postgres + auth + storage, focus on ML differentiator |
| Why Celery for case assessment? | LLM calls are slow, async processing, retry on failure |
| Zero-auth design? | Public Drive folders, backend proxies metadata, API key hidden server-side |
| Cache strategy? | TTL-based in-memory, 98% hit ratio, serves stale on API failure |

---

## Infosys-Specific Tips

| Tip | Detail |
|-----|--------|
| Online test pattern? | Aptitude (15-20 Qs, 25 min) + Coding (2-3 Qs, 45 min) |
| Difficulty level? | Easy to Medium (LeetCode Easy/Medium) |
| Technical interview focus? | DSA (1-2 problems), OOP, SQL queries, project walkthrough |
| Time allocation? | Problem 1: 10 min, Problem 2: 15 min, Problem 3: 20 min |
| Most asked SQL? | Second highest salary, joins, window functions |
| Most asked DSA? | Two pointer, sliding window, HashMap, binary search |
| Project depth? | They'll ask end-to-end flow, trade-offs, and why you made certain choices |

---

## Power Phrases for Interview

- "In my LawPrix project, I faced a similar challenge where..."
- "The approach I'd take is... and here's why"
- "I haven't implemented this yet, but here's how I would approach it"
- "A trade-off I considered was..."
- "That's a great question - let me think about that for a moment"
- "I'm not entirely sure, but based on my understanding..."
- "Could you clarify what you mean by...?"

---

## Last 5 Minutes Before Interview

1. Breathe. You've prepared well.
2. Remember: They called you because your resume impressed them.
3. You built 3 production projects with real users. That's rare.
4. Show your thought process, not just answers.
5. Be honest about gaps - it builds trust.

---

> **You've got this, Vishal. Good luck at PDEU!**
