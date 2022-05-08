#### [257. 二叉树的所有路径](https://leetcode-cn.com/problems/binary-tree-paths/)



给定一个二叉树，返回所有从根节点到叶子节点的路径。

**说明:** 叶子节点是指没有子节点的节点。

**示例:**

```
输入:

   1
 /   \
2     3
 \
  5

输出: ["1->2->5", "1->3"]

解释: 所有根节点到叶子节点的路径为: 1->2->5, 1->3
```



**思路：回溯**

```java
class Solution {
    public List<String> binaryTreePaths(TreeNode root) {
        List<String> paths = new ArrayList<>();
        constructPaths(root,"",paths);
        return paths;
    }

    public void constructPaths(TreeNode root, String path, List<String> paths) {
        if (root != null){
            StringBuffer sb=new StringBuffer(path);
            sb.append(root.val);
            if (root.left == null && root.right == null){
                paths.add(sb.toString());
            }else {
                sb.append("->");
                constructPaths(root.left,sb.toString(),paths);
                constructPaths(root.right,sb.toString(),paths);
            }
        }
    }
}
```

