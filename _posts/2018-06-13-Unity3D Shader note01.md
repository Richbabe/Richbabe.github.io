---
layout:     post
title:      Unity3D Shader学习日记（1）
subtitle:   初探Standard Surface Shader
date:       2018-06-13
author:     Richbabe
header-img: img/post-bg-desk.jpg
catalog: true
tags:
    - 计算机图形学
    - Unity
---
# 引言
这学期的计算机图形学学了OpenGL的一些基本操作，但是想要应用到自己的游戏里面还是有点难度，幸好伟大的unity有自带的Shader编程，在这里开个Unity3d Shader学习记录，希望自己能坚持下去。

# 参考博客
[参考blog1](https://onevcat.com/2013/07/shader-tutorial-1/)

[参考blog2](https://blog.csdn.net/poem_qianmo/article/details/49405909)

# 基本概念
## Shader 和 Material
Shader，即着色器，它负责将输入的Mesh（网格）以指定的方式和输入的贴图或者颜色等组合作用，然后输出。绘图单元可以依据这个输出来将图像绘制到屏幕上。输入的贴图或者颜色等，加上对应的Shader，以及对Shader的特定的参数设置，将这些内容（Shader及输入参数）打包存储在一起，得到的就是一个Material（材质）。之后，我们便可以将材质赋予合适的Renderer（渲染器）来进行渲染（输出）了。

总的来说，Shader就是一段规定好输入（颜色、贴图等）和输出（渲染器能够读懂的点和颜色的对应关系）的程序。而Shader开发者要做的就是根据输入，进行计算变换，产生输出而已。

在Unity中，Shader模板分为四类：
* Standard Surface Shader 标准表面着色器
* Unlit Shader 无灯光着色器
* Image Effect Shader 图像特效着色器
* compute Shader 
    
# Shader程序的基本结构
一个普通的着色器的结构应该如下：
![image](https://github.com/Richbabe/Richbabe.github.io/blob/master/img/U3d%20Shader/Shader%E7%A8%8B%E5%BA%8F%E7%9A%84%E5%9F%BA%E6%9C%AC%E7%BB%93%E6%9E%84.jpg?raw=true)
## 名词解释
（1）属性定义

用来指定这段代码将有哪些输入

（2）子着色器

在实际运行中，哪一个子着色器被使用是由运行的平台所决定的。子着色器是代码的主体，每一个子着色器中包含一个或者多个的Pass。在计算着色时，平台先选择最优先可以使用的着色器，然后依次运行其中的Pass，然后得到输出的结果

（3）回滚

用来处理所有子着色器都不能运行的情况（比如目标设备实在太老，所有Subshader中都有其不支持的特性）。

# 新建第一个Shader程序
在Project面板新建一个StandardSurfaceShader，命名为DiffuseTexture，Shader程序如下

```
Shader "Custom/DiffuseTexture" {
	Properties {
		//主颜色，由RBGA（红蓝绿和透明度）四个量来定义,初始化为白色
		_Color ("Color", Color) = (1,1,1,1)
		//主贴图(在写贴图的输入时至少要一对什么都不含的空白的{}),初始化为白色
		_MainTex ("Albedo (RGB)", 2D) = "white" {}
		//光泽度
		_Glossiness ("Smoothness", Range(0,1)) = 0.5
		//金属度
		_Metallic ("Metallic", Range(0,1)) = 0.0
	}
	SubShader {
		//硬件通过标签来决定什么时候调用该着色器
		Tags { "RenderType"="Opaque" }
		LOD 200

		//******开始CG着色器语言编写模块******
		CGPROGRAM
		// Physically based Standard lighting model, and enable shadows on all light types
		//编译指令：告知编译器表面着色器函数的名称为surf
		//Standard表示光照模型为Unity标准版光照模型
		//fullforwardshadows表示在正向渲染路径中支持所有阴影类型
		#pragma surface surf Standard fullforwardshadows

		// Use shader model 3.0 target, to get nicer looking lighting
		//编译指令：指定着色器编译目标为Shader Model 3.0
		#pragma target 3.0

		sampler2D _MainTex;

		//表面输入结构体
		struct Input {
			float2 uv_MainTex;//纹理坐标
		};

		half _Glossiness;
		half _Metallic;
		fixed4 _Color;

		// Add instancing support for this shader. You need to check 'Enable Instancing' on materials that use the shader.
		// See https://docs.unity3d.com/Manual/GPUInstancing.html for more information about instancing.
		// #pragma instancing_options assumeuniformscaling
		UNITY_INSTANCING_BUFFER_START(Props)
			// put more per-instance properties here
		UNITY_INSTANCING_BUFFER_END(Props)

		//--------------------------------【表面着色函数】-----------------------------  
		//输入：表面输入结构体  
		//输出：Unity内置的SurfaceOutputStandard结构体  
		//SurfaceOutputStandard原型如下：  
		/*
			struct SurfaceOutputStandard
			{
				fixed3 Albedo;                  // 漫反射颜色
				fixed3 Normal;                  // 切线空间法线
				half3 Emission;                 //自发光
				half Metallic;                           // 金属度；取0为非金属, 取1为金属
				half Smoothness;             // 光泽度；取0为非常粗糙, 取1为非常光滑
				half Occlusion;                 // 遮挡(默认值为1)
				fixed Alpha;                      // 透明度
			};
		*/
		//---------------------------------------------------------------------------------  
		void surf (Input IN, inout SurfaceOutputStandard o) {
			// Albedo comes from a texture tinted by color
			//漫反射颜色为主纹理对应的纹理坐标，并乘以主颜色
			fixed4 c = tex2D (_MainTex, IN.uv_MainTex) * _Color;
			//将准备好的颜色的rgb分量作为漫反射颜色
			o.Albedo = c.rgb;
			// Metallic and smoothness come from slider variables
			//金属度取自属性值
			o.Metallic = _Metallic;
			//光泽度取自属性值
			o.Smoothness = _Glossiness;
			//将准备好的颜色的alpha分量作为Alpha分量值
			o.Alpha = c.a;
		}
		//*****结束CG着色器语言编写模块******
		ENDCG
	}
	//备胎为漫反射
	FallBack "Diffuse"
}
```
可以看到该Shader代码由三部分构成：
* Properties
* SubShader
* FallBack

PS：第一行是这个Shader的声明，并为其制定了一个名字（DiffuseTexture）。你可以在材质面板选择Shader时在对应的位置找到这个Shader。

接下来我们来对这个Shader进行逐一击破！

## 属性 Properties
在Properties{}中定义着色器属性，在这里定义的属性将被作为输入提供给所有的子着色器。每一条属性的定义的语法是这样的：

```
_Name("Display Name", type) = defaultValue[{options}]
```
* _Name - 属性的名字，简单说就是变量名，在之后整个Shader代码中将使用这个名字来获取该属性的内容
* Display Name - 这个字符串将显示在Unity的材质编辑器中作为Shader的使用者可读的内容
* type - 这个属性的类型，可能的type所表示的内容有以下几种：
    * Color - 一种颜色，由RGBA（红绿蓝和透明度）四个量来定义；
    * 2D - 一张2的阶数大小（256，512之类）的贴图。这张贴图将在采样后被转为对应基于模型UV的每个像素的颜色，最终被显示出来；
    * Rect - 一个非2阶数大小的贴图；
    * Cube - 即Cube map texture（立方体纹理），简单说就是6张有联系的2D贴图的组合，主要用来做反射效果（比如天空盒和动态反射），也会被转换为对应点的采样；
    * Range(min, max) - 一个介于最小值和最大值之间的浮点数，一般用来当作调整Shader某些特性的参数（比如透明度渲染的截止值可以是从0至1的值等）；
    * Float - 任意一个浮点数；
    * Vector - 一个四维数；
* defaultValue -  定义了这个属性的默认值，通过输入一个符合格式的默认值来指定对应属性的初始值（某些效果可能需要某些特定的参数值来达到需要的效果，虽然这些值可以在之后在进行调整，但是如果默认就指定为想要的值的话就省去了一个个调整的时间，方便很多）。
    * Color - 以0～1定义的rgba颜色，比如(1,1,1,1)；
    * 2D/Rect/Cube - 对于贴图来说，默认值可以为一个代表默认tint颜色的字符串，可以是空字符串或者”white”,”black”,”gray”,”bump”中的一个
    * Float，Range - 某个指定的浮点数
    * Vector - 一个4维数，写为 (x,y,z,w)
* 另外还有一个{option}，它只对2D，Rect或者Cube贴图有关，在写输入时我们最少要在贴图之后写一对什么都不含的空白的{}，当我们需要打开特定选项时可以把其写在这对花括号内。如果需要同时打开多个选项，可以使用空白分隔。可能的选择有ObjectLinear, EyeLinear, SphereMap, CubeReflect, CubeNormal中的一个，这些都是OpenGL中TexGen的模式，具体的留到后面有机会再说。

现在再看回我们Shader中的属性部分：

```
Properties {
		//主颜色，由RBGA（红蓝绿和透明度）四个量来定义,初始化为白色
		_Color ("Color", Color) = (1,1,1,1)
		//主贴图(在写贴图的输入时至少要一对什么都不含的空白的{}),初始化为白色
		_MainTex ("Albedo (RGB)", 2D) = "white" {}
		//光泽度
		_Glossiness ("Smoothness", Range(0,1)) = 0.5
		//金属度
		_Metallic ("Metallic", Range(0,1)) = 0.0
	}
```
基本可以看懂了吧！接下来我们来看看子着色器SubShader部分。

## 子着色器 SubShader
### Tags

表面着色器可以被若干个的标签（tags）所修饰，而硬件将通过判定这些标签来决定什么时候调用该着色器。

比如我们的着色器程序中SubShader的第一句：

```
//硬件通过标签来决定什么时候调用该着色器
		Tags { "RenderType"="Opaque" }
```
Opaque表示非透明物体渲染，该行告诉了系统应该在渲染非透明物体时调用该SubShader。

与"RenderType = Opaque"相反的是"RenderType = Transparent"，其告诉系统在渲染含有透明效果的物体时调用该物体。

在这里Tags其实暗示了你的Shader输出的是什么，如果输出中都是非透明物体，那写在Opaque里，如果想渲染透明或半透明的物体，那应该写在Transparent中。

另外比较有用的标签还有：
（1）"IgnoreProjector"="True"（不被Projectors影响）

（2）"ForceNoShadowCasting"="True"（从不产生阴影）

（3）Queue"="xxx"（指定渲染顺序队列）需要注意的是，当我们使用Unity做一些透明和不透明物体的混合的话，很可能遇到过不透明物体无法呈现在透明物体之后的情况。这种情况很可能是由于Shader的渲染顺序不正确导致的。Queue指定了物体的渲染顺序，预定义的Queue有：
* Background - 最早被调用的渲染，用来渲染天空盒或者背景
* Geometry - 这是默认值，用来渲染非透明物体（普通情况下，场景中的绝大多数物体应该是非透明的）
* AlphaTest - 用来渲染经过Alpha Test的像素，单独为AlphaTest设定一个Queue是出于对效率的考虑
* Transparent - 以从后往前的顺序渲染透明物体
* Overlay - 用来渲染叠加的效果，是渲染的最后阶段（比如镜头光晕等特效）

这些预定义的值本质上是一组定义整数，Background = 1000，Geometry = 2000，AlphaTest = 2450，Transparent = 3000，最后Overlay = 4000。

在我们实际设置Queue值时，不进能使用上面的几个预定义值，我们也可以指定自己的Queue值，写成：

```
"Queue" = "Transparent+100"
```
表示在一个Transparent之后100的Queue上进行调用。通过调整Queue值，我们可以确保某些物体一定在另一些物体之前或者之后渲染，这个技巧有时候很有用处。

### LOD
LOD（Level of Detail），多细节层次。在游戏场景中，根据摄像机与模型的距离，来决定显示哪一个模型，一般距离近的时候显示高精度多细节模型，距离远的时候显示低精度低细节模型。

在我们的Shader程序中我们令它的值为200（其实这是Unity内建着色器的设定值）：

```
LOD 200
```
这个数值决定了我们能用什么样的Shader。在Unity的Quality Setting中我们可以设定允许的最大LOD（[设定方法戳我](https://blog.csdn.net/huutu/article/details/52106468)），当设定的LOD小于SubShader所指定的LOD时，这个SubShader将不可用。Unity内建Shader定义了一组LOD的数值，我们在实现自己的Shader的时候可以将其作为参考来设定自己的LOD数值，这样在之后调整根据设备图形性能来调整画质时可以进行比较精确的控制。
* VertexLit及其系列 = 100
* Decal, Reflective VertexLit = 150
* Diffuse = 200
* Diffuse Detail, Reflective Bumped Unlit, Reflective Bumped VertexLit = 250
* Bumped, Specular = 300
* Bumped Specular = 400
* Parallax = 500
* Parallax Specular = 600

### Shader本体
在开始看SurfaceShader的模板之前，需要强调一句：
> SurfaceShader不能使用Pass，一使用就报错，我们直接在SubShader中实现和填充代码就可以了！

接下来看看Standard Surface Shader模板里的Shader本体：

```
        //******开始CG着色器语言编写模块******
		CGPROGRAM
		// Physically based Standard lighting model, and enable shadows on all light types
		//编译指令：告知编译器表面着色器函数的名称为surf
		//Standard表示光照模型为Unity标准版光照模型
		//fullforwardshadows表示在正向渲染路径中支持所有阴影类型
		#pragma surface surf Standard fullforwardshadows

		// Use shader model 3.0 target, to get nicer looking lighting
		//编译指令：指定着色器编译目标为Shader Model 3.0
		#pragma target 3.0

		sampler2D _MainTex;

		//表面输入结构体
		struct Input {
			float2 uv_MainTex;//纹理坐标
		};

		half _Glossiness;
		half _Metallic;
		fixed4 _Color;

		// Add instancing support for this shader. You need to check 'Enable Instancing' on materials that use the shader.
		// See https://docs.unity3d.com/Manual/GPUInstancing.html for more information about instancing.
		// #pragma instancing_options assumeuniformscaling
		UNITY_INSTANCING_BUFFER_START(Props)
			// put more per-instance properties here
		UNITY_INSTANCING_BUFFER_END(Props)

		//--------------------------------【表面着色函数】-----------------------------  
		//输入：表面输入结构体  
		//输出：Unity内置的SurfaceOutputStandard结构体  
		//SurfaceOutputStandard原型如下：  
		/*
			struct SurfaceOutputStandard
			{
				fixed3 Albedo;                  // 漫反射颜色
				fixed3 Normal;                  // 切线空间法线
				half3 Emission;                 //自发光
				half Metallic;                           // 金属度；取0为非金属, 取1为金属
				half Smoothness;             // 光泽度；取0为非常粗糙, 取1为非常光滑
				half Occlusion;                 // 遮挡(默认值为1)
				fixed Alpha;                      // 透明度
			};
		*/
		//---------------------------------------------------------------------------------  
		void surf (Input IN, inout SurfaceOutputStandard o) {
			// Albedo comes from a texture tinted by color
			//漫反射颜色为主纹理对应的纹理坐标，并乘以主颜色
			fixed4 c = tex2D (_MainTex, IN.uv_MainTex) * _Color;
			//将准备好的颜色的rgb分量作为漫反射颜色
			o.Albedo = c.rgb;
			// Metallic and smoothness come from slider variables
			//金属度取自属性值
			o.Metallic = _Metallic;
			//光泽度取自属性值
			o.Smoothness = _Glossiness;
			//将准备好的颜色的alpha分量作为Alpha分量值
			o.Alpha = c.a;
		}
		//*****结束CG着色器语言编写模块******
		ENDCG
```
首先是CGPROGRAM，这是一个开始标记，表明从这里开始是一段CG程序（我们在写Unity的Shader时用的是CG/HLSL语言）。最后一行的ENDCG与CGPROGRAM是对应的，表明CG程序到此结束。

接下来是一个编译指令：

```
        // Physically based Standard lighting model, and enable shadows on all light types
        //编译指令：告知编译器表面着色器函数的名称为surf
		//Standard表示光照模型为Unity标准版光照模型
		//fullforwardshadows表示在正向渲染路径中支持所有阴影类型
		#pragma surface surf Standard fullforwardshadows
```
其中：
* surface 表示该着色器为表面着色器
* surf 告知编译器表面着色器的函数的名称为surf
* Standard 表示光照模型为Unity标准版光照模型
* fullforwardshadows 表示在正向渲染路径中支持所有阴影类型

然后又是一个编译指令：

```
        //编译指令：指定着色器编译目标为Shader Model 3.0
		#pragma target 3.0
```
表示着色器编译目标为Shader Model 3.0

接下来是：

```
//主贴图变量声明
sampler2D _MainTex;
```
sampler2D是个啥？

其实在CG中，sampler2D就是和texture所绑定的一个数据容器接口。

等等..这个说法还是太复杂了，简单理解的话，所谓加载以后的texture（贴图）说白了不过是一块内存存储的，使用了RGB（也许还有A）通道，且每个通道8bits的数据。而具体地想知道像素与坐标的对应关系，以及获取这些数据，我们总不能一次一次去自己计算内存地址或者偏移，因此可以通过sampler2D来对贴图进行操作。

更简单地理解，sampler2D就是GLSL中的2D贴图的类型，相应的，还有sampler1D，sampler3D，samplerCube等等格式。

我们在之前的Properties里不是已经声明了_MainTex是贴图了吗，为什么还要在这里重复声明一次呢？

答案是我们用来实例的这个shader其实是由两个相对独立的块组成的，外层的属性声明，回滚等等是Unity可以直接使用和编译的ShaderLab；而现在我们是在CGPROGRAM...ENDCG这样一个代码块中，这是一段CG程序。对于这段CG程序，要想访问在Properties中所定义的变量的话，必须使用和之前变量相同的名字进行声明。于是其实sampler2D _MainTex;做的事情就是再次声明并链接了_MainTex，使得接下来的CG程序能够使用这个变量。

终于可以继续了。接下来是一个struct结构体。相信大家对于结构体已经很熟悉了，我们先跳过之，直接看下面的的surf函数。上面的#pragma段已经指出了我们的着色器代码的方法的名字叫做surf，那没跑儿了，就是这段代码是我们的着色器的工作核心。我们已经说过不止一次，着色器就是给定了输入，然后给出输出进行着色的代码。CG规定了声明为表面着色器的方法（就是我们这里的surf）的参数类型和名字，因此我们没有权利决定surf的输入输出参数的类型，只能按照规定写。这个规定就是第一个参数是一个Input结构，第二个参数是一个inout的SurfaceOutput结构。

它们分别是什么呢？Input其实是需要我们去定义的结构，这给我们提供了一个机会，可以把所需要参与计算的数据都放到这个Input结构中，传入surf函数使用；SurfaceOutput是已经定义好了里面类型输出结构，但是一开始的时候内容暂时是空白的，我们需要向里面填写输出，这样就可以完成着色了。先仔细看看INPUT吧，现在可以跳回来看上面定义的INPUT结构体了：

```
//表面输入结构体
	struct Input {
		float2 uv_MainTex;//纹理坐标
	};
```
作为输入的结构体必须命名为Input，这个结构体中定义了一个float2的变量…你没看错我也没打错，就是float2，表示浮点数的float后面紧跟一个数字2，这又是什么意思呢？其实没什么魔法，float和vec都可以在之后加入一个2到4的数字，来表示被打包在一起的2到4个同类型数。比如下面的这些定义：

```
//Define a 2d vector variable
vec2 coordinate;
//Define a color variable
float4 color;
//Multiply out a color
float3 multipliedColor = color.rgb * coordinate.x;
```
在访问这些值时，我们即可以只使用名称来获得整组值，也可以使用下标的方式（比如.xyzw，.rgba或它们的部分比如.x等等）来获得某个值。在这个例子里，我们声明了一个叫做uv_MainTex的包含两个浮点数的变量。

如果你对3D开发稍有耳闻的话，一定不会对uv这两个字母感到陌生。UV mapping的作用是将一个2D贴图上的点按照一定规则映射到3D模型上，是3D渲染中最常见的一种顶点处理手段。在CG程序中，我们有这样的约定，在一个贴图变量（在我们例子中是_MainTex）之前加上uv两个字母，就代表提取它的uv值（其实就是两个代表贴图上点的二维坐标 ）。我们之后就可以在surf程序中直接通过访问uv_MainTex来取得这张贴图当前需要计算的点的坐标值了。

然后是一些变量的声明：

```
        //光泽度变量声明
		half _Glossiness;
		//金属度变量声明
		half _Metallic;
		//颜色变量声明
		fixed4 _Color;
```
这里用到几个新的变量类型：half和fixed，其实他们和float和double一样都表示浮点数，只不过精度不同。这些精度将决定计算结果的数值范围，精度范围如下表：
![image](https://img-blog.csdn.net/20170622135811237?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvaHVhaXhpYW9uaXU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)
上面的精度范围并不是绝对正确的，在不同的平台和GPU上，可能会有所不同。

尽可能使用精度较低的类型，因为这可以优化Shader的性能，这一点在移动平台上尤其重要。从它们大体的值域范围来看，我们可以使用fixed类型来存储颜色和单位矢量，如果要存储更大范围的数据可以选择half 类型，最差情况下再选择使用float。

最后，我们来看看surf函数：

```
//--------------------------------【表面着色函数】-----------------------------  
		//输入：表面输入结构体  
		//输出：Unity内置的SurfaceOutputStandard结构体  
		//SurfaceOutputStandard原型如下：  
		/*
			struct SurfaceOutputStandard
			{
				fixed3 Albedo;                  // 漫反射颜色
				fixed3 Normal;                  // 切线空间法线
				half3 Emission;                 //自发光
				half Metallic;                           // 金属度；取0为非金属, 取1为金属
				half Smoothness;             // 光泽度；取0为非常粗糙, 取1为非常光滑
				half Occlusion;                 // 遮挡(默认值为1)
				fixed Alpha;                      // 透明度
			};
		*/
		//---------------------------------------------------------------------------------  
		void surf (Input IN, inout SurfaceOutputStandard o) {
			// Albedo comes from a texture tinted by color
			//漫反射颜色为主纹理对应的纹理坐标，并乘以主颜色
			fixed4 c = tex2D (_MainTex, IN.uv_MainTex) * _Color;
			//将准备好的颜色的rgb分量作为漫反射颜色
			o.Albedo = c.rgb;
			// Metallic and smoothness come from slider variables
			//金属度取自属性值
			o.Metallic = _Metallic;
			//光泽度取自属性值
			o.Smoothness = _Glossiness;
			//将准备好的颜色的alpha分量作为Alpha分量值
			o.Alpha = c.a;
		}
```
surf函数有两个参数，第一个是Input，在计算输出时Shader会多次调用surf函数，每次给入一个贴图上的点坐标，来计算输出。第二个参数是一个可写的SurfaceOutputStandard，SurfaceOutputStandard是预定义的输出结构，我们surf函数的目标就是根据输入把这个输出结构填上。SurfaceOutputStandard结构体的定义如下：

```
//SurfaceOutputStandard原型如下： 
		struct SurfaceOutputStandard
		{
			fixed3 Albedo;                  // 漫反射颜色
			fixed3 Normal;                  // 切线空间法线
			half3 Emission;                 //自发光
			half Metallic;                           // 金属度；取0为非金属, 取1为金属
			half Smoothness;             // 光泽度；取0为非常粗糙, 取1为非常光滑
			half Occlusion;                 // 遮挡(默认值为1)
			fixed Alpha;                      // 透明度
		};
```
在Surf函数中，我们执行以下赋值：

```
            // Albedo comes from a texture tinted by color
			//漫反射颜色为主纹理对应的纹理坐标，并乘以主颜色
			fixed4 c = tex2D (_MainTex, IN.uv_MainTex) * _Color;
			//将准备好的颜色的rgb分量作为漫反射颜色
			o.Albedo = c.rgb;
			// Metallic and smoothness come from slider variables
			//金属度取自属性值
			o.Metallic = _Metallic;
			//光泽度取自属性值
			o.Smoothness = _Glossiness;
			//将准备好的颜色的alpha分量作为Alpha分量值
			o.Alpha = c.a;
```
可见我们通过计算或者用属性值赋值的方式得到输出的SurfaceOutputStandard类型的变量的每个值（有一些是默认值）。

# 结语
经过了Standard Surface Shader模板的学习，现在已经初步掌握了一些简单Shader写法，接下来我将会接着学习其他Shader模板，并尝试用unity Shader实现一些简单的光照模型之类的效果~





