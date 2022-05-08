#### [200. 岛屿数量](https://leetcode-cn.com/problems/number-of-islands/)



给你一个由 `'1'`（陆地）和 `'0'`（水）组成的的二维网格，请你计算网格中岛屿的数量。

岛屿总是被水包围，并且每座岛屿只能由水平方向和/或竖直方向上相邻的陆地连接形成。

此外，你可以假设该网格的四条边均被水包围。

 

**示例 1：**

```
输入：grid = [
  ["1","1","1","1","0"],
  ["1","1","0","1","0"],
  ["1","1","0","0","0"],
  ["0","0","0","0","0"]
]
输出：1
```

**示例 2：**

```
输入：grid = [
  ["1","1","0","0","0"],
  ["1","1","0","0","0"],
  ["0","0","1","0","0"],
  ["0","0","0","1","1"]
]
输出：3
```

 

**提示：**

- `m == grid.length`
- `n == grid[i].length`
- `1 <= m, n <= 300`
- `grid[i][j]` 的值为 `'0'` 或 `'1'`



**思路：用队列进行广度优先搜索,或用递归进行深度优先搜索**

```java
/* BFS */
class Solution {
    static class Block {
        int x;
        int y;

        public Block(int x, int y) {
            this.x = x;
            this.y = y;
        }
    }

    public int numIslands(char[][] grid) {
        int row = grid.length;
        int col = grid[0].length;
        int[][] state = new int[row][col];
        int[] fx = {1, -1, 0, 0};
        int[] fy = {0, 0, 1, -1};
        Queue<Block> que = new LinkedList<>();
        int count = 0;
        for (int i = 0; i < row; i++) {
            for (int j = 0; j < col; j++) {
                if (grid[i][j] == '1' && state[i][j] == 0) {
                    count++;
                    que.offer(new Block(i, j));
                    state[i][j] = 1;
                    while (!que.isEmpty()) {
                        Block tmp = que.poll();
                        for (int k = 0; k < 4; k++) {
                            int nx = tmp.x + fx[k];
                            int ny = tmp.y + fy[k];
                            if (nx >= 0 && nx < row && ny >= 0 && ny < col && grid[nx][ny] == '1' && state[nx][ny] == 0) {
                                que.offer(new Block(nx, ny));
                                state[nx][ny] = 1;
                            }
                        }

                    }
                }
            }
        }
        return count;
    }
}

/* DFS */
class Solution {
    public int numIslands(char[][] grid) {
        int count = 0;
        for (int i = 0; i < grid.length; i++){
            for (int j = 0; j < grid[0].length; j++){
                if(grid[i][j] == '1'){
                    count++;
                    dfs(grid, i, j);
                }
            }
        }
        return count;
    }

    void dfs(char[][] grid, int x, int y){
        int row = grid.length;
        int col = grid[0].length;
        if (x < 0 || x >= row || y < 0 || y >= col || grid[x][y] == '0')
            return;
        grid[x][y] = '0';
        dfs(grid, x - 1, y);
        dfs(grid, x + 1, y);
        dfs(grid, x, y - 1);
        dfs(grid, x, y + 1);
    }
}
```

