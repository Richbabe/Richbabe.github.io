---
layout:     post
title:      LeetCode76_Minimum Window Substring
subtitle:   
date:       2018-10-18
author:     Richbabe
header-img: img/post-bg-universe.jpg
catalog: true
tags:
    - LeetCode
---
# 题目
题目地址：[LeetCode 76:Minimum Window Substring](https://leetcode.com/problems/minimum-window-substring/description/)
![image](https://github.com/Richbabe/Richbabe.github.io/blob/master/img/LeetCode/LeetCode76.png?raw=true)

# 思路
我们可以使用双指针实现滑动窗口来在O（n）的时间复杂度解决这道题。

具体做法如下：
1. 先扫描一边T，把对应的字符及其出现的次数存到哈希表中
2. 用一个count来记录当前T中还有多少个字母未匹配，用start和end分别表示滑动窗口的左边界和右边界
3. 开始遍历S，先向右移动窗口的右边界end，把遍历到的字母对应的哈希表中的value-1，如果减了1后value>=0说明该字母已经匹配完，count--
4. 当count == 0时说明T中所有字符全部匹配完，此时滑动左窗口，寻找以当前end为右边界的最小窗口子串。我们用head和length来表示最小窗口子串的头部和长度，每次更新这两个值。在滑动左边界start时，每遍历到一个字母便把该字母在哈希表中的value+1（这是因为我们在加入字母到窗口时对哈希表-1，所以在从窗口中移除字母时需要对哈希表+1）。如果+1后value>0说明该字母在T中出现过，则未匹配字母数+1即count+1
5. 重复这个过程直到end遍历到S的最后一个字母且count=0

有人可能会问到：对于不在T串中的字母的映射值也这么加呀减呀的，真的没问题吗？其实对于不在T串中的字母，我们减了1之后value变成-1，此时count不会增加，同理+1后valu为0，count也不会减少。而只有count的变化才会影响我们的最终结果，所以并没有什么关系。

# 代码
C++版本如下：

```
class Solution {
public:
    string minWindow(string s, string t) {
        unordered_map<char,int> m;
        for(int i = 0;i < t.size();i++){
            m[t[i]]++;
        }
        int start = 0,end = 0;//窗口左边界和右边界
        int length = INT_MAX;//最小子串长度
        int head = 0;//最小子串起点
        int count = t.size();//记录t中还有多少个字母未匹配
        for(int end = 0;end < s.size();end++){
            //滑动右边界直到t中字母全部匹配完
            if(--m[s[end]] >= 0){
                count--;
            }
            //滑动左边界找以当前end为右边界的最小子串
            while(count == 0){
                //更新最小子串长度和最小子串的起点
                if(end - start + 1 < length){
                    length = end - start + 1;
                    head = start;
                }
                //如果滑过的字母在t中出现过，则未匹配字母数+1
                if(++m[s[start]] > 0){
                    count++;
                }
                start++;
            }
        }
        return length == INT_MAX ? "" : s.substr(head,length);
    }
};
```

Python版本如下：

```
class Solution(object):
    def minWindow(self, s, t):
        """
        :type s: str
        :type t: str
        :rtype: str
        """
        m = collections.Counter(t)
        count = len(t)
        start = end = head = 0
        length = sys.maxsize
        while end < len(s) :
            if m[s[end]] > 0 :
                count -= 1
            m[s[end]] -= 1
            end += 1
            while count == 0 :
                if end - start < length :
                    length = end - start
                    head = start
                if m[s[start]] == 0 :
                    count += 1
                m[s[start]] += 1
                start += 1
        if length == sys.maxsize :
            return ""
        else :
            return s[head:head + length]
        
```
