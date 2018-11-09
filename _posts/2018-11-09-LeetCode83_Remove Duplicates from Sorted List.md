---
layout:     post
title:      LeetCode83_Remove Duplicates from Sorted List
subtitle:   
date:       2018-11-09
author:     Richbabe
header-img: img/post-bg-universe.jpg
catalog: true
tags:
    - LeetCode
---
# 题目
题目地址：[LeetCode 83:Remove Duplicates from Sorted List](https://leetcode.com/problems/remove-duplicates-from-sorted-list/)
![image](https://github.com/Richbabe/Richbabe.github.io/blob/master/img/LeetCode/LeetCode83.png?raw=true)

# 思路
当遍历当前节点时，将其next节点往后移直到next节点的值与当前节点不同为止。然后令当前节点=next节点。重复此操作直到遍历完整个链表。

# 代码
C++版本代码如下：

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
    ListNode* deleteDuplicates(ListNode* head) {
        ListNode* p = head;
        while(p){
            if(p -> next == nullptr){
                break;
            }
            if(p -> next && p -> val == p -> next -> val){
                p -> next = p -> next -> next;
            }
            else{
                p = p -> next;
            }
        }
        return head;
    }
};
```

Python版本代码如下：

```
# Definition for singly-linked list.
# class ListNode(object):
#     def __init__(self, x):
#         self.val = x
#         self.next = None

class Solution(object):
    def deleteDuplicates(self, head):
        """
        :type head: ListNode
        :rtype: ListNode
        """
        p = head
        while p is not None:
            if p.next is None:
                break
            if p.next is not None and p.val == p.next.val:
                p.next = p.next.next
            else:
                p = p.next
        return head
```

# 扩展
题目地址：[LeetCode 82:Remove Duplicates from Sorted List II](https://leetcode.com/problems/remove-duplicates-from-sorted-list-ii/)
![image](https://github.com/Richbabe/Richbabe.github.io/blob/master/img/LeetCode/LeetCode82.png?raw=true)

# 思路
这道题和上一道题的区别在于：他要把重复的元素一个不留的删掉。那么我们可以先声明一个pre节点用于保存当前节点的上一个节点。然后遍历当前节点cur，使其不断后移直到cur->val != cur->next->val。此时判断pre的下一个节点是否cur，如果是说明没有重复节点出现，令pre = pre.next；如果不是说明pre和cur中间有重复节点。令pre->next = cur->next。

# 代码
C++版本代码如下：

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
    ListNode* deleteDuplicates(ListNode* head) {
        if(head == nullptr){
            return nullptr;
        }
        ListNode preHead(0);
        preHead.next = head;
        ListNode* pre = &preHead;
        ListNode* cur = head;
        while(cur){
            while(cur -> next && cur -> val == cur -> next -> val){
                cur = cur -> next;
            }
            if(pre -> next == cur){
                pre = pre -> next;
            }
            else{
                pre -> next = cur -> next;
            }
            cur = cur -> next;
        }
        return preHead.next;
    }
};
```

Python版本代码如下：

```
# Definition for singly-linked list.
# class ListNode(object):
#     def __init__(self, x):
#         self.val = x
#         self.next = None

class Solution(object):
    def deleteDuplicates(self, head):
        """
        :type head: ListNode
        :rtype: ListNode
        """
        if head is None:
            return None
        preHead = ListNode(0)
        preHead.next = head
        cur = head
        pre = preHead
        while cur is not None:
            while cur.next is not None and cur.val == cur.next.val:
                cur = cur.next
            if pre.next == cur:
                pre = pre.next
            else:
                pre.next = cur.next
            cur = cur.next
        return preHead.next
        
```
