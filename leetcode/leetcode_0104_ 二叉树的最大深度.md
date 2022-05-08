#### [104. 二叉树的最大深度](https://leetcode-cn.com/problems/maximum-depth-of-binary-tree/)



给定一个二叉树，找出其最大深度。

二叉树的深度为根节点到最远叶子节点的最长路径上的节点数。

**说明:** 叶子节点是指没有子节点的节点。

**示例：**
给定二叉树 `[3,9,20,null,null,15,7]`，

```
    3
   / \
  9  20
    /  \
   15   7
```

返回它的最大深度 3 。



**思路：递归或层序遍历（迭代）**

```java
//递归
public int maxDepth(TreeNode root) {
        if (root == null)
            return 0;
        return Math.max(maxDepth(root.left), maxDepth(root.right)) + 1;
}

//层序遍历
public int maxDepth(TreeNode root) {
        Queue<TreeNode> que=new LinkedList<>();
        if (root!=null)
            que.offer(root);
        int depth=0;
        while(!que.isEmpty()){
            depth++;
            int size=que.size();
            for(int i=0;i<size;i++){
                TreeNode tmp=que.poll();
                if(tmp.left!=null)
                    que.offer(tmp.left);
                if(tmp.right!=null)
                    que.offer(tmp.right);
            }
        }
        return depth;
}
```

