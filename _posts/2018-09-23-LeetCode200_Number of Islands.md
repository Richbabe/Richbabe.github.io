---
layout:     post
title:      LeetCode200_Number of Islands
subtitle:   
date:       2018-09-11
author:     Richbabe
header-img: img/post-bg-universe.jpg
catalog: true
tags:
    - LeetCode
---
# 题目
题目地址：[LeetCode 200:Number of Islands](https://leetcode.com/problems/number-of-islands/description/)
![image](https://github.com/Richbabe/Richbabe.github.io/blob/master/img/LeetCode/LeetCode200.png?raw=true)

# 思路
## 方法一：DFS
将遍历过的置为0，每次往上下左右四个方向进行dfs遍历。

代码如下：

```
class Solution {
public:
    int numIslands(vector<vector<char>>& grid) {
        int row = grid.size();
        if(row <= 0){
            return 0;
        }
        int col = grid[0].size();
        int res = 0;
        for(int i = 0;i < row;i++){
            for(int j = 0;j < col;j++){
                if(grid[i][j] == '1'){
                    dfs(grid,i,j);
                    res++;
                }
            }
        }
        return res;
    }
    void dfs(vector<vector<char>> &grid,int i,int j){
        if(i < 0 || i >= grid.size() || j < 0 || j >= grid[0].size() || grid[i][j] == '0'){
            return;
        }
        if(grid[i][j] == '1'){
            grid[i][j] = '0';
        }
        dfs(grid,i - 1,j);
        dfs(grid,i,j - 1);
        dfs(grid,i + 1,j);
        dfs(grid,i,j + 1);
    }
};
```

## 方法二：BFS
用一个队列实现BFS，代码如下：

```
class Solution {
public:
    struct Point {
        int r, c;
        Point(int _r, int _c): r(_r), c(_c) {} 
    };

    void bfs(int row, int col, vector<vector<char>>& grid, int rd, int cd) {
        queue<Point> q;
        q.push(Point(row, col));
        while (!q.empty()) {
            auto p = q.front();
            q.pop();
            if (p.r >= 0 && p.r < rd && p.c >= 0 && p.c < cd && grid[p.r][p.c] == '1') {
                grid[p.r][p.c] = '0';
                q.push(Point(p.r + 1, p.c));
                q.push(Point(p.r - 1, p.c));
                q.push(Point(p.r, p.c + 1));
                q.push(Point(p.r, p.c - 1));
            }
        }
    }

    int numIslands(vector<vector<char>>& grid) {
        int rd = grid.size();
        if (rd == 0)
            return 0;
        
        int cd = grid[0].size();
        if (cd == 0)
            return 0;

        int row, col;
        int res = 0;
        for (row = 0; row < rd; row++) {
            for (col = 0; col < cd; col++) {
                if (grid[row][col] == '1') {
                    bfs(row, col, grid, rd, cd);
                    res++;
                }
            }
        }
        return res;
    }
};
```

# 总结
DFS和BFS两种方法的时间复杂度都为O(row * col)，但是BFS的空间复杂度更高一点