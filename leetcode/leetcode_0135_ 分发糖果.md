#### [135. 分发糖果](https://leetcode-cn.com/problems/candy/)



老师想给孩子们分发糖果，有 *N* 个孩子站成了一条直线，老师会根据每个孩子的表现，预先给他们评分。

你需要按照以下要求，帮助老师给这些孩子分发糖果：

- 每个孩子至少分配到 1 个糖果。
- 评分更高的孩子必须比他两侧的邻位孩子获得更多的糖果。

那么这样下来，老师至少需要准备多少颗糖果呢？

 

**示例 1：**

```
输入：[1,0,2]
输出：5
解释：你可以分别给这三个孩子分发 2、1、2 颗糖果。
```

**示例 2：**

```
输入：[1,2,2]
输出：4
解释：你可以分别给这三个孩子分发 1、2、1 颗糖果。
     第三个孩子只得到 1 颗糖果，这已满足上述两个条件。
```

**思路：贪心**

```java
class Solution {
    public int candy(int[] ratings) {
        if (ratings.length == 1)
            return 1;
        int[] count = new int[ratings.length];
        Arrays.fill(count, 1);
        int[] opea = new int[ratings.length];
        for (int i = 0; i < ratings.length; i++)
            opea[i] = ratings[i];
        Arrays.sort(opea);
        ArrayList<Integer> arr = new ArrayList<>();
        for (int i = 0; i < opea.length; i++){
            if (i > 0 && opea[i] == opea[i - 1])
                continue;
            arr.add(opea[i]);
        }
        for (int i = 0; i < arr.size(); i++){
            for (int j = 0; j < ratings.length; j++){
                if (arr.get(i) == ratings[j]){
                    if (j == 0){
                        if (ratings[j] > ratings[j + 1])
                            count[j] = count[j + 1] + 1;
                    }
                    else if (j == ratings.length - 1){
                        if (ratings[j] > ratings[j - 1])
                            count[j] = count[j - 1] + 1;
                    }
                    else{
                        if (ratings[j] > ratings[j - 1] && ratings[j] > ratings[j + 1])
                            count[j] = Math.max(count[j - 1], count[j + 1]) + 1;
                        else if (ratings[j] > ratings[j - 1] && ratings[j] <= ratings[j + 1])
                            count[j] = count[j - 1] + 1;
                        else if (ratings[j] <= ratings[j - 1] && ratings[j] > ratings[j + 1])
                            count[j] = count[j + 1] + 1;

                    }
                }
            }
        }
        int res = 0;
        for (Integer i : count)
            res += i;
        return res;
    }
}
```

