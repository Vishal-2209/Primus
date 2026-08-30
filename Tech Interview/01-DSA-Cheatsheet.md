---
created: 2026-08-30
purpose: DSA patterns and algorithms for Infosys technical interview
---
# DSA Cheatsheet - Infosys Focus

## Time Complexity Quick Reference

| Complexity | Name         | Example                            |
| ---------- | ------------ | ---------------------------------- |
| O(1)       | Constant     | Array index access, HashMap lookup |
| O(log n)   | Logarithmic  | Binary search                      |
| O(n)       | Linear       | Single loop through array          |
| O(n log n) | Linearithmic | Merge sort, Quick sort (avg)       |
| O(n^2)     | Quadratic    | Nested loops, Bubble sort          |
| O(2^n)     | Exponential  | Subsets generation                 |
| O(n!)      | Factorial    | Permutations                       |

---

## 1. Arrays & Strings

### Two Pointer Technique
**When to use**: Sorted array, pair/triplet problems, palindrome check.

```python
# Two Sum (Sorted Array)
def two_sum_sorted(arr, target):
    left, right = 0, len(arr) - 1
    while left < right:
        curr = arr[left] + arr[right]
        if curr == target:
            return [left, right]
        elif curr < target:
            left += 1
        else:
            right -= 1
    return []
```

**LeetCode**: [#1 Two Sum](https://leetcode.com/problems/two-sum/), [#167 Two Sum II](https://leetcode.com/problems/two-sum-ii-input-array-is-sorted/)

### Sliding Window
**When to use**: Subarray/substring with condition, max/min window, fixed window size.

```python
# Max Sum Subarray of Size K
def max_sum_subarray(arr, k):
    window_sum = sum(arr[:k])
    max_sum = window_sum
    for i in range(k, len(arr)):
        window_sum += arr[i] - arr[i - k]
        max_sum = max(max_sum, window_sum)
    return max_sum
```

**Pattern**:
```
Initialize window with first k elements
Slide window: add new element, remove old element
Track max/min/count during slide
```

**LeetCode**: [#3 Longest Substring Without Repeating](https://leetcode.com/problems/longest-substring-without-repeating-characters/), [#209 Minimum Size Subarray Sum](https://leetcode.com/problems/minimum-size-subarray-sum/)

### Kadane's Algorithm (Maximum Subarray)
```python
def max_subarray(arr):
    max_sum = curr_sum = arr[0]
    for num in arr[1:]:
        curr_sum = max(num, curr_sum + num)
        max_sum = max(max_sum, curr_sum)
    return max_sum
```

**LeetCode**: [#53 Maximum Subarray](https://leetcode.com/problems/maximum-subarray/)

---

## 2. Hashing / HashMap

**When to use**: Frequency counting, duplicate detection, two-sum pattern, grouping.

```python
# Frequency Count
from collections import Counter
freq = Counter(arr)

# Two Sum using HashMap
def two_sum(nums, target):
    seen = {}
    for i, num in enumerate(nums):
        complement = target - num
        if complement in seen:
            return [seen[complement], i]
        seen[num] = i
```

**LeetCode**: [#1 Two Sum](https://leetcode.com/problems/two-sum/), [#49 Group Anagrams](https://leetcode.com/problems/group-anagrams/), [#128 Longest Consecutive Sequence](https://leetcode.com/problems/longest-consecutive-sequence/)

---

## 3. Binary Search

**When to use**: Sorted array, search space reduction, finding boundary.

```python
def binary_search(arr, target):
    left, right = 0, len(arr) - 1
    while left <= right:
        mid = left + (right - left) // 2
        if arr[mid] == target:
            return mid
        elif arr[mid] < target:
            left = mid + 1
        else:
            right = mid - 1
    return -1
```

**Variants**:
- Find first/last occurrence
- Find minimum in rotated sorted array
- Search in infinite sorted array

**LeetCode**: [#704 Binary Search](https://leetcode.com/problems/binary-search/), [#33 Search in Rotated Sorted Array](https://leetcode.com/problems/search-in-rotated-sorted-array/), [#153 Find Minimum in Rotated Sorted Array](https://leetcode.com/problems/find-minimum-in-rotated-sorted-array/)

---

## 4. Sorting Algorithms

### Quick Reference
| Algorithm | Time (Avg) | Time (Worst) | Space | Stable |
|-----------|-----------|--------------|-------|--------|
| Bubble Sort | O(n^2) | O(n^2) | O(1) | Yes |
| Selection Sort | O(n^2) | O(n^2) | O(1) | No |
| Insertion Sort | O(n^2) | O(n^2) | O(1) | Yes |
| Merge Sort | O(n log n) | O(n log n) | O(n) | Yes |
| Quick Sort | O(n log n) | O(n^2) | O(log n) | No |
| Heap Sort | O(n log n) | O(n log n) | O(1) | No |

### Merge Sort (Always O(n log n))
```python
def merge_sort(arr):
    if len(arr) <= 1:
        return arr
    mid = len(arr) // 2
    left = merge_sort(arr[:mid])
    right = merge_sort(arr[mid:])
    return merge(left, right)

def merge(left, right):
    result = []
    i = j = 0
    while i < len(left) and j < len(right):
        if left[i] <= right[j]:
            result.append(left[i])
            i += 1
        else:
            result.append(right[j])
            j += 1
    result.extend(left[i:])
    result.extend(right[j:])
    return result
```

---

## 5. Linked List

### Core Operations
```python
class ListNode:
    def __init__(self, val=0, next=None):
        self.val = val
        self.next = next
```

### Reverse Linked List
```python
def reverse_list(head):
    prev, curr = None, head
    while curr:
        next_temp = curr.next
        curr.next = prev
        prev = curr
        curr = next_temp
    return prev
```

### Detect Cycle (Floyd's Algorithm)
```python
def has_cycle(head):
    slow = fast = head
    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next
        if slow == fast:
            return True
    return False
```

**LeetCode**: [#206 Reverse Linked List](https://leetcode.com/problems/reverse-linked-list/), [#141 Linked List Cycle](https://leetcode.com/problems/linked-list-cycle/), [#21 Merge Two Sorted Lists](https://leetcode.com/problems/merge-two-sorted-lists/)

---

## 6. Trees

### Traversals
```python
# Inorder (Left, Root, Right) - Sorted order for BST
def inorder(root):
    if root:
        inorder(root.left)
        print(root.val)
        inorder(root.right)

# Preorder (Root, Left, Right) - Used for tree construction
# Postorder (Left, Right, Root) - Used for deletion
# Level Order (BFS) - Used for level-by-level processing
```

### Binary Search Tree (BST)
- Left child < Parent < Right child
- Search: O(log n) average, O(n) worst
- Inorder traversal gives sorted order

**LeetCode**: [#94 Inorder Traversal](https://leetcode.com/problems/binary-tree-inorder-traversal/), [#102 Level Order Traversal](https://leetcode.com/problems/binary-tree-level-order-traversal/), [#226 Invert Binary Tree](https://leetcode.com/problems/invert-binary-tree/), [#236 LCA of Binary Tree](https://leetcode.com/problems/lowest-common-ancestor-of-a-binary-tree/)

---

## 7. Graphs

### Representations
```python
# Adjacency List (preferred for sparse graphs)
graph = {
    'A': ['B', 'C'],
    'B': ['A', 'D'],
    'C': ['A'],
    'D': ['B']
}

# Adjacency Matrix (for dense graphs)
# graph[i][j] = 1 if edge exists
```

### BFS (Breadth-First Search)
**Use for**: Shortest path in unweighted graph, level-order traversal.

```python
from collections import deque

def bfs(graph, start):
    visited = set([start])
    queue = deque([start])
    while queue:
        node = queue.popleft()
        for neighbor in graph[node]:
            if neighbor not in visited:
                visited.add(neighbor)
                queue.append(neighbor)
```

**Time**: O(V + E), **Space**: O(V)

### DFS (Depth-First Search)
**Use for**: Cycle detection, topological sort, path finding.

```python
def dfs(graph, node, visited=None):
    if visited is None:
        visited = set()
    visited.add(node)
    for neighbor in graph[node]:
        if neighbor not in visited:
            dfs(graph, neighbor, visited)
    return visited
```

**Time**: O(V + E), **Space**: O(V)

### Topological Sort (DAG only)
```python
from collections import defaultdict, deque

def topological_sort(num_nodes, edges):
    graph = defaultdict(list)
    in_degree = [0] * num_nodes
    for u, v in edges:
        graph[u].append(v)
        in_degree[v] += 1
    
    queue = deque([i for i in range(num_nodes) if in_degree[i] == 0])
    result = []
    while queue:
        node = queue.popleft()
        result.append(node)
        for neighbor in graph[node]:
            in_degree[neighbor] -= 1
            if in_degree[neighbor] == 0:
                queue.append(neighbor)
    return result
```

**LeetCode**: [#200 Number of Islands](https://leetcode.com/problems/number-of-islands/), [#207 Course Schedule](https://leetcode.com/problems/course-schedule/), [#210 Course Schedule II](https://leetcode.com/problems/course-schedule-ii/)

---

## 8. Dynamic Programming

### Two Types
1. **Memoization** (Top-Down): Recursive + cache
2. **Tabulation** (Bottom-Up): Iterative + table

### Pattern 1: 0/1 Knapsack
```python
def knapsack(weights, values, capacity):
    n = len(weights)
    dp = [[0] * (capacity + 1) for _ in range(n + 1)]
    
    for i in range(1, n + 1):
        for w in range(1, capacity + 1):
            if weights[i-1] <= w:
                dp[i][w] = max(
                    values[i-1] + dp[i-1][w - weights[i-1]],  # Take item
                    dp[i-1][w]                                   # Skip item
                )
            else:
                dp[i][w] = dp[i-1][w]
    return dp[n][capacity]
```

**LeetCode**: [#416 Partition Equal Subset Sum](https://leetcode.com/problems/partition-equal-subset-sum/)

### Pattern 2: Longest Common Subsequence (LCS)
```python
def lcs(text1, text2):
    m, n = len(text1), len(text2)
    dp = [[0] * (n + 1) for _ in range(m + 1)]
    
    for i in range(1, m + 1):
        for j in range(1, n + 1):
            if text1[i-1] == text2[j-1]:
                dp[i][j] = dp[i-1][j-1] + 1
            else:
                dp[i][j] = max(dp[i-1][j], dp[i][j-1])
    return dp[m][n]
```

**LeetCode**: [#1143 Longest Common Subsequence](https://leetcode.com/problems/longest-common-subsequence/)

### Pattern 3: Climbing Stairs / Fibonacci
```python
def climb_stairs(n):
    if n <= 2:
        return n
    a, b = 1, 2
    for _ in range(3, n + 1):
        a, b = b, a + b
    return b
```

**LeetCode**: [#70 Climbing Stairs](https://leetcode.com/problems/climbing-stairs/)

### Pattern 4: Coin Change
```python
def coin_change(coins, amount):
    dp = [float('inf')] * (amount + 1)
    dp[0] = 0
    for i in range(1, amount + 1):
        for coin in coins:
            if coin <= i and dp[i - coin] != float('inf'):
                dp[i] = min(dp[i], dp[i - coin] + 1)
    return dp[amount] if dp[amount] != float('inf') else -1
```

**LeetCode**: [#322 Coin Change](https://leetcode.com/problems/coin-change/)

### Pattern 5: Longest Palindromic Subsequence
```python
def longest_palindrome_subseq(s):
    n = len(s)
    dp = [[0] * n for _ in range(n)]
    
    for i in range(n):
        dp[i][i] = 1
    
    for length in range(2, n + 1):
        for i in range(n - length + 1):
            j = i + length - 1
            if s[i] == s[j]:
                dp[i][j] = dp[i+1][j-1] + 2
            else:
                dp[i][j] = max(dp[i+1][j], dp[i][j-1])
    return dp[0][n-1]
```

**LeetCode**: [#516 Longest Palindromic Subsequence](https://leetcode.com/problems/longest-palindromic-subsequence/)

---

## 9. Greedy Algorithms

**When to use**: Local optimal choice leads to global optimal. Activity selection, interval scheduling.

```python
# Activity Selection Problem
def max_activities(activities):
    activities.sort(key=lambda x: x[1])  # Sort by end time
    result = [activities[0]]
    last_end = activities[0][1]
    
    for i in range(1, len(activities)):
        if activities[i][0] >= last_end:
            result.append(activities[i])
            last_end = activities[i][1]
    return result
```

**LeetCode**: [#55 Jump Game](https://leetcode.com/problems/jump-game/), [#45 Jump Game II](https://leetcode.com/problems/jump-game-ii/)

---

## 10. Backtracking

**When to use**: All permutations, all combinations, constraint satisfaction.

```python
# Generate All Permutations
def permutations(nums):
    result = []
    
    def backtrack(start):
        if start == len(nums):
            result.append(nums[:])
            return
        for i in range(start, len(nums)):
            nums[start], nums[i] = nums[i], nums[start]
            backtrack(start + 1)
            nums[start], nums[i] = nums[i], nums[start]
    
    backtrack(0)
    return result
```

**LeetCode**: [#46 Permutations](https://leetcode.com/problems/permutations/), [#78 Subsets](https://leetcode.com/problems/subsets/), [#39 Combination Sum](https://leetcode.com/problems/combination-sum/)

---

## Infosys-Specific Focus Areas

### Must-Know Patterns (High Probability)
1. **Two Pointer** - Almost guaranteed to appear
2. **Sliding Window** - Very common
3. **HashMap operations** - Frequency counting
4. **Binary Search** - Medium difficulty expected
5. **BFS/DFS** - Graph traversal questions
6. **Basic DP** - Fibonacci, Knapsack, Coin Change

### Infosys Interview Difficulty
- Easy to Medium LeetCode problems
- Focus on **approach explanation** over code
- They value **optimization thinking** (brute force -> optimized)

---

> **Strategy**: For Infosys, focus on patterns over individual problems. If you understand Two Pointer, Sliding Window, and HashMap, you can solve 60% of their DSA questions.
