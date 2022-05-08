#### [763. 划分字母区间](https://leetcode-cn.com/problems/partition-labels/)



字符串 `S` 由小写字母组成。我们要把这个字符串划分为尽可能多的片段，同一字母最多出现在一个片段中。返回一个表示每个字符串片段的长度的列表。

 

**示例：**

```
输入：S = "ababcbacadefegdehijhklij"
输出：[9,7,8]
解释：
划分结果为 "ababcbaca", "defegde", "hijhklij"。
每个字母最多出现在一个片段中。
像 "ababcbacadefegde", "hijhklij" 的划分是错误的，因为划分的片段数较少。
```

 

**提示：**

- `S`的长度在`[1, 500]`之间。
- `S`只包含小写字母 `'a'` 到 `'z'` 。



**思路：计算每个元素的最后下标**

```java
//自己想的写法，时间复杂度有点高
class Solution {
    public List<Integer> partitionLabels(String s) {
        List<Integer> res = new ArrayList<>();
        int start = 0;
        while (start < s.length()) {
            int cur_last = getLast(s, s.charAt(start));
            for (int i = start + 1; i < cur_last; i++)
                cur_last = Math.max(cur_last, getLast(s, s.charAt(i)));
            res.add(cur_last - start + 1);
            start = cur_last + 1;
        }
        return res;
    }

    public int getLast(String s, char c) {
        int last = 0;
        for (int i = 0; i < s.length(); i++)
            if (s.charAt(i) == c)
                last = i;
        return last;
    }
}

//巧妙的写法，利用字符与下标的映射
class Solution {
    public List<Integer> partitionLabels(String s) {
        List<Integer> res = new ArrayList<>();
        int[] edge = new int[123];
        char[] chars = s.toCharArray();
        for (int i = 0; i < chars.length; i++)
            edge[chars[i] - 0] = i;
        int index = 0;
        int last = -1;
        for (int i = 0; i < chars.length; i++){
            index = Math.max(index, edge[chars[i] - 0]);
            if (i == index){
                res.add(i - last);
                last = i;
            }
        }
        return res;
    }

}
```

