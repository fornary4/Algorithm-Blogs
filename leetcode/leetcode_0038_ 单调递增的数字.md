#### [38. 单调递增的数字](https://leetcode-cn.com/problems/monotone-increasing-digits/)

难度中等206

给定一个非负整数 `N`，找出小于或等于 `N` 的最大的整数，同时这个整数需要满足其各个位数上的数字是单调递增。

（当且仅当每个相邻位数上的数字 `x` 和 `y` 满足 `x <= y` 时，我们称这个整数是单调递增的。）

**示例 1:**

```
输入: N = 10
输出: 9
```

**示例 2:**

```
输入: N = 1234
输出: 1234
```

**示例 3:**

```
输入: N = 332
输出: 299
```

**说明:** `N` 是在 `[0, 10^9]` 范围内的一个整数。



**思路：递归**

```java
class Solution {
    public int monotoneIncreasingDigits(int n) {
        char[] arr = Integer.toString(n).toCharArray();
        for (int i = 0; i < arr.length - 1; i++){
            if (arr[i] > arr[i + 1]){
                for (int j = i + 1; j < arr.length; j++){
                    arr[j] = '9';
                }
                String pre = new String(arr).substring(0, i + 1);
                int preint = monotoneIncreasingDigits(Integer.parseInt(pre) - 1);
                String tmp = Integer.toString(preint);
                if (tmp.length() < i + 1){
                    arr[0] = '0';
                    int k = 1;
                    for (char c : tmp.toCharArray()){
                        arr[i] = c;
                        k++;
                    }
                }
                else{
                for (int j = 0; j <= i; j++)
                    arr[j] = tmp.charAt(j);
                }
            }
        }
        return Integer.parseInt(new String(arr));
    }
}

//巧妙解法
class Solution {
    public int monotoneIncreasingDigits(int n) {
        char[] arr = Integer.toString(n).toCharArray();
        int flag = arr.length;
        for (int i = arr.length - 1; i > 0; i--){
            if (arr[i] < arr[i - 1]){
                flag = i;
                arr[i - 1]--;
            }
        }
        for (int i = flag; i < arr.length; i++)
            arr[i] = '9';
        return Integer.parseInt(new String(arr));
    }
}
```

