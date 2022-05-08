#### [239. 滑动窗口最大值](https://leetcode-cn.com/problems/sliding-window-maximum/)

难度困难1207

给你一个整数数组 `nums`，有一个大小为 `k` 的滑动窗口从数组的最左侧移动到数组的最右侧。你只可以看到在滑动窗口内的 `k` 个数字。滑动窗口每次只向右移动一位。

返回滑动窗口中的最大值。

 

**示例 1：**

```
输入：nums = [1,3,-1,-3,5,3,6,7], k = 3
输出：[3,3,5,5,6,7]
解释：
滑动窗口的位置                最大值
---------------               -----
[1  3  -1] -3  5  3  6  7       3
 1 [3  -1  -3] 5  3  6  7       3
 1  3 [-1  -3  5] 3  6  7       5
 1  3  -1 [-3  5  3] 6  7       5
 1  3  -1  -3 [5  3  6] 7       6
 1  3  -1  -3  5 [3  6  7]      7
```

**示例 2：**

```
输入：nums = [1], k = 1
输出：[1]
```

**示例 3：**

```
输入：nums = [1,-1], k = 1
输出：[1,-1]
```

**示例 4：**

```
输入：nums = [9,11], k = 2
输出：[11]
```

**示例 5：**

```
输入：nums = [4,-2], k = 2
输出：[4]
```

 

**提示：**

- `1 <= nums.length <= 105`
- `-104 <= nums[i] <= 104`
- `1 <= k <= nums.length`



**思路：单调队列**

```java
class MyQueue{
    Deque<Integer> deq = new LinkedList<>();
    void poll(int val){
        if (!deq.isEmpty() && val == deq.peek())
            deq.poll();
    }
    void add(int val){
        while (!deq.isEmpty() && val > deq.getLast())
            deq.removeLast();
        deq.offer(val);
    }
    int peek(){
        return deq.peek();
    }
}

class Solution{
    public int[] maxSlidingWindow(int[] nums, int k){
        if (nums.length == 1)
            return nums;
        int len = nums.length - k + 1;
        int[] res = new int[len];
        MyQueue myQueue = new MyQueue();
        for (int i = 0; i < k; i++)
            myQueue.add(nums[i]);
        int num = 0;
        res[num++] = myQueue.peek();
        for (int i = k; i < nums.length; i++){
            myQueue.poll(nums[i - k]);
            myQueue.add(nums[i]);
            res[num++] = myQueue.peek();
        }
        return res;
        
    }
}
```

