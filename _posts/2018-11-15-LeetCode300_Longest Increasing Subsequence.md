---
layout:     post
title:      LeetCode300_Longest Increasing Subsequence
subtitle:   
date:       2018-11-15
author:     Richbabe
header-img: img/post-bg-universe.jpg
catalog: true
tags:
    - LeetCode
---
# 题目
题目地址：[LeetCode 300:Longest Increasing Subsequence](https://leetcode.com/problems/longest-increasing-subsequence)
![image](https://github.com/Richbabe/Richbabe.github.io/blob/master/img/LeetCode/LeetCode300.png?raw=true)

# 思路
LIS问题是比较简单的动态规划问题了。设dp[i]为以nums[i]结尾的LIS长度（i∈[0,n-1]），其状态转移方程如下：
> dp[i] = max(dp[j]) + 1（0 <= j < i && nums[j] < nums[i]）

最后只要返回最大的dp[i]（i∈[0,n-1]）即可。

# 代码
C++版本如下：

```
class Solution {
public:
    int lengthOfLIS(vector<int>& nums) {
        int n = nums.size();
        if(n <= 0){
            return 0;
        }
        int dp[n];//dp[i]表示以nums[i]为结尾的LIS长度
        for(int i = 0;i < n;i++){
            dp[i] = 1;
        }
        for(int i = 1;i < n;i++){
            int maxLength = 0;
            for(int j = 0;j < i;j++){
                if(nums[j] < nums[i]){
                    maxLength = max(maxLength,dp[j]);
                }
            }
            dp[i] = maxLength + 1;
        }
        int res = 0;
        for(int i = 0;i < n;i++){
            res = max(res,dp[i]);
        }
        return res;
    }
};
```

python版本如下：

```
class Solution(object):
    def lengthOfLIS(self, nums):
        """
        :type nums: List[int]
        :rtype: int
        """
        n = len(nums)
        if n <= 0 :
            return 0
        dp = [1] * n
        for i in range(1,n):
            maxLength = 0
            for j in range(0,i):
                if nums[j] < nums[i]:
                    maxLength = max(maxLength,dp[j])
            dp[i] = maxLength + 1
        dp.sort()
        return dp[n - 1]
```
