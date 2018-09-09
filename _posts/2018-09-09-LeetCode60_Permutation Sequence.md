---
layout:     post
title:      LeetCode60_Permutation Sequence
subtitle:   
date:       2018-09-09
author:     Richbabe
header-img: img/post-bg-universe.jpg
catalog: true
tags:
    - LeetCode
---
# 题目
题目地址：[LeetCode 60:Permutation Sequence](https://leetcode.com/problems/permutation-sequence/description/)
![image](https://github.com/Richbabe/Richbabe.github.io/blob/master/img/LeetCode/LeetCode60.png?raw=true)

# 思路
一开始使用了backtracking的方法构造出全排列，当构造出第K个排列时返回，此时显示超时！代码如下：

```
class Solution {
public:
    int count = 0;
    string getPermutation(int n, int k) {
        vector<int> nums;
        for(int i = 0;i < n;i++){
            nums.push_back(i + 1);
        }
        vector<int> resTemp;//第i个全排列
        vector<int> res;//第k个全排列
        vector<bool> hasSeen(n,false);
        fullArray(nums,n,k,resTemp,res,hasSeen);
        string s = "";
        for(int i = 0;i < res.size();i++){
            s += to_string(res[i]);
        }
        return s;
    }
    void fullArray(vector<int> nums,int n,int k,vector<int> &resTemp,vector<int> &res,vector<bool> &hasSeen){
        if(resTemp.size() == n){
            count++;
            if(count == k){
                res = resTemp;
                return;
            }
        }
        else{
            for(int i = 0;i < n;i++){
                if(hasSeen[i]){
                    continue;
                }
                resTemp.push_back(nums[i]);
                hasSeen[i] = true;
                fullArray(nums,n,k,resTemp,res,hasSeen);
                resTemp.pop_back();
                hasSeen[i] = false;
            }
        }
    }
};
```
后面仔细观察了一下全排列的集合，发现这其中可以找到一些规律来最快解决这道题。
举个例子，当n = 4，k = 17时，一共有4! = 24种排列方式：

```
1234
1243
1324
1342
1423
1432
2134
2143
2314 
2341
2413
2431
3124
3142
3214
3241
3412	<--- k = 17
3421
4123
4132
4213
4231
4312
4321
```
通过观察可以发现。每一位上的1，2，3，4分别出现了6次。当第一位上的数字确定了，后面三位上每个数字都出现了2次，当第二位也确定了，后面的数字都只出现了1次，当第三位确定了，那么第四位上的数字也只能出现一次，那么下面我们来看k = 17这种情况的每位数字如何确定，由于k = 17是转化为数组下标为16：

（1）最高位可取1,2,3,4中的一个，每个数字出现3！= 6次，所以k = 16的第一位数字的下标为16 / 6 = 2，即3被取出

（2）第二位此时从1,2,4中取一个，k = 16是此时的k' = 16 % (3!) = 4，而剩下的每个数字出现2！= 2次，所以第二数字的下标为4 / 2 = 2，即4被取出

（3）第三位此时从1,2中去一个，k' = 4是此时的k'' = 4 % (2!) = 0，而剩下的每个数字出现1！= 1次，所以第三个数字的下标为 0 / 1 = 0，即1被取出

（4）第四位是从2中取一个，k'' = 0是此时的k''' = 0 % (1!) = 0，而剩下的每个数字出现0！= 1次，所以第四个数字的下标为0 / 1= 0，即2被取出

因此第17个数为3412！

那么我们便可以找出规律：

```
a1 = k / (n - 1)!
k1 = k

a2 = k1 / (n - 2)!
k2 = k1 % (n - 2)!
...

an-1 = kn-2 / 1!
kn-1 = kn-2 / 1!

an = kn-1 / 0!
kn = kn-1 % 0!
```
代码如下：

```
class Solution {
public:
    string getPermutation(int n, int k) {
        string res;
        string nums = "123456789";
        vector<int> f(n,1);//保存0!到(n-1)!
        for(int i = 1;i < n;i++){
            f[i] = f[i - 1] * i;
        }
        k--;//数组下标为k-1
        for(int i = n;i >= 1;i--){
            int j = k / f[i - 1];
            k = k % f[i - 1];
            res += nums[j];
            nums.erase(j,1);//记得擦除已经使用过的数
        }
        return res;
    }
};
```
可以发现，该算法时间复杂度为O(n)，比backtracking快了很多！



