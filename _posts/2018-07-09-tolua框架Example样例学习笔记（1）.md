---
layout:     post
title:      tolua框架Example样例学习笔记（1）
subtitle:   Unity用C#调用lua代码的方法
date:       2018-07-07
author:     Richbabe
header-img: img/u3d技术博客背景.jpg
catalog: true
tags:
    - Lua
    - tolua
    - Unity
---
# 前言
本系列博客我将通过tolua框架中的Example样例来简单入门tolua。框架下载地址[戳我](https://github.com/jarjin/LuaFramework_UGUI)

Example样例的位置在：
![image](https://github.com/Richbabe/Richbabe.github.io/blob/master/img/tolua/Example/toLuaExample%E7%9B%AE%E5%BD%95.png?raw=true)

运行第一个样例时报错如下：
![image](https://github.com/Richbabe/Richbabe.github.io/blob/master/img/tolua/Example/%E6%B2%A1%E6%9C%89Lua%E6%96%87%E4%BB%B6%E5%A4%B9%E6%8A%A5%E9%94%99.png?raw=true)
根据报错信息在Assets目录下新建一个Lua文件夹即可，该文件夹后面我们将用来保存我们游戏的lua逻辑代码。

# 01_HelloWorld
HelloWorld.cs:

```
using UnityEngine;
using LuaInterface;
using System;

public class HelloWorld : MonoBehaviour
{
    void Awake()
    {
        LuaState lua = new LuaState();//创建lua虚拟机
        lua.Start();//启动lua虚拟机
        //lua脚本
        string hello =
            @"                
                print('hello tolua#')                                  
            ";
        
        lua.DoString(hello, "HelloWorld.cs");//运行lua脚本
        lua.CheckTop();//判断lua虚拟机栈是否为空
        lua.Dispose();//析构掉lua虚拟机
        lua = null;
    }
}


```
运行效果图为：
![image](https://github.com/Richbabe/Richbabe.github.io/blob/master/img/tolua/Example/01.png?raw=true)

总结：
* 使用Tolua的相关类和方法都需要调用命名空间LuaInterface
* 调用lua脚本必须先创建一个lua虚拟机，创建步骤为：

```
LuaState lua = new LuaState();
```
* 在C#中运行一段lua脚本最简单的方法就是lua.DoString，该方法声明如下：

```
public object[] DoString(string chunk, string chunkName = "LuaState.DoString")
```
* 使用完lua虚拟机之后记得要销毁，具体操作如下：
    * 先进行lua虚拟机的判空，具体做法为lua.CheckTop 
    * 析构掉lua虚拟机，具体做法为：lua.Dispose

# 02_ScriptsFromFile
核心代码为：
```
lua = new LuaState();                
lua.Start();        
//如果移动了ToLua目录，自己手动修复吧，只是例子就不做配置了
string fullPath = Application.dataPath + "\\LuaFramework/ToLua/Examples/02_ScriptsFromFile";
lua.AddSearchPath(fullPath);
```
和

```
if (GUI.Button(new Rect(50, 50, 120, 45), "DoFile"))
        {
            strLog = "";
            lua.DoFile("ScriptsFromFile.lua");                        
        }
        else if (GUI.Button(new Rect(50, 150, 120, 45), "Require"))
        {
            strLog = "";            
            lua.Require("ScriptsFromFile");            
        }

```
点击DoFile和Require按钮的执行效果一样，都为：
![image](https://github.com/Richbabe/Richbabe.github.io/blob/master/img/tolua/Example/02.png?raw=true)

总结：
* 调用lua.Start方法完成lua虚拟机的一些基础初始化，里面的内容主要包括一些环境的配置和一些lua的基本库的加载，默认一般工程中创建该虚拟机时都要初始化。
* 重要方法lua.AddSearchPath ，通过此方法添加lua文件的路径，只有添加了文件路径之后，在该路径上的lua文件才可以被读取
* lua文件的2个加载方法，lua.DoFile ， lua.Require  ,参数为lua文件名，推荐使用Require（[关于Require可以看我这篇博客](http://richbabe.top/2018/07/05/Lua-%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0-%E4%BA%8C/)），因为Require 读取文件是会检查该文件是否被加载过，如果被加载过，则直接返回一个索引，否则则加载并返回一个索引，而Dofile则是每次调用都会重新加载使用，相对来说对性能等的消耗都会大一些，而且感觉不利于一些后面代码的书写

# 结语
通过上面的介绍，我们知道在C#中调用lua代码方法有两种，一种是用lua.Dostring直接执行一段保存在string类型里的lua脚本，另一种是通过读取lua文件来执行lua脚本（包括lua.Dofile和lua.Require,后者效率更高）。

