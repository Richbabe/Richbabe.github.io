---
layout:     post
title:      tolua框架Example样例学习笔记（6）
subtitle:   lua中操作c#的委托事件与lua中对Unity的GameObject的操作
date:       2018-07-12
author:     Richbabe
header-img: img/u3d技术博客背景.jpg
catalog: true
tags:
    - Lua
    - tolua
    - Unity
---
# 11_UseDelegate
该样例主要讲解如何在lua中操作C#的委托事件，关于C#的委托事件可以看看[我这篇博客](http://richbabe.top/2018/07/07/%E6%8E%A2%E7%A9%B6C-%E4%B8%AD%E7%9A%84Delegate-Event%E5%92%8CUnityEvent/)。

由于这个样例代码过多，所以我们分开来分析，首先来看看TestEventListener.cs:

```
public class TestEventListener : MonoBehaviour
{
    public delegate void VoidDelegate(GameObject go);
    public delegate void OnClick(GameObject go);    
    public OnClick onClick = delegate { };
 
    public event OnClick onClickEvent = delegate { };    
    
    public void SetOnFinished(OnClick click)
    {
        Debugger.Log("SetOnFinished OnClick");
    }
    
    public void SetOnFinished(VoidDelegate click)
    {
        Debugger.Log("SetOnFinished VoidDelegate");
    }
 
    [NoToLuaAttribute]
    public void OnClickEvent(GameObject go)
    {
        onClickEvent(go);
    }
}
```
这是作者自己封装的一个类，里面包含了两种类型的委托事件和一些基础的方法。在后面我们会用到这个类，接着我们来看TestDelegate.cs，在Awake里作者和以往一样做完了所有的基本加载操作：

```
void Awake()
    {               
        state = new LuaState();
        state.Start();
        LuaBinder.Bind(state);   //加载基本的Warp文件
        Bind(state);
 
        state.LogGC = true;
        state.DoString(script);
        GameObject go = new GameObject("TestGo");
        listener = (TestEventListener)go.AddComponent(typeof(TestEventListener));
 
        //获取lua中的函数
        SetClick1 = state.GetFunction("SetClick1");
        AddClick1 = state.GetFunction("AddClick1");
        AddClick2 = state.GetFunction("AddClick2");
        RemoveClick1 = state.GetFunction("RemoveClick1");
        RemoveClick2 = state.GetFunction("RemoveClick2");
        TestOverride = state.GetFunction("TestOverride");
        AddEvent = state.GetFunction("AddEvent");
        RemoveEvent = state.GetFunction("RemoveEvent");        
    }
```
首先是创建lua虚拟机，然后调用state.Start()为虚拟机加载一些标准库。接着使用LuaBinder.Bind(LuaState state)为该虚拟机中加载一些基本的Class类型，就是之前在CustomSetting中添加并Warp的类型。之后的Bind()函数则是为lua虚拟机加载一些该例子特有的类型，平时我们完全可以不通过该方法添加，直接在CustomSetting中添加即可。最后将一开始说的TestEventListener脚本添加到我们创建的物体上。

然后我们来看看编译读取的lua脚本：

```
            function DoClick1(go)                
                print('click1 gameObject is '..go.name)                    
            end

            function DoClick2(go)                
                print('click2 gameObject is '..go.name)                              
            end                       

            -- 添加点击事件DoClick1
            function AddClick1(listener)       
                if listener.onClick then
                    listener.onClick = listener.onClick + DoClick1                                                    
                else
                    listener.onClick = DoClick1                      
                end                
            end

            -- 添加点击事件DoClick2
            function AddClick2(listener)
                if listener.onClick then
                    listener.onClick = listener.onClick + DoClick2                      
                else
                    listener.onClick = DoClick2
                end                
            end

            -- 设置点击事件DoClick1
            function SetClick1(listener)
                if listener.onClick then
                    listener.onClick:Destroy()
                end

                listener.onClick = DoClick1         
            end

            -- 移除点击事件DoClick1
            function RemoveClick1(listener)
                if listener.onClick then
                    listener.onClick = listener.onClick - DoClick1      
                else
                    print('empty delegate')
                end
            end

            -- 移除点击事件DoClick2
            function RemoveClick2(listener)
                if listener.onClick then
                    listener.onClick = listener.onClick - DoClick2    
                else
                    print('empty delegate')                                
                end
            end

            --测试重载问题
            function TestOverride(listener)
                listener:SetOnFinished(TestEventListener.OnClick(DoClick1))
                listener:SetOnFinished(TestEventListener.VoidDelegate(DoClick2))
            end

            -- 测试事件
            function TestEvent()
                print('this is a event')
            end

            -- 添加事件
            function AddEvent(listener)
                listener.onClickEvent = listener.onClickEvent + TestEvent
            end

            -- 移除事件
            function RemoveEvent(listener)
                listener.onClickEvent = listener.onClickEvent - TestEvent
            end

            local t = {name = 'byself'}

            function t:TestSelffunc()
                print('callback with self: '..self.name)
            end       

            function AddSelfClick(listener)
                if listener.onClick then
                    listener.onClick = listener.onClick + TestEventListener.OnClick(t.TestSelffunc, t)
                else
                    listener.onClick = TestEventListener.OnClick(t.TestSelffunc, t)
                end   
            end     

            function RemoveSelfClick(listener)
                if listener.onClick then
                    listener.onClick = listener.onClick - TestEventListener.OnClick(t.TestSelffunc, t)
                else
                    print('empty delegate')
                end   
            end
```
具体方法的含义我已经给了注释，接着便是在C#中获得这些lua方法：

```
 //获取lua中的函数
        SetClick1 = state.GetFunction("SetClick1");
        AddClick1 = state.GetFunction("AddClick1");
        AddClick2 = state.GetFunction("AddClick2");
        RemoveClick1 = state.GetFunction("RemoveClick1");
        RemoveClick2 = state.GetFunction("RemoveClick2");
        TestOverride = state.GetFunction("TestOverride");
        AddEvent = state.GetFunction("AddEvent");
        RemoveEvent = state.GetFunction("RemoveEvent");

        AddSelfClick = state.GetFunction("AddSelfClick");
        RemoveSelfClick = state.GetFunction("RemoveSelfClick");
```
接着我们来看看在OnGUI中是如何调用这些方法的：

```
//UI面板绘制方法
    void OnGUI()
    {
        if (GUI.Button(new Rect(10, 10, 120, 40), " = OnClick1"))
        {
            CallLuaFunction(SetClick1);
        }
        else if (GUI.Button(new Rect(10, 60, 120, 40), " + Click1"))
        {
            CallLuaFunction(AddClick1);
        }
        else if (GUI.Button(new Rect(10, 110, 120, 40), " + Click2"))
        {
            CallLuaFunction(AddClick2);
        }
        else if (GUI.Button(new Rect(10, 160, 120, 40), " - Click1"))
        {
            CallLuaFunction(RemoveClick1);
        }
        else if (GUI.Button(new Rect(10, 210, 120, 40), " - Click2"))
        {
            CallLuaFunction(RemoveClick2);
        }
        else if (GUI.Button(new Rect(10, 260, 120, 40), "+ Click1 in C#"))
        {
            LuaFunction func = state.GetFunction("DoClick1");
            TestEventListener.OnClick onClick = (TestEventListener.OnClick)DelegateFactory.CreateDelegate(typeof(TestEventListener.OnClick), func);
            listener.onClick += onClick;
        }        
        else if (GUI.Button(new Rect(10, 310, 120, 40), " - Click1 in C#"))
        {
            LuaFunction func = state.GetFunction("DoClick1");
            listener.onClick = (TestEventListener.OnClick)DelegateFactory.RemoveDelegate(listener.onClick, func);
            func.Dispose();
            func = null;
        }
        else if (GUI.Button(new Rect(10, 360, 120, 40), "OnClick"))
        {
            if (listener.onClick != null)
            {
                listener.onClick(gameObject);
            }
            else
            {
                Debug.Log("empty delegate!!");
            }
        }
        else if (GUI.Button(new Rect(10, 410, 120, 40), "Override"))
        {
            CallLuaFunction(TestOverride);
        }
        else if (GUI.Button(new Rect(10, 460, 120, 40), "Force GC"))
        {
            //自动gc log: collect lua reference name , id xxx in thread 
            state.LuaGC(LuaGCOptions.LUA_GCCOLLECT, 0);
            GC.Collect();
        }
        else if (GUI.Button(new Rect(10, 510, 120, 40), "event +"))
        {
            CallLuaFunction(AddEvent);
        }
        else if (GUI.Button(new Rect(10, 560, 120, 40), "event -"))
        {
            CallLuaFunction(RemoveEvent);
        }
        else if (GUI.Button(new Rect(10, 610, 120, 40), "event call"))
        {
            listener.OnClickEvent(gameObject);
        }
    }
```
这里面涉及到一个方法CallLuaFunction():

```
void CallLuaFunction(LuaFunction func)
    {        
        func.BeginPCall();
        func.Push(listener);
        func.PCall();
        func.EndPCall();                
    }
```
该方法其实就是对luaFunction调用的一种封装，使得其默认总是以之前的那个TestEventListener组件作为lua函数的参数来调用LuaFunction。

然后我们来分析每个按钮的运行机制：
* 按钮"= OnClick1"

对应的方法是以TestEventListener组件为参数来调用lua的方法 SetClick1(listener)，该lua方法的代码为：

```
function SetClick1(listener)       --设置点击事件
    if listener.onClick then
        listener.onClick:Destroy()
    end
    listener.onClick = DoClick1         
end
```
1. 在这段代码中我们可以看到C#中的委托可以直接使用if语句判空，并且可以使用Delegate:Distory()来销毁该委托
2. 对于C#中的委托的赋值我们可以直接使用lua中的方法以赋值运算符赋值，例如：listener.onClick = DoClick1 

* 按钮 " + Click1" 和 按钮 " + Click2 " 

这两个按钮主要是演示了如何为委托绑定额外的方法，相当于C#中对委托的+=操作，该部分lua方法的代码如下：

```
function AddClick1(listener)       --添加点击事件DoClick1
    if listener.onClick then
        listener.onClick = listener.onClick + DoClick1                                                    
    else
        listener.onClick = DoClick1    
    end                
end
 
function AddClick2(listener)       --添加点击事件DoClick2
    if listener.onClick then
        listener.onClick = listener.onClick + DoClick2                      
    else
        listener.onClick = DoClick2
    end                
end
```
由于lua中不支持+=运算符，所以在这里使用了Delegate = Delegate + 方法这种写法来代替。

* 按钮 " - Click1" 和 按钮 " - Click2 " 

相对于上面的绑定功能自然也有取消绑定的功能，lua代码如下：

```
function RemoveClick1(listener)    --移除点击事件DoClick1
    if listener.onClick then
        listener.onClick = listener.onClick - DoClick1      
    else
        print('empty delegate')
    end
end

function RemoveClick2(listener)    --移除点击事件DoClick2
    if listener.onClick then
        listener.onClick = listener.onClick - DoClick2    
    else
        print('empty delegate')                                
    end
end
```
同样的，lua之中也不支持-=运算符，所以使用这样的写法 ： Delegate = Delegate - 方法

* 按钮 "OnClick"

该按钮即是最基本的C#委托调用，当点击gameObject时触发相应的方法：

```
 else if (GUI.Button(new Rect(10, 360, 120, 40), "OnClick"))
        {
            if (listener.onClick != null)
            {
                listener.onClick(gameObject);
            }
            else
            {
                Debug.Log("empty delegate!!");
            }
        }
```

* 按钮 "Override"

该按钮主要是向大家证实在lua中重载函数可以正常使用。在Tolua中重载函数是完全兼容的，同时还演示了如何在lua中创建委托的写法：

```
委托类型(参数列表)
```
即可在lua中创建一个新的委托变量。

具体代码为：

```
function TestOverride(listener)
    listener:SetOnFinished(TestEventListener.OnClick(DoClick1))
    listener:SetOnFinished(TestEventListener.VoidDelegate(DoClick2))
end
```

* 按钮 " + Click in C# " 和 " - Click in C#"

演示了如何在C#中绑定或者解除绑定lua方法：

```

        if (GUI.Button(new Rect(10, 260, 120, 40), "+ Click1 in C#"))
        {
            LuaFunction func = state.GetFunction("DoClick1");
            TestEventListener.OnClick onClick = (TestEventListener.OnClick)DelegateFactory.CreateDelegate(typeof(TestEventListener.OnClick), func);
            listener.onClick += onClick;
        }        
        else if (GUI.Button(new Rect(10, 310, 120, 40), " - Click1 in C#"))
        {
            LuaFunction func = state.GetFunction("DoClick1");
            listener.onClick = (TestEventListener.OnClick)DelegateFactory.RemoveDelegate(listener.onClick, func);
            func.Dispose();
            func = null;
        }
```
其实就是将lua方法取出到C#中，然后转成LuaFunction，然后直接使用：

```
DelegateFactory.CreateDelegate 和 DelegateFactory.RemoveDelegate
```
进行绑定和解绑，这里需要注意的是，每次使用完LuaFunction后要记得手动释放，否则会有GC！

剩下的按钮就是演示了如何在lua中操作C#事件，其操作方法和委托十分类似，大家看看代码就可以学会了。

最后，需要注意的是在这个例子中需要用到的委托类型也是要单独导出Wrap的，具体做法就是将需要的委托类型添加到CustomSetting.cs的customDelegateList中：

```
    public static DelegateType[] customDelegateList =
    {        
        _DT(typeof(Action)),
        //_DT(typeof(Action)).SetAbrName("ActionGo"),
        _DT(typeof(UnityEngine.Events.UnityAction)),      
        
        _DT(typeof(TestEventListener.OnClick)),
        _DT(typeof(TestEventListener.VoidDelegate)),
    };
```
因为运行效果和具体的操作顺序有关，所以运行截图我这里就不放了。

# 12_GameObject
这个样例主要演示了在lua中如何对Unity的GameObject进行操作

核心代码为：

```

    private string script =
        @"                                    
            local GameObject = UnityEngine.GameObject          
            local ParticleSystem = UnityEngine.ParticleSystem            
            local go = GameObject('go')
            go:AddComponent(typeof(ParticleSystem))
            local node = go.transform
            node.position = Vector3.one      
            print('gameObject is: '..tostring(go))    
            GameObject.Destroy(go, 2)                        
        ";
 
    LuaState lua = null;
 
    void Start()
    {        
        lua = new LuaState();
        lua.LogGC = true;
        lua.Start();
        LuaBinder.Bind(lua);
        lua.DoString(script);            
    }
```
从代码中我们很容易看出我们在lua代码中创建了一个GameObject对象并命名为go，接着给他添加了粒子系统，然后设置其坐标为(1,1,1)，接着过2s把他Destroy掉。

总结：
* GameObject:AddComponent(typeof(Component))  在lua中给游戏物体添加组件
* GameObject("游戏物体名")  在lua中创建新的游戏物体
* GameObject.Destroy(游戏对象, 延迟时间)  延时销毁的指定的游戏物体



    
