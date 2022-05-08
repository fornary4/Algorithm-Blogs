#### [63. 不同路径 II](https://leetcode-cn.com/problems/unique-paths-ii/)



一个机器人位于一个 *m x n* 网格的左上角 （起始点在下图中标记为“Start” ）。

机器人每次只能向下或者向右移动一步。机器人试图达到网格的右下角（在下图中标记为“Finish”）。

现在考虑网格中有障碍物。那么从左上角到右下角将会有多少条不同的路径？

![img](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2018/10/22/robot_maze.png)

网格中的障碍物和空位置分别用 `1` 和 `0` 来表示。

 

**示例 1：**

![img](https://assets.leetcode.com/uploads/2020/11/04/robot1.jpg)

```
输入：obstacleGrid = [[0,0,0],[0,1,0],[0,0,0]]
输出：2
解释：
3x3 网格的正中间有一个障碍物。
从左上角到右下角一共有 2 条不同的路径：
1. 向右 -> 向右 -> 向下 -> 向下
2. 向下 -> 向下 -> 向右 -> 向右
```

**示例 2：**

![img](https://assets.leetcode.com/uploads/2020/11/04/robot2.jpg)

```
输入：obstacleGrid = [[0,1],[0,0]]
输出：1
```

 

**提示：**

- `m == obstacleGrid.length`
- `n == obstacleGrid[i].length`
- `1 <= m, n <= 100`
- `obstacleGrid[i][j]` 为 `0` 或 `1`



**踩坑:遍历二维数组的正确方式**

**不要把数组和坐标搞混**

```java
for(int i=0;i<num.length;i++)
    for(int j=nums[0].length;j++)
        System.out.println(nums[i][j]);
```



```java
class Solution {
    public static int uniquePathsWithObstacles(int[][] obstacleGrid) {
        int m=obstacleGrid[0].length;
        int n=obstacleGrid.length;
        if(obstacleGrid[0][0]==1)
            return 0;
        int i=0,j=0;
        for(i=0;i<n;i++)
            for(j=0;j<m;j++)
                if(obstacleGrid[i][j]==1)
                    obstacleGrid[i][j]=-1;

        for(i=0;i<n;i++){
            if(obstacleGrid[i][0]==-1){
                obstacleGrid[i][0]=0;
                break;
            }
            else
                obstacleGrid[i][0]=1;
        }
        for(i=i+1;i<n;i++)
            obstacleGrid[i][0]=0;

        for(i=0;i<m;i++){
            if(obstacleGrid[0][i]==-1){
                obstacleGrid[0][i]=0;
                break;
            }
            else
                obstacleGrid[0][i]=1;
        }
        for(i=i+1;i<m;i++)
            obstacleGrid[0][i]=0;
        for(i=1;i<n;i++){
            for(j=1;j<m;j++){
                if(obstacleGrid[i][j]==-1)
                    obstacleGrid[i][j]=0;
                else
                    obstacleGrid[i][j]=obstacleGrid[i-1][j]+obstacleGrid[i][j-1];
            }
        }
        return obstacleGrid[n-1][m-1];
    }
}
```



