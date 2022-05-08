#### [968. 监控二叉树](https://leetcode-cn.com/problems/binary-tree-cameras/)

难度困难326

给定一个二叉树，我们在树的节点上安装摄像头。

节点上的每个摄影头都可以监视**其父对象、自身及其直接子对象。**

计算监控树的所有节点所需的最小摄像头数量。

 

**示例 1：**

![img](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2018/12/29/bst_cameras_01.png)

```
输入：[0,0,null,0,0]
输出：1
解释：如图所示，一台摄像头足以监控所有节点。
```

**示例 2：**

![img](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2018/12/29/bst_cameras_02.png)

```
输入：[0,0,null,0,null,0,null,null,0]
输出：2
解释：需要至少两个摄像头来监视树的所有节点。 上图显示了摄像头放置的有效位置之一。
```


**提示：**

1. 给定树的节点数的范围是 `[1, 1000]`。
2. 每个节点的值都是 0。

**思路：贪心**

```java
class Solution {
    private int count = 0;
    public int minCameraCover(TreeNode root) {
        if (trval(root) == 0) count++;
        return count;
    }

    private int trval(TreeNode root) {
        if (root == null) return -1;

        int left = trval(root.left);
        int right = trval(root.right);

        if (left == 0 || right == 0) {
            count++;
            return 2;
        }

        if (left == 2 || right == 2) {
            return 1;
        }

        return 0;
    }
}
```

