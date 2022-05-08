#### [513. 找树左下角的值](https://leetcode-cn.com/problems/find-bottom-left-tree-value/)



给定一个二叉树的 **根节点** `root`，请找出该二叉树的 **最底层 最左边** 节点的值。

假设二叉树中至少有一个节点。

 

**示例 1:**

![img](https://assets.leetcode.com/uploads/2020/12/14/tree1.jpg)

```
输入: root = [2,1,3]
输出: 1
```

**示例 2:**

![img](https://assets.leetcode.com/uploads/2020/12/14/tree2.jpg)

```
输入: [1,2,3,4,null,5,6,null,null,7]
输出: 7
```

 

**提示:**

- 二叉树的节点个数的范围是 `[1,104]`
- `-231 <= Node.val <= 231 - 1` 



**思路：层序遍历or递归**



```java
/* 层序遍历 */
class Solution {
    public int findBottomLeftValue(TreeNode root) {
        int result = 0;
        ArrayList<ArrayList<Integer>> res = new ArrayList<>();
        Queue<TreeNode> que = new LinkedList<>();
        que.offer(root);
        while(!que.isEmpty()){
            int size = que.size();
            ArrayList<Integer> arr = new ArrayList<>();
            for(int i = 0; i < size; i++){
                TreeNode tmp = que.poll();
                arr.add(tmp.val);
                if (tmp.left != null)
                    que.offer(tmp.left);
                if (tmp.right != null)
                    que.offer(tmp.right);
            }
            res.add(arr);
        }
        result = res.get(res.size()-1).get(0);
        return result;
    }
}

/* 递归 */
class Solution {
    int DEPTH = -1;
    int value = 0;
    public int findBottomLeftValue(TreeNode root) {
        value = root.val;
        traversal(root, 0);
        return value;
    }
    public void traversal(TreeNode root, int depth){
        if (root == null)
            return;
        if (root.left == null && root.right == null){
            if (depth > DEPTH){
                DEPTH = depth;
                value = root.val;
            }
        }
        if (root.left != null)
            traversal(root.left, depth + 1);
        if (root.right != null)
            traversal(root.right, depth + 1);
    }
}
```

