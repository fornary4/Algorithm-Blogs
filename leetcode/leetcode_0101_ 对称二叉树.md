#### [101. 对称二叉树](https://leetcode-cn.com/problems/symmetric-tree/)



给定一个二叉树，检查它是否是镜像对称的。

 

例如，二叉树 `[1,2,2,3,4,4,3]` 是对称的。

```
    1
   / \
  2   2
 / \ / \
3  4 4  3
```

 

但是下面这个 `[1,2,2,null,3,null,3]` 则不是镜像对称的:

```
    1
   / \
  2   2
   \   \
   3    3
```

 

**进阶：**

你可以运用递归和迭代两种方法解决这个问题吗？



**思路：**

方法1：暴力解法，判断原树和转置后的树是否一样，注意判断树相等不能仅根据节点值来判断，还需要判断节点的左右指针

```java
//特例

//原树   1 
       / \
      2   2
     /   /
    2   2
    
//转置后 1 
       / \
      2   2
       \   \
        2   2
    
//如果仅使用节点值，两颗树的中序和后序遍历结果完全一致，导致无法区分
```

方法2：递归解法，一次判断两个节点







```java
//暴力解法
class Solution {
    static class MyNode{
        int val;
        String left;
        String right;

        public MyNode() {
        }

        public MyNode(int val, String left, String right) {
            this.val = val;
            this.left = left;
            this.right = right;
        }
        
    }
    
    public boolean isSymmetric(TreeNode root) {
        ArrayList<MyNode> origin_post = postOrderTraversal(root);
        TreeNode invert = invertTree(root);
        ArrayList<MyNode> current_post = postOrderTraversal(invert);
        return arrayListEqual(origin_post, current_post);
    }

    public boolean arrayListEqual(ArrayList<MyNode> a, ArrayList<MyNode> b) {
        if (a.size() != b.size())
            return false;
        for (int i = 0; i < a.size(); i++)
            if (a.get(i).val!=b.get(i).val||a.get(i).left!=b.get(i).left||a.get(i).right!=b.get(i).right)
                return false;
        return true;
    }

    public TreeNode invertTree(TreeNode root) {
        Queue<TreeNode> que = new LinkedList<>();
        if (root != null)
            que.offer(root);
        while (!que.isEmpty()) {
            int size = que.size();
            for (int i = 0; i < size; i++) {
                TreeNode tmp = que.poll();
                swap(tmp);
                if (tmp.left != null)
                    que.offer(tmp.left);
                if (tmp.right != null)
                    que.offer(tmp.right);
            }
        }
        return root;
    }

    public void swap(TreeNode root) {
        TreeNode tmp = root.left;
        root.left = root.right;
        root.right = tmp;
    }

    public ArrayList<MyNode> postOrderTraversal(TreeNode root) {
        ArrayList<MyNode> res = new ArrayList<>();
        Stack<TreeNode> stack = new Stack<>();
        stack.push(root);
        while (!stack.empty()) {
            TreeNode tmp = stack.pop();
            MyNode myNode = new MyNode(tmp.val, tmp.left == null ? "null" : "have", tmp.right == null ? "null" : "have");
            res.add(myNode);
            if (tmp.left != null)
                stack.push(tmp.left);
            if (tmp.right != null)
                stack.push(tmp.right);
        }
        Collections.reverse(res);
        return res;
    }

}

//递归解法
class Solution {
    public boolean isSymmetric(TreeNode root) {
        if (root == null)
            return false;
        return compare(root.left, root.right);
    }

    public boolean compare(TreeNode left, TreeNode right) {
        if (left == null && right == null)
            return true;
        else if (left == null)
            return false;
        else if (right == null)
            return false;
        else if (left.val != right.val)
            return false;
        else
            return compare(left.left, right.right) && compare(left.right, right.left);

    }
}
```

