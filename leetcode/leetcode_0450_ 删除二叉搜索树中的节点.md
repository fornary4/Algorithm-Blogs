#### [450. 删除二叉搜索树中的节点](https://leetcode-cn.com/problems/delete-node-in-a-bst/)

难度中等516收藏分享切换为英文接收动态反馈

给定一个二叉搜索树的根节点 **root** 和一个值 **key**，删除二叉搜索树中的 **key** 对应的节点，并保证二叉搜索树的性质不变。返回二叉搜索树（有可能被更新）的根节点的引用。

一般来说，删除节点可分为两个步骤：

1. 首先找到需要删除的节点；
2. 如果找到了，删除它。

**说明：** 要求算法时间复杂度为 O(h)，h 为树的高度。

**示例:**

```
root = [5,3,6,2,4,null,7]
key = 3

    5
   / \
  3   6
 / \   \
2   4   7

给定需要删除的节点值是 3，所以我们首先找到 3 这个节点，然后删除它。

一个正确的答案是 [5,4,6,2,null,null,7], 如下图所示。

    5
   / \
  4   6
 /     \
2       7

另一个正确答案是 [5,2,6,null,4,null,7]。

    5
   / \
  2   6
   \   \
    4   7
```

**思路：采用递归求解，分3种情况讨论**

- 要删除的节点是叶节点，令root=null
- 要删除的节点有一个子节点，令root=不为空的叶节点
- 要删除的节点有两个字节点，令root.val=右子树的最大节点值，并删除右子树值最大的节点

==在第三种情况下，不要直接删除节点，先替换节点的值，再做递归删除，这是一种很巧妙的处理方法==

```java
/* 数据结构课本上的解法 */
class Solution {
    TreeNode tmp;
    public TreeNode deleteNode(TreeNode root, int key) {
        if (root == null)
            return null;
        if (key < root.val)
            root.left = deleteNode(root.left, key);
        else if (key > root.val)
            root.right = deleteNode(root.right, key);
        else{
            if(root.left != null && root.right != null){ //有两个子节点的情况
                tmp = findMin(root.right); //获取右子树值最小的节点
                root.val = tmp.val;
                root.right = deleteNode(root.right, root.val);//删除右子树值最小的节点
            }
            else{
                if (root.left == null) //没有子节点或只有右节点
                    root = root.right;
                else				   //只有左节点
                    root = root.left;
            }
        }
        return root;
    }

    public TreeNode findMin(TreeNode root){
        if (root == null)
            return null;
        while (root.left != null){
            root = root.left;
        }
        return root;
    }
}

/* 简化 */
class Solution {
    TreeNode tmp;
    public TreeNode deleteNode(TreeNode root, int key) {
        if (root == null)
            return null;
        if (key < root.val)
            root.left = deleteNode(root.left, key);
        else if (key > root.val)
            root.right = deleteNode(root.right, key);
        else{
            if(root.left != null && root.right != null){
                tmp = root.right;
                while (tmp.left != null)
                    tmp = tmp.left;
                root.val = tmp.val;
                root.right = deleteNode(root.right, root.val);
            }
            else{
                if (root.left == null)
                    root = root.right;
                else
                    root = root.left;
            }
        }
        return root;
    }
}
```

