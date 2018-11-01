---
layout:     post
title:      LeetCode79_Word Search
subtitle:   
date:       2018-11-01
author:     Richbabe
header-img: img/post-bg-universe.jpg
catalog: true
tags:
    - LeetCode
---
# 题目
题目地址：[LeetCode 79:Word Search](https://leetcode.com/problems/word-search/description/)
![image](https://github.com/Richbabe/Richbabe.github.io/blob/master/img/LeetCode/Leetcode79.png?raw=true)

# 思路
看到这种搜索类题目，第一反应就是DFS和BFS，在这里我使用DFS。在遍历完矩阵一个元素时，我将其置为'\0'，以防下次再次遍历到，这样做可以避免用一个二维数组存储遍历状态，减小空间复杂度。

# 代码
C++版本如下：

```
class Solution {
public:
    bool exist(vector<vector<char>>& board, string word) {
        if(word.size() <= 0){
            return true;
        }
        if(board.size() <= 0 || board[0].size() <= 0){
            return false;
        }
        for(int i = 0;i < board.size();i++){
            for(int j = 0;j < board[0].size();j++){
                if(dfs(board,word,0,i,j)){
                    return true;
                }
            }
        }
        return false;
    }
    bool dfs(vector<vector<char> >& board,string word,int index,int x,int y){
        if(index == word.size()){
            return true;
        }
        if(x < 0 || x >= board.size() || y < 0 || y >= board[0].size()){
            return false;
        }
        if(board[x][y] != word[index] || board[x][y] == '\0'){
            return false;
        }
        char temp = board[x][y];
        board[x][y] = '\0';
        bool res = dfs(board,word,index + 1,x - 1,y) ||
                   dfs(board,word,index + 1,x + 1,y) ||
                   dfs(board,word,index + 1,x,y - 1) ||
                   dfs(board,word,index + 1,x,y + 1);
        board[x][y] = temp;
        return res;
    }
};
```

Python版本如下：

```
class Solution(object):
    def dfs(self, board, word, index, x, y):
        if index == len(word) : 
            return True
        if x < 0 or x >= len(board) or y < 0 or y >= len(board[0]) : 
            return False
        if board[x][y] != word[index] or board[x][y] == '\0' :
            return False
        temp = board[x][y]
        board[x][y] = '\0'
        res = self.dfs(board,word,index + 1,x - 1,y) or self.dfs(board,word,index + 1,x + 1,y) or self.dfs(board,word,index + 1,x,y - 1) or self.dfs(board,word,index + 1,x,y + 1)
        board[x][y] = temp
        return res
    
    def exist(self, board, word):
        """
        :type board: List[List[str]]
        :type word: str
        :rtype: bool
        """
        if len(word) <= 0 :
            return true
        for i in range(len(board)) :
            for j in range(len(board[0])) :
                if(self.dfs(board,word,0,i,j)) :
                    return True
        return False
    
        
```
