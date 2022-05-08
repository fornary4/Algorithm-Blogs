#### [59. 螺旋矩阵 II](https://leetcode-cn.com/problems/spiral-matrix-ii/)



给你一个正整数 `n` ，生成一个包含 `1` 到 `n2` 所有元素，且元素按顺时针顺序螺旋排列的 `n x n` 正方形矩阵 `matrix` 。

 

**示例 1：**

![img](https://assets.leetcode.com/uploads/2020/11/13/spiraln.jpg)

```
输入：n = 3
输出：[[1,2,3],[8,9,4],[7,6,5]]
```

**示例 2：**

```
输入：n = 1
输出：[[1]]
```

 

**提示：**

- `1 <= n <= 20`

## 思路：模拟

**将矩阵分层，再将每层分为四个序列**

```java
class Solution {
    public int[][] generateMatrix(int n) {
        int[][] a = new int[n][n];
        int up = n - 1;
        int times = n / 2;
        int num = 1;
        int i;
        for (int t = 0; t < times; t++) {
            for (i = t; i < up; i++)//用t和up表示一个序列的起点和终点
                a[t][i] = num++;
            for (i = t; i < up; i++)
                a[i][up] = num++;
            for (i = up; i > t; i--)
                a[up][i] = num++;
            for (i = up; i > t; i--)
                a[i][t] = num++;
            up--;
        }
        if (n % 2 == 1)
            a[n / 2][n / 2] = num;
        return a;
    }
}
```

