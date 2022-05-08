#### [494. 目标和](https://leetcode-cn.com/problems/target-sum/)

难度中等859收藏分享切换为英文接收动态反馈

给你一个整数数组 `nums` 和一个整数 `target` 。

向数组中的每个整数前添加 `'+'` 或 `'-'` ，然后串联起所有整数，可以构造一个 **表达式** ：

- 例如，`nums = [2, 1]` ，可以在 `2` 之前添加 `'+'` ，在 `1` 之前添加 `'-'` ，然后串联起来得到表达式 `"+2-1"` 。

返回可以通过上述方法构造的、运算结果等于 `target` 的不同 **表达式** 的数目。

 

**示例 1：**

```
输入：nums = [1,1,1,1,1], target = 3
输出：5
解释：一共有 5 种方法让最终目标和为 3 。
-1 + 1 + 1 + 1 + 1 = 3
+1 - 1 + 1 + 1 + 1 = 3
+1 + 1 - 1 + 1 + 1 = 3
+1 + 1 + 1 - 1 + 1 = 3
+1 + 1 + 1 + 1 - 1 = 3
```

**示例 2：**

```
输入：nums = [1], target = 1
输出：1
```

 

**提示：**

- `1 <= nums.length <= 20`
- `0 <= nums[i] <= 1000`
- `0 <= sum(nums[i]) <= 1000`
- `-1000 <= target <= 1000`



**思路：这是一个背包问题，也是一个排列组合问题，可以抽象为装满一个指定容量的背包有多少种不同的方法**



```java
class Solution {
    public int findTargetSumWays(int[] nums, int target) {
        int sum = 0;
        for (int num : nums)
            sum += num;
        if ((sum + target) % 2 == 1)
            return 0;
        int size = (sum + target) / 2;
        int[] dp = new int[size + 1];
        dp[0] = 1;
        for (int num : nums) {
            for (int j = size; j >= num; j--) {
                dp[j] += dp[j - num];
            }
        }
        return dp[size];
    }
}
```

