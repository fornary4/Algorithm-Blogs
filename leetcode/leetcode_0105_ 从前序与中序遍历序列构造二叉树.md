#### [105. 从前序与中序遍历序列构造二叉树](https://leetcode-cn.com/problems/construct-binary-tree-from-preorder-and-inorder-traversal/)



给定一棵树的前序遍历 `preorder` 与中序遍历 `inorder`。请构造二叉树并返回其根节点。

 

**示例 1:**

![img](https://assets.leetcode.com/uploads/2021/02/19/tree.jpg)

```
Input: preorder = [3,9,20,15,7], inorder = [9,3,15,20,7]
Output: [3,9,20,null,null,15,7]
```

**示例 2:**

```
Input: preorder = [-1], inorder = [-1]
Output: [-1]
```

 

**提示:**

- `1 <= preorder.length <= 3000`
- `inorder.length == preorder.length`
- `-3000 <= preorder[i], inorder[i] <= 3000`
- `preorder` 和 `inorder` 均无重复元素
- `inorder` 均出现在 `preorder`
- `preorder` 保证为二叉树的前序遍历序列
- `inorder` 保证为二叉树的中序遍历序列



**思路：与106类似**

```java
class Solution {
    public TreeNode buildTree(int[] preorder, int[] inorder) {
        return build(inorder, 0, inorder.length, preorder, 0, preorder.length);
    }

    public TreeNode build(int[] inorder, int inleft, int inright, int[] preorder, int preleft, int preright) {
        if (inright - inleft < 1)
            return null;
        if (inright - inleft == 1)
            return new TreeNode(inorder[inleft]);
        int rootval = preorder[preleft];
        TreeNode root = new TreeNode(rootval);
        int rootindex = 0;
        for (int i = 0; i < inright; i++) {
            if (inorder[i] == rootval) {
                rootindex = i;
            }
        }
        root.left = build(inorder, inleft, rootindex, preorder, preleft + 1, preleft + 1 + rootindex - inleft);
        root.right = build(inorder, rootindex + 1, inright, preorder, preleft + 1 + rootindex - inleft, preright);
        return root;
    }
}
```

