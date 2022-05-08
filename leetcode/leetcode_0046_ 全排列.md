#### [46. 全排列](https://leetcode-cn.com/problems/permutations/)

难度中等1547收藏分享切换为英文接收动态反馈

给定一个不含重复数字的数组 `nums` ，返回其 **所有可能的全排列** 。你可以 **按任意顺序** 返回答案。

 

**示例 1：**

```
输入：nums = [1,2,3]
输出：[[1,2,3],[1,3,2],[2,1,3],[2,3,1],[3,1,2],[3,2,1]]
```

**示例 2：**

```
输入：nums = [0,1]
输出：[[0,1],[1,0]]
```

**示例 3：**

```
输入：nums = [1]
输出：[[1]]
```

 

**提示：**

- `1 <= nums.length <= 6`
- `-10 <= nums[i] <= 10`
- `nums` 中的所有整数 **互不相同**

**思路：回溯**

```java
class Solution {
    List<List<Integer>> res = new LinkedList<>();
    LinkedList<Integer> path = new LinkedList<>();
    public List<List<Integer>> permute(int[] nums) {
        backTrack(nums);
        return res;
    }
    
    private void backTrack(int[] nums){
        if (path.size() == nums.length){
            res.add(new LinkedList<>(path));
            return;
        }
        for (int i = 0; i < nums.length; i++){
            if(!contains(nums[i])){
                path.add(nums[i]);
                backTrack(nums);
                path.removeLast();
            }
        }
    }
    
    private boolean contains(int x){
        boolean flag = false;
        for (int i = 0; i < path.size(); i++){
            if (x == path.get(i))
                flag = true;
        }
        return flag;
    }
}
```

