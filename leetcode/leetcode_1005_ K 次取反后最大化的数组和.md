#### [1005. K 次取反后最大化的数组和](https://leetcode-cn.com/problems/maximize-sum-of-array-after-k-negations/)

难度简单105

给定一个整数数组 A，我们**只能**用以下方法修改该数组：我们选择某个索引 `i` 并将 `A[i]` 替换为 `-A[i]`，然后总共重复这个过程 `K` 次。（我们可以多次选择同一个索引 `i`。）

以这种方式修改数组后，返回数组可能的最大和。

 

**示例 1：**

```
输入：A = [4,2,3], K = 1
输出：5
解释：选择索引 (1,) ，然后 A 变为 [4,-2,3]。
```

**示例 2：**

```
输入：A = [3,-1,0,2], K = 3
输出：6
解释：选择索引 (1, 2, 2) ，然后 A 变为 [3,1,0,2]。
```

**示例 3：**

```
输入：A = [2,-3,-1,5,-4], K = 2
输出：13
解释：选择索引 (1, 4) ，然后 A 变为 [2,3,-1,5,4]。
```

 

**提示：**

1. `1 <= A.length <= 10000`
2. `1 <= K <= 10000`
3. `-100 <= A[i] <= 100`



**思路：贪心**

```java
class Solution {
    public int largestSumAfterKNegations(int[] nums, int k) {
        Arrays.sort(nums);
        int res = 0;
        int index = -1;
        for (int i = 0; i < nums.length; i++){
            if (nums[i] >= 0){
                index = i;
                break;
            }
        }
        int bound = index;
        if (index == -1)
            bound = nums.length;
        for (int i = 0; i < bound; i++){
            nums[i] = -nums[i];
            k--;
            if (k == 0)
                break;
        }
        if (k > 0){
            if (k % 2 == 1){
                if (index == -1)
                    nums[nums.length - 1] = -nums[nums.length - 1];
                else{
                int choose = index;
                if (index > 0 && nums[index - 1] < nums[index])
                    choose = index - 1;
                nums[choose] = -nums[choose];
                }
            }
        }
        for (Integer i : nums)
            res += i;
        return res;
    }
}
```

