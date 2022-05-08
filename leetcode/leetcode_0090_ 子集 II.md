#### [90. 子集 II](https://leetcode-cn.com/problems/subsets-ii/)



给你一个整数数组 `nums` ，其中可能包含重复元素，请你返回该数组所有可能的子集（幂集）。

解集 **不能** 包含重复的子集。返回的解集中，子集可以按 **任意顺序** 排列。

 

**示例 1：**

```
输入：nums = [1,2,2]
输出：[[],[1],[1,2],[1,2,2],[2],[2,2]]
```

**示例 2：**

```
输入：nums = [0]
输出：[[],[0]]
```

 

**提示：**

- `1 <= nums.length <= 10`
- `-10 <= nums[i] <= 10`



**思路：回溯**

==先对nums进行排序，再去重==

```java
//方法一
class Solution {
    List<List<Integer>> res = new LinkedList<>();
    LinkedList<Integer> path = new LinkedList<>();

    public List<List<Integer>> subsetsWithDup(int[] nums) {
        Arrays.sort(nums);
        backTrack(nums, 0);
        return res;
    }

    private void backTrack(int[] nums, int start) {
        res.add(new LinkedList(path));
        if (start >= nums.length)
            return;
        for (int i = start; i < nums.length;  i++) {
            if (i > start && nums[i] == nums [i - 1])
                continue;
            path.add(nums[i]);
            backTrack(nums, i + 1);
            path.removeLast();
        }
    }
}

// 方法e,HashSet暴力去重
class Solution {
    List<List<Integer>> res = new LinkedList<>();
    LinkedList<Integer> path = new LinkedList<>();

    public List<List<Integer>> subsetsWithDup(int[] nums) {
        Arrays.sort(nums);
        backTrack(nums, 0);
        Set<List<Integer>> set = new HashSet<>();
        for (List<Integer> list : res){
            List<Integer> tmp = new LinkedList<>(list);
            Collections.sort(tmp);
            set.add(tmp);
        }
        List<List<Integer>> result = new LinkedList<>();
        for (List<Integer> list : set)
            result.add(new LinkedList<>(list));
        return result;
    }

    private void backTrack(int[] nums, int start) {
        res.add(new LinkedList(path));
        if (start >= nums.length)
            return;
        for (int i = start; i < nums.length;  i++) {
            path.add(nums[i]);
            backTrack(nums, i + 1);
            path.removeLast();
        }
    }
}
```

