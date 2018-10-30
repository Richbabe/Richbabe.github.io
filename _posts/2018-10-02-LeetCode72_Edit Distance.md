---
layout:     post
title:      LeetCode72_Edit Distance
subtitle:   
date:       2018-10-02
author:     Richbabe
header-img: img/post-bg-universe.jpg
catalog: true
tags:
    - LeetCode
---
# 题目
题目地址：[LeetCode 72:Edit Distance](https://leetcode.com/problems/edit-distance/)
![image](https://github.com/Richbabe/Richbabe.github.io/blob/master/img/LeetCode/LeetCode72.png?raw=true)

# 思路
这一题可以用动态规划的方法来做。
首先来定义dp矩阵

```
定义dp[i][j]：word1这个字符串的前i个字符转换成word2的前j个字符所需要的最小步数
```
接着要设定一下状态转移方程,一共有以下几种情况：
1. 当word1[i - 1] == word2[j - 1]时（注意这里的-1，因为i和j表示第几个字符，如果是索引则要-1）：
    
    dp[i][j] = dp[i - 1][j - 1]

2. 当word1[i - 1] != word2[j - 1]时，则要进行题目所述的三种操作：
* add = dp[i][j - 1] + 1，代表插入一个字符
* delete = dp[i - 1][j] + 1，代表删除一个字符
* update = dp[i - 1][j - 1] + 1，代表替换一个字符

    dp[i][j] = min(add,delete,update)

最后返回dp[word1.size()][word2.size()即可

# 代码
C++版本如下：

```
class Solution {
public:
    int minDistance(string word1, string word2) {
        int m = word1.size();
        int n = word2.size();
        int dp[m + 1][n + 1];//dp[i][j]表示word1的前i个字符转换成word2前j个字符的编辑路径
        //初始化dp矩阵第一行(即添加操作)
        for(int i = 0;i <= n;i++){
            dp[0][i] = i;
        }
        //初始化dp矩阵的第一列（即删除操作）
        for(int i = 0;i <= m;i++){
            dp[i][0] = i;
        }
        for(int i = 1;i <= m;i++){
            for(int j = 1;j <= n;j++){
                //不需要进行增删改操作
                if(word1[i - 1] == word2[j - 1]){
                    dp[i][j] = dp[i - 1][j - 1];
                }
                //需要进行增删改操作
                else{
                    int add = dp[i][j - 1] + 1;
                    int del = dp[i - 1][j] + 1;
                    int update = dp[i - 1][j - 1] + 1;
                    dp[i][j] = min(add,min(del,update));
                }
            }
        }
        return dp[m][n];
    }
};
```
Python版本如下：

```
class Solution(object):
    def minDistance(self, word1, word2):
        """
        :type word1: str
        :type word2: str
        :rtype: int
        """
        m = len(word1)
        n = len(word2)
        #dp[i][j]表示word1的前i个字符转换成word2前j个字符的编辑路径
        dp = [([0] * (n + 1)) for i in range(m + 1)]
        #初始化dp矩阵第一行(即添加操作)
        for i in range(n + 1):
            dp[0][i] = i
        #初始化dp矩阵第一列（即删除操作）
        for i in range(m + 1):
            dp[i][0] = i
        #计算dp矩阵
        for i in range(1,m + 1):
            for j in range(1,n + 1):
                if word1[i - 1] == word2[j - 1]:
                    dp[i][j] = dp[i - 1][j - 1]
                else:
                    add = dp[i][j - 1] + 1
                    delete = dp[i - 1][j] + 1
                    update = dp[i - 1][j - 1] + 1
                    dp[i][j] = min(add,delete,update)
        return dp[m][n]
        
        
```
