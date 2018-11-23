---
layout:     post
title:      Stylized Highlights for Cartoon Rendering and Animation
subtitle:   基于卡通渲染的风格化高光
date:       2018-11-22
author:     Richbabe
header-img: img/okami.png
catalog: true
tags:
    - NPR实验室
---

> 中山大学 数据科学与计算机学院 软件工程数字媒体 洪鹏圳

---

# 参考论文
> Stylized Highlights for Cartoon Rendering and Animation[1]

# Abstract
在以往的卡通渲染中，我们通过判断高光因子的范围来得到一块纯白的高光区域。但在很多卡通动画中，高光区域的形状是千奇百怪的。2003年K.I. Anjyo等人在其论文Stylized Highlights for Cartoon Rendering and Animation中提出一种算法，通过对高光区域平移、旋转、有向性缩放、分裂、方块化来实现高光区域的风格化。

# Introduction
在传统的卡通插画中，光照和阴影往往是艺术家对场景和人物的一种特征的表现手法。比如说，光照在渲染环境氛围上起了很重要的作用，而阴影通常指示了人物站的位置。高光也不例外，其表现了场景和物体的许多不同方面的性质。下面给出一些卡通动画中高光的例子：

![image](https://github.com/Richbabe/NPR_Lab/blob/master/Image/%E7%AC%AC%E4%BA%94%E5%91%A8%E5%91%A8%E6%8A%A5/%E8%AE%BA%E6%96%87%E6%88%AA%E5%9B%BE/figure1.png?raw=true)

比如在Figure1a中，我们可以看到主角的剑上有着交错的高光条纹，这预示着剑十分的锋利。在Figure1b中，我们可以看到巨钳螳螂的钳子上有着月牙形的高光，这预示着其钳子十分坚硬，具有强大的攻击性。在Figure1c中，我们可以看到车后窗变形了的高光区域，这表明车后窗不是平面矩形的，而是一个环形梯形。从这些例子可以看到，卡通渲染中的高光并不是追求反应物体本身的物体特征（而PBR中往往真实的模拟了物体的金属性和非金属性），而是为了表达某种艺术效果。

但是，传统的卡通渲染技术并不能很好的实现高光的风格化。我们拿波克比蛋壳上的高光作为例子：

![image](https://github.com/Richbabe/NPR_Lab/blob/master/Image/%E7%AC%AC%E4%BA%94%E5%91%A8%E5%91%A8%E6%8A%A5/%E8%AE%BA%E6%96%87%E6%88%AA%E5%9B%BE/figure2.png?raw=true)

在Figure2a中，我们使用了标准光照模型，可以看到高光区域过于真实。在Figure2b中，我们使用了传统卡通渲染的光照模型，可以看到相较于Figure2a更偏向卡通风格，但仍过于简单。而Figure3c则是应用了我们的风格化高光算法，可以看到其和Figure1中的图一样，都具有符合卡通动画的高光形状。

所以，为了实现卡通动画中高光的风格化，我们必须同时满足以下要求：
1. 形状（Shape）。我们必须绘制带有清晰轮廓且形状简单的高光区域，它不是一直是圆形的，而是有着丰富的变化，比如月牙型或者方形。
2. 动画连续性。我们必须保证在卡通动画中高光区域具有平滑的动态变化。这种暂时变形的高光区域，突出的是卡通感而非真实感。

如果使用传统的卡通渲染光照模型，我们并不能修改高光区域的形状。一些传统的方法可能可以做到修改高光区域的形状，比如环境映射加上程序形状动画技术可以很好的做到，但是我们需要的是更加简单的逐帧渲染方法。因此，我们的算法基于传统的卡通渲染技术，在高光部分在传统Blinn-Phong光照模型上加以改进，通过对高光区域的仿射变换和一些布尔操作最终实现我们想要的高光形状。

# Related Work
在很早的时候迪斯尼公司就将3D手绘人物加入到动画电影中，比如《The Great Mouse Detective》(1986)和《Beauty and the Beast》(1991)。从那以后，就有许许多多动画电影采用这种计算机图形渲染技术手段[2]。

而目前被应用于卡通电影制作的计算机图形技术称之为卡通渲染，该技术因能高效的渲染出卡通电影而被广泛使用。其中，Softimage's Toon Shaders[3]成就了许多著名卡通电影比如Ghibli工作室的《The Princess Mononoke》(1997)和Universal公司的《The Road to EI Dorado》（2000）。另一种卡通渲染技术是将复杂的纹理映射到3D物体上来模拟卡通风格[4]。

L. Petrovic提出了一种卡通阴影生成技术，在克服卡通风格阴影生成的困难的同时，还让阴影更加好看。[5]

但是这些技术都没有考虑让高光部分的更加卡通化。Lake et al.推出了一种实时卡通渲染方法[6]，其能使高光更加卡通化，但却很难去控制高光区域的形状。其他使高光卡通化的方法包括使用投影纹理、light maps、虚拟光源等，但这些方法都过于复杂，计算成本高，而且当高光区域形状变得复杂时，实现难度很高。

因此，我们的方法从Blinn-Phong高光模型出发，通过一些简单仿射变换和其他对高光区域的操作来使高光风格化更加简单高效。

# Stylized Highlights
我们在上面曾经提到过要实现高光风格化我们需要对高光区域做一些仿射变换和操作，那么我们该如何对哪些变量进行操作呢？在Blinn-Phong光照模型中，高光部分的计算公式如下：
> I(specular) = I * k * pow(max(0,dot(N,H)), gloss)

其中H是光照方向L和视线方向V的半向量，N是片段的法向量，gloss控制了高光区域的光泽度。

![image](https://github.com/Richbabe/NPR_Lab/blob/master/Image/%E7%AC%AC%E4%BA%94%E5%91%A8%E5%91%A8%E6%8A%A5/%E8%AE%BA%E6%96%87%E6%88%AA%E5%9B%BE/figure3.png?raw=true)

我们的方法是通过对Blinn-Phong光照模型中的半向量H进行一些修改操作，然后通过判断半向量和法线方向点乘结果的范围计算得到高光区域，从而使得高光区域的形状可以发生变化：

![image](https://github.com/Richbabe/NPR_Lab/blob/master/Image/%E7%AC%AC%E4%BA%94%E5%91%A8%E5%91%A8%E6%8A%A5/%E8%AE%BA%E6%96%87%E6%88%AA%E5%9B%BE/%E9%AB%98%E5%85%89%E5%8C%BA%E5%9F%9F%E5%85%AC%E5%BC%8F.png?raw=true)

这是因为，当H给定时，视线方向L = 2(H,V)H - V。所以，当V一定时，我们可以通过修改H来使L进行变换（相当于移动光源），最终使投射到物体上的高光区域形状发生改变。


我们一般将高光风格化算法对高光区域的操作分成两种。一种称为局部操作（local operation），这是因为我们只对当前一个片元进行操作，例如Figure4中给出的例子。另一种称为后处理操作(post operations)，这是因为操作是基于高光区域多个片元的直接操作。

![image](https://github.com/Richbabe/NPR_Lab/blob/master/Image/%E7%AC%AC%E4%BA%94%E5%91%A8%E5%91%A8%E6%8A%A5/%E8%AE%BA%E6%96%87%E6%88%AA%E5%9B%BE/Figure4.png?raw=true)

Figure4中给出的是5中local operation，其中橙色箭头（平移、旋转、有向性缩放）的操作称为局部仿射变换，黄色箭头（分裂、方块化）的操作称为风格化变换。在实现过程中，这些操作往往有顺序，通常的实现顺序是缩放、旋转、平移、分裂、方块化。但是，我在实现的时候发现这个顺序并不重要，只要能实现最终效果就行，我的实现顺序如下：
![image](https://github.com/Richbabe/NPR_Lab/blob/master/Image/%E7%AC%AC%E4%BA%94%E5%91%A8%E5%91%A8%E6%8A%A5/%E8%BF%90%E8%A1%8C%E7%BB%93%E6%9E%9C%E6%88%AA%E5%9B%BE/%E8%BF%87%E7%A8%8B.png?raw=true)

# 对半向量进行局部仿射操作
## 平移（Translation）
定义平移操作t(H):

![image](https://github.com/Richbabe/NPR_Lab/blob/master/Image/%E7%AC%AC%E4%BA%94%E5%91%A8%E5%91%A8%E6%8A%A5/%E8%AE%BA%E6%96%87%E6%88%AA%E5%9B%BE/%E5%B9%B3%E7%A7%BB%E5%85%AC%E5%BC%8F.png?raw=true)

其中du和dv分别是切线空间下的x轴和y轴。简单来说就是把切线空间下的半向量H的x和y分量加上某个参数值，再进行归一化，着色器代码为：

```
//平移操作(Translation)
tangentHalfDir = tangentHalfDir + fixed3(_TranslationX, _TranslationY, 0);
tangentHalfDir = normalize(tangentHalfDir);
```
其中，_TranslationX和_TranslationY是输入参数（对应了公式里的α和β），用于控制切线空间下x和y方向的高光区域平移程度。

平移操作其实就是相当于移动生成高光区域的光源，使得高光区域位置发生变化。对于输入参数α和β，并没有规定大小，但是当α和β过大时半角向量将会和切平面平行，会产生不好的结果。因此，在实现中我将_TranslationX和_TranslationY的范围设成[-1,1]。

## 旋转（Rotation）
旋转操作r(H)其实就是对切线空间下的半向量H和旋转矩阵相乘，实现绕切线空间某个坐标轴的旋转。着色器代码如下：

```
//旋转操作(Rotation)
float xRad = _RotationX * DegreeToRadian;//绕切线空间的x轴旋转多少弧度
//绕x轴旋转的旋转矩阵
float3x3 xRotation = float3x3(1, 0, 0,
                              0, cos(xRad), sin(xRad),
							  0, -sin(xRad), cos(xRad));
float yRad = _RotationY * DegreeToRadian;//绕切线空间的y轴旋转多少弧度
//绕y轴旋转的旋转矩阵
float3x3 yRotation = float3x3(cos(yRad), 0, -sin(yRad),
							  0, 1, 0,
					          sin(yRad), 0, cos(yRad));
float zRad = _RotationZ * DegreeToRadian;//绕切线空间的z轴旋转多少弧度
//绕z轴旋转的旋转矩阵
float3x3 zRotation = float3x3(cos(zRad), -sin(zRad), 0,
					          sin(zRad), cos(zRad), 0,
					          0, 0, 1);
tangentHalfDir = mul(zRotation, mul(yRotation, mul(xRotation, tangentHalfDir)));
tangentHalfDir = normalize(tangentHalfDir);
```
其中，_RotationX、_RotationY和_RotationZ是用于控制半向量切线空间的x轴、y轴和z轴的旋转角度。实际上，绕x轴和y轴的旋转和平移效果很类似，我们通常只需要调整z轴的旋转即可。

## 有向性缩放（Directional scaling）
我们通常需要给高光区域赋予方向属性（比如各项异性）来反映光照方向。我们定义沿着切线空间x轴方向的缩放如下：
![image](https://github.com/Richbabe/NPR_Lab/blob/master/Image/%E7%AC%AC%E4%BA%94%E5%91%A8%E5%91%A8%E6%8A%A5/%E8%AE%BA%E6%96%87%E6%88%AA%E5%9B%BE/%E7%BC%A9%E6%94%BE%E5%85%AC%E5%BC%8F.png?raw=true)

其推导过程如下：

我们用H+δ(N-H)来代替H，这样当δ越大时，N和H+δ(N-H)的点积越接近1，即半向量H更接近法线N，反之当δ越小时半向量H更远离法线N。当只在切线空间x轴方向上进行缩放时，我们有：
![image](https://github.com/Richbabe/NPR_Lab/blob/master/Image/%E7%AC%AC%E4%BA%94%E5%91%A8%E5%91%A8%E6%8A%A5/%E8%AE%BA%E6%96%87%E6%88%AA%E5%9B%BE/%E7%BC%A9%E6%94%BE%E5%85%AC%E5%BC%8F%E6%8E%A8%E5%AF%BC.png?raw=true)

因为(N,du) = 0（法线和切线空间x轴垂直，点积为0），所以我们可以得到上面提到的缩放公式。当δ从0逐渐变大时，半向量的x分量逐渐变小，从而在x方向上更靠近法线方向，高光区域在x方向上变大。通过这种方法，我们可以沿任意方向放缩高光区域（通过和旋转矩阵r相乘），但在实现中我只实现了对x方向和y方向的缩放。着色器代码如下：

```
//缩放操作(Scale)
tangentHalfDir = tangentHalfDir - _ScaleX * tangentHalfDir.x * fixed3(1, 0, 0);//在切线空间x轴方向上缩放
tangentHalfDir = normalize(tangentHalfDir);
tangentHalfDir = tangentHalfDir - _ScaleY * tangentHalfDir.y * fixed3(0, 1, 0);//在切线空间y轴方向上缩放
tangentHalfDir = normalize(tangentHalfDir);
```

# 对半向量进行风格化变换
上面使用的局部仿射变换让我们的高光区域位置和形状得以改变，接着我们加上风格化变换中的分裂（Spilt）和方块化（Squaring）来使高光区域更加卡通化。

## 分裂(Spilt)
分裂操作会把一个高光区域分成两个或者四个区域，它的公式其实就是缩放公式的修改版：
![image](https://github.com/Richbabe/NPR_Lab/blob/master/Image/%E7%AC%AC%E4%BA%94%E5%91%A8%E5%91%A8%E6%8A%A5/%E8%AE%BA%E6%96%87%E6%88%AA%E5%9B%BE/%E5%88%86%E8%A3%82%E5%85%AC%E5%BC%8F.png?raw=true)

其中，sgn[x]操作会判断x的符号，如果x为负返回-1，否则返回1。和缩放操作不同，分块操作会把半向量沿着不同方向让它们远离法线。着色器代码如下：

```
//分裂操作(Spilt)
fixed signX = 1;
if (tangentHalfDir.x < 0) {
	signX = -1;
}
fixed signY = 1;
if (tangentHalfDir.y < 0) {
	signY = -1;
}
tangentHalfDir = tangentHalfDir - _SplitX * signX * fixed3(1, 0, 0) - _SplitY * signY * fixed3(0, 1, 0);
tangentHalfDir = normalize(tangentHalfDir);
```
其中_SplitX和_SplitY用于控制高光区域在x方向和y方向上的分离程度，当_SplitX > 0时高光区域会在x方向上分裂成两部分，当_SplitY > 0时高光区域会在y方向上分裂成两部分。因此你如果只想让高光区域在某个方向上只分成两部分，就让_SplitX和_SplitY其中一个大于0，另一个等于0;如果你想让高光区域分成四部分，就让_SplitX和_SplitY都大于0。

## 方块化(Squaring)
这是最复杂的一个操作，我们将公式定义如下：
![image](https://github.com/Richbabe/NPR_Lab/blob/master/Image/%E7%AC%AC%E4%BA%94%E5%91%A8%E5%91%A8%E6%8A%A5/%E8%AE%BA%E6%96%87%E6%88%AA%E5%9B%BE/%E6%96%B9%E5%9D%97%E5%8C%96%E5%85%AC%E5%BC%8F.png?raw=true)

当给定正数的n和σ时，高光区域会沿着切线空间的x轴和y轴进行方块化调整，结合旋转操作我们就可以得到我们最终想要的高光图案。其中，n越大，方块尖锐程度越大；σ越大，方块面积越大。

着色器代码如下：

```
float sqrThetaX = acos(tangentHalfDir.x);
float sqrThetaY = acos(tangentHalfDir.y);
float theta = min(sqrThetaX, sqrThetaY);
fixed sqrnorm = sin(pow(2 * theta, _SquareN));
tangentHalfDir = tangentHalfDir - _SquareScale * sqrnorm * (tangentHalfDir.x * fixed3(1, 0, 0) + tangentHalfDir.y * fixed3(0, 1, 0));
tangentHalfDir = normalize(tangentHalfDir);
```
实现效果如下：
![image](https://github.com/Richbabe/NPR_Lab/blob/master/Image/%E7%AC%AC%E4%BA%94%E5%91%A8%E5%91%A8%E6%8A%A5/%E8%BF%90%E8%A1%8C%E7%BB%93%E6%9E%9C%E6%88%AA%E5%9B%BE/%E9%94%99%E8%AF%AF%E6%96%B9%E5%9D%97%E5%8C%96.png?raw=true)

可以看到我很难通过调整参数和旋转程度来使高光区域变成矩形，因此我对方块化公式进行了修改，不计算两个角度的最小值，而是同时使用两个角度：

![image](https://github.com/Richbabe/NPR_Lab/blob/master/Image/%E7%AC%AC%E4%BA%94%E5%91%A8%E5%91%A8%E6%8A%A5/%E8%AE%BA%E6%96%87%E6%88%AA%E5%9B%BE/%E4%BF%AE%E6%94%B9%E7%89%88%E6%96%B9%E5%9D%97%E5%8C%96%E5%85%AC%E5%BC%8F.png?raw=true)

着色器代码如下：

```
//方块化（Squaring）
float sqrThetaX = acos(tangentHalfDir.x);
float sqrThetaY = acos(tangentHalfDir.y);
/*
float theta = min(sqrThetaX, sqrThetaY);
fixed sqrnorm = sin(pow(2 * theta, _SquareN));
tangentHalfDir = tangentHalfDir - _SquareScale * sqrnorm * (tangentHalfDir.x * fixed3(1, 0, 0) + tangentHalfDir.y * fixed3(0, 1, 0));
tangentHalfDir = normalize(tangentHalfDir);
*/
fixed sqrnormX = sin(pow(2 * sqrThetaX, _SquareN));
fixed sqrnormY = sin(pow(2 * sqrThetaY, _SquareN));
tangentHalfDir = tangentHalfDir - _SquareScale * (sqrnormX * tangentHalfDir.x * fixed3(1, 0, 0) + sqrnormY * tangentHalfDir.y * fixed3(0, 1, 0));
tangentHalfDir = normalize(tangentHalfDir);
```
其中，_SquareScale对应公式中的σ，用于控制方块的大小，_SquareN对应公式中的n，用于控制方块的尖锐程度。运行效果如下：

![image](https://github.com/Richbabe/NPR_Lab/blob/master/Image/%E7%AC%AC%E4%BA%94%E5%91%A8%E5%91%A8%E6%8A%A5/%E8%BF%90%E8%A1%8C%E7%BB%93%E6%9E%9C%E6%88%AA%E5%9B%BE/%E6%AD%A3%E7%A1%AE%E6%96%B9%E5%9D%97%E5%8C%96.png?raw=true)

可以看到高光形状相对于上面的更加方正，我们的高光风格化也达到了预期的效果！

# 后处理操作（Post operation）
通过重复的使用我们上面提到的局部操作，我们可以获得不同形状的高光形状。但是，有些时候我们采取后处理操作直接对高光区域面积进行操作来替代对半向量的操作来使计算更高效。比如说我们想要生成一个月牙型高光，我们可以用一块圆形高光减去一块小扇形高光获得月牙型高光，这样做更直接有效。因此，我们定义后处理操作为对高光面积的布尔操作(Boolean operation)，包括加操作（类似于集合的并集）和减操作(类似于集合之间做差)。当我们发现有某一部分的高光区域不需要时，我们可以通过这些布尔操作来消除它。

![image](https://github.com/Richbabe/NPR_Lab/blob/master/Image/%E7%AC%AC%E4%BA%94%E5%91%A8%E5%91%A8%E6%8A%A5/%E8%AE%BA%E6%96%87%E6%88%AA%E5%9B%BE/figure6.png?raw=true)

在Figure6中，我们可以看到飞船的机翼部分的高光区域是通过一块正方形的面积减去一个圆形的1/4面积得到的。

# Future work
我们提出的方法是基于Blinn-Phong高光模型，通过对半向量的操作实现高光的风格化。希望在以后我们这套现有的方法可以帮助我们统一一套高效的卡通渲染光照和阴影算法。

# Reference
* [1] K.I. Anjyo，Katsuaki Hiramitsu. Stylized Highlights for Cartoon Rendering and Animation
* [2]  K. Anjyo et al., “Digital Cel Animation in Japan,” Siggraph 2000
Conf. Abstracts and Applications, ACM Press, 2000, pp.115-117.
* [3]  M. Arias, “Non-Photorealistic Rendering,” Softimage|XSI Power
Creators’ Guide, Aspects Corp., 2002, pp. 284-290.
* [4]  W. Corrêa et al., “Texture Mapping for Cel Animation,” Proc. Sig-
graph 98, ACM Press, 1998, pp. 435-446.
* [5]  L. Petrovic et al., “Shadows for Cel Animation,” Proc. Siggraph
2000, ACM Press, 2000, pp. 511-516.
* [6]  A. Lake et al., “Stylized Rendering Techniques For Scalable Real-
Time 3D Animation,” Proc. 1st Int’l Symp. Non-Photorealistic Ren-
dering (NPAR 00), ACM Press, 2000, pp. 13-20.

# 结语
本博客的代码和资源均可在我的[github](https://github.com/Richbabe/NPR_Lab)上下载，别忘了点颗Star哟！