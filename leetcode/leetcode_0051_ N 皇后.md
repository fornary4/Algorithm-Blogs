#### [51. N 皇后](https://leetcode-cn.com/problems/n-queens/)

难度困难1007收藏分享切换为英文接收动态反馈

**n 皇后问题** 研究的是如何将 `n` 个皇后放置在 `n×n` 的棋盘上，并且使皇后彼此之间不能相互攻击。

给你一个整数 `n` ，返回所有不同的 **n 皇后问题** 的解决方案。

每一种解法包含一个不同的 **n 皇后问题** 的棋子放置方案，该方案中 `'Q'` 和 `'.'` 分别代表了皇后和空位。

 

**示例 1：**

![img](https://assets.leetcode.com/uploads/2020/11/13/queens.jpg)

```
输入：n = 4
输出：[[".Q..","...Q","Q...","..Q."],["..Q.","Q...","...Q",".Q.."]]
解释：如上图所示，4 皇后问题存在两个不同的解法。
```

**示例 2：**

```
输入：n = 1
输出：[["Q"]]
```

 

**提示：**

- `1 <= n <= 9`
- 皇后彼此不能相互攻击，也就是说：任何两个皇后都不能处于同一条横行、纵行或斜线上。



**思路：回溯**

==弄清攻击条件==

```java
class Solution {
    List<List<Integer>> mark = new LinkedList<>();
    LinkedList<Integer> path = new LinkedList<>();
    public List<List<String>> solveNQueens(int n) {
        backTrack(n);
        List<List<String>> res = new LinkedList<>();
        for (List<Integer> list : mark){
            List<String> tmp = new LinkedList<>();
            for (Integer i : list){
                StringBuilder sb = new StringBuilder();
                sb.append(".".repeat(Math.max(0, n)));
                sb.setCharAt(i, 'Q');
                tmp.add(sb.toString());
            }
            res.add(tmp);
        }
        return res;

    }

    private void backTrack(int n){
        if (path.size() == n){
            mark.add(new LinkedList<>(path));
            return;
        }
        for (int i = 0; i < n; i++){
            if(check(i)){
                path.add(i);
                backTrack(n);
                path.removeLast();
            }
        }
    }

    private boolean check(int x) {
        boolean flag = true;
        for (int i = 0; i < path.size(); i++) {
            if (x == path.get(i) || Math.abs(path.size() - i) == Math.abs(x - path.get(i))) {
                flag = false;
                break;
            }
        }
        return flag;
    }
}
```

