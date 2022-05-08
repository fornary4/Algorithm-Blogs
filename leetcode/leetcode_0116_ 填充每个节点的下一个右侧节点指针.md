#### [116. 填充每个节点的下一个右侧节点指针](https://leetcode-cn.com/problems/populating-next-right-pointers-in-each-node/)



给定一个 **完美二叉树** ，其所有叶子节点都在同一层，每个父节点都有两个子节点。二叉树定义如下：

```
struct Node {
  int val;
  Node *left;
  Node *right;
  Node *next;
}
```

填充它的每个 next 指针，让这个指针指向其下一个右侧节点。如果找不到下一个右侧节点，则将 next 指针设置为 `NULL`。

初始状态下，所有 next 指针都被设置为 `NULL`。

 

**进阶：**

- 你只能使用常量级额外空间。
- 使用递归解题也符合要求，本题中递归程序占用的栈空间不算做额外的空间复杂度。

 

**示例：**

![img](https://assets.leetcode.com/uploads/2019/02/14/116_sample.png)

```
输入：root = [1,2,3,4,5,6,7]
输出：[1,#,2,3,#,4,5,6,7,#]
解释：给定二叉树如图 A 所示，你的函数应该填充它的每个 next 指针，以指向其下一个右侧节点，如图 B 所示。序列化的输出按层序遍历排列，同一层节点由 next 指针连接，'#' 标志着每一层的结束。
```

 

**提示：**

- 树中节点的数量少于 `4096`
- `-1000 <= node.val <= 1000`



**思路：层序遍历**

```java
//arraylist遍历
class Solution {
    public Node connect(Node root) {
        Queue<Node> que = new LinkedList<>();
        if (root != null)
            que.offer(root);
        while (!que.isEmpty()) {
            int size = que.size();
            ArrayList<Node> arrayList = new ArrayList<>();
            for (int i = 0; i < size; i++) {
                Node tmp = que.poll();
                arrayList.add(tmp);
                if (tmp.left != null)
                    que.offer(tmp.left);
                if (tmp.right != null)
                    que.offer(tmp.right);
            }
            for (int i = 0; i < arrayList.size() - 1; i++) {
                arrayList.get(i).next = arrayList.get(i + 1);
            }
            arrayList.get(arrayList.size() - 1).next = null;
        }
        return root;
    }
}

//指针遍历
class Solution {
    public Node connect(Node root) {
        Queue<Node> que = new LinkedList<>();
        if (root != null)
            que.offer(root);
        while (!que.isEmpty()) {
            int size = que.size();
            Node tmp;
            Node pre=null;
            for (int i = 0; i < size; i++) {
                if (i==0){
                    pre=que.poll();
                    tmp=pre;
                }else {
                    tmp=que.poll();
                    pre.next=tmp;
                    pre=pre.next;
                }
                if (tmp.left != null)
                    que.offer(tmp.left);
                if (tmp.right != null)
                    que.offer(tmp.right);
            }
            pre.next=null;
        }
        return root;
    }
}
```

