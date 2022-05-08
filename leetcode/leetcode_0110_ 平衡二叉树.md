#### [110. 平衡二叉树](https://leetcode-cn.com/problems/balanced-binary-tree/)



给定一个二叉树，判断它是否是高度平衡的二叉树。

本题中，一棵高度平衡二叉树定义为：

> 一个二叉树*每个节点* 的左右两个子树的高度差的绝对值不超过 1 。

 

**示例 1：**

![img](https://assets.leetcode.com/uploads/2020/10/06/balance_1.jpg)

```
输入：root = [3,9,20,null,null,15,7]
输出：true
```

**示例 2：**

![img](https://assets.leetcode.com/uploads/2020/10/06/balance_2.jpg)

```
输入：root = [1,2,2,3,3,null,null,4,4]
输出：false
```

**示例 3：**

```
输入：root = []
输出：true
```

 

**提示：**

- 树中的节点数在范围 `[0, 5000]` 内
- `-104 <= Node.val <= 104`

通过次数244,318

提交次数434,379



**思路：递归，先求高度**

```java
class Solution {
    public boolean isBalanced(TreeNode root) {
        if(root == null)
            return true;
        if (Math.abs(getHeight(root.left) - getHeight(root.right)) > 1)
            return false;
        return isBalanced(root.left) && isBalanced(root.right);//递归的本质，让原函数和子函数的返回值产生关联
    }

    public int getHeight(TreeNode root){
        if (root == null)
            return 0;
        return Math.max(getHeight(root.left),getHeight(root.right)) + 1;
    }
}
```

