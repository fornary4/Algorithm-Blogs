#### [541. 反转字符串 II](https://leetcode-cn.com/problems/reverse-string-ii/)

难度简单194

给定一个字符串 `s` 和一个整数 `k`，从字符串开头算起，每计数至 `2k` 个字符，就反转这 `2k` 字符中的前 `k` 个字符。

- 如果剩余字符少于 `k` 个，则将剩余字符全部反转。
- 如果剩余字符小于 `2k` 但大于或等于 `k` 个，则反转前 `k` 个字符，其余字符保持原样。

 

**示例 1：**

```
输入：s = "abcdefg", k = 2
输出："bacdfeg"
```

**示例 2：**

```
输入：s = "abcd", k = 2
输出："bacd"
```

 

**提示：**

- `1 <= s.length <= 104`
- `s` 仅由小写英文组成
- `1 <= k <= 104`



**思路：分类讨论**

```java
class Solution {
    public String reverseStr(String s, int k) {
        char[] arr = s.toCharArray();
        int start = 0;
        while (start + 2 * k <= arr.length){
            reverse(arr, start, start + k);
            start += 2 * k;
        }
        if (start < arr.length){
            if (start + k <= arr.length )
                reverse(arr, start, start + k);
            else
                reverse(arr, start, arr.length);
        }
        return new String(arr);
    }

    public void reverse(char[] arr, int start, int end){
        int i = start, j = end - 1;
        while (i < j){
            char tmp = arr[i];
            arr[i] = arr[j];
            arr[j] = tmp;
            i++;
            j--;
        }
    }
}
```

