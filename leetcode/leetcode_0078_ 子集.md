#### [78. 子集](https://leetcode-cn.com/problems/subsets/)



给你一个整数数组 `nums` ，数组中的元素 **互不相同** 。返回该数组所有可能的子集（幂集）。

解集 **不能** 包含重复的子集。你可以按 **任意顺序** 返回解集。

 

**示例 1：**

```
输入：nums = [1,2,3]
输出：[[],[1],[2],[1,2],[3],[1,3],[2,3],[1,2,3]]
```

**示例 2：**

```
输入：nums = [0]
输出：[[],[0]]
```

 

**提示：**

- `1 <= nums.length <= 10`
- `-10 <= nums[i] <= 10`
- `nums` 中的所有元素 **互不相同**



**思路：回溯**

==分为两种情况，选择当前元素或不选择当前元素==

```java
//第一种写法，每个元素分两种情况讨论
class Solution {
    List<List<Integer>> res = new ArrayList<>();
    List<Integer> path = new ArrayList<>();

    public List<List<Integer>> subsets(int[] nums) {
        backTrack(nums, 0);
        return res;
    }

    private void backTrack(int[] nums, int start) {
        if (start == nums.length){
            res.add(new ArrayList(path));
            return;
        }
        backTrack(nums, start + 1);
        path.add(nums[start]);
        backTrack(nums, start + 1);
        path.remove(path.size() - 1);
    }
}

//第二种写法，搜集回溯树所有的节点
class Solution {
    List<List<Integer>> res = new ArrayList<>();
    List<Integer> path = new ArrayList<>();

    public List<List<Integer>> subsets(int[] nums) {
        backTrack(nums, 0);
        return res;
    }

    private void backTrack(int[] nums, int start){
        res.add(new ArrayList(path));
        if (start >= nums.length)   
            return;
        for (int i = start; i < nums.length; i++){
            path.add(nums[i]);
            backTrack(nums, i + 1);
            path.remove(path.size() - 1);
        }
    }
}
```

