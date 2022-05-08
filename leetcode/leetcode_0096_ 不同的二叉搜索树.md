#### [96. 不同的二叉搜索树](https://leetcode-cn.com/problems/unique-binary-search-trees/)



给你一个整数 `n` ，求恰由 `n` 个节点组成且节点值从 `1` 到 `n` 互不相同的 **二叉搜索树** 有多少种？返回满足题意的二叉搜索树的种数。

 

**示例 1：**

![img](https://assets.leetcode.com/uploads/2021/01/18/uniquebstn3.jpg)

```
输入：n = 3
输出：5
```

**示例 2：**

```
输入：n = 1
输出：1
```

 

**提示：**

- `1 <= n <= 19`



**思路：动态规划，根据根节点进行分类**

​		例如dp[5]=dp[0]*dp[4]

​						 +dp[1]*dp[3]

​						 +dp[2]*dp[2]

​						 +dp[3]*dp[1]

​					     +dp[4]*dp[0]



```java
class Solution {
    public int numTrees(int n) {
        int[] dp = new int[n+1];
        dp[0] = 1;
        dp[1] = 1;
        for(int i = 2;i <= n;i++)
            for(int j = 0;j < i;j++)
                dp[i] += dp[j] * dp[i-1-j];
        return dp[n];
    }
}
```

