---
layout:     post
title:      tolua框架Example样例学习笔记（2）
subtitle:   C#与lua的函数，变量交互方法
date:       2018-07-09
author:     Richbabe
header-img: img/u3d技术博客背景.jpg
catalog: true
tags:
    - Lua
    - tolua
    - Unity
---
# 03_CallLuaFunction
该样例主要表现了C#怎么调用lua中的函数，核心代码如下：

```
    private string script = 
        @"  function luaFunc(num)                        
                return num + 1
            end
            test = {}
            test.luaFunc = luaFunc
        ";
 
    LuaFunction func = null;
    LuaState lua = null;
	
	void Start () 
       {
        lua = new LuaState();
        lua.Start();
        lua.DoString(script);
 
        //Get the function object
        func = lua.GetFunction("test.luaFunc");
 
        if (func != null)
        {
            //有gc alloc
            object[] r = func.Call(123456);
            Debugger.Log(r[0]);
 
            // no gc alloc
            int num = CallFunc();
            Debugger.Log(num);
        }
 
        lua.CheckTop();
	}
 
    int CallFunc()
    {        
        func.BeginPCall();                
        func.Push(123456);
        func.PCall();        
        int num = (int)func.CheckNumber();                    
        func.EndPCall();
        return (int)num;        
    }
```
运行效果图为：
![image](https://github.com/Richbabe/Richbabe.github.io/blob/master/img/tolua/Example/03.png?raw=true)

总结：
* 对于lua的函数在C#中的调用，我们可以把lua中的函数看成一个对象，在虚拟机初始化完成后，加载对应的lua文件，接着需要创建一个LuaFunction类型的对象来表示这个lua函数，可以通过调用：

```
lua.GetFunction("方法名"); 
```
来获取lua文件中对应的函数对象，接下来就是调用了。

* C#中调用lua函数的方法有两种：
1. 直接调用LuaFunction类型的对象的func.Call 方法，完整声明为：
```
public object[] Call(params object[] args)
```
这种调用方法比较简单，但是有一个缺点，lua对象的内存无法被自动释放，所以当使用完这个lua函数对象之后，我们需要手动的调用LuaFunction类型的对象的func.Dispose();方法，释放掉垃圾内存，否则会造成内存泄漏！
2. 如样例中的CallFunc()所示，代码如下：

```
int CallFunc()
    {        
        func.BeginPCall();                
        func.Push(123456);
        func.PCall();        
        int num = (int)func.CheckNumber();                    
        func.EndPCall();
        return (int)num;        
    }
```
这种调用方法比较麻烦，首先我们必须先以func.BeginPCall;开始，通过func.Push(参数)来给方法传参，然后需要通过func.PCall();来运行，接着通过对应的func.Checkxxx()方法来获取返回值,最后通过func.EndPCall();结束。整个流程比较繁琐且不易封装，不过优点是不会有垃圾内存，所以不用手动释放GC。

# 04_AccessingLuaVariables
这个样例主要是介绍C#中如何获取、声明、使用lua中的变量，核心代码如下：

```
public class AccessingLuaVariables : MonoBehaviour 
{
    private string script =
        @"
            print('Objs2Spawn is: '..Objs2Spawn)
            var2read = 42
            varTable = {1,2,3,4,5}
            varTable.default = 1
            varTable.map = {}
            varTable.map.name = 'map'
            
            meta = {name = 'meta'}
            setmetatable(varTable, meta)
            
            function TestFunc(strs)
                print('get func by variable')
            end
        ";
 
	void Start () 
    {        
        LuaState lua = new LuaState();
        lua.Start();
        lua["Objs2Spawn"] = 5;
        lua.DoString(script);
 
        //通过LuaState访问
        Debugger.Log("Read var from lua: {0}", lua["var2read"]);
        Debugger.Log("Read table var from lua: {0}", lua["varTable.default"]);//LuaState 拆串式table
 
        LuaFunction func = lua["TestFunc"] as LuaFunction;
        func.Call();
        func.Dispose();
 
        //cache成LuaTable进行访问
        LuaTable table = lua.GetTable("varTable");
        Debugger.Log("Read varTable from lua, default: {0} name: {1}", table["default"], table["map.name"]);//没有map.name这个键，因此table["map.name"]为nil
        table["map.name"] = "new";//table 字符串只能是key
        Debugger.Log("Modify varTable name: {0}", table["map.name"]);
 
        table.AddTable("newmap");
        LuaTable table1 = (LuaTable)table["newmap"];
        table1["name"] = "table1";
        Debugger.Log("varTable.newmap name: {0}", table1["name"]);
        table1.Dispose();
 
        table1 = table.GetMetaTable();
 
        if (table1 != null)
        {
            Debugger.Log("varTable metatable name: {0}", table1["name"]);
        }
 
        object[] list = table.ToArray();
 
        for (int i = 0; i < list.Length; i++)
        {
            Debugger.Log("varTable[{0}], is {1}", i, list[i]);
        }
 
        table.Dispose();
        lua.CheckTop();
        lua.Dispose();
	}
}
```
运行效果为：
![image](https://github.com/Richbabe/Richbabe.github.io/blob/master/img/tolua/Example/04.png?raw=true)

总结：
* 在C#中创建lua虚拟机的全局变量的方法：

    在lua虚拟机创建完成且初始化完毕（调用Start方法）之后，可以直接通过以下格式声明一个lua虚拟机的全局变量:

```
虚拟机名["变量名"] = 值
```
则创建了一个名为“变量名”的lua全局变量，其值为右式的值
* 访问lua变量和创建lua虚拟机的全局变量的格式相同，这一点和lua中关于变量的定义是相同的，如果没有就创建，否则就获取对应的值。其中对于lua中的函数的使用是需要经过强制转换的，例如要获得lua中的函数，可以：

```
LuaFunction func = lua["TestFunc"] as LuaFunction;
```
可见我们又多了一个获取lua函数的方法。
* lua中table的获取和创建：通过调用虚拟机的方法lua.GetTable 来获取lua中的table，用LuaTable类型来储存lua中的Table，通过调用Luatable的成员方法table.AddTable来创建lua中的table , 除了通过虚拟机的GetTable方法我访问之外，直接通过 LuaTable  型变量按字典的类似方法也可以调用table ， 例如如下：

```
LuaTable table1 = (LuaTable)table["newmap"];
```
对于lua元表也是可以获取的，方法为调用LuaTable的table.GetMetaTable();方法，则可以获得table的元表。
* 需要注意，在样例中的lua代码中：

```
varTable.map = {}
varTable.map.name = 'map'
```
在C#代码中：
```
Debugger.Log("Read varTable from lua, default: {0} name: {1}", table["default"], table["map.name"]);//没有map.name这个键，因此table["map.name"]为nil
```
我们想要通过table["map.name"]来获取varTable.map.name是不可行的，因为其把双引号内的map.name看成table的一个key，其值是nil。正确做法是：

```
LuaTable table = lua.GetTable("varTable");
LuaTable map = (LuaTable)table["map"];
Debugger.Log("Read varTable from lua, default: {0} name: {1}", table["default"], map["name"]);
```
* 对LuaTable类型的变量，在使用完后需要手动释放内存，否则会因为内存未自动释放造成内存泄漏，具体方法为调用LuaTable对象的方法table.Dispose();





