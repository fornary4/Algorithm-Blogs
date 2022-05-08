#### [106. 从中序与后序遍历序列构造二叉树](https://leetcode-cn.com/problems/construct-binary-tree-from-inorder-and-postorder-traversal/)



根据一棵树的中序遍历与后序遍历构造二叉树。

**注意:**
你可以假设树中没有重复的元素。

例如，给出

```
中序遍历 inorder = [9,3,15,20,7]
后序遍历 postorder = [9,15,7,20,3]
```

返回如下的二叉树：

```
    3
   / \
  9  20
    /  \
   15   7
```



**思路：递归，数组区间为左闭右开**

```java
class Solution {
    public TreeNode buildTree(int[] inorder, int[] postorder) {
        return build(inorder, 0, inorder.length, postorder, 0, postorder.length);
    }
    public TreeNode build(int[] inorder, int inleft, int inright, int[] postorder, int postleft, int postright){
        if (inright - inleft == 0)
            return null;
        if (inright - inleft == 1)
            return new TreeNode(inorder[inleft]);
        int rootval = postorder[postright - 1];
        TreeNode root = new TreeNode(rootval);
        int rootindex = 0;
        for (int i = inleft; i<inright; i++){
            if (inorder[i] == rootval){
                rootindex = i;
            }
        }
        root.left = build(inorder, inleft, rootindex, postorder, postleft, postleft + rootindex - inleft);
        root.right = build(inorder, rootindex + 1, inright, postorder, postleft + rootindex-inleft, postright - 1);
        return root;
    }
}
```

