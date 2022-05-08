#### [226. 翻转二叉树](https://leetcode-cn.com/problems/invert-binary-tree/)

难度简单955收藏分享切换为英文接收动态反馈

翻转一棵二叉树。

**示例：**

输入：

```
     4
   /   \
  2     7
 / \   / \
1   3 6   9
```

输出：

```
     4
   /   \
  7     2
 / \   / \
9   6 3   1
```



**思路：二叉树的遍历**

```java
class Solution {
    public TreeNode invertTree(TreeNode root) {
        if (root == null) {
            return null;
        }
        invertTree(root.left);
        invertTree(root.right);
        swap(root);
        return root;
    }

//这里的swap一定不能写成swap(TreeNode left,TreeNode right)  
    public void swap(TreeNode root) {
        TreeNode tmp = root.left;
        root.left = root.right;
        root.right = tmp;
    }
}
```

