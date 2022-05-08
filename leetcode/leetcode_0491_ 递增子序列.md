#### [491. 递增子序列](https://leetcode-cn.com/problems/increasing-subsequences/)

难度中等330收藏分享切换为英文接收动态反馈

给你一个整数数组 `nums` ，找出并返回所有该数组中不同的递增子序列，递增子序列中 **至少有两个元素** 。你可以按 **任意顺序** 返回答案。

数组中可能含有重复元素，如出现两个整数相等，也可以视作递增序列的一种特殊情况。

 

**示例 1：**

```
输入：nums = [4,6,7,7]
输出：[[4,6],[4,6,7],[4,6,7,7],[4,7],[4,7,7],[6,7],[6,7,7],[7,7]]
```

**示例 2：**

```
输入：nums = [4,4,3,2,1]
输出：[[4,4]]
```

 

**提示：**

- `1 <= nums.length <= 15`
- `-100 <= nums[i] <= 100`



**思路：回溯**

==用set去重==

```java
// 原始人写法，非常简单粗暴
class Solution {
    List<List<Integer>> res = new LinkedList<>();
    LinkedList<Integer> path = new LinkedList<>();

    public List<List<Integer>> findSubsequences(int[] nums) {
        backTrack(nums, 0);
        Set<List<Integer>> set = new HashSet<>();
        for (List<Integer> list : res)
            set.add(list);
        List<List<Integer>> simplify = new LinkedList<>();
        for (List<Integer> list : set)
            simplify.add(list);
        return simplify;
    }
    private void backTrack(int[] nums, int start) {
        if (path.size() > 1)
            res.add(new LinkedList<>(path));
        for (int i = start; i < nums.length; i++){
            if (path.size() > 0 && nums[i] < path.getLast())
                continue;
            path.add(nums[i]);
            backTrack(nums, i + 1);
            path.removeLast();
        }
    }
}

//优化版本,用局部hashSet去重
class Solution {
    List<List<Integer>> res = new LinkedList<>();
    LinkedList<Integer> path = new LinkedList<>();

    public List<List<Integer>> findSubsequences(int[] nums) {
        backTrack(nums, 0);
        
        return res;
    }
    private void backTrack(int[] nums, int start) {
        if (path.size() > 1)
            res.add(new LinkedList<>(path));
        Set<Integer> set = new HashSet<>();
        for (int i = start; i < nums.length; i++){
            if (set.contains(nums[i]) || path.size() > 0 && nums[i] < path.getLast())
                continue;
            set.add(nums[i]);
            path.add(nums[i]);
            backTrack(nums, i + 1);
            path.removeLast();
        }
    }
}
```

