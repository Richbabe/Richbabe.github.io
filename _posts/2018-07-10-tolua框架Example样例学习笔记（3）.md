---
layout:     post
title:      tolua框架Example样例学习笔记（3）
subtitle:   tolua协程的用法
date:       2018-07-10
author:     Richbabe
header-img: img/u3d技术博客背景.jpg
catalog: true
tags:
    - Lua
    - tolua
    - Unity
---
# 05_LuaCoroutine
由于Lua的协同程序功能相对有限，所以ToLua帮我们对Lua的协同程序进行了扩展，具体文件是在：

ToLua/Lua/System/coroutine.lua

它对Lua原生的coroutine进行了扩展。

下面我们来看看样例的代码：

lua部分：

```

function fib(n)
    local a, b = 0, 1
    while n > 0 do
        a, b = b, a + b
        n = n - 1
    end

    return a
end

function CoFunc()
    print('Coroutine started')    
    for i = 0, 10, 1 do
        print(fib(i))                    
        coroutine.wait(0.1)						
    end	
	print("current frameCount: "..Time.frameCount)
	coroutine.step()
	print("yield frameCount: "..Time.frameCount)

	local www = UnityEngine.WWW("http://www.baidu.com")
	coroutine.www(www)
	local s = tolua.tolstring(www.bytes)
	print(s:sub(1, 128))
    print('Coroutine ended')
end

function TestCortinue()	
    coroutine.start(CoFunc)
end

local coDelay = nil

function Delay()
	local c = 1

	while true do
		coroutine.wait(1) 
		print("Count: "..c)
		c = c + 1
	end
end

function StartDelay()
	coDelay = coroutine.start(Delay)
end

function StopDelay()
	coroutine.stop(coDelay)
end

```
c#部分：

```
using UnityEngine;
using System;
using System.Collections;
using LuaInterface;

//例子5和6展示的两套协同系统勿交叉使用，此为推荐方案
public class TestCoroutine : MonoBehaviour 
{
    public TextAsset luaFile = null;
    private LuaState lua = null;
    private LuaLooper looper = null;

	void Awake () 
    {
#if UNITY_5 || UNITY_2017 || UNITY_2018
        Application.logMessageReceived += ShowTips;
#else
        Application.RegisterLogCallback(ShowTips);
#endif        
        new LuaResLoader();
        lua  = new LuaState();
        lua.Start();
        LuaBinder.Bind(lua);
        DelegateFactory.Init();         
        looper = gameObject.AddComponent<LuaLooper>();
        looper.luaState = lua;

        lua.DoString(luaFile.text, "TestLuaCoroutine.lua");
        LuaFunction f = lua.GetFunction("TestCortinue");
        f.Call();
        f.Dispose();
        f = null;        
    }

    void OnApplicationQuit()
    {
        looper.Destroy();
        lua.Dispose();
        lua = null;
#if UNITY_5 || UNITY_2017 || UNITY_2018
        Application.logMessageReceived -= ShowTips;
#else
        Application.RegisterLogCallback(null);
#endif
    }

    string tips = null;

    void ShowTips(string msg, string stackTrace, LogType type)
    {
        tips += msg;
        tips += "\r\n";
    }

    void OnGUI()
    {
        GUI.Label(new Rect(Screen.width / 2 - 300, Screen.height / 2 - 200, 600, 400), tips);

        if (GUI.Button(new Rect(50, 50, 120, 45), "Start Counter"))
        {
            tips = null;
            LuaFunction func = lua.GetFunction("StartDelay");
            func.Call();
            func.Dispose();
        }
        else if (GUI.Button(new Rect(50, 150, 120, 45), "Stop Counter"))
        {
            LuaFunction func = lua.GetFunction("StopDelay");
            func.Call();
            func.Dispose();
        }
        else if (GUI.Button(new Rect(50, 250, 120, 45), "GC"))
        {
            lua.DoString("collectgarbage('collect')", "TestCoroutine.cs");
            Resources.UnloadUnusedAssets();
        }
    }
}


```
运行效果为：
![image](https://github.com/Richbabe/Richbabe.github.io/blob/master/img/tolua/Example/05.png?raw=true)
且在点击StartCounter之后会进行计数（通过协程调用lua脚本中的Delay方法），点击StopCounter后停止计数（停止调用Delay方法的协程）。

总结：
* 要在Lua中使用C#中的方法或者变量，需要将C#中对应的类在ToLua中的CustomSetting.cs文件中注册，如果是调用C#中的变量（或者静态方法），则用C#类名.变量名（静态方法名）获得；如果是调用C#中的非静态方法（函数），则用C#类名:函数名来进行调用。这样区别是因为Lua中没有类的概念，只能通过table来获取具体的对象。在样例的lua脚本中，我们使用到：Time.frameCount这个变量，该变量是Unity.Time类中的变量，表示帧数，所以我们需要在CustomSetting.cs注册Time这个类：

```
_GT(typeof(Time))
```
* tolua协程的准备工作
    
    在创建完lua虚拟机之后，记得做以下几步：

```

lua.Start();
LuaBinder.Bind(lua);                
looper = gameObject.AddComponent<LuaLooper>();
looper.luaState = lua;
```
首先调用虚拟机的lua.Start函数进行初始化，然后调用LuaBinder的静态方法LuaBinder.Bind(lua)；参数就是你创建的虚拟机，接着为你的一个游戏对象添加组件LuaLooper，并将该LuaLooper的内部虚拟机引用指定为我们创建的虚拟机，然后我们就可以正常的使用Lua中的协程了，它会在C#每一帧驱动lua的协程完成所有的协程功能，这里的协程已经不再是lua自身功能，而是用tolua#模拟Unity的所有协程功能。

* 在[我的Lua学习博客](http://richbabe.top/2018/07/05/Lua-%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0-%E4%BA%8C/)里已经介绍了lua自身的协程使用方法，而tolua在这里又重写并扩展了lua的协程功能。tolua中协程的使用如下：
    * 协程函数的开启： coroutine.start(协程函数)
    * 协程函数的挂起：   coroutine.step()
    * 协程函数的延时：   coroutine.wait(延时时间)    注意：时间的单位是秒   
    * 协程函数的结束：   coroutine.stop(协程对象)   注意：协程函数关闭的协程对象是对应的协程开启函数的返回值（返回对象是一个协程）
    * 协程下载：               coroutine.www(网址)

需要注意的是，除了coroutine.start(协程函数) 和  coroutine.stop(协程对象)  之外，其他的协程方法只允许在协程函数的内部使用！！

# 06_LuaCoroutine2
这个样例介绍了tolua的第二套协程使用方法，而样例5是tolua的第一套协程使用方法。作者说明这两套协程方案不要交叉使用，而且样例6这套使用效率较低，推荐使用样例5的方法。

在运行代码之前，因为我们之前创建的Lua文件夹（保存lua逻辑脚本）是放在LuaFramework文件夹下，而tolua默认框架是放在asset下，因此要在LuaConst.cs修改Lua文件夹的绝对路径：

```
public static string luaDir = Application.dataPath + "/LuaFramework/Lua";
```
如果你路径没错直接无视

接下来看看样例脚本：

```
using UnityEngine;
using System.Collections;
using LuaInterface;

//两套协同勿交叉使用，类unity原生，大量使用效率低
public class TestCoroutine2 : LuaClient 
{
    string script =
    @"
        function CoExample()            
            WaitForSeconds(1)
            print('WaitForSeconds end time: '.. UnityEngine.Time.time)            
            WaitForFixedUpdate()
            print('WaitForFixedUpdate end frameCount: '..UnityEngine.Time.frameCount)
            WaitForEndOfFrame()
            print('WaitForEndOfFrame end frameCount: '..UnityEngine.Time.frameCount)
            Yield(null)
            print('yield null end frameCount: '..UnityEngine.Time.frameCount)
            Yield(0)
            print('yield(0) end frameCime: '..UnityEngine.Time.frameCount)
            local www = UnityEngine.WWW('http://www.baidu.com')
            Yield(www)
            print('yield(www) end time: '.. UnityEngine.Time.time)
            local s = tolua.tolstring(www.bytes)
            print(s:sub(1, 128))
            print('coroutine over')
        end

        function TestCo()            
            StartCoroutine(CoExample)                                   
        end

        local coDelay = nil

        function Delay()
	        local c = 1

	        while true do
		        WaitForSeconds(1) 
		        print('Count: '..c)
		        c = c + 1
	        end
        end

        function StartDelay()
	        coDelay = StartCoroutine(Delay)            
        end

        function StopDelay()
	        StopCoroutine(coDelay)
            coDelay = nil
        end
    ";

    protected override LuaFileUtils InitLoader()
    {
        return new LuaResLoader();
    }

    protected override void OnLoadFinished()
    {
        base.OnLoadFinished();

        luaState.DoString(script, "TestCoroutine2.cs");
        LuaFunction func = luaState.GetFunction("TestCo");
        func.Call();
        func.Dispose();
        func = null;
    }

    //屏蔽，例子不需要运行
    protected override void CallMain() { }

    bool beStart = false;
    string tips = null;

    void Start()
    {
#if UNITY_5 || UNITY_2017 || UNITY_2018
        Application.logMessageReceived += ShowTips;
#else
        Application.RegisterLogCallback(ShowTips);
#endif
    }

    void ShowTips(string msg, string stackTrace, LogType type)
    {
        tips += msg;
        tips += "\r\n";
    }

    new void OnApplicationQuit()
    {
#if UNITY_5 || UNITY_2017 || UNITY_2018
        Application.logMessageReceived -= ShowTips;
#else
        Application.RegisterLogCallback(null);
#endif
        base.OnApplicationQuit();
    }

    void OnGUI()
    {
        GUI.Label(new Rect(Screen.width / 2 - 300, Screen.height / 2 - 200, 600, 400), tips);

        if (GUI.Button(new Rect(50, 50, 120, 45), "Start Counter"))
        {
            if (!beStart)
            {
                beStart = true;
                tips = "";
                LuaFunction func = luaState.GetFunction("StartDelay");
                func.Call();
                func.Dispose();
            }
        }
        else if (GUI.Button(new Rect(50, 150, 120, 45), "Stop Counter"))
        {
            if (beStart)
            {
                beStart = false;
                LuaFunction func = luaState.GetFunction("StopDelay");
                func.Call();
                func.Dispose();
            }
        }
    }
}

```
运行效果如下：
![image](https://github.com/Richbabe/Richbabe.github.io/blob/master/img/tolua/Example/06.png?raw=true)
两个按钮的效果和样例5一样。

总结：
* 这一套协程使用方法的好处在于：在lua代码中协程的使用方法和C#较为类似，而且在C#里面不用之前的准备工作（比如样例5中虚拟机的创建和初始化以及LuaBinder的绑定等待）
* 这一套协程使用方法的坏处在于：效率低下，而且在C#端还要将对应的对象继承类LuaClient，而LuaClient中就封装了方案1中那些准备操作，除此之外还要加载一些其他的库，估计这些就是写法改变的核心。




