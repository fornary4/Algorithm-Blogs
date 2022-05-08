#### [98. 验证二叉搜索树](https://leetcode-cn.com/problems/validate-binary-search-tree/)



给你一个二叉树的根节点 `root` ，判断其是否是一个有效的二叉搜索树。

**有效** 二叉搜索树定义如下：

- 节点的左子树只包含 **小于** 当前节点的数。
- 节点的右子树只包含 **大于** 当前节点的数。
- 所有左子树和右子树自身必须也是二叉搜索树。

 

**示例 1：**

![img](https://assets.leetcode.com/uploads/2020/12/01/tree1.jpg)

```
输入：root = [2,1,3]
输出：true
```

**示例 2：**

![img](https://assets.leetcode.com/uploads/2020/12/01/tree2.jpg)

```
输入：root = [5,1,4,null,null,3,6]
输出：false
解释：根节点的值是 5 ，但是右子节点的值是 4 。
```

 

**提示：**

- 树中节点数目范围在`[1, 104]` 内
- `-231 <= Node.val <= 231 - 1`



---

==哈哈，感觉自己智商有点低，多多练习吧==

**思路：递归**

<font color=red>二叉搜索树的重要性质：中序遍历有序</font>

```java
/* 复杂解法 */
class Solution {
    public boolean isValidBST(TreeNode root) {
        if (root == null)
            return true;
        if (judge(root.left, 1, root.val) == false || judge(root.right, 2, root.val) == false )
            return false;
        return isValidBST(root.left) && isValidBST(root.right);
    }

    public boolean judge(TreeNode root, int flag, int val){
        if (root == null)
            return true;
        if (flag == 1 && root.val >= val || flag == 2 && root.val <= val)
            return false;
        return judge(root.left, flag, val) && judge(root.right, flag, val);
    }
}

/* 巧解, 利用性质 */
class Solution {
    ArrayList<Integer> arr = new ArrayList<>();
    public boolean isValidBST(TreeNode root) {
        if (root == null)
            return true;
        boolean left = isValidBST(root.left); //左
        if (arr.size() != 0 && root.val <= arr.get(arr.size() - 1))
            return false;
        arr.add(root.val); //中

        boolean right = isValidBST(root.right); //y
        return left && right;
    }
}

```

