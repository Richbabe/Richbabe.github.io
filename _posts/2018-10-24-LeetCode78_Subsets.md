---
layout:     post
title:      LeetCode78_Subsets
subtitle:   
date:       2018-10-18
author:     Richbabe
header-img: img/post-bg-universe.jpg
catalog: true
tags:
    - LeetCode
---
# 题目
题目地址：[LeetCode 78:Subsets](https://leetcode.com/problems/subsets/description/)
![image](https://github.com/Richbabe/Richbabe.github.io/blob/master/img/LeetCode/lc78.png?raw=true)

# 思路
当我们遍历nums中的每一个数时，我们需要将这个数插入到我们的子集中的每个集合中。因此我们可以先声明一个空子集，每遍历一个数我们便把子集中的每个集合复制一份，加入该数再压入到子集中。

# 代码
C++版本如下：

```
class Solution {
public:
    vector<vector<int>> subsets(vector<int>& nums) {
        vector<vector<int> > res(1);
        int n = nums.size();
        if(n <= 0){
            return res;
        }
        for(int i = 0;i < n;i++){
            int size = res.size();
            for(int j = 0;j < size;j++){
                res.push_back(res[j]);
                res.back().push_back(nums[i]);
            }
        }
        return res;
    }
};
```
Python版本如下：

```
class Solution(object):
    def subsets(self, nums):
        """
        :type nums: List[int]
        :rtype: List[List[int]]
        """
        res = [[]]
        for n in nums:
            res += [r+[n] for r in res]
        
        return res
```

# 扩展
当集合包含重复元素时，我们用上述的办法可能会出现重复的子集，那要怎么求得的所有子集是唯一的呢？

题目地址：[LeetCode 90:Subsets II](https://leetcode.com/problems/subsets-ii/description/)
![image](https://github.com/Richbabe/Richbabe.github.io/blob/master/img/LeetCode/leetcode90.png?raw=true)

# 思路
可以将原始集合排序，然后每次选择元素时，判断是否与之前选择的元素相同，从而进行去重复。可以通过count来记录添加上一个元素时的集合个数，当遍历到当前元素时，如果当前元素 == 上一个元素，则只对count之后的集合进行添加元素的操作。

# 代码
C++代码如下：
```
class Solution {
public:
    vector<vector<int>> subsetsWithDup(vector<int>& nums) {
        vector<int> v(nums);
        sort(v.begin(),v.end());
        vector<vector<int> > res(1);
        int n = v.size();
        if(n <= 0){
            return res;
        }
        int count = 0;
        for(int i = 0;i < n;i++){
            int size = res.size();
            for(int j = 0;j < size;j++){
                if(i == 0 || v[i] != v[i - 1] || j >= count){
                    res.push_back(res[j]);
                    res.back().push_back(v[i]);
                }
            }
            count = size;
        }
        return res;
    }
};
```
Python代码如下：

```
class Solution(object):
    def subsetsWithDup(self, nums):
        """
        :type nums: List[int]
        :rtype: List[List[int]]
        """
        res = [[]]
        nums.sort()
    
        for n in nums:
            res += [r+[n] for r in res if r+[n] not in res]
        
        return res
```



