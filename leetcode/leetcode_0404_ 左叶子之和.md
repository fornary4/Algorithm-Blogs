#### [404. 左叶子之和](https://leetcode-cn.com/problems/sum-of-left-leaves/)



计算给定二叉树的所有左叶子之和。

**示例：**

```
    3
   / \
  9  20
    /  \
   15   7

在这个二叉树中，有两个左叶子，分别是 9 和 15，所以返回 24
```



**思路：设置一个flag来判定是否为左节点**



```java
class Solution {
    int count=0;
    public int sumOfLeftLeaves(TreeNode root) {
        cal(root, false);
        return count;
    }

    public void cal(TreeNode root, boolean flag){
        if(root != null){
            if (root.left == null && root.right == null && flag == true)
                count += root.val;
            cal(root.left, true);
            cal(root.right, false);
        }
        
    }
}
```

