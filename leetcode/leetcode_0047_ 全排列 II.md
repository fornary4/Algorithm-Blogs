#### [47. 全排列 II](https://leetcode-cn.com/problems/permutations-ii/)



给定一个可包含重复数字的序列 `nums` ，**按任意顺序** 返回所有不重复的全排列。

 

**示例 1：**

```
输入：nums = [1,1,2]
输出：
[[1,1,2],
 [1,2,1],
 [2,1,1]]
```

**示例 2：**

```
输入：nums = [1,2,3]
输出：[[1,2,3],[1,3,2],[2,1,3],[2,3,1],[3,1,2],[3,2,1]]
```

 

**提示：**

- `1 <= nums.length <= 8`
- `-10 <= nums[i] <= 10`



**思路：回溯**

**用used数组去重**

**注意理解去重逻辑**

```java
if (i > 0 && nums[i] == nums[i - 1] && used[i - 1] == false)
    continue;
```

```java
//简单粗暴写法
class Solution {
    List<List<Integer>> res = new LinkedList<>();
    LinkedList<Integer> path = new LinkedList<>();
    Map<Integer,Integer> map = new HashMap<>();
    public List<List<Integer>> permuteUnique(int[] nums) {
        Arrays.sort(nums);
        for (Integer i : nums){
            if (map.containsKey(i))
                map.put(i, map.get(i) + 1);
            else
                map.put(i, 1);
        }
        backTrack(nums);
        return res;
    }
    
    private void backTrack(int[] nums){
        if (path.size() == nums.length){
            res.add(new LinkedList<>(path));
            return;
        }
        for (int i = 0; i < nums.length; i++){
            if (i > 0 && nums[i] == nums[i - 1])
                continue;
            if(!contains(nums[i])){
                path.add(nums[i]);
                backTrack(nums);
                path.removeLast();
            }
        }
    }
    
    private boolean contains(int x){
        return count(x) == map.get(x); 
    }
    
    private int count(int x){
        int res = 0;
        for (Integer i : path)
            if (i == x)
                res++;
        return res;
    }
}

//简洁写法，用used数组去重
class Solution {
    List<List<Integer>> res = new LinkedList<>();
    LinkedList<Integer> path = new LinkedList<>();

    public List<List<Integer>> permuteUnique(int[] nums) {
        boolean[] used = new boolean[nums.length];
        Arrays.fill(used, false);
        Arrays.sort(nums);
        backTrack(nums, used);
        return res;
    }

    public void backTrack(int[] nums, boolean[] used){
        if (path.size() == nums.length){
            res.add(new LinkedList<>(path));
            return;
        }
        for (int i = 0; i < nums.length; i++){
            if (i > 0 && nums[i] == nums[i - 1] && used[i - 1] == false)
                continue;
            if (used[i] == false){
                used[i] = true;
                path.add(nums[i]);
                backTrack(nums, used);
                path.removeLast();
                used[i] = false;
            }
        }
    }
}


```

