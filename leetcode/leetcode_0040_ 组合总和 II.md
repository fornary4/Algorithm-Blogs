#### [40. 组合总和 II](https://leetcode-cn.com/problems/combination-sum-ii/)



给定一个数组 `candidates` 和一个目标数 `target` ，找出 `candidates` 中所有可以使数字和为 `target` 的组合。

`candidates` 中的每个数字在每个组合中只能使用一次。

**注意：**解集不能包含重复的组合。 

 

**示例 1:**

```
输入: candidates = [10,1,2,7,6,1,5], target = 8,
输出:
[
[1,1,6],
[1,2,5],
[1,7],
[2,6]
]
```

**示例 2:**

```
输入: candidates = [2,5,2,1,2], target = 5,
输出:
[
[1,2,2],
[5]
]
```

 

**提示:**

- `1 <= candidates.length <= 100`
- `1 <= candidates[i] <= 50`
- `1 <= target <= 30`



**思路：与组合总和1类似**

==修改一下去重思路==

```java
class Solution {
    List<List<Integer>> res = new ArrayList<>();
    List<Integer> path = new ArrayList<>();
    public List<List<Integer>> combinationSum2(int[] candidates, int target) {
        Arrays.sort(candidates);
        backTrack(candidates, 0, target, 0);
        return res;
    }

    private void backTrack(int[] candidates, int sum, int target, int index){
        if(sum >= target){
            if (sum == target)
                res.add(new ArrayList(path));
            return;
        }
        for (int i = index; i < candidates.length; i++){
            if (i > 0 && i != index && candidates[i] == candidates[i - 1]) //去掉重复值
                continue;
            path.add(candidates[i]);
            backTrack(candidates, sum + candidates[i], target,i + 1);
            path.remove(path.size() - 1);
        }
    }
}
```

