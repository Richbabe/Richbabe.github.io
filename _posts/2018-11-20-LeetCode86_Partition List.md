---
layout:     post
title:      LeetCode86_Partition List
subtitle:   
date:       2018-11-20
author:     Richbabe
header-img: img/post-bg-universe.jpg
catalog: true
tags:
    - LeetCode
---
# 题目
题目地址：[LeetCode 86:Partition List](https://leetcode.com/problems/partition-list/)
![image](https://github.com/Richbabe/Richbabe.github.io/blob/master/img/LeetCode/LeetCode86.png?raw=true)

# 思路
我们可以将结果链表分成两部分，第一部分是比x小的数组成，第二部分是>=x的数组成。那么我们现在需要做的就是遍历原链表，将元素连接到对应的部分中。

链表的题目可以分四步走：
1. 连过来。把想要的节点链接到你需要的链表上。
2. 指针走。该节点处理完了，在原来的链表上要走一步。
3. 断后路。当前链表不要和原来链表有连接。
4. 调状态。将当前链表的指针往后走一步。

所以，我们可以将第一部分的链表头声明为preDummy,第二部分的链表头声明为dummy，接着按照上面提到的四步遍历原链表即可。

# 代码
C++版本如下：

```
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode(int x) : val(x), next(NULL) {}
 * };
 */
class Solution {
public:
    ListNode* partition(ListNode* head, int x) {
        if(head == nullptr || head -> next == nullptr){
            return head;
        }
        ListNode* preDummy = new ListNode(-1);
        ListNode* dummy = new ListNode(-1);
        preDummy -> next = dummy;
        ListNode* res = preDummy;
        ListNode* tail = dummy;
        ListNode* current = head;
        
        while(current != nullptr){
            ListNode* temp = current;
            //链接第一部分
            if(temp -> val < x){
                //连过来
                preDummy -> next = temp;
                //指针走
                current = current -> next;
                //断后路
                temp -> next = dummy;
                //调状态
                preDummy = preDummy -> next;
            }
            //链接第二部分
            else{
                //连过来
                tail -> next = temp;
                //指针走
                current = current -> next;
                //断后路
                temp -> next = nullptr;
                //调状态
                tail = tail -> next;
            }
        }
        
        //链接第一部分和第二部分
        preDummy -> next = dummy -> next;
        return res -> next;
    }
};
```

Python版本如下：

```
# Definition for singly-linked list.
# class ListNode(object):
#     def __init__(self, x):
#         self.val = x
#         self.next = None

class Solution(object):
    def partition(self, head, x):
        """
        :type head: ListNode
        :type x: int
        :rtype: ListNode
        """
        if head is None or head.next is None:
            return head
        preDummy = ListNode(-1)
        dummy = ListNode(-1)
        preDummy.next = dummy
        res = preDummy
        tail = dummy
        current = head
        
        while current is not None:
            temp = current
            # 链接第一部分
            if temp.val < x:
                # 连过来
                preDummy.next = temp
                # 指针走
                current = current.next
                # 断后路
                temp.next = preDummy
                # 调状态：
                preDummy = preDummy.next
            # 链接第二部分
            else:
                # 连过来
                tail.next = temp
                # 指针走
                current = current.next
                # 断后路
                temp.next = None
                # 调状态：
                tail = tail.next
        
        # 链接第一部分和第二部分
        preDummy.next = dummy.next
        return res.next
                
```
