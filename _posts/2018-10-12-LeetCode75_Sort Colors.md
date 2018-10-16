---
layout:     post
title:      LeetCode75_Sort Colors
subtitle:   
date:       2018-10-12
author:     Richbabe
header-img: img/post-bg-universe.jpg
catalog: true
tags:
    - LeetCode
---
# 题目
题目地址：[LeetCode 75:Sort Colors](https://leetcode.com/problems/sort-colors/)
![image](https://github.com/Richbabe/Richbabe.github.io/blob/master/img/LeetCode/LeetCode75.png?raw=true)

# 思路
这一题也被称为荷兰国旗问题，是可以通过双指针的方法在O(n)的时间复杂度和O（1）的空间复杂度解决的。具体思路如下：

1. 在数组左侧设置0区域为-1，右侧设置2区域为n

![image](https://github.com/Richbabe/Richbabe.github.io/blob/master/img/LeetCode/lc75-1.png?raw=true)

2. 从前到后遍历数组，如果当前数为0的话，与0区域的下一个位置进行交换

![image](https://github.com/Richbabe/Richbabe.github.io/blob/master/img/LeetCode/lc75-2.png?raw=true)

3.然后让0区域往右扩大即+1

![image](https://github.com/Richbabe/Richbabe.github.io/blob/master/img/LeetCode/lc75-3.png?raw=true)

4. 同理当前数为2的时候就和2区域的前一个数进行交换

![image](https://github.com/Richbabe/Richbabe.github.io/blob/master/img/LeetCode/lc75-4.png?raw=true)

需要注意的是：因为2区域的前一个数为未遍历过的数，因此我们应该停留在当前位置去看该数是不是0或者2，即currentIndex不能++

# 代码
C++版本如下：

```
class Solution {
public:
    void sortColors(vector<int>& nums) {
        int n = nums.size();
        if(n <= 0){
            return;
        }
        int left = -1;//左区域
        int right = n;//右区域
        int currentIndex = 0;//当前数组下标
        while(currentIndex < right){
            if(nums[currentIndex] == 1){
                currentIndex++;
            }
            else if(nums[currentIndex] == 0){
                swap(nums,currentIndex,left + 1);
                currentIndex++;
                left++;
            }
            else{
                swap(nums,currentIndex,right - 1);
                right--;
                //停留在当前位置，currentIndex不++
            }
        }
    }
    void swap(vector<int> &nums,int i,int j){
        int temp = nums[i];
        nums[i] = nums[j];
        nums[j] = temp;
    }
};
```

Python版本如下：

```
class Solution(object):
    def sortColors(self, nums):
        """
        :type nums: List[int]
        :rtype: void Do not return anything, modify nums in-place instead.
        """
        n = len(nums)
        if n <= 0 :
            return
        left = -1
        right = n
        currentIndex = 0
        while currentIndex < right :
            if nums[currentIndex] == 1 :
                currentIndex += 1
            elif nums[currentIndex] == 0 :
                nums[left + 1],nums[currentIndex] = nums[currentIndex],nums[left + 1]
                left += 1
                currentIndex += 1
            else:
                nums[right - 1],nums[currentIndex] = nums[currentIndex],nums[right - 1]
                right -= 1
               
```



