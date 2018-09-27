---
layout:     post
title:      LeetCode71_Simplify Path
subtitle:   
date:       2018-09-27
author:     Richbabe
header-img: img/post-bg-universe.jpg
catalog: true
tags:
    - LeetCode
---
# 题目
题目地址：[LeetCode 71:Simplify Path](https://leetcode.com/problems/simplify-path/description/)
![image](https://github.com/Richbabe/Richbabe.github.io/blob/master/img/LeetCode/LeetCode71.png?raw=true)

# 思路
这道题我们可以用C++的库函数getline来将'/'（或者'//'）之间的字符串分隔开。
> getline()的原型是istream& getline ( istream &is , string &str , char delim );
其中 istream &is 表示一个输入流，譬如cin；
string&str表示把从输入流读入的字符串存放在这个字符串中（可以自己随便命名，str什么的都可以）；
char delim表示遇到这个字符停止读入，在不设置的情况下系统默认该字符为'\n'，也就是回车换行符（遇到回车停止读入）。

接着我们声明一个存储文件夹名的vector<string> v

然后我们判断'/'或者'//'分割开的字符串：
* 该字符串是'.'，说明为当前文件夹，跳过
* 该字符串为'..'，说明要跳到上个文件夹，即把v的最后一个元素（当前文件夹名）pop掉
* 该字符串为文件夹名，则把该文件夹名push_back到v中

最后只要遍历v便可得到当前路径的所有文件夹名。其实很容易发现，v其实是一个栈，当遇到..时弹出栈顶，当遇到文件夹名时入栈，该方法时间复杂度为O(n)

代码如下：

```
class Solution {
public:
    string simplifyPath(string path) {
        istringstream ss(path);
        vector<string> v;
        string p;
        while(getline(ss,p,'/')){
            if(p.empty() || p == "."){
                continue;
            }
            if(p == ".."){
                if(v.size() > 0){
                    v.pop_back();
                }
            }
            else{
                v.push_back(p);
            }
        }
        if(v.empty()){
            return "/";
        }
        string res = "";
        for(int i = 0;i < v.size();i++){
            res = res + "/" + v[i];
        }
        return res;
    }
};
```
