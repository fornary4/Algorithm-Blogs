#### [15. 三数之和](https://leetcode-cn.com/problems/3sum/)



给你一个包含 `n` 个整数的数组 `nums`，判断 `nums` 中是否存在三个元素 *a，b，c ，*使得 *a + b + c =* 0 ？请你找出所有和为 `0` 且不重复的三元组。

**注意：**答案中不可以包含重复的三元组。

 

**示例 1：**

```
输入：nums = [-1,0,1,2,-1,-4]
输出：[[-1,-1,2],[-1,0,1]]
```

**示例 2：**

```
输入：nums = []
输出：[]
```

**示例 3：**

```
输入：nums = [0]
输出：[]
```

 

**提示：**

- `0 <= nums.length <= 3000`
- `-10^5 <= nums[i] <= 10^5`



**思路：双指针法，注意去重的细节**

```java
class Solution {
    public List<List<Integer>> threeSum(int[] nums) {
        List<List<Integer>> result = new ArrayList<>();
        Arrays.sort(nums);//对数组进行排序
        for (int i = 0; i < nums.length; i++) {
            if (nums[i] > 0)//如果第一个数大于0没有必要再进行下去
                return result;
            if (i > 0 && nums[i - 1] == nums[i])//遇到重复数字直接跳过，注意这里一定不能用nums[i]==nums[i+1]判定
                continue;
            int left = i + 1;
            int right = nums.length - 1;
            while (left < right) {
                int tmp = nums[i] + nums[left] + nums[right];
                if (tmp > 0)
                    right--;
                else if (tmp < 0)
                    left++;
                else {
                    result.add(Arrays.asList(nums[i], nums[left], nums[right]));//使用asList将结果转为List
                    while (left < right && nums[left] == nums[left + 1])
                        left++;
                    while (left < right && nums[right] == nums[right - 1])
                        right--;
                    left++;
                    right--;
                }
            }

        }
        return result;
    }
}
```

