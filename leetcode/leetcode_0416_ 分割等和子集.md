#### [416. 分割等和子集](https://leetcode-cn.com/problems/partition-equal-subset-sum/)



给你一个 **只包含正整数** 的 **非空** 数组 `nums` 。请你判断是否可以将这个数组分割成两个子集，使得两个子集的元素和相等。

 

**示例 1：**

```
输入：nums = [1,5,11,5]
输出：true
解释：数组可以分割成 [1, 5, 5] 和 [11] 。
```

**示例 2：**

```
输入：nums = [1,2,3,5]
输出：false
解释：数组不能分割成两个元素和相等的子集。
```

 

**提示：**

- `1 <= nums.length <= 200`
- `1 <= nums[i] <= 100`



**思路：转化为背包问题，利用动态规划解体**

若能分割，则两个子集的元素和相等，首先可以得到原集合的元素和一定为偶数（为奇数直接返回false）

问题可以转化为能否找到一个子集，其元素和等于原集合的一半

即能否装满容量为原集合一半的背包

```java
class Solution {
    public boolean canPartition(int[] nums) {
        int sum = 0;
        for (int num : nums) sum += num;
        if (sum % 2 == 1)
            return false;
        int target = sum / 2;
        int[][] dp = new int[nums.length][target + 1];
        for (int i = nums[0]; i <= target; i++)
            dp[0][i] = nums[0];
        for (int i = 1; i < nums.length; i++) {
            for (int j = 0; j <= target; j++) {
                if (j < nums[i])
                    dp[i][j] = dp[i - 1][j];
                else
                    dp[i][j] = Math.max(dp[i - 1][j], dp[i - 1][j - nums[i]] + nums[i]);
            }
        }
        return dp[nums.length - 1][target] == target;
    }
}
```

