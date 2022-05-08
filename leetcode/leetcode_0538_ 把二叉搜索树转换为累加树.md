#### [538. 把二叉搜索树转换为累加树](https://leetcode-cn.com/problems/convert-bst-to-greater-tree/)

难度中等583收藏分享切换为英文接收动态反馈

给出二叉 **搜索** 树的根节点，该树的节点值各不相同，请你将其转换为累加树（Greater Sum Tree），使每个节点 `node` 的新值等于原树中大于或等于 `node.val` 的值之和。

提醒一下，二叉搜索树满足下列约束条件：

- 节点的左子树仅包含键 **小于** 节点键的节点。
- 节点的右子树仅包含键 **大于** 节点键的节点。
- 左右子树也必须是二叉搜索树。

**注意：**本题和 1038: https://leetcode-cn.com/problems/binary-search-tree-to-greater-sum-tree/ 相同

 

**示例 1：**

**![img](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2019/05/03/tree.png)**

```
输入：[4,1,6,0,2,5,7,null,null,null,3,null,null,null,8]
输出：[30,36,21,36,35,26,15,null,null,null,33,null,null,null,8]
```

**示例 2：**

```
输入：root = [0,null,1]
输出：[1,null,1]
```

**示例 3：**

```
输入：root = [1,0,2]
输出：[3,3,2]
```

**示例 4：**

```
输入：root = [3,2,4,1]
输出：[7,9,4,10]
```

 

**提示：**

- 树中的节点数介于 `0` 和 `104` 之间。
- 每个节点的值介于 `-104` 和 `104` 之间。
- 树中的所有值 **互不相同** 。
- 给定的树为二叉搜索树。

**思路：找出规律，递归求解**

二叉搜索树的性质

==右左中遍历为递减序列==

```java
/* 暴力解法，只能应急 */
class Solution {
    ArrayList<Integer> arr = new ArrayList<>();
    public TreeNode convertBST(TreeNode root) {
        traversal(root);
        modify(root);
        return root;
    }
    public void traversal(TreeNode root){
        if (root != null){
            arr.add(root.val);
            traversal(root.left);
            traversal(root.right);
        }
    }
    public int cal(int val){
        int res = 0;
        for (Integer i : arr)
            if (i >= val)
                res += i;
        return res;
    }
    public void modify(TreeNode root){
        if(root != null){
            root.val = cal(root.val);
            modify(root.left);
            modify(root.right);
        }
    }
}

/* 递归,巧解，利用二叉搜索树的性质 */
class Solution {
    int tmp = 0;
    public TreeNode convertBST(TreeNode root) {
        traversal(root);
        return root;
    }

    public void traversal(TreeNode root){
        if (root != null){
            traversal(root.right);
            tmp = tmp + root.val;
            root.val = tmp;
            traversal(root.left);
        }
    }
}

```

