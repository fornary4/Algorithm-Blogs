#### [93. 复原 IP 地址](https://leetcode-cn.com/problems/restore-ip-addresses/)

难度中等676收藏分享切换为英文接收动态反馈

给定一个只包含数字的字符串，用以表示一个 IP 地址，返回所有可能从 `s` 获得的 **有效 IP 地址** 。你可以按任何顺序返回答案。

**有效 IP 地址** 正好由四个整数（每个整数位于 0 到 255 之间组成，且不能含有前导 `0`），整数之间用 `'.'` 分隔。

例如："0.1.2.201" 和 "192.168.1.1" 是 **有效** IP 地址，但是 "0.011.255.245"、"192.168.1.312" 和 "192.168@1.1" 是 **无效** IP 地址。

 

**示例 1：**

```
输入：s = "25525511135"
输出：["255.255.11.135","255.255.111.35"]
```

**示例 2：**

```
输入：s = "0000"
输出：["0.0.0.0"]
```

**示例 3：**

```
输入：s = "1111"
输出：["1.1.1.1"]
```

**示例 4：**

```
输入：s = "010010"
输出：["0.10.0.10","0.100.1.0"]
```

**示例 5：**

```
输入：s = "101023"
输出：["1.0.10.23","1.0.102.3","10.1.0.23","10.10.2.3","101.0.2.3"]
```

 

**提示：**

- `0 <= s.length <= 3000`
- `s` 仅由数字组成



**思路：回溯**

==这道题细节比较多，需要掌握String，StringBuild的常用操作==

```java
class Solution {
    List<String> res = new ArrayList<>();
    List<String> path = new ArrayList<>();

    public List<String> restoreIpAddresses(String s) {
        if (s.length() > 12)//如果字符串长度大于12，不可n
            return res;
        backTrack(s, 0);
        return res;
    }

    private void backTrack(String s, int start) {
        if (path.size() == 4) {
            if (totalLength(path) == s.length() && judge(path))
                res.add(changeToString(path));
            return;
        }
        for (int i = start; i < s.length() && i <= start + 2; i++) {
            path.add(s.substring(start, i + 1));
            backTrack(s, i + 1);
            path.remove(path.size() - 1);
        }
    }

    //计算list的总长度
    private int totalLength(List<String> list) {
        int res = 0;
        for (String s : list)
            res += s.length();
        return res;
    }

    //将path转成ip字符串
    private String changeToString(List<String> list) {
        StringBuilder sb = new StringBuilder();
        for (String s : list) {
            sb.append(s);
            sb.append(".");
        }
        sb.deleteCharAt(sb.length() - 1);
        return sb.toString();
    }

    //判断数字的合法性
    private boolean judge(List<String> list) {
        boolean res = true;
        for (String s : list)
            if (s.length() > 1 && s.charAt(0) == '0' || Integer.parseInt(s) > 255)
                res = false;
        return res;
    }
}
```

