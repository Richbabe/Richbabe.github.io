---
layout:     post
title:      LeetCode542_01Matrix
subtitle:   
date:       2018-12-04
author:     Richbabe
header-img: img/post-bg-universe.jpg
catalog: true
tags:
    - LeetCode
---
# 题目
题目地址：[LeetCode 542:01Matrix List](https://leetcode.com/problems/01-matrix/)
![image](https://github.com/Richbabe/Richbabe.github.io/blob/master/img/LeetCode/LeetCode542_1.png?raw=true)
![image](https://github.com/Richbabe/Richbabe.github.io/blob/master/img/LeetCode/LeetCode542_2.png?raw=true)

# 思路
要求这种矩阵元素之间的最短距离，第一反应就是用bfs遍历，而bfs都是通过队列实现的。一开始我们先把matrix[i][j]为0的坐标值作为起点存入队列q中，把非0的值替换成INT_MAX（方便后面的最短距离值比较更新）。然后我们在队列中取队头，遍历其上下左右四个点，如果遍历的点的最短距离值小于等于当前点的最短距离值，则跳过。否则令其等于当前最短距离值+1。重复这个过程直至队列为空。

# 代码
C++版本如下：

```
class Solution {
public:
    vector<vector<int>> updateMatrix(vector<vector<int>>& matrix) {
        int row = matrix.size();
        int col = matrix[0].size();
        queue<pair<int,int> > q;//遍历点队列
        vector<vector<int> > dirs = { {0,-1},{-1,0},{1,0},{0,1} };//保存四个方向
        for(int i = 0;i < row;i++){
            for(int j = 0;j < col;j++){
                if(matrix[i][j] == 0){
                    q.push(make_pair(i,j));//作为起点
                }
                else{
                    matrix[i][j] = INT_MAX;
                }
            }
        }
        //BFS
        while(!q.empty()){
            auto t = q.front();
            q.pop();
            for(auto d : dirs){
                int x = t.first + d[0];
                int y = t.second + d[1];
                if(x < 0 || x >= row || y < 0 || y >= col || 
                    matrix[x][y] <= matrix[t.first][t.second]){
                    continue;
                }
                matrix[x][y] = matrix[t.first][t.second] + 1;
                q.push(make_pair(x,y));
            }
        }
    return matrix;
    }
};
```

Python版本如下：

```
class Solution(object):
    def updateMatrix(self, matrix):
        """
        :type matrix: List[List[int]]
        :rtype: List[List[int]]
        """
        row = len(matrix)
        col = len(matrix[0])
        q = []
        for i in xrange(row):
            for j in xrange(col):
                if matrix[i][j] != 0:
                    matrix[i][j] = 0x7fffffff
                else:
                    q.append((i, j))
        for i,j in q:
            for r,c in((i,j+1),(i,j-1),(i+1,j),(i-1,j)):
                if 0 <= r < row and 0 <= c < col and matrix[r][c] > matrix[i][j] + 1:
                    matrix[r][c] = matrix[i][j] + 1
                    q.append((r,c))
        return matrix
        
```


