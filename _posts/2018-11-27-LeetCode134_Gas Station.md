---
layout:     post
title:      LeetCode134_Gas Station
subtitle:   
date:       2018-11-27
author:     Richbabe
header-img: img/post-bg-universe.jpg
catalog: true
tags:
    - LeetCode
---
# 题目
题目地址：[LeetCode 86:Partition List](https://leetcode.com/problems/gas-station/)
![image](https://github.com/Richbabe/Richbabe.github.io/blob/master/img/LeetCode/LeetCode134_1.png?raw=true)
![image](https://github.com/Richbabe/Richbabe.github.io/blob/master/img/LeetCode/LeetCode134_2.png?raw=true)
![image](https://github.com/Richbabe/Richbabe.github.io/blob/master/img/LeetCode/LeetCode134_3.png?raw=true)

# 思路
这道题打上的是贪心的tag，那么我们可以这么想：首先如果要走完整个环，那么gas总和肯定要大过cost的总和。因此我们声明一个totalSum来记录每对gas和cost的差的总和。然后我们假设开始的起点res = 0，并从这里出发。接着我们声明一个curSum记录当前路径上每对gas和cost的差。如果遍历到某个点时curSum < 0，说明该路径（从起点到当前点）上任何一点都不能作为起点走完整个环，此时令curSum重置为0，res为下一个点i+1。

# 代码
C++版本：

```
class Solution {
public:
    int canCompleteCircuit(vector<int>& gas, vector<int>& cost) {
        int res = 0;
        int curSum = 0;
        int totalSum = 0;
        for(int i = 0;i < gas.size();i++){
            curSum += gas[i] - cost[i];
            totalSum += gas[i] - cost[i];
            if(curSum < 0){
                res = i + 1;
                curSum = 0;
            }
        }
        return totalSum >= 0 ? res : -1;
    }
};
```

Python版本：

```
class Solution(object):
    def canCompleteCircuit(self, gas, cost):
        """
        :type gas: List[int]
        :type cost: List[int]
        :rtype: int
        """
        res = 0
        curSum = 0
        totalSum = 0
        for i in range(0,len(gas)) :
            curSum += gas[i] - cost[i]
            totalSum += gas[i] - cost[i]
            if curSum < 0 :
                res = i + 1
                curSum = 0
        if totalSum < 0 :
            return -1
        else :
            return res
        
```
