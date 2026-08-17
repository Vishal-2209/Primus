---
created: 2026-08-17
status: in-progress
---

# DSA Practice — 17 Aug 2026

Practiced Dynamic Programming problems today. Key insight: in DP, you need to think about what the opponent will do, not just what you should do. You have to be a little cunning. For example, in a game, don't just think about how you can score more points — figure out how the opponent can score fewer points.

---

## Problems Solved

### Divisor Game

Alice and Bob take turns playing a game, with Alice starting first.
Initially, there is a number `n` on the chalkboard. On each player's turn, that player makes a move consisting of:
- Choosing any integer `x` with `0 < x < n` and `n % x == 0`.
- Replacing the number `n` on the chalkboard with `n - x`.
Also, if a player cannot make a move, they lose the game.
Return `true` _if and only if Alice wins the game, assuming both players play optimally_.

```
Example 1:
Input: n = 2
Output: true
Explanation: Alice chooses 1, and Bob has no more moves.

```

Answer
```Python
class Solution:
    def divisorGame(self, n: int) -> bool:
        return n%2 == 0
```

---

### Min Cost Climbing Stairs

You are given an integer array `cost` where `cost[i]` is the cost of `ith` step on a staircase. Once you pay the cost, you can either climb one or two steps.
You can either start from the step with index `0`, or the step with index `1`.
Return _the minimum cost to reach the top of the floor_.
```Text
Example 1:

Input: cost = [10,15,20]
Output: 15
Explanation: You will start at index 1.
Pay 15 and climb two steps to reach the top.
The total cost is 15.

```

```Python
class Solution:
    def minCostClimbingStairs(self, cost: List[int]) -> int:
        prev2 = cost[0]
        prev1 = cost[1]
        for i in range(2, len(cost)):
            curr = cost[i] + min(prev1, prev2)
            prev2 = prev1
            prev1 = curr
        return min(prev1, prev2)
```

---

## Dynamic Programming Notes

### Basic Pattern

For basic DP problems, the pattern is recursion-based. You just need to find the lowest base cases and redirect all other cases to them, similar to factorial.

### Climbing Stairs

> **Difficulty:** Easy
> [LeetCode](https://leetcode.com/problems/climbing-stairs/)
> You are climbing a staircase. It takes `n` steps to reach the top.
> Each time you can either climb `1` or `2` steps. In how many distinct ways can you climb to the top?
> 
> **Example 1**
> ```text
> Input: n = 2
> Output: 2
> Explanation: There are two ways to climb to the top.
> 1. 1 step + 1 step
> 2. 2 steps
> ```

For this type of problem, the conclusion is: you only need to work with edge cases like n = 1, 2, and redirect all other cases to these, just like factorial.

```Python
class Solution:
    def climbStairs(self, n: int) -> int:
        if n == 1 or n == 2:
            return n
        else:
            return self.climbStairs(n-1) + self.climbStairs(n-2)
```

### Pascal's Triangle II

> **Difficulty:** Easy
> [LeetCode](https://leetcode.com/problems/pascals-triangle-ii/)
> Given an integer `rowIndex`, return the `rowIndex`th (0-indexed) row of Pascal's triangle.

```Python
class Solution:
    def getRow(self, rowIndex: int) -> List[int]:
        x = [[1], [1, 1]]
        if rowIndex <= 1:
            return x[rowIndex]
        else:
            for i in range(rowIndex - 1):
                a = x[-1]
                b = [1]
                for j in range(len(a) - 1):
                    b.append(a[j] + a[j + 1])
                b.append(1)
                x.pop(-1)
                x.append(b)
        return x[-1]
```

### Is Subsequence

> **Difficulty:** Easy
> [LeetCode](https://leetcode.com/problems/is-subsequence/)
> Given two strings `s` and `t`, return `true` if `s` is a subsequence of `t`, otherwise return `false`.

```Python
class Solution:
    def isSubsequence(self, s: str, t: str) -> bool:
        j = 0
        for i in range(len(t)):
            if j < len(s) and s[j] == t[i]:
                j += 1
        return j == len(s)
```

---

## Takeaway

Every problem needs a different approach. Some problems have patterns you can recognize, like [Fibonacci Series](https://leetcode.com/problems/fibonacci-number?envType=problem-list-v2&envId=dsoytzzc), Factorial, and [Climbing Stairs](https://leetcode.com/problems/climbing-stairs?envType=problem-list-v2&envId=dsoytzzc).

---

*Last updated: 17 Aug 2026*
