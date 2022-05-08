#### [515. 在每个树行中找最大值](https://leetcode-cn.com/problems/find-largest-value-in-each-tree-row/)



给定一棵二叉树的根节点 `root` ，请找出该二叉树中每一层的最大值。

 

**示例1：**

```
输入: root = [1,3,2,5,3,null,9]
输出: [1,3,9]
解释:
          1
         / \
        3   2
       / \   \  
      5   3   9 
```

**示例2：**

```
输入: root = [1,2,3]
输出: [1,3]
解释:
          1
         / \
        2   3
```

**示例3：**

```
输入: root = [1]
输出: [1]
```

**示例4：**

```
输入: root = [1,null,2]
输出: [1,2]
解释:      
           1 
            \
             2     
```

**示例5：**

```
输入: root = []
输出: []
```

 

**提示：**

- 二叉树的节点个数的范围是 `[0,104]`
- `-231 <= Node.val <= 231 - 1`



**思路：层序遍历**

```java
class Solution {
    public List<Integer> largestValues(TreeNode root) {
        List<Integer> res = new ArrayList<>();
        Queue<TreeNode> que = new LinkedList<>();
        if (root != null)
            que.offer(root);
        while (!que.isEmpty()) {
            int size = que.size();
            ArrayList<Integer> arrayList = new ArrayList<>();
            for (int i = 0; i < size; i++) {
                TreeNode tmp = que.peek();
                que.poll();
                arrayList.add(tmp.val);
                if (tmp.left != null)
                    que.offer(tmp.left);
                if (tmp.right != null)
                    que.offer(tmp.right);
            }
            Collections.sort(arrayList);//排序
            Collections.reverse(arrayList);//反转arraylist
            res.add(arrayList.get(0));
        }
        return res;
    }
}
```

