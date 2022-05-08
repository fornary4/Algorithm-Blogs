#### [701. 二叉搜索树中的插入操作](https://leetcode-cn.com/problems/insert-into-a-binary-search-tree/)



给定二叉搜索树（BST）的根节点和要插入树中的值，将值插入二叉搜索树。 返回插入后二叉搜索树的根节点。 输入数据 **保证** ，新值和原始二叉搜索树中的任意节点值都不同。

**注意**，可能存在多种有效的插入方式，只要树在插入后仍保持为二叉搜索树即可。 你可以返回 **任意有效的结果** 。

 

**示例 1：**

![img](https://assets.leetcode.com/uploads/2020/10/05/insertbst.jpg)

```
输入：root = [4,2,7,1,3], val = 5
输出：[4,2,7,1,3,5]
解释：另一个满足题目要求可以通过的树是：
```

**示例 2：**

```
输入：root = [40,20,60,10,30,50,70], val = 25
输出：[40,20,60,10,30,50,70,null,null,25]
```

**示例 3：**

```
输入：root = [4,2,7,1,3,null,null,null,null,null,null], val = 5
输出：[4,2,7,1,3,5]
```

 

 

**提示：**

- 给定的树上的节点数介于 `0` 和 `10^4` 之间
- 每个节点都有一个唯一整数值，取值范围从 `0` 到 `10^8`
- `-10^8 <= val <= 10^8`
- 新值和原始二叉搜索树中的任意节点值都不同



**思路：递归或迭代，使用迭代要保存上一个节点的值**

```java
/* 迭代 */
class Solution {
    public TreeNode insertIntoBST(TreeNode root, int val) {
        if (root == null)
            return new TreeNode(val);
        TreeNode tmp = root;
        while (tmp != null){
            if (val > tmp.val){
                if (tmp.right == null){  //在这里就要进行判断，否则会插入不成功
                    tmp.right = new TreeNode(val);
                    break;
                }
                tmp = tmp .right;
            }
                
            else {
                if (tmp.left == null){
                    tmp.left = new TreeNode(val);
                    break;
                }
                tmp = tmp.left;
            }
        }
        return root;
    }
}

/* 递归，较好的解法 */
class Solution {
    public TreeNode insertIntoBST(TreeNode root, int val) {
        if (root == null)
            return new TreeNode(val);
        if (root.val > val)
            root.left = insertIntoBST(root.left, val);
        else if (root.val < val)
            root.right = insertIntoBST(root.right, val);
        return root;
    }
}
```

