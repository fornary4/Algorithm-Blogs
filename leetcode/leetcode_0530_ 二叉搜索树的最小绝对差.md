#### [530. 二叉搜索树的最小绝对差](https://leetcode-cn.com/problems/minimum-absolute-difference-in-bst/)

难度简单270收藏分享切换为英文接收动态反馈

给你一棵所有节点为非负值的二叉搜索树，请你计算树中任意两节点的差的绝对值的最小值。

 

**示例：**

```
输入：

   1
    \
     3
    /
   2

输出：
1

解释：
最小绝对差为 1，其中 2 和 1 的差的绝对值为 1（或者 2 和 3）。
```

**思路：利用二叉搜索树性质**

```java
//非常不好的解法
class Solution {
    int[] nodes = new int[10005];
    int count = 0;
    public int getMinimumDifference(TreeNode root) {
        dfs(root);
        int min = 1000000;
        for(int i = 0; i < count - 1; i++){
            if (nodes[i + 1] - nodes[i] < min)
                min = nodes[i + 1] - nodes[i];
        }
        return min;
    }
    public void dfs(TreeNode root){
        if(root != null){
            dfs(root.left);
            nodes[count] = root.val;
            count++;
            dfs(root.right);
        }
    }
}

//正常解法
class Solution {
    int min = 1000000;
    TreeNode pre = null;//利用指针保存上一个节点
    public int getMinimumDifference(TreeNode root) {
        dfs(root);
        return min;
    }
    public void dfs(TreeNode root){
        if(root != null){
            dfs(root.left);
            if(pre != null && root.val - pre.val < min)
                min = root.val - pre.val;
            pre = root;
            dfs(root.right);
        }
    }
}
```



