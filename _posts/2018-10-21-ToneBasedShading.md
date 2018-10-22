---
layout:     post
title:      ToneBasedShading
subtitle:   基于色调的光照模型
date:       2018-10-21
author:     Richbabe
header-img: img/okami.jpg
catalog: true
tags:
    - NPR实验室
---

> 中山大学 数据科学与计算机学院 软件工程数字媒体 洪鹏圳

---


# 参考论文
> A Non-Photorealistic Lighting Model For Automatic Technical illustration

## Abstract
在1998年，Gooch等人发了一篇论文中提出并实现了基于色调的光照模型。基于色调的光照模型主要是通过亮度和色调的变化来指示物体表面方向：使轮廓和高光点极暗或极亮并让光照模型仅应用在中间色调上来让轮廓和高光点保持可见性。同时在渲染金属模型时基于色调的光照模型也会做出相应的变化。

基于色调的光照模型比传统的Phong光照模型更能模拟人们手绘插画技术。

## Introduction
在该论文中，我们采用一种基于冷色调到暖色调的变化（类似Figure 1中的画作）的算法，同时加上黑色轮廓线和高光点来实现模拟插画风格的渲染。
![image](https://github.com/Richbabe/NPR_Lab/blob/master/Image/%E7%AC%AC%E4%B8%80%E5%91%A8%E5%91%A8%E6%8A%A5/%E8%AE%BA%E6%96%87%E6%88%AA%E5%9B%BE/lw1.png?raw=true)

## Related Work
在计算机图形学领域里，有许多人也曾经做过用CG模拟插画风格的相关工作：
* Saito和Takahashi[19]使用了一些方法来展示物体的几何属性，但其渲染效果不遵循插画的特征。
* Seligmann和Feiner[21]提出了一套自动生成绘画风格图像的系统，但该系统首要关注的是要画什么，其次才是关注视觉效果，这与我们首要关注视觉效果的工作目标有所出入。
* Dooley和Cohen[7]所做的工作与我们很相近。他们提出了一套由用户自定义目标图像的特征，比如线条宽度、透明度和生成轮廓的条件，并生成对应的图像。但这种前期参数配置显得比较繁琐，我们的基于色调的光照模型需要更简单和更加自动化一点。
* Williams[25]提出了用暖色调到冷色调来近似模拟全局光照并绘制出物体的高光特征。

## Illustration Techiques
要模拟插画风格，我们首先要熟知插画的特征。根据对插画师的探访和画作的研究我们总结了插画所具有的特征：
* 轮廓，在不同表面的交接或者表面的不连续处，我们需要用黑色曲线来表示这些地方。
* 非金属物体用强度不同于黑和白的暖色或者冷色来表示表面发现，由单一光源提供白色高光点。
* 需要将金属的各向异性绘制出来。

在绝大多数NPR技术中，物体的表面形状信息比反射率信息（比如PBR中的粗糙度、金属度等）更为重要，因此色调的变化往往被用来指明表面朝向而不是反射率。

Tuffe[23]在他的书里指出一个简单的低动态范围着色模型（low dynamic-range shading model）需要遵循以下原则：

```
使得所有视觉差异尽可能细微，但仍清晰有效
```
这个原则也解释了为什么素描画与黑白画较为相近，和彩色画大相径庭。这是因为彩色画通过色调的变化来表现表面的朝向，通过一种更加细微却又十分清晰有效的方法表现一幅画作的视觉差异。

# Automatic Lighting Model

## Traditional Shading of Matte Objects
要渲染一个卡通风格的物体，我们除了要加上轮廓和高光点外，最重要的是要渲染这个物体的表面。传统的漫反射着色令漫反射强度与光线方向和表面法线之间的夹角的余弦值成正比：


$$
I = k_dk_a + k_dmax(0,\vec{l} \cdot \vec{n})
$$
（公式1）

其中I是物体表面片段的颜色的RGB值，Kd是漫反射系数，ka是环境光的RGB值，**l**是光线方向的单位向量，**n**是顶点法线的单位向量。

![image](https://github.com/Richbabe/NPR_Lab/blob/master/Image/%E7%AC%AC%E4%B8%80%E5%91%A8%E5%91%A8%E6%8A%A5/%E8%AE%BA%E6%96%87%E6%88%AA%E5%9B%BE/lw2.png?raw=true)
在Figure3中，kd=1，ka=0.可以看到这张图片在暗处没有把物体的形状和材质属性表现出来。比如在爪子那里丢失了许多的细节，同时在明亮处同样也出现了部分细节丢失的现象。

![image](https://github.com/Richbabe/NPR_Lab/blob/master/Image/%E7%AC%AC%E4%B8%80%E5%91%A8%E5%91%A8%E6%8A%A5/%E8%AE%BA%E6%96%87%E6%88%AA%E5%9B%BE/lw3.png?raw=true)
在Figure4中给出了Figure3物体的轮廓和高光部分，当把Figure4中的轮廓和高光部分加入Figure3中时，我们并不能得到想要的效果。这是因为黑暗区域太暗导致我们不能看清轮廓，而亮处的高光点也因太亮而看不到。

要在应用了公式（1）的渲染效果中可以看清物体的轮廓和高光点，我们分为以下两个步骤：
1. 要看到轮廓线，我们可以增大ka的值（即提高环境光的亮度）直到暗处可以清晰看到黑色轮廓。
2. 为了看到高光点，我们可以减小kd的值直到在亮处可以清晰看到高光点。

![image](https://github.com/Richbabe/NPR_Lab/blob/master/Image/%E7%AC%AC%E4%B8%80%E5%91%A8%E5%91%A8%E6%8A%A5/%E8%AE%BA%E6%96%87%E6%88%AA%E5%9B%BE/lw4.png?raw=true)
在使用了这两种方法后，在Figure5中，kd=0.5，ka=0.1，这是目前使用单一光源和传统着色模型渲染效果最好的非彩色图片，但这仍无法很好地表达一些细节，比如在图片底部的爪子处仍有一些细节丢失了。

## Tone-based Shading of Matte Objects
画家通常使用画笔给作品添加色彩和亮度变化：
* 给颜色加点黑色的操作称作 **shades**
* 给颜色加点白色的操作称作 **tints**
* 给颜色加点灰色的操作称作 **tones**

总的来说，给某种颜色添加一种颜色使其成为另一种颜色就被称为色调(*tones*)。这些色调通常只会导致色彩不同而不会导致亮度不同。因此当受限于很小的亮度变化范围时，色调对画家有着重要意义。

另一种颜色的性质为色温(*temperature* of the color)。颜色根据色温不同分为暖色（红、橙、紫）和冷色（蓝、紫、绿）。这一定义来自于人类对于颜色的自然感知，因为不同温度的光照射在物体表面所显示的颜色是不同的。

我们根据上述颜色的性质和插画常用技术作为我们基于色调光照模型的基础。

我们可以使用公式（1）中的变量**l**·**n**作为参数来混合暖色和冷色得到最终颜色值：

```math
I = \frac{1 + \vec{l} \cdot \vec{n}}{2}k_{cool} +(1 - \frac{1 + \vec{l} \cdot \vec{n}}{2})k_{warm}            
```
(公式2)

注意到**l**·**n**的范围是[-1,1]，因为光照方向一般在上方，所以我们假定光线方向**l**垂直于实现方向。

![image](https://github.com/Richbabe/NPR_Lab/blob/master/Image/%E7%AC%AC%E4%B8%80%E5%91%A8%E5%91%A8%E6%8A%A5/%E8%AE%BA%E6%96%87%E6%88%AA%E5%9B%BE/l25.png?raw=true)
Figure6中的图片通过极小的亮度变化和明显的色彩变化传递了深度信息。可以看到轮廓和高光点很明显，但是缺乏的强烈的冷色调到暖色调的转换和亮度变化使得爪子的细节很细微，且颜色不太自然。

为了自动化这种色彩偏移技术并增加一些亮度变化，我们可以加入两种颜色变化过程：蓝色到黄色的色调变化和黑色到物体自身颜色的色调变化。我们最终的光照模型是这两种变化的线性组合。

加入蓝色到黄色的色调变化是为了模拟除了物体自身颜色之外的冷色调到暖色调的转换。其中：

```
蓝色指纯饱和蓝色：Kblue=(0,0,b),b∈[0,1]
黄色指纯饱和黄色：Kyellow=(y,y,0),y∈[0,1]
```
这两种颜色是和物体自身漫反射系数kd无关的，与kd相关的是黑色到物体颜色的自身变化，因此我们将这两种颜色变化组合起来得到Kcool和Kwarm：

```math
k_{cool} = k_{blue} + \alpha k_d
```
```math
k_{warm} = k_{yellow} + \beta k_d
```
公式(3)

将公式（3）带入公式（2）可得到我们最终的基于色调的光照模型有4个自由变量b、y、α和β。其中b和y将决定色温的变化强度；α和β将决定物体颜色和亮度的变化强度。

![image](https://github.com/Richbabe/NPR_Lab/blob/master/Image/%E7%AC%AC%E4%B8%80%E5%91%A8%E5%91%A8%E6%8A%A5/%E8%AE%BA%E6%96%87%E6%88%AA%E5%9B%BE/lw6.png?raw=true)
Figure2展示了一个纯红物体应用了我们基于色调的光照模型后的最终色调。

![image](https://github.com/Richbabe/NPR_Lab/blob/master/Image/%E7%AC%AC%E4%B8%80%E5%91%A8%E5%91%A8%E6%8A%A5/%E8%AE%BA%E6%96%87%E6%88%AA%E5%9B%BE/lw7.png?raw=true)
Figure7中，我们可以看到边缘线、高光点、爪子处的细节都清晰可见。

![image](https://github.com/Richbabe/NPR_Lab/blob/master/Image/%E7%AC%AC%E4%B8%80%E5%91%A8%E5%91%A8%E6%8A%A5/%E8%AE%BA%E6%96%87%E6%88%AA%E5%9B%BE/lw8.png?raw=true)
Figure8中，我们可以看到更明显的色温和亮度变化。

![image](https://github.com/Richbabe/NPR_Lab/blob/master/Image/%E7%AC%AC%E4%B8%80%E5%91%A8%E5%91%A8%E6%8A%A5/%E8%AE%BA%E6%96%87%E6%88%AA%E5%9B%BE/lw9.png?raw=true)
在Figure9中，我们可以看到基于色调的光照模型在保留了物体自身颜色的同时，将轮廓和高光点清晰地渲染出来。

## Shading of Medal Objects
在实践中，插画家通常用明暗带的交界来表示铣削过的金属物体（铣削过的物体通常会有各向异性的镜面反射）。为了模拟一个铣削过的金属，我们绘制了一组20条条纹的贴图，这些条纹方向沿着强度变化曲线的最大斜率延伸。条纹的强度在0.0和0.5之间取随机值，最接近光源的条纹为全白色。

![image](https://github.com/Richbabe/NPR_Lab/blob/master/Image/%E7%AC%AC%E4%B8%80%E5%91%A8%E5%91%A8%E6%8A%A5/%E8%AE%BA%E6%96%87%E6%88%AA%E5%9B%BE/lw10.png?raw=true)
在Figure10中给出了由Phong光照模型，没有轮廓的金属模型、带轮廓的金属模型、结合冷色调到暖色调变化的金属光照模型各自的渲染效果。

可以看到在图（c）中物体的金属感比图（a）好很多，图（d）虽然没有图（c）那么有说服力，但是当金属和非金属一起渲染时会显得更加自然。

## Approximation to new model
如果在使用了Phong光照模型的高级图形包中，由于将内部算法封装了起来，我们并不能直接应用基于色调的光照模型。这时我们可以用Phong光照模型为基础来近似我们的光照模型，在许多图形接口（比如OpengGL）中，我们可以将光源颜色设为负值。我们可以通过以下方法来近似公式（2）：

使用两个光源，一个光源强度为(kwarm - kcool) / 2，方向为**l**；另一个光源强度为(kcool - kwarm) / 2，方向为-**l**；环境光强度为(kcool + kwarm) / 2

该近似模型假定物体自身颜色为白色，同时需要关闭镜面反射。这是因为负值的蓝色光会产生很糟糕的高光效果，我们可以在颜色缓冲中添加高光。

![image](https://github.com/Richbabe/NPR_Lab/blob/master/Image/%E7%AC%AC%E4%B8%80%E5%91%A8%E5%91%A8%E6%8A%A5/%E8%AE%BA%E6%96%87%E6%88%AA%E5%9B%BE/lw11.png?raw=true)
Figure11为近似光照模型的渲染效果与其他光照模型渲染之间的比较。

## Future work and conclusion
目前基于色调的光照模型的缺点是当要渲染一整个插画风格的场景时不能自适应不同模型的不同性质。这项工作虽充满挑战但却十分有趣。

# 论文实现
在这里我使用Unity的Unity shader实现了非金属的基于色调的光照模型。

## 轮廓线绘制
在《Real-Time Rendering，third edition》中作者提出了五种类型的轮廓线绘制方法。在这里，我使用较为简单的过程式几何轮廓线渲染。这种方法的核心是使用两个Pass渲染，第一个Pass渲染背面的面片，并使用某些技术让它的轮廓可见；第二个Pass再正常渲染正面的面片。

在第一个Pass中，我们会使用轮廓线颜色渲染整个背面的面片，并在视角空间下把模型模型顶点沿着法线方向向外扩张一段距离，以此来让背部轮廓线可见。具体的代码如下：

```
//渲染轮廓线的Pass,该Pass只渲染背面的三角面片（过程式几何轮廓线渲染）
		Pass{
			NAME "OUTLINE" //该Pass的名称，后面只需调用名字即可绘制轮廓不用重写该Pass

			Cull Front //剔除正面，只渲染背面

			CGPROGRAM

			#pragma vertex vert
			#pragma fragment frag

			#include "UnityCG.cginc"

			//定义Properties属性
			float _Outline;
			fixed4 _OutlineColor;

			//顶点着色器输入
			struct a2v {
				float4 vertex : POSITION;
				float3 normal : NORMAL;
			};

			//顶点着色器输出
			struct v2f {
				float4 pos : SV_POSITION;
			};

			//定义顶点着色器
			v2f vert(a2v v) {
				v2f o;

				float4 pos = mul(UNITY_MATRIX_MV, v.vertex);//将坐标从模型空间转换到视角空间
				float3 normal = mul((float3x3)UNITY_MATRIX_IT_MV, v.normal);//将法线从模型空间转换到视角空间

				normal.z = -0.5;//让法线的z分量等于一个定值，避免背面扩张后的顶点挡住正面的面片

				pos = pos + float4(normalize(normal), 0) * _Outline;//将顶点沿法线方向扩张

				o.pos = mul(UNITY_MATRIX_P, pos);//将顶点从视角空间转换到裁剪空间

				return o;
			}

			//定义片段着色器
			float4 frag(v2f i) : SV_Target{
				return float4(_OutlineColor.rgb,1);//使用轮廓线颜色渲染整个背面
			}

			ENDCG
		}
```
需要注意的是，如果直接使用顶点法线进行扩展，对于一些内凹的模型，就可能发生背面面片遮挡正面面片的情况。为了尽可能防止出现这样的情况，在扩张背面顶点之前，我们首先对顶点法线的z分量进行处理，使他们等于一个定值，然后把法线归一化后再对顶点进行扩张。这样的好处在于，扩展后的表面更加扁平化，从而降低了遮挡正面面片的可能性，所以代码如下：

```
normal.z = -0.5;//让法线的z分量等于一个定值，避免背面扩张后的顶点挡住正面的面片

pos = pos + float4(normalize(normal), 0) * _Outline;//将顶点沿法线方向扩张
```

## 用基于色调的光照模型渲染物体正面
由于基于色调的光照模型和Phong光照模型只有漫反射部分不同，因此环境光和镜面反射部分我们照搬Phong光照模型，我们重点来看漫反射部分：

```
fixed diff = dot(worldNormal, worldLightDir);
diff = (diff * 0.5 + 0.5);//计算漫反射分量

fixed3 k_d = c.rgb * _Color.rgb;//计算物体漫反射系数

//计算基于色调的光照模型的自由变量
fixed3 k_blue = fixed3(0, 0, _Blue);
fixed3 k_yellow = fixed3(_Yellow, _Yellow, 0);
fixed3 k_cool = k_blue + _Alpha * k_d;
fixed3 k_warm = k_yellow + _Beta * k_d;

//计算漫反射光照强度
fixed3 diffuse = _LightColor0.rgb * (diff * k_cool + (1 - diff) * k_warm);
```
为了让效果更加显著，我增加多了一个Pass将效果叠加了一下。

## 运行效果
在这里我找了几个模型来测试基于色调的光照模型，参数为：物体自身颜色为白色，b = 0.55，y = 0.3，α = 0.25，β = 0.5。

![image](https://github.com/Richbabe/NPR_Lab/blob/master/Image/%E7%AC%AC%E4%B8%80%E5%91%A8%E5%91%A8%E6%8A%A5/%E8%BF%90%E8%A1%8C%E7%BB%93%E6%9E%9C%E6%88%AA%E5%9B%BE/jg1.png?raw=true)

![image](https://github.com/Richbabe/NPR_Lab/blob/master/Image/%E7%AC%AC%E4%B8%80%E5%91%A8%E5%91%A8%E6%8A%A5/%E8%BF%90%E8%A1%8C%E7%BB%93%E6%9E%9C%E6%88%AA%E5%9B%BE/jg2.png?raw=true)

![image](https://github.com/Richbabe/NPR_Lab/blob/master/Image/%E7%AC%AC%E4%B8%80%E5%91%A8%E5%91%A8%E6%8A%A5/%E8%BF%90%E8%A1%8C%E7%BB%93%E6%9E%9C%E6%88%AA%E5%9B%BE/jg3.png?raw=true)

![image](https://github.com/Richbabe/NPR_Lab/blob/master/Image/%E7%AC%AC%E4%B8%80%E5%91%A8%E5%91%A8%E6%8A%A5/%E8%BF%90%E8%A1%8C%E7%BB%93%E6%9E%9C%E6%88%AA%E5%9B%BE/jg4.png?raw=true)

可以看到效果和Figure8差不多，我尝试着把公式（2）修改成：

```
//计算漫反射光照强度
//fixed3 diffuse = _LightColor0.rgb * (diff * k_cool + (1 - diff) * k_warm);
fixed3 diffuse = _LightColor0.rgb * (diff * k_warm + (1 - diff) * k_cool);
```
并换了个自身颜色，得到的运行效果为：
![image](https://github.com/Richbabe/NPR_Lab/blob/master/Image/%E7%AC%AC%E4%B8%80%E5%91%A8%E5%91%A8%E6%8A%A5/%E8%BF%90%E8%A1%8C%E7%BB%93%E6%9E%9C%E6%88%AA%E5%9B%BE/jg6.png?raw=true)

看着好像效果好了一点。

# Reference
* Amy Gooch, Bruce Gooch, Peter Shirley, Elaine Cohen. A non-photorealistic lighting model for automatic technical illustration
* Tomas Akenine-Moller , Eric Haines , Naty Hoffman. 《Real-Time Rendering 3rd》

# 结语
本博客的代码和资源均可在我的[github](https://github.com/Richbabe/NPR_Lab)上下载，别忘了点颗Star哟！






