#### [501. 二叉搜索树中的众数](https://leetcode-cn.com/problems/find-mode-in-binary-search-tree/)

难度简单341收藏分享切换为英文接收动态反馈

给定一个有相同值的二叉搜索树（BST），找出 BST 中的所有众数（出现频率最高的元素）。

假定 BST 有如下定义：

- 结点左子树中所含结点的值小于等于当前结点的值
- 结点右子树中所含结点的值大于等于当前结点的值
- 左子树和右子树都是二叉搜索树

例如：
给定 BST `[1,null,2,2]`,

```
   1
    \
     2
    /
   2
```

`返回[2]`.

**提示**：如果众数超过1个，不需考虑输出顺序

**进阶：**你可以不使用额外的空间吗？（假设由递归产生的隐式调用栈的开销不被计算在内）



**思路：HashMap**

```java
class Solution {
    Map<Integer, Integer> map = new HashMap<>();
    public int[] findMode(TreeNode root) {
        dfs(root);
        ArrayList<Integer> arr = new ArrayList<>();
        int mode = 0;
        for (Integer i : map.values())
            if (i > mode)
                mode = i;
        for (Integer i : map.keySet())
            if (map.get(i) == mode)
                arr.add(i);
        int[] res = new int[arr.size()];
        for (int i = 0; i < arr.size(); i++)
            res[i] = arr.get(i);
        return res;
        
    }
    private void dfs(TreeNode root){
        if (root != null){
            dfs(root.left);
            if (map.containsKey(root.val))
                map.put(root.val, map.get(root.val) + 1);
            else
                map.put(root.val, 1);
            dfs(root.right);
        }
    }
}
```

