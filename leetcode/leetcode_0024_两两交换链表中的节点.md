#### [24. 两两交换链表中的节点](https://leetcode-cn.com/problems/swap-nodes-in-pairs/)



给定一个链表，两两交换其中相邻的节点，并返回交换后的链表。

**你不能只是单纯的改变节点内部的值**，而是需要实际的进行节点交换。

 

**示例 1：**

![img](https://assets.leetcode.com/uploads/2020/10/03/swap_ex1.jpg)

```
输入：head = [1,2,3,4]
输出：[2,1,4,3]
```

**示例 2：**

```
输入：head = []
输出：[]
```

**示例 3：**

```
输入：head = [1]
输出：[1]
```

 

**提示：**

- 链表中节点的数目在范围 `[0, 100]` 内
- `0 <= Node.val <= 100`

 

**进阶：**你能在不修改链表节点值的情况下解决这个问题吗?（也就是说，仅修改节点本身。）



**思路：增加虚拟头结点，两两交换节点**

```java
class Solution {
    public ListNode swapPairs(ListNode head) {
        ListNode dump=new ListNode(-1,head);//虚拟头结点
        ListNode cur=dump;
        while(cur.next!=null&&cur.next.next!=null){
            ListNode tmp=cur.next;
            ListNode tmp1=cur.next.next.next;
            cur.next=cur.next.next;
            cur.next.next=tmp;
            cur.next.next.next=tmp1;
            cur=cur.next.next;
        }
        return dump.next;
    }
}

```

