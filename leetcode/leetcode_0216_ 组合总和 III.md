#### [216. 组合总和 III](https://leetcode-cn.com/problems/combination-sum-iii/)

难度中等348收藏分享切换为英文接收动态反馈

找出所有相加之和为 ***n*** 的 ***k\*** 个数的组合***。\***组合中只允许含有 1 - 9 的正整数，并且每种组合中不存在重复的数字。

**说明：**

- 所有数字都是正整数。
- 解集不能包含重复的组合。 

**示例 1:**

```
输入: k = 3, n = 7
输出: [[1,2,4]]
```

**示例 2:**

```
输入: k = 3, n = 9
输出: [[1,2,6], [1,3,5], [2,3,4]]
```



**思路：回溯**

```java
//简单写法
class Solution {
    List<List<Integer>> res =new ArrayList<>();
    List<Integer> path = new ArrayList<>();
    public List<List<Integer>> combinationSum3(int k, int n) {
        backTrack(k, n, 1);
        return res;
    }
    private void backTrack(int k, int n, int count){
        if (path.size() == k){
            if (sum(path) == n)
                res.add(new ArrayList(path));
            return;
        }
        for (int i = count; i <= 9 - (k - path.size()) + 1; i++){
            path.add(i);
            backTrack(k, n, i + 1);
            path.remove(path.size() - 1);
        }
    }

    private int sum(List<Integer> arr){
        int res = 0;
        for (Integer i : arr)
            res += i;
        return res;
    }
}

//优化版本
class Solution {
    List<List<Integer>> res =new ArrayList<>();
    List<Integer> path = new ArrayList<>();
    public List<List<Integer>> combinationSum3(int k, int n) {
        backTrack(k, n, 1, 0);
        return res;
    }
    private void backTrack(int k, int n, int count, int sum){
        if (sum > n) //若sum > n 直接返回，这是一个剪枝操作
            return;
        if (path.size() == k){
            if (sum == n)
                res.add(new ArrayList(path));
            return;
        }
        for (int i = count; i <= 9 - (k - path.size()) + 1; i++){
            path.add(i);
            // 在这里写sum = sum + i是错误的，参数的变化要写在下面的递归函数里面
            backTrack(k, n, i + 1, sum + i);
            path.remove(path.size() - 1);
        }
    }
}
```

