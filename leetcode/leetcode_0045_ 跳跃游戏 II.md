#### [45. 跳跃游戏 II](https://leetcode-cn.com/problems/jump-game-ii/)

难度中等1174

给你一个非负整数数组 `nums` ，你最初位于数组的第一个位置。

数组中的每个元素代表你在该位置可以跳跃的最大长度。

你的目标是使用最少的跳跃次数到达数组的最后一个位置。

假设你总是可以到达数组的最后一个位置。

 

**示例 1:**

```
输入: nums = [2,3,1,1,4]
输出: 2
解释: 跳到最后一个位置的最小跳跃数是 2。
     从下标为 0 跳到下标为 1 的位置，跳 1 步，然后跳 3 步到达数组的最后一个位置。
```

**示例 2:**

```
输入: nums = [2,3,0,1,4]
输出: 2
```

 

**提示:**

- `1 <= nums.length <= 104`
- `0 <= nums[i] <= 1000`

**思路：贪心**

```java
class Solution {
    public int jump(int[] nums) {
        int res = 0;
        int index = 0;
        while (index < nums.length - 1){
            int tMax = 0;
            int tIndex = 0;
            if ( index + nums[index] >= nums.length - 1){
                res++;
                break;
            }
            for (int i = index + 1; i <= index + nums[index] && i < nums.length; i++){
                if (i + nums[i] >= tMax){
                    tMax = i + nums[i];
                    tIndex = i;
                }
            }
            res++;
            index = tIndex;
        }
        return res;
    }
}
```

