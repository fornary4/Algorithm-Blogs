#### [39. 组合总和](https://leetcode-cn.com/problems/combination-sum/)

难度中等1522收藏分享切换为英文接收动态反馈

给定一个**无重复元素**的正整数数组 `candidates` 和一个正整数 `target` ，找出 `candidates` 中所有可以使数字和为目标数 `target` 的唯一组合。

`candidates` 中的数字可以无限制重复被选取。如果至少一个所选数字数量不同，则两种组合是唯一的。 

对于给定的输入，保证和为 `target` 的唯一组合数少于 `150` 个。

 

**示例 1：**

```
输入: candidates = [2,3,6,7], target = 7
输出: [[7],[2,2,3]]
```

**示例 2：**

```
输入: candidates = [2,3,5], target = 8
输出: [[2,2,2,2],[2,3,3],[3,5]]
```

**示例 3：**

```
输入: candidates = [2], target = 1
输出: []
```

**示例 4：**

```
输入: candidates = [1], target = 1
输出: [[1]]
```

**示例 5：**

```
输入: candidates = [1], target = 2
输出: [[1,1]]
```

 

**提示：**

- `1 <= candidates.length <= 30`
- `1 <= candidates[i] <= 200`
- `candidate` 中的每个元素都是独一无二的。
- `1 <= target <= 500`



**思路：回溯**

<font color=red>使path为递增序列来去重</font>

```java
/* 原始版本 */
class Solution {
    List<List<Integer>> res = new ArrayList<>();
    List<Integer> path = new ArrayList<>();
    public List<List<Integer>> combinationSum(int[] candidates, int target) {
        backTrack(candidates, 0, target);
        return res;
    }

    private void backTrack(int[] candidates, int sum, int target){
        if(sum >= target){
            if (sum == target)
                res.add(new ArrayList(path));
            return;
        }
        for (int i = 0; i < candidates.length; i++){
            if (path.size() != 0 && candidates[i] < path.get(path.size() - 1))//保证path递增
                continue;
            path.add(candidates[i]);
            backTrack(candidates, sum + candidates[i], target);
            path.remove(path.size() - 1);
        }
    }
}

/* 优化版本 */
class Solution {
    List<List<Integer>> res = new ArrayList<>();
    List<Integer> path = new ArrayList<>();
    public List<List<Integer>> combinationSum(int[] candidates, int target) {
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
            path.add(candidates[i]);
            backTrack(candidates, sum + candidates[i], target,i);
            path.remove(path.size() - 1);
        }
    }
}
```

