#### [131. 分割回文串](https://leetcode-cn.com/problems/palindrome-partitioning/)



给你一个字符串 `s`，请你将 `s` 分割成一些子串，使每个子串都是 **回文串** 。返回 `s` 所有可能的分割方案。

**回文串** 是正着读和反着读都一样的字符串。

 

**示例 1：**

```
输入：s = "aab"
输出：[["a","a","b"],["aa","b"]]
```

**示例 2：**

```
输入：s = "a"
输出：[["a"]]
```

 

**提示：**

- `1 <= s.length <= 16`
- `s` 仅由小写英文字母组成



**思路：回溯**

```java
/* 简单粗暴的解法 */
class Solution {
    List<List<String>> res = new ArrayList<>();
    List<String> path = new ArrayList<>();
    public List<List<String>> partition(String s) {
        backTrack(s, 0);
        return res;
    }

    private void backTrack(String s, int start){
        if (start == s.length()){
            if (judge(path))
                res.add(new ArrayList(path));
            return;
        }
        for (int i = start; i < s.length(); i++){
            path.add(s.substring(start, i + 1)); //用substring截取
            backTrack(s, i + 1);
            path.remove(path.size() - 1);
        }
    }

    private boolean judge(List<String> list){
        boolean flag = true;
        for (String s : list)
            if (judge(s) == false)
                flag = false;
        return flag;
    }

    private boolean judge (String s){
        String r = new StringBuilder(s).reverse().toString(); // 判断是否为回文字符串
        return r.equals(s);
    }

}
```

lambda是定义函数的，可以看成函数f(x)的f

例如，lambda x : x + 1可以理解为f(x) = x + 1

通常map和lambda搭配使用，表示将满足条件的元素加入集合

如果是lambda x ： x +  1

表示将x 换成x + 1加入map

trade_date表示日期，是string类型,例如20181228 ，其中第四位和第五位表示月份

lambda x: x.find('12', 4, 6) >= 0的意思是如果trade_date的月份是12，就将trade_date加入map

简单来说，就是筛选出12月的数据

