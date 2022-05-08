#### [343. 整数拆分](https://leetcode-cn.com/problems/integer-break/)



给定一个正整数 *n*，将其拆分为**至少**两个正整数的和，并使这些整数的乘积最大化。 返回你可以获得的最大乘积。

**示例 1:**

```
输入: 2
输出: 1
解释: 2 = 1 + 1, 1 × 1 = 1。
```

**示例 2:**

```
输入: 10
输出: 36
解释: 10 = 3 + 3 + 4, 3 × 3 × 4 = 36。
```

**说明:** 你可以假设 *n* 不小于 2 且不大于 58。



**数学归纳**

```java
class Solution {
    public int pow(int a,int n){
        int res=1;
        for(int i=0;i<n;i++)
            res*=a;
        return res;
    }
    public int integerBreak(int n) {
        if(n==2)
            return 1;
        if(n==3)
            return 2;
        if(n%3==0)
            return pow(3,n/3);
        else if(n%3==1)
            return pow(3,(n-1)/3-1)*4;
        else
            return pow(3,(n-2)/3)*2;

    }
}
```

**动态规划**

```java
class Solution {
   public int integerBreak(int n) {
        int[] dp = new int[n + 1];
        dp[2] = 1;
        for (int i = 3; i <= n; i++)
            for (int j = 1; j < i - 1; j++)
                dp[i] = Math.max(dp[i], Math.max(j * (i - j), j * dp[i - j]));//是否要拆分
        return dp[n];
    }
}
```

<font color=red>疑问：为什么不拆分j</font>

