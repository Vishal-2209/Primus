Dynamic Programming mai mene abhi tak ye seekha hai ki hume iske ander ye samajhna hota hai ki saamne wala kya karega, sirf ye samajh ke ki hum kya kare usse kaam nahi chalta, aapko iske liye kameena banna padta hai. For example kisi game mai aapko sirf ye sochna nahi hai ki aap kese zyada points jeet sakte hai, balki aapko ye seekhna hai ki saamne wala kese kam points jeet sakta hai.
## Basic

Agar basic sawaalo ki baat ki jaaye, to Dynamic Programming ke andar maine abhi tak recursion ke upar based sawaal dekhe hain jo kaafi basic hote hain.

Inke andar ek hi pattern follow hota hai, aur inke andar aapko sirf aur sirf sabse lower-most cases dhundhne hote hain, jaise ki ye sawaal:

---
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
> 71. 1 step + 1 step
> 72. 2 steps
> ```

And iss type ke sawaalo k liye mai isi nateeje par pahocha hu ki hume sirf edge cases jes n = 1, 2 ko leke chalna hota hai, and baaki cases ko inke upar redirect krna hota hai, just like factorial

>```Python
>class Solution:
>    def climbStairs(self, n: int) -> int:
>       if n == 1 or n == 2:
>            return n
>       else:
>            return self.climbStairs(n-1) + self.climbStairs(n-2)
>```
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

Inn sabhi sawaalo ke ander mai isi nateeje par aaya hu ki hume har question ke ander alag alag approaches ka use karna hoga, kuchh sawaalo ke ander patterns mil sakte hai jese ki [Fibonacci Series](https://leetcode.com/problems/fibonacci-number?envType=problem-list-v2&envId=dsoytzzc), Factorial, [Climbing Stairs](https://leetcode.com/problems/climbing-stairs?envType=problem-list-v2&envId=dsoytzzc)
