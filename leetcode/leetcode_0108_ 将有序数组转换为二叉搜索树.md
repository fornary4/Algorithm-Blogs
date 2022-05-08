#### [108. 将有序数组转换为二叉搜索树](https://leetcode-cn.com/problems/convert-sorted-array-to-binary-search-tree/)

难度简单833收藏分享切换为英文接收动态反馈

给你一个整数数组 `nums` ，其中元素已经按 **升序** 排列，请你将其转换为一棵 **高度平衡** 二叉搜索树。

**高度平衡** 二叉树是一棵满足「每个节点的左右两个子树的高度差的绝对值不超过 1 」的二叉树。

 

**示例 1：**

![img](https://assets.leetcode.com/uploads/2021/02/18/btree1.jpg)

```
输入：nums = [-10,-3,0,5,9]
输出：[0,-3,9,-10,null,5]
解释：[0,-10,5,null,-3,null,9] 也将被视为正确答案：
```

**示例 2：**

![img](https://assets.leetcode.com/uploads/2021/02/18/btree.jpg)

```
输入：nums = [1,3]
输出：[3,1]
解释：[1,3] 和 [3,1] 都是高度平衡二叉搜索树。
```

 

**提示：**

- `1 <= nums.length <= 104`
- `-104 <= nums[i] <= 104`
- `nums` 按 **严格递增** 顺序排列



**思路：递归，与构造二叉树类似**

==每次创建数组正中间的节点==

```java
class Solution {
    public TreeNode sortedArrayToBST(int[] nums) {
        return insert(nums, 0, nums.length);
    }
    public TreeNode insert(int[] nums, int left, int right){
        if ((right - left) == 0) //0个元素
            return null;
        if ((right - left) == 1) //1个元素
            return new TreeNode(nums[left]);
        int mid = (right + left) / 2;
        TreeNode root = new TreeNode(nums[mid]);
        root.left = insert(nums, left, mid);
        root.right = insert(nums, mid + 1, right);
        return root;
    }
}
```

