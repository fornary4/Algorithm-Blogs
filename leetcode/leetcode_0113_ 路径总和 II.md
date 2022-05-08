#### [113. 路径总和 II](https://leetcode-cn.com/problems/path-sum-ii/)



给你二叉树的根节点 `root` 和一个整数目标和 `targetSum` ，找出所有 **从根节点到叶子节点** 路径总和等于给定目标和的路径。

**叶子节点** 是指没有子节点的节点。

 

**示例 1：**

![img](https://assets.leetcode.com/uploads/2021/01/18/pathsumii1.jpg)

```
输入：root = [5,4,8,11,null,13,4,7,2,null,null,5,1], targetSum = 22
输出：[[5,4,11,2],[5,8,4,5]]
```

**示例 2：**

![img](https://assets.leetcode.com/uploads/2021/01/18/pathsum2.jpg)

```
输入：root = [1,2,3], targetSum = 5
输出：[]
```

**示例 3：**

```
输入：root = [1,2], targetSum = 0
输出：[]
```

 

**提示：**

- 树中节点总数在范围 `[0, 5000]` 内
- `-1000 <= Node.val <= 1000`
- `-1000 <= targetSum <= 1000`

**思路:递归**

```java
class Solution {
    List<List<Integer>> res = new ArrayList<>();
    List<Integer> path = new LinkedList<>();
    public List<List<Integer>> pathSum(TreeNode root, int targetSum) {
        preorderDFS(root, targetSum);
        return res;
    }

    public void preorderDFS(TreeNode root, int targetSum) {
        if (root == null)
            return;
        path.add(root.val);
        if (root.left == null && root.right == null && targetSum - root.val == 0) {
            res.add(new ArrayList<>(path));
            return;
        }
        if (root.left != null) {
            preorderDFS(root.left, targetSum - root.val);
            path.remove(path.size() - 1); // 回溯
        }
        if (root.right != null) {
            preorderDFS(root.right, targetSum - root.val);
            path.remove(path.size() - 1); // 回溯
        }
    }
}

//简洁写法
class Solution {
    List<List<Integer>> ret = new ArrayList<>();
    List<Integer> path = new LinkedList<>();

    public List<List<Integer>> pathSum(TreeNode root, int sum) {
        dfs(root,sum);
        return ret;
    }

    public void dfs(TreeNode root, int sum) {
        if (root == null) {
            return;
        }
        path.add(root.val);
        if (root.left == null && root.right == null && sum == root.val)
            ret.add(new ArrayList<Integer>(path));
        dfs(root.left, sum - root.val);
        dfs(root.right,sum - root.val);
        path.remove(path.size()-1);
    }
}
```

