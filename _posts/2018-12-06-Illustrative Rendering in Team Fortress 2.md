---
layout:     post
title:      Illustrative Rendering in Team Fortress 2
subtitle:   《军团要塞2》中的卡通渲染技术
date:       2018-12-06
author:     Richbabe
header-img: img/okami.png
catalog: true
tags:
    - NPR实验室
---

> 中山大学 数据科学与计算机学院 软件工程数字媒体 洪鹏圳

---

# 参考论文
> Illustrative Rendering in Team Fortress 2[1]

# Abstract
在2007年，Value公司发行了一款名为《军团要塞2》的游戏。在该游戏中，Value公司采取了一种全新的卡通渲染技术，使得《军团要塞2》因画质好而火了起来。因此Value公司写了这篇论文，并介绍了在这个游戏里面使用的Shader技术以及他们如何使用这种技术来和美术部门配合。同时，为了得到更好的效果，他们的Shading技术还使用了边缘高光技术（rim highlights）和在亮度、色调上的变化来快速传递不同模型的几何信息。

![image](https://github.com/Richbabe/NPR_Lab/blob/master/Image/%E7%AC%AC%E4%B8%83%E5%91%A8%E5%91%A8%E6%8A%A5/%E8%AE%BA%E6%96%87%E6%88%AA%E5%9B%BE/Figure1.png?raw=true)

Figure1图片中展示了《军团要塞2》中人物的概念图和游戏实况图。

# Introduction
在此之前，大部分NPR技术都只能在一个给定的特定模型下生效。但是这篇论文介绍了一系列可交互的渲染技术，而且还添加了在光照和色彩方面的变化，在提高模型适应性的同时让玩家可以在不同的光照条件下直观地区分不同的场景。

在《军团要塞2》中，他们选用了20世纪中期几位著名插画师的艺术风格。这种风格的插画角色使用了强烈而且鲜明的轮廓，并且着重强调衣服的褶皱。同时，他们通过使用边缘高光来代替黑色轮廓从而强调了物体的内部形状，如Figure1(a)那样。

这篇论文的主要贡献是整理这种商业风格到游戏的转换。在技术上，实现了一种diffuse light warping function的插画式渲染函数，rim lighting的公式，以及一个总体来说在真实感和非真实感之间获得平衡的shading技术。

# Related Work
非真实感渲染的风格虽然变化多样，但都是源自于真实世界的艺术风格，前提是这些技术都是由人类发展而来，具有内在价值的。《军团要塞2》使用的商业插画风格是和1998年Gooch等人提出的Tone Based Shading[2]紧密相关的。在Tone Based Shading中，传统的Phong光照模型（由Phong在1975年提出）被一种由冷到暖的色彩变化所修改，来暗示在某个光源下的平面方向。因此，非常明亮和黑暗区域分别被保留为高光和边缘轮廓。相比于传统的光照模型，在一些比较差的光照条件下，这种方法渲染出来的3D模型更加清晰。有关Tone Based Shading的具体细节可看我这篇博客：
[ToneBasedShading-基于色调的光照模型](http://richbabe.top/2018/10/21/ToneBasedShading/)

# Commercial Illustration Techniques
《军团要塞2》的技术人员根据20世纪早期的插画家J. C. Leyendecker, Dean Cornwell , Norman Rockwell 以及他们美术部门自己的原画定义了《军团要塞2》中的风格：
* Shading遵循一个由暖倒冷的色调转换。阴影是趋向于冷色调而不是黑色。
* 在给定光源生成的明暗交界处，饱和度增加。交替处通常是骗红色。
* 高频度出现的细节尽可能省略。
* 对于角色的内部细节，像衣服的褶皱等，重复绘制轮廓线条。
* 在轮廓处使用边缘高光，而非深色的轮廓线进行强调。

基于以上的原则，他们进行了美术资源的创建和实时着色算法的实现。

下一节将阐述美术资源的创建，在第五节将介绍技术细节的实现。

# Creating Art Assets
本节介绍3D人物和场景模型的建模，以及生成贴图时遵循的规范。

## Character Modeling
在像《军团要塞2》这样的多人战斗游戏中，我们需要在各种距离和视角上能快速识别出不同的人物，以便做出不同的应对。在《军团要塞2》中，英雄类型——Demo、Engineer、Heavy、Medic、Pyro、Spy、Sniper、Solider和Scout——尤其重要。因此，这九种类型的轮廓要专门设计成差别很大的样子，如下图所示。

![image](https://github.com/Richbabe/NPR_Lab/blob/master/Image/%E7%AC%AC%E4%B8%83%E5%91%A8%E5%91%A8%E6%8A%A5/%E8%AE%BA%E6%96%87%E6%88%AA%E5%9B%BE/Figure2.png?raw=true)

由鞋子、帽子和衣服褶皱确定的身体比例，武器以及轮廓线条给予每一个角色独特的轮廓。在角色被挡住的内部区域，衣服褶皱重复了轮廓线条以强调轮廓，正如我们在商业插画里观察到一样。在第5节中，用于这些角色的着色算法补足了我们的模型来提高阴影比率。

## World Modeling
《军团要塞2》中的场景颜色和两个队伍自身颜色有关（蓝队和红队）。因此为两个队伍定义了对比鲜明的场景模型。红队基本使用暖色调，天然材质和棱角分明的地图模型。而蓝队则主要使用冷色调，工业材质以及正交形状的地图模型，如下图所示。

![image](https://github.com/Richbabe/NPR_Lab/blob/master/Image/%E7%AC%AC%E4%B8%83%E5%91%A8%E5%91%A8%E6%8A%A5/%E8%AE%BA%E6%96%87%E6%88%AA%E5%9B%BE/Figure3.png?raw=true)

尽管实际建模有更多的细节，但他们尽可能避免复杂或者几何失衡的模型，因为那样会增加不必要的视觉噪点以及给内存造成计算负担。并且，使用重复的结构，例如桥架、电线杆、铁路轨道对我们的风格更合适，因为重复的空间感觉比表达每一个细节更重要。

## Texture Painting
在《军团要塞2》中，角色和游戏的色彩跟真实材质比，色彩饱和度和对比度更强。红蓝两队在游戏中都有各自的基调，它们的参考样色样本如下图所示。可以看出，所取的大多都是柔和的颜色。

![image](https://github.com/Richbabe/NPR_Lab/blob/master/Image/%E7%AC%AC%E4%B8%83%E5%91%A8%E5%91%A8%E6%8A%A5/%E8%AE%BA%E6%96%87%E6%88%AA%E5%9B%BE/Figure4.png?raw=true)

除了上述两种主要的红蓝两个色调，还有其他的颜色被用于一些环境道具，如灭火器、电话等。通常，用于3D场景中的贴图是印象派的，意味着它们是绘画式的并保持最小化的视觉噪声。这种风格被应用在很多动画电影中，比如宫崎骏的作品，在这些作品里，地图都用很大的笔刷绘制。对于我们的3D游戏，我们也使用同样的技术来绘制场景。

大量的环境贴图来源于手绘的反射贴图，他们内部具有少量的细节，通过使用较大的笔触来表现特定平面的触感，如下图所示。在早期的开发中，许多这些2D贴图是在画板上使用水彩和扫描完成的。后来，美术部分转而使用了真实的图片作为参考，再使用一系列的滤镜和数字笔刷来得到最终的目标贴图。

![image](https://github.com/Richbabe/NPR_Lab/blob/master/Image/%E7%AC%AC%E4%B8%83%E5%91%A8%E5%91%A8%E6%8A%A5/%E8%AE%BA%E6%96%87%E6%88%AA%E5%9B%BE/Figure5.png?raw=true)

# Interactive Character and Model Shading
本节将会介绍《军团要塞2》中非真实感渲染算法。对于《军团要塞2》中的人物角色和大多数模型，我们结合了一系列依赖视角和非依赖视角的部分，如下图所示。

![image](https://github.com/Richbabe/NPR_Lab/blob/master/Image/%E7%AC%AC%E4%B8%83%E5%91%A8%E5%91%A8%E6%8A%A5/%E8%AE%BA%E6%96%87%E6%88%AA%E5%9B%BE/Figure6_1.png?raw=true)
![image](https://github.com/Richbabe/NPR_Lab/blob/master/Image/%E7%AC%AC%E4%B8%83%E5%91%A8%E5%91%A8%E6%8A%A5/%E8%AE%BA%E6%96%87%E6%88%AA%E5%9B%BE/Figure6_@.png?raw=true)

非依赖视角的部分由一个空间变化的平行环境光部分加上一个修改后的兰伯特光照部分。依赖视角的部分由Phong高光和自定义的边缘光照部分组成。所有这些光照部分都是逐像素计算的，并且大多数材质特性，包括法线、反射率、镜面反射部分以及各种各样的遮罩均由贴图采样而得。在下面的两个章节中，我们将会讨论每一个光照部分和传统方法有什么不同，并且是如何影响我们的效果的。

## View Independent Lighting Terms（非依赖视角的光照部分）

非依赖视角的部分由一个空间变化的平行环境光部分加上一个修改后的兰伯特光照部分。它可以用下列等式（1）表示：

![image](https://github.com/Richbabe/NPR_Lab/blob/master/Image/%E7%AC%AC%E4%B8%83%E5%91%A8%E5%91%A8%E6%8A%A5/%E8%AE%BA%E6%96%87%E6%88%AA%E5%9B%BE/%E5%85%AC%E5%BC%8F1.png?raw=true)

其中，L是光源数量，i是光源索引，![image](https://github.com/Richbabe/NPR_Lab/blob/master/Image/%E7%AC%AC%E4%B8%83%E5%91%A8%E5%91%A8%E6%8A%A5/%E8%AE%BA%E6%96%87%E6%88%AA%E5%9B%BE/%E5%8F%98%E9%87%8F1.png?raw=true)是光源的颜色，![image](https://github.com/Richbabe/NPR_Lab/blob/master/Image/%E7%AC%AC%E4%B8%83%E5%91%A8%E5%91%A8%E6%8A%A5/%E8%AE%BA%E6%96%87%E6%88%AA%E5%9B%BE/%E5%8F%98%E9%87%8F2.png?raw=true)是由贴图映射得到的模型片段的反射率(albedo)，![image](https://github.com/Richbabe/NPR_Lab/blob/master/Image/%E7%AC%AC%E4%B8%83%E5%91%A8%E5%91%A8%E6%8A%A5/%E8%AE%BA%E6%96%87%E6%88%AA%E5%9B%BE/%E5%8F%98%E9%87%8F3.png?raw=true)是一个传统的光源i的unclamped Lamertian term（半兰伯特因子），常量α、β、γ分别是Lambertian term的放缩、偏移和指数部分，![image](https://github.com/Richbabe/NPR_Lab/blob/master/Image/%E7%AC%AC%E4%B8%83%E5%91%A8%E5%91%A8%E6%8A%A5/%E8%AE%BA%E6%96%87%E6%88%AA%E5%9B%BE/%E5%8F%98%E9%87%8F4.png?raw=true)是一个为每一个片段的法线n评估平行环境光部分的函数，w()则是一个将[0,1]范围的标量映射到一个RGB颜色的变形函数。

### 半兰伯特（Half Lambert）
等式（1）中第一个不同寻常的部分就是应用到![image](https://github.com/Richbabe/NPR_Lab/blob/master/Image/%E7%AC%AC%E4%B8%83%E5%91%A8%E5%91%A8%E6%8A%A5/%E8%AE%BA%E6%96%87%E6%88%AA%E5%9B%BE/%E5%8F%98%E9%87%8F3.png?raw=true)的放缩、偏移和指数部分。我们在我们的一个游戏《半条命》中使用了0.5倍的放缩，0.5倍的偏移量以及平方来防止角色在背光面时丢失形状，显示全黑（α = 0.5、β = 0.5、γ = 2）。尽管在现在的游戏大多都追求真实感，但我们仍然采用这些设置，以便让点乘的结果（范围是[-1,1]）转换到[0,1]，并且有一个令人满意的衰减区。由于兰伯特部分的0.5倍缩放和0.5大小的偏移，我们将这种技术称为“半兰伯特”。在《军团要塞2》中，我们让α = 0.5、β = 0.5但γ = 1,因为我们可以通过w()函数得到任何想要的阴影。

### 漫反射变形函数（Diffuse Warping Function）
等式（1）中第二个有趣的地方是变形函数w()。这个函数的目标是在引入商业插画中那些漂亮的明暗交接带的同时，保持半兰伯特的阴影信息。在《军团要塞2》中，该变形函数使用一个一维贴图进行查找映射，如下图所示。这种方式间接地创建了一个“硬阴影”，我们在保留所有法线对光照的变化的同时，收紧了明暗交界处从亮到暗的跳变，如Figure6(b)所示。（可以看到在明暗交替处的变化很快，因此达到了收紧的目的）。

![image](https://github.com/Richbabe/NPR_Lab/blob/master/Image/%E7%AC%AC%E4%B8%83%E5%91%A8%E5%91%A8%E6%8A%A5/%E8%AE%BA%E6%96%87%E6%88%AA%E5%9B%BE/Figure7.png?raw=true)

除了上述常见的“阴影”作用外，这个一维的变形贴图还有一些有趣的特性。首先，最右侧的值不是白色，而是只比Mid-gray稍稍明亮点的颜色。这是因为在查找映射完这张贴图后还需要将值乘以2（这样会变得更明亮了），这样使得美术人员可以更好地控制明暗。还有一点很重要，这张贴图被分成了三个部分：右侧的灰度梯度部分，左侧的冷色梯度部分以及中间较小的发红的分界线部分。这和我们之前观察的插画阴影的变化是一致的，也就是说趋于冷色而不是黑色，并且在分解处通常是有一点微红的。正如等式(1)所示，变形函数作用域半兰伯特因子后，得到一个RGB颜色，然后再和微调过的![image](https://github.com/Richbabe/NPR_Lab/blob/master/Image/%E7%AC%AC%E4%B8%83%E5%91%A8%E5%91%A8%E6%8A%A5/%E8%AE%BA%E6%96%87%E6%88%AA%E5%9B%BE/%E5%8F%98%E9%87%8F5.png?raw=true)(光源颜色)相乘，最终得到一个漫反射光照部分，如图6b所示。

### 平行环境光部分（Directional Ambient Term）
除了简单的变形漫反射光照部分的和以外，我们还应用了一个平行环境光照部分![image](https://github.com/Richbabe/NPR_Lab/blob/master/Image/%E7%AC%AC%E4%B8%83%E5%91%A8%E5%91%A8%E6%8A%A5/%E8%AE%BA%E6%96%87%E6%88%AA%E5%9B%BE/%E5%8F%98%E9%87%8F4.png?raw=true)。尽管表现方式不同，但是我们的平行环境光部分和一个环境辐射光照映射（inrradiance environment map，[Ramamoorthi and Hanrahan 2001][3]）是等同的。但是，我们没有使用一个9项的球面调谐公式（a 9 term spherical harmonic basis），而是使用了一个修改后的6项公式，我们称为“环境光盒子”（ambient cube），它使用了沿x、y、z正负方向延伸的余弦平方来实现[McTag- gart 2004][4], [Mitchell et al. 2006][5]。我们使用离线的radiosity solver计算这些环境盒子，并把它们存储在一个辐射量（an irradiance volume）中，以便实时访问[Greger et al. 1998][6]。尽管这个光照部分很简单，如图Figure6c所示，这个环境光盒子部分提供了反射光线的信息，而这是渲染游戏中的人物和其他模型的关键基础。

这些视角无关的光照部分相加起来的最终结果如图Figure6d所示，然后再和模型的颜色反射率(albedo,也叫自身漫反射颜色，图Figure6a)相乘，得到的结果如图Figure6e所示。


## View Dependent Lighting Terms（依赖视角的光照部分）

我们的依赖视角的光照部分由传统的Phong镜面高光模型和一个自定义的边缘高光部分组合而成。等式（2）如下：

![image](https://github.com/Richbabe/NPR_Lab/blob/master/Image/%E7%AC%AC%E4%B8%83%E5%91%A8%E5%91%A8%E6%8A%A5/%E8%AE%BA%E6%96%87%E6%88%AA%E5%9B%BE/%E5%85%AC%E5%BC%8F2.png?raw=true)

其中，L是光源数量，i是光源索引，![image](https://github.com/Richbabe/NPR_Lab/blob/master/Image/%E7%AC%AC%E4%B8%83%E5%91%A8%E5%91%A8%E6%8A%A5/%E8%AE%BA%E6%96%87%E6%88%AA%E5%9B%BE/%E5%8F%98%E9%87%8F1.png?raw=true)是光源i的颜色，![image](https://github.com/Richbabe/NPR_Lab/blob/master/Image/%E7%AC%AC%E4%B8%83%E5%91%A8%E5%91%A8%E6%8A%A5/%E8%AE%BA%E6%96%87%E6%88%AA%E5%9B%BE/%E5%8F%98%E9%87%8F6.png?raw=true)是被嵌入到一个纹理通道中的镜面高光遮罩（specular mask，用于决定是否使用高光），![image](https://github.com/Richbabe/NPR_Lab/blob/master/Image/%E7%AC%AC%E4%B8%83%E5%91%A8%E5%91%A8%E6%8A%A5/%E8%AE%BA%E6%96%87%E6%88%AA%E5%9B%BE/%E5%8F%98%E9%87%8F7.png?raw=true)是视角向量，![image](https://github.com/Richbabe/NPR_Lab/blob/master/Image/%E7%AC%AC%E4%B8%83%E5%91%A8%E5%91%A8%E6%8A%A5/%E8%AE%BA%E6%96%87%E6%88%AA%E5%9B%BE/%E5%8F%98%E9%87%8F8.png?raw=true)是一个由美术调整的为镜面高光而设的菲涅尔（Fresnel）因子，![image](https://github.com/Richbabe/NPR_Lab/blob/master/Image/%E7%AC%AC%E4%B8%83%E5%91%A8%E5%91%A8%E6%8A%A5/%E8%AE%BA%E6%96%87%E6%88%AA%E5%9B%BE/%E5%8F%98%E9%87%8F9.png?raw=true)是光源i的光线方向向量相对于![image](https://github.com/Richbabe/NPR_Lab/blob/master/Image/%E7%AC%AC%E4%B8%83%E5%91%A8%E5%91%A8%E6%8A%A5/%E8%AE%BA%E6%96%87%E6%88%AA%E5%9B%BE/%E5%8F%98%E9%87%8F10.png?raw=true)的反射向量，![image](https://github.com/Richbabe/NPR_Lab/blob/master/Image/%E7%AC%AC%E4%B8%83%E5%91%A8%E5%91%A8%E6%8A%A5/%E8%AE%BA%E6%96%87%E6%88%AA%E5%9B%BE/%E5%8F%98%E9%87%8F11.png?raw=true)是世界坐标系中的方向向上的单位向量（up vector），![image](https://github.com/Richbabe/NPR_Lab/blob/master/Image/%E7%AC%AC%E4%B8%83%E5%91%A8%E5%91%A8%E6%8A%A5/%E8%AE%BA%E6%96%87%E6%88%AA%E5%9B%BE/%E5%8F%98%E9%87%8F12.png?raw=true)是一个由一张纹理映射而得的镜面反射的指数部分，![image](https://github.com/Richbabe/NPR_Lab/blob/master/Image/%E7%AC%AC%E4%B8%83%E5%91%A8%E5%91%A8%E6%8A%A5/%E8%AE%BA%E6%96%87%E6%88%AA%E5%9B%BE/%E5%8F%98%E9%87%8F13.png?raw=true)是一个常量指数部分，用于控制边缘高光的宽度（越小越宽），![image](https://github.com/Richbabe/NPR_Lab/blob/master/Image/%E7%AC%AC%E4%B8%83%E5%91%A8%E5%91%A8%E6%8A%A5/%E8%AE%BA%E6%96%87%E6%88%AA%E5%9B%BE/%E5%8F%98%E9%87%8F14.png?raw=true)是另一个菲涅耳因子，用于修饰边缘高光（通常就是使用![image](https://github.com/Richbabe/NPR_Lab/blob/master/Image/%E7%AC%AC%E4%B8%83%E5%91%A8%E5%91%A8%E6%8A%A5/%E8%AE%BA%E6%96%87%E6%88%AA%E5%9B%BE/%E5%8F%98%E9%87%8F15.png?raw=true)），![image](https://github.com/Richbabe/NPR_Lab/blob/master/Image/%E7%AC%AC%E4%B8%83%E5%91%A8%E5%91%A8%E6%8A%A5/%E8%AE%BA%E6%96%87%E6%88%AA%E5%9B%BE/%E5%8F%98%E9%87%8F16.png?raw=true)是一个边缘遮罩纹理(rim mask texture),用于减弱边缘高光对模型上某些部分的影响；最后，![image](https://github.com/Richbabe/NPR_Lab/blob/master/Image/%E7%AC%AC%E4%B8%83%E5%91%A8%E5%91%A8%E6%8A%A5/%E8%AE%BA%E6%96%87%E6%88%AA%E5%9B%BE/%E5%8F%98%E9%87%8F17.png?raw=true)是一个通过使用观察方向对环境盒子(ambient cube)的取值函数。

### 多重Phong部分（Multiple Phong Term）
等式（2）的左半部分包含了使用常见的表达式——来计算Phong高光，并且使用了适当的常量以及一个菲涅尔因子对它进行调整。然而，我们在求和的内部还使用了max()函数将Phong高光和额外的Phong lobes（使用了不同的指数、菲涅尔系数以及遮罩）结合在一起。在《军团要塞2》中，![image](https://github.com/Richbabe/NPR_Lab/blob/master/Image/%E7%AC%AC%E4%B8%83%E5%91%A8%E5%91%A8%E6%8A%A5/%E8%AE%BA%E6%96%87%E6%88%AA%E5%9B%BE/%E5%8F%98%E9%87%8F13.png?raw=true)对一个模型整体来说是一个常量，并且比![image](https://github.com/Richbabe/NPR_Lab/blob/master/Image/%E7%AC%AC%E4%B8%83%E5%91%A8%E5%91%A8%E6%8A%A5/%E8%AE%BA%E6%96%87%E6%88%AA%E5%9B%BE/%E5%8F%98%E9%87%8F12.png?raw=true)小很多，来由光源的到一个边缘高光轮廓而不受材质属性的影响。我们使用了一个菲涅尔系数对这些边缘高光进行遮罩，以保证它们只在切线角（grazing angles）出现。这种使用Phong高光和边缘高光的组合使得《军团要塞2》得到了非常出色的画面效果。最后的效果可见图Figure6(f)。

### 专用的边缘光照（Dedicated Rim Lighting）
当角色逐渐移动远离光源时，仅仅基于Phong部分的边缘高光可能不会向我们想要的那么明显了。为此，我们还加入了一个专用的边缘高光部分（等式（2）的右半部分）。这个部分使用了观察方向对环境盒子（ambient cube）进行了取值，并且使用了一个由美术控制的遮罩纹理、菲涅尔因子以及表达式进行调整。这个最后的表达式仅仅是逐像素的法线和空间向上向量（up vector）点乘的结果，再约束到正数范围（clamped to be positive）。这使得这种专用的边缘高光看起来包含了环境中的间接光源，但仅仅适用于朝上的法线。这种方法是一种基于美学和感性的选择，我们希望这样可以让人感觉这些光照好像是从上方照下来的一样，可见图Figure6(g)。加上左半部分的效果图为Figure6(h)。
 
用于角色和其他模型渲染的完整的片段着色器就是等式（1）和等式（2）的和，以及一些其他类似于可选的环境映射的操作。最后的效果为Figure6(j)。

# Reference
* [1] Jason Mitchell, Moby Francke, Dhabih Eng. Illustrative Rendering in Team Fortress 2
* [2] Amy Gooch, Bruce Gooch, Peter Shirley, Elaine Cohen. A non-photorealistic lighting model for automatic technical illustration
* [3] R AMAMOORTHI , R., AND H ANRAHAN , P. 2001. An Effi-
cient Representation for Irradiance Environment Maps. In SIG-
GRAPH 2001: Proceedings of the 28th annual conference on
Computer graphics and interactive techniques, 497–500.
* [4] M C T AGGART , G. 2004. Half-Life 2 Shading. In Game Developers
Conference. Direct3D Tutorial.
* [5] M ITCHELL , J. L., M C T AGGART , G., AND G REEN , C. 2006.
Shading in Valve’s Source Engine. In SIGGRAPH Course on
Advanced Real-Time Rendering in 3D Graphics and Games.
* [6] G REGER , G., S HIRLEY , P., H UBBARD , P. M., AND G REENBERG ,
D. P. 1998. The Irradiance Volume. IEEE Computer Graphics
and Applications 18, 2 (/), 32–43.


