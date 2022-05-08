#### [17. 电话号码的字母组合](https://leetcode-cn.com/problems/letter-combinations-of-a-phone-number/)

难度中等1500收藏分享切换为英文接收动态反馈

给定一个仅包含数字 `2-9` 的字符串，返回所有它能表示的字母组合。答案可以按 **任意顺序** 返回。

给出数字到字母的映射如下（与电话按键相同）。注意 1 不对应任何字母。

![img](https://assets.leetcode-cn.com/aliyun-lc-upload/original_images/17_telephone_keypad.png)

 

**示例 1：**

```
输入：digits = "23"
输出：["ad","ae","af","bd","be","bf","cd","ce","cf"]
```

**示例 2：**

```
输入：digits = ""
输出：[]
```

**示例 3：**

```
输入：digits = "2"
输出：["a","b","c"]
```

 

**提示：**

- `0 <= digits.length <= 4`
- `digits[i]` 是范围 `['2', '9']` 的一个数字。



**思路：利用回溯法解题**

---

==这道题非常有意思，涉及的知识点有很多，里面有很多值得注意的细节==

- 为方便处理，使用StringBuild，要熟悉StringBuild和String的常用方法
- java是引用传递，类就是指针，当类为参数时，要考虑为null的情况
- '9' - '0'可以把字符9转化为数字9

```java
class Solution {
    String[] relation = {"","","abc","def","ghi","jkl","mno","pqrs","tuv","wxyz"};
    List<String> res = new ArrayList<>();
    StringBuilder path = new StringBuilder();
    public List<String> letterCombinations(String digits) {
        if (digits == null || digits == "")  //这里一定要注意digits == null的情况，否则会出错
            return res;
        backTrack(digits, 0);
        return res;
    }

    private void backTrack(String digits, int index){
        if (path.length() == digits.length()){
            res.add(path.toString()); // StringBuilder转String
            return;
        }
        for (int i = 0; i < relation[digits.charAt(index) - '0'].length(); i++){ //字符转数字
            path.append(relation[digits.charAt(index) - '0'].charAt(i));
            backTrack(digits, index + 1);
            path.deleteCharAt(path.length() - 1);
        }
    }
}
```

