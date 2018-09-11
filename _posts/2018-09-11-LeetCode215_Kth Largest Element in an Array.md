---
layout:     post
title:      2018-09-11-LeetCode215_Kth Largest Element in an Array
subtitle:   
date:       2018-09-11
author:     Richbabe
header-img: img/post-bg-universe.jpg
catalog: true
tags:
    - LeetCode
---
# 题目
题目地址：[LeetCode 215:Kth Largest Element in an Array](https://leetcode.com/problems/kth-largest-element-in-an-array/description/)
![image](https://github.com/Richbabe/Richbabe.github.io/blob/master/img/LeetCode/LeetCode215.png?raw=true)

# 思路
## 方法一：排序后取出第K大的数
* 时间复杂度为O（nlogn）

直接调用sort函数获取第K大的数。

代码如下：

```
class Solution {
public:
    int findKthLargest(vector<int>& nums, int k) {
        sort(nums.begin(),nums.end());
        return nums[nums.size() - k];
    }
};
```

其实这道题可以用TopK的做法来解决，可以不通过排序直接获得第K最大的数。

## 方法二：基于快排划分的思想
* 时间复杂度为O（n）

通过Partition函数，基于数组第n-k个数字来调整，使得比第n-k个数字小的所有数字都位于数组的左边，比第n-k个数字大的所有数字都位于数组的右边。这样调整之后，位于数组中第n-k个数字即是第k最大的数

代码如下：

```
class Solution {
public:
    int findKthLargest(vector<int>& nums, int k) {
        int res = -1;
        if(nums.size() <= 0 || k <= 0 ||k > nums.size()){
            return res;
        }
        int n = nums.size();
        int start = 0;
        int end = n - 1;
        int index = Partition(nums,n,start,end);
        while(index != n - k){
            if(index >= n - k){
                end = index - 1;
                index = Partition(nums,n,start,end);
            }
           else{
               start = index + 1;
               index = Partition(nums,n,start,end);
           }
        }
        res = nums[n - k];
        return res;

    }
    void Swap(vector<int> &a,int i,int j){
        int temp = a[i];
        a[i] = a[j];
        a[j] = temp;
    }
    int Partition(vector<int> &a,int n,int start,int end){
        int index = rand() % (end - start + 1) + start;
        Swap(a,index,end);
        int small = start - 1;
        //保证small及small之前的数小于基准数
        for(index = start;index < end;index++){
            if(a[index] < a[end]){
                small++;
                if(small != index){
                    Swap(a,index,small);
                }
            }
        }
        small++;
        Swap(a,small,end);
        return small; 
    } 

};
```

## 方法三：使用小根堆或者红黑树，适合海量数据处理
* 时间复杂度为O(nlogk)

使用一个大小为k的容器来保存最大k个数，该容器可以是小根堆或者是红黑树，这样我们在取容器中最小元素（即数组中第k最大数），调整数时用O（logk），这样遍历n个数下来可以在O（nlogk）时间复杂度得到k个最大数。在这里我是用multiset作为容器来保存k个最大数（multiset内部实现为红黑树，不用set是因为有可能数组中有相同值的数，set会自动清除重复值！）

代码如下：

```
class Solution {
public:
    int findKthLargest(vector<int>& nums, int k) {
        int res = -1;
        if(nums.size() <= 0 || k <= 0 ||k > nums.size()){
            return res;
        }
        multiset<int,less<int>> leastNumbers;//按照升序排序，保存最小k个数
        leastNumbers.clear();
        for(vector<int>::iterator iter = nums.begin();iter != nums.end();iter++){
            if(leastNumbers.size() < k){
                leastNumbers.insert(*iter);
            }
            else{
                multiset<int,greater<int>>::iterator iterGreatest = leastNumbers.begin();//k个数中最小数
                if(*iter > *iterGreatest){
                    leastNumbers.erase(iterGreatest);
                    leastNumbers.insert(*iter);
                }
            }
        }
        multiset<int,greater<int>>::iterator iter = leastNumbers.begin();
        res = *iter;
        return res;
    }
};
```

# 几种方法的比较
## 方法一：排序后取出第K大的数
不用说，这肯定是最差的方法，要是在面试中提出这样的方法肯定被面试官fail了！

## 方法二：基于快排划分的思想
优点：
时间复杂度小，只用O（n）

缺点：
* 需要修改原数组
* 不适合处理海量数据，当内存受限小于n时不适用

## 方法三：使用小根堆或者红黑树，适合海量数据处理
优点：
* 不需要修改原数组
* 适合处理海量数据，只要内存满足大于k即可，其他数可以分布式存储

缺点：时间复杂度大，需要O（nlogk）
