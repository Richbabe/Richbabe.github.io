---
layout:     post
title:      探究C#中的Delegate、Event和UnityEvent
subtitle:   
date:       2018-07-07
author:     Richbabe
header-img: img/u3d技术博客背景.jpg
catalog: true
tags:
    - C#
    - Unity
---
# 前言
Event作为C#语言特性的一部分，在.Net开发中具有比较重要的作用。当我们使用C#作为脚本语言编写Unity游戏时，也经常会需要使用Event解决各种问题。然而，相比于传统C#语言的Event，UnityEvent基于Unity引擎做了一些改变，并且更加适合我们的游戏开发。为了帮助读者深入理解UnityEvent，本文会从Delegate讲起，并逐步介绍C# Event 与UnityEvent的相似与不同。 

本文参考了Unity论坛以及其他前辈的文章和视频，想要进一步了解的可以自行查阅:
* [Unity3D中使用委托和事件](https://www.cnblogs.com/murongxiaopifu/p/4149659.html)
* [Delegate Events VS UnityEvent, which one is superior?](https://www.reddit.com/r/Unity3D/comments/35oekm/delegate_events_vs_unityevent_which_one_is/)
* [C# Events and Delegates Made Simple](https://www.youtube.com/watch?v=jQgwEsJISy0)
* [Events - Unity Official Tutorials](https://www.youtube.com/watch?v=6qyR73EO68w)

# 一切的渊源——Delegate
要理解Event是什么，首先必须得知道它们的前身——Delegate是啥，中文翻译即“委托”。用一句话让你理解Delegate的作用就是“Delegate是一个可以存放函数的容器”。众所周知，变量是程序在内存中为数据开辟的一块空间，面向对象语言中变量可以存放一个具体的数值，或者某个对象的引用。C#则在该基础上更进一步，使用Delegate的机制让存放“函数（Function）”成为可能。 
使用Delegate一般分为三步：
1. 定义一种委托类型
2. 声明一个该类型的委托函数
3. 通过声明的委托调用函数执行相关操作

还是通过一个简单的例子来了解委托的用法。假设现在我们分别要对奇数和偶数做不同的处理方案：

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class PrintEvenNum : MonoBehaviour {

	// Use this for initialization
	void Start () {
        PrintNum(1);
        PrintNum(2);
	}
	
	// Update is called once per frame
	void Update () {
		
	}

    //最终输出函数
    void PrintNum(int num)
    {
        //如果num是奇数，则调用奇数的输出函数
        if(num % 2 != 0)
        {
            PrintOdd(num);
        }
        //如果num是偶数函数，则调用偶数的输出函数
        else
        {
            PrintEven(num);
        }
    }

    //处理奇数函数
    void PrintOdd(int num)
    {
        Debug.Log(num + " is odd number！");
        //...其他处理奇数函数的部分
    }
    
    //处理偶数函数
    void PrintEven(int num)
    {
        Debug.Log(num + " is even number！");
        //...其他处理偶数函数的部分
    }
}


```
可以看到我们在PrintNum函数中通过if-else来调用不同的函数，那么当输入的数有多种情况需要分别处理时，我们是不是要引用大量的if-elseif呢？

如果我们能够在PrintNum函数中传入不同类型数的处理函数，即PrintNum的第一个参数是num，第二个参数是处理该数的函数，那么就可以省去if和else语句了。现在的问题就是传入参数为函数时，该参数的类型是什么呢？

事实上，我们可以通过委托解决这个问题。我们知道区分一个函数，即区分他的返回值和传入参数的类型，因此我们可以通过定义一个与该函数具有相同类型的返回值和参数类型的委托来声明该函数作为参数时参数的类型：

```
public delegate void PrintNumDelegate(int num);
```
然后修改PrintNum函数为：

```
//最终输出函数
    void PrintNum(int num, PrintNumDelegate func)
    {
        func(num);
    }
```
总的代码是这样的：

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class PrintEvenNum : MonoBehaviour {
    public delegate void PrintNumDelegate(int num);

    // Use this for initialization
    void Start () {
        PrintNum(1, PrintOdd);
        PrintNum(2, PrintEven);
	}
	
	// Update is called once per frame
	void Update () {
		
	}

    //最终输出函数
    void PrintNum(int num, PrintNumDelegate func)
    {
        func(num);
    }

    //处理奇数函数
    void PrintOdd(int num)
    {
        Debug.Log(num + " is odd number！");
        //...其他处理奇数函数的部分
    }
    
    //处理偶数函数
    void PrintEven(int num)
    {
        Debug.Log(num + " is even number！");
        //...其他处理偶数函数的部分
    }
}

```
可以发现我们在减少代码量、提高可扩展性的同时也实现了功能。可见所谓委托就是一个类，它定义了方法的类型，使得可以将方法当作另一个方法的参数来进行传递，这种将方法动态地赋给参数的做法，可以避免在程序中大量使用If … Else(Switch)语句，同时使得程序具有更好的可扩展性。

实际上，委托还能这样使用：

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class PrintEvenNum : MonoBehaviour {
    public delegate void PrintNumDelegate(int num);

    //声明一个PrintNumDelegate的变量
    public PrintNumDelegate myDelegate;

    // Use this for initialization
    void Start () {
        myDelegate = PrintOdd;
        myDelegate(1);

        myDelegate = PrintEven;
        myDelegate(2);
	}
	
	// Update is called once per frame
	void Update () {
		
	}

    //最终输出函数
    void PrintNum(int num, PrintNumDelegate func)
    {
        func(num);
    }

    //处理奇数函数
    void PrintOdd(int num)
    {
        Debug.Log(num + " is odd number！");
        //...其他处理奇数函数的部分
    }
    
    //处理偶数函数
    void PrintEven(int num)
    {
        Debug.Log(num + " is even number！");
        //...其他处理偶数函数的部分
    }
}

```
运行上述代码，你会发现使用一个delegate变量让我们执行了两种函数实现。这就是Delegate的妙处所在， Delegate定义了一个用于存放相同函数原型（相同参数类型，相同返回值）的容器。因为他们的函数原型相等，所以向delegate传递一次参数，就可以让所有添加到delegate上的函数正常执行。 
在上述例子中，我们第二次向myDelegate赋值时覆盖掉了第一次赋值的函数，所以第二次引用myDelegate只会调用PrintEven(int num)函数。实际上，delegate作为函数容器，并不仅仅只能容纳一个函数，而是可以同时被委任多个函数。例如，当你把上述代码中的第二次赋值改为 
```
myDelegate += PrintEven;
```
 就可以实现同时打印两条语句的效果。这种delegate一般被称为multicast delegate。

# 基于delegate实现的Event（C# Event）
事实上，event就是在Multicast delegate的基础上演变来的。关于Event，一个比较形象的比喻就是广播者和订阅者。我们把Event想象成一个主播（比如说PUBG的GodV？），那么他就会有一堆热情的粉丝（等待狙击他的水友...）。为了在第一时间知道伟神什么时候开播，水友必须在直播间订阅（关注）伟神，这样在伟神开播时水友便能收到伟神开播的消息。

在程序的世界里，伟神（Event）为他的水友（具有相同函数类型和相同参数的函数）提供了订阅他的途径（即把自身加入到event的函数容器中），这样无论他有什么动向，都可以直接通知所有他知道的粉丝（调用event会立即引用所有函数容器中的函数）。当然，一千个观众心中就有一千个哈姆雷特，就如同真爱粉无论如何都会支持自己的主播，而黑粉无时无刻不在直播间带节奏一样（比如说：刚刚这波操作绝对开挂了），event只负责告诉每个函数什么时候被调用，这些函数到底干了什么，event并不关心。

下面我们来看看在C#中如何实现event：

在Idol.cs(模拟伟神)中

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

//伟神
public class Idol : MonoBehaviour {
    public delegate void IdolBehaviour(string behaviour);//定义委托
    public static event IdolBehaviour IdolDoSomethingHandler;//定义事件

	// Use this for initialization
	void Start () {
		//伟神开播了，通知他的全部粉丝
        if(IdolDoSomethingHandler != null)
        {
            IdolDoSomethingHandler("伟神开播啦！！");
        }
	}
	
	// Update is called once per frame
	void Update () {
		
	}
}

```
将该脚本挂载在一个命名为Idol的空物体上。

在SubscriberA.cs(模拟伟神的真爱粉)中

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

//真爱粉
public class SubscriberA : MonoBehaviour {

    // OnEnable在该脚本被启用时调用,你可以把它看做路转粉的开端
    private void OnEnable()
    {
        //粉丝通过订阅偶像来获取偶像的咨询, 并在得到讯息后执行相应的动作
        Idol.IdolDoSomethingHandler += LikeIdol;
    }

    // OnDisable在该脚本被禁用时调用,你可以把它看做粉转路的开端
    private void OnDisable()
    {
        Idol.IdolDoSomethingHandler -= LikeIdol;
    }

    // 真爱粉针对伟神开播的处理函数
    public void LikeIdol(string idolAction)
    {
        print(idolAction + " ,快点围观伟神吃鸡没");
    }

    // Use this for initialization
    void Start () {
		
	}
	
	// Update is called once per frame
	void Update () {
		
	}
}

```
将该脚本挂载在一个命名为SubscriberA的空物体上。

在SubscriberB.cs(模拟伟神的黑粉)中

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

//黑粉
public class SubscriberB : MonoBehaviour {

    // OnEnable在该脚本被启用时调用,你可以把它看做路转黑的开端
    private void OnEnable()
    {
        //粉丝通过订阅偶像来获取偶像的咨询, 并在得到讯息后执行相应的动作
        Idol.IdolDoSomethingHandler += HateIdol;
    }

    // OnDisable在该脚本被禁用时调用,你可以把它看做黑转路的开端
    private void OnDisable()
    {
        Idol.IdolDoSomethingHandler -= HateIdol;
    }

    // 黑粉针对伟神开播的处理函数
    public void HateIdol(string idolAction)
    {
        print(idolAction + " ,快点去直播间带伟神开挂的节奏");
    }

    // Use this for initialization
    void Start()
    {

    }

    // Update is called once per frame
    void Update()
    {

    }
}

```
将该脚本挂载在一个命名为SubscriberB的空物体上。

运行，结果为：

![image](https://github.com/Richbabe/Richbabe.github.io/blob/master/img/%E5%A7%94%E6%89%98%E4%B8%8E%E4%BA%8B%E4%BB%B6/event.png?raw=true)

可以看到两位订阅者分别对他们订阅的事件做出了不同的处理。下面为使用event的注意事项：
* 想让爱豆直接通知你TA的近况，你就必须在他发出消息前完成订阅。在本例中，虽然两个粉丝和爱豆位于不同的GameObject上，但是他们都提前订阅了Idol，所以Idol才能正确通知到他们（OnEnable()在执行顺序上优先于Start()，关于OnEnable/Start/Awake的辨析，可以看看 [Awake/Start/OnEnable 辨析](http://blog.csdn.net/qq_28849871/article/details/78137261) ）
* 在本例中两个粉丝均采用了调用静态字段IdolDoSomethingHandler的方法实现订阅，实际上你也可以为每个粉丝添加一个public Idol idol;然后在Editor上直接绑定。（这点是Unity特有的）;
* 偶像并不关心他的粉丝对自己的行为作出何种反映。甚至在他发出消息时，除了确认一下自己还没有过气（IdolDoSomethingHandler != null）之外，对粉丝的行为不会有任何了解。（在降低耦合性loose decoupling 的同时，隐藏了事件函数的实现细节）
* 你并不需要担心偶像受不受得了同时给那么多粉丝发邮件，因为一般有经纪人代办（误）。细心的人可能会发现我们在声明event delegate时并没有给它分配内存，使用时直接赋值或添加即可。

# UnityEvent
经过上面的解释，你应该对event有个大概的了解。那么接下来我们来看阿奎那Unity在Event的基础上进行的改良，即UnityEvent。Event设计之初并不会想到应用于Unity游戏开发，所以它的弊端就在于纯代码编程，没有通过使用Unity Editor提高工作效率。而UnityEvent就可以看做是发挥Editor作用的正确改良。还记得上一节中粉丝是怎么订阅的嘛？你必须在每个粉丝对象中访问Idol的IdolDoSomethingHandler，然后把自己将采取的行动添加上去。这样有两个坏处——其一就是你必须时刻提防订阅的时机，假如不小心在Idol发动态之后才订阅，那你就永远收不到那条动态了。其二就是不方便管理，想要查看订阅偶像的所有粉丝，我们就得查找项目中所有IdolDoSomethingHandler的引用，然后再把每个粉丝的文件打开，可以说是非常麻烦了。 

为了避免上述的缺点，UnityEvent使用Serializable让用户可以在Editor中直接绑定所有粉丝的调用，即一目了然又不用担心把握不准订阅的时机。

话不多说，我们直接上代码：

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.Events;

//使用Serializable序列化IdolEvent,否则无法在Editor中显示
[System.Serializable]
public class IdolEvent : UnityEvent<string>
{

}

//伟神
public class Idol : MonoBehaviour {
    //public delegate void IdolBehaviour(string behaviour);//定义委托
    //public static event IdolBehaviour IdolDoSomethingHandler;//定义事件

    public IdolEvent idolEvent;//定义UnityEvent

	// Use this for initialization
	void Start () {
		//伟神开播了，通知他的全部粉丝
        if(idolEvent == null)
        {
            idolEvent = new IdolEvent();
            //IdolDoSomethingHandler("伟神开播啦！！");
        }
        idolEvent.Invoke("伟神开播啦！！");
	}
	
	// Update is called once per frame
	void Update () {
		
	}
}

```

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

//真爱粉
public class SubscriberA : MonoBehaviour {
    /*
    // OnEnable在该脚本被启用时调用,你可以把它看做路转粉的开端
    private void OnEnable()
    {
        //粉丝通过订阅偶像来获取偶像的咨询, 并在得到讯息后执行相应的动作
        Idol.IdolDoSomethingHandler += LikeIdol;
    }

    // OnDisable在该脚本被禁用时调用,你可以把它看做粉转路的开端
    private void OnDisable()
    {
        Idol.IdolDoSomethingHandler -= LikeIdol;
    }
    */
    // 真爱粉针对伟神开播的处理函数
    public void LikeIdol(string idolAction)
    {
        print(idolAction + " ,快点围观伟神吃鸡没");
    }

    // Use this for initialization
    void Start () {
		
	}
	
	// Update is called once per frame
	void Update () {
		
	}
}

```

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

//黑粉
public class SubscriberB : MonoBehaviour {
    /*
    // OnEnable在该脚本被启用时调用,你可以把它看做路转黑的开端
    private void OnEnable()
    {
        //粉丝通过订阅偶像来获取偶像的咨询, 并在得到讯息后执行相应的动作
        Idol.IdolDoSomethingHandler += HateIdol;
    }

    // OnDisable在该脚本被禁用时调用,你可以把它看做黑转路的开端
    private void OnDisable()
    {
        Idol.IdolDoSomethingHandler -= HateIdol;
    }
    */
    // 黑粉针对伟神开播的处理函数
    public void HateIdol(string idolAction)
    {
        print(idolAction + " ,快点去直播间带伟神开挂的节奏");
    }
    
    // Use this for initialization
    void Start()
    {

    }

    // Update is called once per frame
    void Update()
    {

    }
}

```
把上面三个脚本绑定到三个GameObject上，但是不要着急立刻运行游戏，因为我们还没有让两个粉丝实现订阅。和使用Event时不同，UnityEvent在序列化后可以在Editor上显示，并且可以让我们在Editor阶段就设置好需要执行的函数。选中Idol所在的GameObject，然后就可以在Inspector中设置IdolEvent可以引用的函数。设置完成后应该如图所示:

![image](https://github.com/Richbabe/Richbabe.github.io/blob/master/img/%E5%A7%94%E6%89%98%E4%B8%8E%E4%BA%8B%E4%BB%B6/unityEvent.png?raw=true)

此时再运行游戏，你会得到和使用基于delegate的Event时相同的效果。

除此之外，UnityEvent依然提供和C# Event 类似的运行时绑定的功能，不过不同的是，UnityEvent是一个对象，向其绑定函数是通过AddListener()方法实现的。以SubscriberB为例，我们可以在代码中实现同等效果的绑定：

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

//黑粉
public class SubscriberB : MonoBehaviour {
    public Idol myIdol;

    // OnEnable在该脚本被启用时调用,你可以把它看做路转黑的开端
    private void OnEnable()
    {
        //粉丝通过订阅偶像来获取偶像的咨询, 并在得到讯息后执行相应的动作
        myIdol.idolEvent.AddListener(HateIdol);
    }

    // OnDisable在该脚本被禁用时调用,你可以把它看做黑转路的开端
    private void OnDisable()
    {
        myIdol.idolEvent.RemoveListener(HateIdol);
    }
    
    // 黑粉针对伟神开播的处理函数
    public void HateIdol(string idolAction)
    {
        print(idolAction + " ,快点去直播间带伟神开挂的节奏");
    }
    
    // Use this for initialization
    void Start()
    {

    }

    // Update is called once per frame
    void Update()
    {

    }
}

```
需要注意的是：

由于UnityEvent是一个对象，所以自然可以允许我们通过继承实现自己的Event，实际上Unity中包括Button在内的许多UI组件的点击事件都是通过继承自UnityEvent来复写的。 

可访问性(public/private)决定了UnityEvent的默认值，当可访问性为public时，默认会为其分配空间(new UnityEvent())；当可访问性为private时，默认UnityEvent为null，需要在Start()中为其分配内存。






