#### [209. 长度最小的子数组](https://leetcode-cn.com/problems/minimum-size-subarray-sum/)



给定一个含有 `n` 个正整数的数组和一个正整数 `target` **。**

找出该数组中满足其和 `≥ target` 的长度最小的 **连续子数组** `[numsl, numsl+1, ..., numsr-1, numsr]` ，并返回其长度**。**如果不存在符合条件的子数组，返回 `0` 。

 

**示例 1：**

```
输入：target = 7, nums = [2,3,1,2,4,3]
输出：2
解释：子数组 [4,3] 是该条件下的长度最小的子数组。
```

**示例 2：**

```
输入：target = 4, nums = [1,4,4]
输出：1
```

**示例 3：**

```
输入：target = 11, nums = [1,1,1,1,1,1,1,1]
输出：0
```

 

**提示：**

- `1 <= target <= 109`
- `1 <= nums.length <= 105`
- `1 <= nums[i] <= 105`

 

**进阶：**

- 如果你已经实现 `O(n)` 时间复杂度的解法, 请尝试设计一个 `O(n log(n))` 时间复杂度的解法。

## 思路：使用滑动窗口求解

**时间复杂度：O（n）**

![](https://img-blog.csdnimg.cn/20210312160441942.png)

#### 繁琐解法

```java
class Solution {
     public int minSubArrayLen(int target, int[] nums) {
        int left=0,right=0;
        int subsum=nums[0];
        int len=nums.length+1;//设置一个不可能的长度
        while(left<=right){
            if(subsum>=target){
                if((right-left+1)<len)
                    len=right-left+1;//更新子串长度
                left++;
                if(left<=right) {
                    subsum -= nums[left-1];
                }
            }
            else{
                if(right==nums.length-1)
                    break;//避免出现死循环
                if(right<nums.length-1){
                    right++;
                    subsum+=nums[right];
                }
            }
        }
        if(len==nums.length+1)
            return 0;
        return len;
    }
}
```

#### 优雅解法 

```java
class Solution {
      public int minSubArrayLen(int target, int[] nums) {
        int left = 0;
        int len = nums.length + 1;//定义一个不可能的长度
        int subSum = 0;
        for (int right = 0; right < nums.length; right++) {
            subSum += nums[right];
            while (subSum >= target) {
                len = Math.min(len, right - left + 1);
                subSum -= nums[left++];
            }
        }
        return len == nums.length + 1 ? 0 : len;
    }
}
```

