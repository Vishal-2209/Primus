---
created: 2026-08-30
purpose: System Design basics for Infosys interview
---
# System Design Basics - Infosys Interview

## 1. LRU Cache (Most Asked)

**Problem**: Design a cache with O(1) get and put operations.

### Solution: HashMap + Doubly Linked List

```python
class Node:
    def __init__(self, key=0, val=0):
        self.key = key
        self.val = val
        self.prev = None
        self.next = None

class LRUCache:
    def __init__(self, capacity):
        self.capacity = capacity
        self.cache = {}  # key -> Node
        self.head = Node()  # dummy head
        self.tail = Node()  # dummy tail
        self.head.next = self.tail
        self.tail.prev = self.head
    
    def _remove(self, node):
        node.prev.next = node.next
        node.next.prev = node.prev
    
    def _add_to_front(self, node):
        node.next = self.head.next
        node.prev = self.head
        self.head.next.prev = node
        self.head.next = node
    
    def get(self, key):
        if key in self.cache:
            node = self.cache[key]
            self._remove(node)
            self._add_to_front(node)
            return node.val
        return -1
    
    def put(self, key, val):
        if key in self.cache:
            self._remove(self.cache[key])
        node = Node(key, val)
        self.cache[key] = node
        self._add_to_front(node)
        if len(self.cache) > self.capacity:
            lru = self.tail.prev
            self._remove(lru)
            del self.cache[lru.key]
```

**LeetCode**: [#146 LRU Cache](https://leetcode.com/problems/lru-cache/)

**Interview Answer**: "HashMap gives O(1) lookup. Doubly linked list maintains access order - recently used items move to front, evict from back when capacity exceeded."

---

## 2. Caching Strategies

### Cache-Aside (Lazy Loading)
```
App checks cache first
  -> Hit: return cached data
  -> Miss: fetch from DB, store in cache, return
```
**Used in**: Most web applications, including your LawPrix.

### Write-Through
```
Write goes to cache AND database simultaneously
Always consistent, but slower writes
```

### Write-Behind (Write-Back)
```
Write goes to cache first, async flush to DB
Fast writes, but risk of data loss
```

### Cache Invalidation Strategies
| Strategy | Description | Trade-off |
|----------|-------------|-----------|
| TTL (Time-to-Live) | Auto-expire after N seconds | May serve stale data |
| Event-based | Invalidate on data change | Complex implementation |
| Manual | Clear cache on deploy | Simple but infrequent |

**Your Context**: "In Nexus, I use TTL-based in-memory caching for Google Drive metadata. TTL is set based on how often the underlying Drive files change. This gives me 98% cache hit ratio."

---

## 3. Load Balancing

### Algorithms
| Algorithm | How it Works | Best For |
|-----------|--------------|----------|
| Round Robin | Sequential distribution | Equal capacity servers |
| Least Connections | Fewest active conns | Varying request duration |
| IP Hash | Same IP -> same server | Session persistence |
| Weighted | Proportional to capacity | Heterogeneous servers |

### Health Checks
Load balancer periodically checks if servers are alive. Unhealthy servers removed from pool until recovered.

---

## 4. Database Scaling

### Vertical Scaling (Scale Up)
- Add more CPU, RAM, disk to existing server
- Simple but has limits
- Single point of failure

### Horizontal Scaling (Scale Out)
- Add more servers
- Requires data distribution

### Sharding
Split database across multiple servers by a shard key.

```
Users 1-1000   -> Shard A
Users 1001-2000 -> Shard B
Users 2001-3000 -> Shard C
```

**Challenges**: Cross-shard queries, rebalancing, hotspots.

### Read Replicas
- Primary handles writes
- Replicas handle reads
- Asynchronous replication

**Your Context**: "LawPrix uses Supabase which handles this automatically. PGPulse uses PostgreSQL which could be extended with read replicas for scaling."

---

## 5. CAP Theorem

**In a distributed system, you can only guarantee 2 out of 3:**

| Property | Description |
|----------|-------------|
| **Consistency** | All nodes see the same data at the same time |
| **Availability** | Every request gets a response (success/failure) |
| **Partition Tolerance** | System works despite network failures |

### Real-World Choices
| System | Choice | Why |
|--------|--------|-----|
| Traditional RDBMS | CA | Consistency + Availability (no partition tolerance) |
| DynamoDB, Cassandra | AP | Availability + Partition tolerance (eventual consistency) |
| MongoDB, HBase | CP | Consistency + Partition tolerance (may sacrifice availability) |

**Interview Answer**: "Since network partitions are unavoidable in distributed systems, the real choice is between CP (consistent but may be unavailable) and AP (available but may be inconsistent). LawPrix prioritizes consistency for case assignments."

---

## 6. Rate Limiting

### Algorithms
| Algorithm | Description |
|-----------|-------------|
| **Token Bucket** | Tokens added at fixed rate, request consumes token |
| **Fixed Window** | Count requests in fixed time window |
| **Sliding Window** | More accurate version of fixed window |
| **Leaky Bucket** | Requests queued, processed at fixed rate |

### Token Bucket Implementation
```python
import time

class TokenBucket:
    def __init__(self, capacity, refill_rate):
        self.capacity = capacity
        self.tokens = capacity
        self.refill_rate = refill_rate
        self.last_refill = time.time()
    
    def allow(self):
        now = time.time()
        elapsed = now - self.last_refill
        self.tokens = min(self.capacity, self.tokens + elapsed * self.refill_rate)
        self.last_refill = now
        
        if self.tokens >= 1:
            self.tokens -= 1
            return True
        return False
```

**Your Context**: "In LawPrix, I'd implement rate limiting on the API to prevent abuse, especially on the LLM-powered case assessment endpoint which has cost implications."

---

## 7. Message Queues

### Why Use Queues?
- Decouple services
- Handle traffic spikes (buffer)
- Async processing
- Retry failed operations

### Queue Options
| Queue | Use Case | Your Context |
|-------|----------|--------------|
| **Redis** | Lightweight, in-memory | LawPrix Celery broker |
| **RabbitMQ** | Feature-rich, routing | Enterprise apps |
| **Kafka** | High throughput, streaming | Large-scale data |
| **SQS** | Managed AWS service | Serverless apps |

**Your Context**: "In LawPrix, when a case is submitted, the RAG assessment task gets queued in Redis via Celery. This allows the API to respond immediately while assessment runs in background."

---

## 8. REST API Design Best Practices

### Versioning
```
/api/v1/cases
/api/v2/cases
```

### Pagination
```
GET /api/cases?page=2&limit=20
Response: { "data": [...], "page": 2, "total": 100 }
```

### Error Response Format
```json
{
    "error": {
        "code": 400,
        "message": "Invalid input",
        "details": {
            "field": "email",
            "reason": "Already exists"
        }
    }
}
```

---

## 9. Microservices vs Monolith

| Aspect | Monolith | Microservices |
|--------|----------|---------------|
| Deployment | Single unit | Independent services |
| Scaling | Scale entire app | Scale individual services |
| Tech Stack | Usually one | Can be different per service |
| Complexity | Simple to start | Complex to manage |
| Team | Easier coordination | Independent teams |

**Your Context**: "LawPrix is a modular Django monolith with 22 apps - gives organization benefits without microservices complexity. PGPulse uses containerized services because the AI pipeline is compute-intensive and needed separate scaling."

---

## 10. System Design Interview Framework

### 1. Requirements Clarification
- Functional: What features?
- Non-functional: Scale, latency, availability?

### 2. High-Level Design
- Draw components (client, load balancer, services, DB)
- Define APIs

### 3. Detailed Design
- Database schema
- Data flow
- Key algorithms

### 4. Trade-offs
- Consistency vs Availability
- Simplicity vs Scalability
- Cost vs Performance

---

## 11. Quick Answers

**Q: What is the difference between horizontal and vertical scaling?**
A: Vertical = adding more resources to existing machine (scale up). Horizontal = adding more machines (scale out). Horizontal is more flexible but adds complexity (distributed system challenges).

**Q: What is a CDN?**
A: Content Delivery Network. Distributed servers that cache content close to users. Reduces latency and server load. Examples: Cloudflare, AWS CloudFront.

**Q: What is the difference between SQL and NoSQL?**
A: SQL: Structured schema, ACID, joins, vertical scaling. NoSQL: Flexible schema, eventual consistency, horizontal scaling. Use SQL for complex queries, NoSQL for flexibility and scale.

**Q: What is eventual consistency?**
A: After a write, reads may temporarily return stale data, but eventually all replicas converge to the same value. Acceptable for many use cases (social media feeds, shopping carts).

---

> **For Infosys**: They typically ask about LRU Cache, basic caching, and REST API design. Be ready to draw system diagrams and explain trade-offs.
