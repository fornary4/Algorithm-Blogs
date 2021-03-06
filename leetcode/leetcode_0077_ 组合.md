#### [77. 组合](https://leetcode-cn.com/problems/combinations/)

难度中等701收藏分享切换为英文接收动态反馈

给定两个整数 `n` 和 `k`，返回范围 `[1, n]` 中所有可能的 `k` 个数的组合。

你可以按 **任何顺序** 返回答案。

 

**示例 1：**

```
输入：n = 4, k = 2
输出：
[
  [2,4],
  [3,4],
  [2,3],
  [1,2],
  [1,3],
  [1,4],
]
```

**示例 2：**

```
输入：n = 1, k = 1
输出：[[1]]
```

 

**提示：**

- `1 <= n <= 20`
- `1 <= k <= n`

**思路：回溯**

```java
/*  
 *未优化版本
 */
class Solution {
    List<List<Integer>> res = new ArrayList<>();
    List<Integer> path = new ArrayList<>();
    public List<List<Integer>> combine(int n, int k) {
        backTrack(n, k, 1);
        return res;
    }

    public void backTrack(int n, int k, int start){
        if (path.size() == k){
            res.add(new ArrayList(path));//重点，这里一定要new ArrayList,不然会报错
            return;
        }
        for (int i = start; i <= n; i++){
            path.add(i);
            backTrack(n, k, i + 1);
            path.remove(path.size() - 1);//回溯
        }
    }
}
/*  
 *改进版本
 *调整i的边界
 */
class Solution {
    List<List<Integer>> res = new ArrayList<>();
    List<Integer> path = new ArrayList<>();
    public List<List<Integer>> combine(int n, int k) {
        backTrack(n, k, 1);
        return res;
    }

    public void backTrack(int n, int k, int start){
        if (path.size() == k){
            res.add(new ArrayList(path));
            return;
        }
        for (int i = start; i <= n - (k - path.size()) + 1; i++){
            path.add(i);
            backTrack(n, k, i + 1);
            path.remove(path.size() - 1);
        }
    }
}

```

