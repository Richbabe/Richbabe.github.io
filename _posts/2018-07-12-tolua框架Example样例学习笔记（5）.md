---
layout:     post
title:      tolua框架Example样例学习笔记（5）
subtitle:   lua中遍历访问c#的Dictionary类型和枚举类型
date:       2018-07-12
author:     Richbabe
header-img: img/u3d技术博客背景.jpg
catalog: true
tags:
    - Lua
    - tolua
    - Unity
---
# 09_Dictionary
该样例主要演示了如何在lua中访问C#的Dictionary类型的变量，已经对他进行遍历，删除等操作。

需要注意的是，该样例中的BindMap（）只是为了让例子可以独立运行，在实际开发中只要将需要的类型添加在CustomSetting列表中即可。

代码如下：

```
using UnityEngine;
using System.Collections.Generic;
using LuaInterface;

public sealed class TestAccount
{
    public int id;
    public string name;
    public int sex;

    public TestAccount(int id, string name, int sex)
    {
        this.id = id;
        this.name = name;
        this.sex = sex;
    }
}

public class UseDictionary : MonoBehaviour 
{
    Dictionary<int, TestAccount> map = new Dictionary<int, TestAccount>();

    string script =
        @"              
            function TestDict(map)
                -- 获得Dictionary类型的迭代器                        
                local iter = map:GetEnumerator() 
                
                -- 输出Dictionary里面的值
                while iter:MoveNext() do
                    local v = iter.Current.Value
                    print('id: '..v.id ..' name: '..v.name..' sex: '..v.sex)                                
                end
                
                -- 获取键值为1的值，如果不存在则为nil
                local flag, account = map:TryGetValue(1, nil)

                if flag then
                    print('TryGetValue result ok: '..account.name)
                end

                -- 遍历Dictionary中的key
                local keys = map.Keys
                iter = keys:GetEnumerator() -- 获得Dictionary的key的迭代器
                print('------------print dictionary keys---------------')
                while iter:MoveNext() do
                    print(iter.Current)
                end
                print('----------------------over----------------------')

                -- 遍历Dictionary中的value
                local values = map.Values
                iter = values:GetEnumerator() -- 获得Dictionary的value的迭代器
                print('------------print dictionary values---------------')
                while iter:MoveNext() do
                    print(iter.Current.name)
                end
                print('----------------------over----------------------')                

                print('kick '..map[2].name)
                map:Remove(2) --移除键值为2的键值对
                iter = map:GetEnumerator() 

                while iter:MoveNext() do
                    local v = iter.Current.Value
                    print('id: '..v.id ..' name: '..v.name..' sex: '..v.sex)                                
                end
            end                        
        ";

	void Awake () 
    {
#if UNITY_5 || UNITY_2017 || UNITY_2018
        Application.logMessageReceived += ShowTips;
#else
        Application.RegisterLogCallback(ShowTips);
#endif
        new LuaResLoader();
        map.Add(1, new TestAccount(1, "水水", 0));
        map.Add(2, new TestAccount(2, "王伟", 1));
        map.Add(3, new TestAccount(3, "王芳", 0));

        LuaState luaState = new LuaState();
        luaState.Start();
        BindMap(luaState);

        luaState.DoString(script, "UseDictionary.cs");
        LuaFunction func = luaState.GetFunction("TestDict");
        func.BeginPCall();
        func.Push(map);
        func.PCall();
        func.EndPCall();

        func.Dispose();
        func = null;
        luaState.CheckTop();
        luaState.Dispose();
        luaState = null;
    }

    void OnApplicationQuit()
    {
#if UNITY_5 || UNITY_2017 || UNITY_2018
        Application.logMessageReceived -= ShowTips;
#else
        Application.RegisterLogCallback(null);
#endif        
    }

    string tips = "";

    void ShowTips(string msg, string stackTrace, LogType type)
    {
        tips += msg;
        tips += "\r\n";
    }

    void OnGUI()
    {
        GUI.Label(new Rect(Screen.width / 2 - 300, Screen.height / 2 - 200, 600, 400), tips);
    }

    //示例方式，方便删除，正常导出无需手写下面代码
    void BindMap(LuaState L)
    {
        L.BeginModule(null);
        TestAccountWrap.Register(L);
        L.BeginModule("System");
        L.BeginModule("Collections");
        L.BeginModule("Generic");
        System_Collections_Generic_Dictionary_int_TestAccountWrap.Register(L);
        System_Collections_Generic_KeyValuePair_int_TestAccountWrap.Register(L);
        L.BeginModule("Dictionary");
        System_Collections_Generic_Dictionary_int_TestAccount_KeyCollectionWrap.Register(L);
        System_Collections_Generic_Dictionary_int_TestAccount_ValueCollectionWrap.Register(L);
        L.EndModule();
        L.EndModule();
        L.EndModule();
        L.EndModule();
        L.EndModule();
    }
}

```
运行效果为：
![image](https://github.com/Richbabe/Richbabe.github.io/blob/master/img/tolua/Example/09.png?raw=true)

总结：
* 要在lua遍历C#中Dictionary的对象，我们可以通过GetEnumerator来获得Dictionary的迭代器，通过iter.Current.key访问当前迭代器指向的键值，通过iter.Current.value访问当前迭代器指向的值，通过iter.MoveNext()来将迭代器指向下一个元素。在lua中，iter:MoveNext()在访问越界时会返回一个nil。（C#中的NULL在lua中会被识别成nil，这样方便了lua中对于NULL的检测）。
* C#中的Dictionary的大多数方法在lua中可以正常使用：

```

Dictionary:get_Item()  --获取与指定的键相关联的值
Dictionary:set_Item()  --设置与指定的键相关联的值
Dictionary:Add()      --将指定的键和值添加到字典中
Dictionary:Clear()    --从 Dictionary中移除所有的键和值
Dictionary:ContainsKey()     --确定 Dictionary是否包含指定的键
Dictionary:ContainsValue()   --确定 Dictionary是否包含特定值
Dictionary:GetObjectData()
Dictionary:OnDeserialization()
Dictionary:Remove() --从 Dictionary中移除所指定的键的值
Dictionary:TryGetValue() --获取与指定的键相关联的值
Dictionary:GetEnumerator() --返回循环访问 Dictionary的枚举数(迭代器)
Dictionary:New() --这个是创建一个字典的方法
Dictionary.this --对象的this引用
Dictionary.__tostring --返回表示当前 Object的 String,这里其实并非C#对象的那个Tostring().而是作者自己重新等装的，但是效果类似，不用纠结，
                      --只不过不是方法而是对象了
Dictionary.Count     --获取包含在 Dictionary中的键/值对的数
Dictionary.Comparer  --获取用于确定字典中的键是否相
Dictionary.Keys     --获取包含 Dictionary中的键的集合
Dictionary.Values   --获取包含 Dictionary中的值的集合
```
需要注意的是，在lua中调用类的成员方法时用":"，调用类的成员对象时用"."，具体为啥可以参见[我这篇博客](http://richbabe.top/2018/07/06/Lua-%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0-%E4%B8%89/)

# 10_Enum
这个样例主要介绍了如何在lua里访问C#中的枚举类型，样例所访问的枚举类型是UnityEngine中自带的Space类型：

```
namespace UnityEngine
{
    //
    // 摘要:
    //     ///
    //     The coordinate space in which to operate.
    //     ///
    public enum Space
    {
        //
        // 摘要:
        //     ///
        //     Applies transformation relative to the world coordinate system.
        //     ///
        World = 0,
        //
        // 摘要:
        //     ///
        //     Applies transformation relative to the local coordinate system.
        //     ///
        Self = 1
    }
}
```
样例代码如下：

```
using UnityEngine;
using System;
using LuaInterface;

public class AccessingEnum : MonoBehaviour 
{
    string script =
        @"
            space = nil

            function TestEnum(e)        
                print('Enum is:'..tostring(e))        

                if space:ToInt() == 0 then
                    print('enum ToInt() is ok')                
                end

                if not space:Equals(0) then
                    print('enum compare int is ok')                
                end

                if space == e then
                    print('enum compare enum is ok')
                end

                local s = UnityEngine.Space.IntToEnum(0)

                if space == s then
                    print('IntToEnum change type is ok')
                end
            end

            function ChangeLightType(light, type)
                print('change light type to '..tostring(type))
                light.type = type
            end
        ";

    LuaState state = null;

    void Start () 
    {
#if UNITY_5 || UNITY_2017 || UNITY_2018
        Application.logMessageReceived += ShowTips;
#else
        Application.RegisterLogCallback(ShowTips);
#endif
        new LuaResLoader();
        state = new LuaState();
        state.Start();
        LuaBinder.Bind(state);

        state.DoString(script);
        state["space"] = Space.World;

        LuaFunction func = state.GetFunction("TestEnum");
        func.BeginPCall();
        func.Push(Space.World);
        func.PCall();
        func.EndPCall();
        func.Dispose();        
        func = null;
	}
    void OnApplicationQuit()
    {
        state.CheckTop();
        state.Dispose();
        state = null;

#if UNITY_5 || UNITY_2017 || UNITY_2018
        Application.logMessageReceived -= ShowTips;
#else
        Application.RegisterLogCallback(null);
#endif        
    }

    string tips = "";
    int count = 1;

    void ShowTips(string msg, string stackTrace, LogType type)
    {
        tips += msg;
        tips += "\r\n";
    }

    void OnGUI()
    {
        GUI.Label(new Rect(Screen.width / 2 - 300, Screen.height / 2 - 200, 600, 400), tips);

        if (GUI.Button(new Rect(0, 60, 120, 50), "ChangeType"))
        {
            GameObject go = GameObject.Find("/Light");
            Light light = go.GetComponent<Light>();
            LuaFunction func = state.GetFunction("ChangeLightType");
            func.BeginPCall();
            func.Push(light);
            LightType type = (LightType)(count++ % 4);
            func.Push(type);
            func.PCall();
            func.EndPCall();
            func.Dispose();
        }
    }
}

```
运行结果为：

![image](https://github.com/Richbabe/Richbabe.github.io/blob/master/img/tolua/Example/10.png?raw=true)

总结：
* 在lua中枚举变量能使用的方法如下：
    * tostring(枚举变量):输出该枚举变量的变量名
    * 枚举变量:ToInt()：将枚举变量转化为对应的整数
    * 枚举变量:Equals(整数)：可以实现整数与枚举变量的比较
    * 枚举变量.IntToEnum()：可以将整数转化为对应的枚举变量
    
    此外，两个枚举变量之间可以用 == 来判断是否相等，并可以直接赋值。赋值方式分为两种：
    * 枚举变量 = 枚举变量
    * 枚举变量 = 枚举类型.枚举值
* 在样例中，我们可以点击ChangeType按钮来改变灯光的类型，这是通过在C#中把Light组件和LightType变量传入lua，在lua的ChangeLightType函数实现灯光类型的设置。从这个例子我们可以看到，Unity的component是可以作为函数参数传递到lua中的。
