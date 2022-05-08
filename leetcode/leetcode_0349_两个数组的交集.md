#### [349. 两个数组的交集](https://leetcode-cn.com/problems/intersection-of-two-arrays/)

给定两个数组，编写一个函数来计算它们的交集。

 

**示例 1：**

```
输入：nums1 = [1,2,2,1], nums2 = [2,2]
输出：[2]
```

**示例 2：**

```
输入：nums1 = [4,9,5], nums2 = [9,4,9,8,4]
输出：[9,4]
```

 

**说明：**

- 输出结果中的每个元素一定是唯一的。
- 我们可以不考虑输出结果的顺序。



**思路：使用unorder_set**

```c++
class Solution {
public:
    vector<int> intersection(vector<int>& num1,vector<int>& num2){
    unordered_set<int> result_set;
    unordered_set<int> num_set(num1.begin(),num1.end());
    for(int num:num2)
        if(num_set.find(num)!=num_set.end())
            result_set.insert(num);//若存在就放入结果set
    return vector<int>(result_set.begin(),result_set.end());   
    }
};
```

```java
class Solution {
   public static int[] intersection(int[] a,int[] b){
        Set<Integer> tmp_set=new HashSet<>();
        Set<Integer> res_set=new HashSet<>();
        for(int i:a)
            tmp_set.add(i);
        for(int i:b)
            if(tmp_set.contains(i))
                res_set.add(i);
            int index=0;
        int[] res=new int[res_set.size()];
        for(int i:res_set)
            res[index++]=i;
        return res;
    }
}
```

