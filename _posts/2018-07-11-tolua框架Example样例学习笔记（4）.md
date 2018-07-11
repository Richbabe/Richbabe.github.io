---
layout:     post
title:      tolua框架Example样例学习笔记（4）
subtitle:   lua中的多线程与数组的获取
date:       2018-07-11
author:     Richbabe
header-img: img/u3d技术博客背景.jpg
catalog: true
tags:
    - Lua
    - tolua
    - Unity
---
# 07_LuaThread
在Lua中不存在传统意义上的多线程，所谓的多线程都是基于协程实现的，所以Lua中的线程也都是那种协作式的多线程，而无法实现那种抢占式的多线程的效果，这也就导致有些效果我们无法得到实现。

如果真的要实现抢占式的效果，可以使用tolua作者封装过的协程。

在这个样例中，我们主要学习如何获得原生的lua线程，并在C#中进行lua线程的挂起、激活等操作。

样例代码如下：

```
using UnityEngine;
using System.Collections;
using LuaInterface;

public class TestLuaThread : MonoBehaviour 
{
    string script =
        @"
            function fib(n)
                local a, b = 0, 1
                while n > 0 do
                    a, b = b, a + b
                    n = n - 1
                end

                return a
            end

            function CoFunc(len)
                print('Coroutine started')                
                local i = 0
                for i = 0, len, 1 do                    
                    local flag = coroutine.yield(fib(i))	                    
                    if not flag then
                        break
                    end                                      
                end
                print('Coroutine ended')
            end

            function Test()                
                local co = coroutine.create(CoFunc)                                
                return co
            end            
        ";

    LuaState state = null;
    LuaThread thread = null;//Lua线程类型
    string tips = null;

    void Start () 
    {
#if UNITY_5 || UNITY_2017 || UNITY_2018
        Application.logMessageReceived += ShowTips;
#else
        Application.RegisterLogCallback(ShowTips);
#endif
        //初始化Lua虚拟机
        new LuaResLoader();
        state = new LuaState();
        state.Start();
        state.LogGC = true;
        state.DoString(script);

        LuaFunction func = state.GetFunction("Test");//从lua虚拟机中获得lua函数对象
        func.BeginPCall();
        func.PCall();
        thread = func.CheckLuaThread();//获取函数func对应的线程对象
        thread.name = "LuaThread";
        func.EndPCall();
        func.Dispose();
        func = null;

        thread.Resume(10);//运行线程并同时传递参数10
	}

    void OnApplicationQuit()
    {
        if (thread != null)
        {
            thread.Dispose();
            thread = null;
        }

        state.Dispose();
        state = null;
#if UNITY_5 || UNITY_2017 || UNITY_2018
        Application.logMessageReceived -= ShowTips;
#else
        Application.RegisterLogCallback(null);
#endif
    }

    void ShowTips(string msg, string stackTrace, LogType type)
    {
        tips += msg;
        tips += "\r\n";
    }

    void Update()
    {
        state.CheckTop();
        state.Collect();
    }

    void OnGUI()
    {
        GUI.Label(new Rect(Screen.width / 2 - 300, Screen.height / 2 - 200, 600, 400), tips);

        if (GUI.Button(new Rect(10, 50, 120, 40), "Resume Thead"))
        {
            int ret = -1;

            if (thread != null && thread.Resume(true, out ret) == (int)LuaThreadStatus.LUA_YIELD)
            {                
                Debugger.Log("lua yield: " + ret);
            }
        }
        else if (GUI.Button(new Rect(10, 150, 120, 40), "Close Thread"))
        {
            if (thread != null)
            {                
                thread.Dispose();                
                thread = null;
            }
        }
    }
}

```
运行效果为：
![image](https://github.com/Richbabe/Richbabe.github.io/blob/master/img/tolua/Example/07.png?raw=true)

总结：
* C#中声明lua线程的方法为：

```
LuaThread thread;
```
获得lua线程的方法为：

```
thread = func.CheckLuaThread();
```
获得的线程是lua函数func对应的线程。

* 我们每点击一次Resume Thead按钮，就会输出斐波那契数列的一个数，这是因为我们在点击按钮之后通过thread.Resume激活线程（等同于lua中的coroutine.resume()），而在输出数字之后，在lua代码中又通过coroutine.yield挂起线程。当输出10个斐波那契数后，该线程结束。当然我们也可以点击Close Thread来提前结束lua中的线程，其通过在C#中调用thread.Dispose()实现。
* 在对lua线程操作时，需要在C#的Update中进行垃圾回收：

```
state.CheckTop();
state.Collect();
```

# 08_AccessingArray
该样例主要介绍了在lua中访问C#中的数组对象的几种方法。

样例代码为：

```
using UnityEngine;
using LuaInterface;

public class AccessingArray : MonoBehaviour 
{
    private string script =
        @"
            function TestArray(array)
                local len = array.Length
                
                -- 通过下标索引访问
                for i = 0, len - 1 do
                    print('Array: '..tostring(array[i]))
                end
                
                -- 通过迭代器访问
                local iter = array:GetEnumerator()

                while iter:MoveNext() do
                    print('iter: '..iter.Current)
                end

                -- 通过转换为luatable访问
                local t = array:ToTable()                
                
                for i = 1, #t do
                    print('table: '.. tostring(t[i]))
                end

                local pos = array:BinarySearch(3) --通过二分搜索找到值为3的元素在数组中的位置
                print('array BinarySearch: pos: '..pos..' value: '..array[pos])

                pos = array:IndexOf(4)  --找到值为4的元素在数组中的索引（位置）
                print('array indexof bbb pos is: '..pos)
                
                return 1, '123', true
            end            
        ";

    LuaState lua = null;
    LuaFunction func = null;
    string tips = null;

    void Start()
    {
#if UNITY_5 || UNITY_2017 || UNITY_2018
        Application.logMessageReceived += ShowTips;
#else
        Application.RegisterLogCallback(ShowTips);
#endif
        new LuaResLoader();
        lua = new LuaState();
        lua.Start();
        lua.DoString(script, "AccessingArray.cs");
        tips = "";

        int[] array = { 1, 2, 3, 4, 5};        
        func = lua.GetFunction("TestArray");

        func.BeginPCall();
        func.Push(array);//把C#中的数组作为lua函数的参数传入lua
        func.PCall();
        double arg1 = func.CheckNumber();//获得func函数double类型的返回值
        string arg2 = func.CheckString();//获得func函数string类型的返回值
        bool arg3 = func.CheckBoolean();//获得func函数bool类型的返回值
        Debugger.Log("return is {0} {1} {2}", arg1, arg2, arg3);
        func.EndPCall();

        //调用通用函数需要转换一下类型，避免可变参数拆成多个参数传递
        object[] objs = func.LazyCall((object)array);

        if (objs != null)
        {
            Debugger.Log("return is {0} {1} {2}", objs[0], objs[1], objs[2]);
        }

        lua.CheckTop();                
    }

    void ShowTips(string msg, string stackTrace, LogType type)
    {
        tips += msg;
        tips += "\r\n";
    }

    void OnGUI()
    {
        GUI.Label(new Rect(Screen.width / 2 - 300, Screen.height / 2 - 300, 600, 600), tips);
    }

    void OnApplicationQuit()
    {
#if UNITY_5 || UNITY_2017 || UNITY_2018
        Application.logMessageReceived -= ShowTips;
#else
        Application.RegisterLogCallback(null);
#endif
        func.Dispose();
        lua.Dispose();
    }
}

```
运行结果为：
![image](https://github.com/Richbabe/Richbabe.github.io/blob/master/img/tolua/Example/08.png?raw=true)

总结：
* 在lua中想要访问C#中的数组，方法有以下几种：
    * 直接通过下标索引访问，写法和C#中完全一致，例如： array[i] ， 同时还可以通过直接调用 array.Length 获取到数组的大小
    * 通过迭代器进行访问，具体方法为 array:GetEnumerator()获取数组的迭代器，通过iter.Current获得当前的迭代器所指向的元素值，通过iter:MoveNext()将迭代器所指位置移至下一个。
    * 对于C#中的数组，我们可以调用 array:ToTable() 实现将数组对象转化为对应的lua中的Table表的形式进行访问。需要注意的是在C#中数组下标从0开始，而在luatable中下标是从1开始！
    * 通过array:BinarySearch(value)方法获得值为value的下标位置
    * 通过array:IndexOf(value)方法获得值为value的索引（下标位置）
* 要在C#中获得lua函数的返回值，可以通过func.Checkxxx()来获得，比如：

```
double arg1 = func.CheckNumber();//获得func函数double类型的返回值
string arg2 = func.CheckString();//获得func函数string类型的返回值
bool arg3 = func.CheckBoolean();//获得func函数bool类型的返回值
Debugger.Log("return is {0} {1} {2}", arg1, arg2, arg3);
```
需要注意的是，tolua作者还加了这段：

```
//调用通用函数需要转换一下类型，避免可变参数拆成多个参数传递
        object[] objs = func.LazyCall((object)array);

        if (objs != null)
        {
            Debugger.Log("return is {0} {1} {2}", objs[0], objs[1], objs[2]);
        }
```
我猜是因为C#的装箱拆箱机制的问题，需要将其他类型转换成object类型来实现变量在通用函数中正常调用？






