---
layout:     post
title:      探究PBR的两种流程以及Unity中的PBS
subtitle:   
date:       2018-06-25
author:     Richbabe
header-img: img/u3d技术博客背景.jpg
catalog: true
tags:
    - 计算机图形学
    - Unity
---
# 前言
通过上一篇博客[PBR原理](http://richbabe.top/2018/06/18/PBR%E5%8E%9F%E7%90%86/)我们大概了解了PBR的一些基础理念，这篇博客就让我们来探究一下PBR的两种流程和Unity中两种PBS材质，毕竟Unity是我主要学习的引擎（希望未来有机会接触虚幻4）

# 预备知识
虽然上一篇博客把一些基础理念都介绍了，但是这里还是要重点介绍一下这篇博客所需要的一些理念。

## Albedo
可以理解为物体的基础色Base Color,即该物体本身颜色，也是漫反射光的颜色。

其实该颜是色光的折射被物质吸收并形成漫反射后，成为不同波长的光，这就赋予了物体颜色。比如说物体呈现蓝色，是因为其把除了蓝色的光都吸收了，散射出来的光的波长是在蓝色所在范围内。

## Metallic
即金属度，表示了反射时发生镜面反射和漫反射的光线的占比。

Metallic度越大，发生镜面反射的占比越大，漫反射diffuse占比越小，一般金属物体的金属度比较大：70% ~ 100%之间；

Metallic越小，发生镜面反射的占比越小，漫反射diffuse占比越大，一般非金属物质金属度比较小；2% ~ 5%之间，宝石的大概8%；

金属是没有漫反射的，折射进表面的光照全部被吸收。但是一些腐蚀性的金属，其腐蚀的部分具有漫反射。

## Specular
这个没什么好说的，就是镜面反射，对于非金属物质，其镜面反射的sRBG范围在40 ~ 75；对于金属物质，其sRGB范围在155 ~ 255

其也是表示0度时的菲涅尔系数RF（0°）。

## Roughness
即粗糙度，其与Smoothness（光滑度，也称为Glossiness光泽度）相反。其表示了物体表面的不规则程度，决定了在发生镜面反射时入射光线与法线的夹角大小。

粗糙度roughness越大，镜面反射的出射光线分散的角度就越大，光照越模糊；

粗糙度roughness越小，镜面反射的出射光线分散的角度就越小，光照越尖锐；

# PBR的两种工作流程
![image](https://github.com/Richbabe/Richbabe.github.io/blob/master/img/PBR/PBR%E7%9A%84%E4%B8%A4%E7%A7%8D%E5%B7%A5%E4%BD%9C%E6%B5%81%E7%A8%8B%E5%92%8CUnity%E4%B8%AD%E7%9A%84PBS/PBR%E7%9A%84%E4%B8%A4%E7%A7%8D%E5%B7%A5%E4%BD%9C%E6%B5%81%E7%A8%8B.png?raw=true)

在PBR内，规范了设计流程 ，使得贴图的设计不再仅仅依靠经验依靠直觉，可以按照固定的工作流程实现真实的贴图效果。

通过上面的介绍，我们可以只知道，在真实的生活中，视觉效果的呈现，主要取决于：
* 自然光照下，物体呈现的颜色（BasedColor/Albedo）；
* 物体表面对光线的镜面反射角度(Roughness)；
* 物体表面对光线镜面反射和漫反射的比例(Metallic/Specular)；

## Metal-Roughness
在Metal-Rougnness流程中，分别对应BaseColor，Roughness，Metallic这三个参数；

在Metal-Roughness流程中，只要按照流程，分别设置好BaseColor，Roughness，Metallic，就可以基本确定物体材质的视觉效果；

## Specular-Glossiness
在Specular-Glossiness流程中，参数发生了变化，分别为Diffuse，Glossiness，Specular三个参数；

在Specular-Glossiness流程中，Diffuse和Specular共同决定了物体的basecolor，和表面镜面反射和漫反射的比例，与第一种流程的区别在于，此流程直接指定确定的占比值，第一种是根据Metallic属性，自动匹配相应的占比值；

## 两个流程的共同贴图
最右侧三张贴图 AO，normal，height都是为了增加材质贴图的细节，使得其看起来更真实。

### AO贴图
AO贴图会根据模型某一部分相对于其他组成部分或者其他模型之间的几何距离，模拟模型的光影效果，比如一些夹角会更暗或者更亮，某一些面因为其他模型部分的影响，可能光照更少。

### Normal贴图
normal贴图会根据光照环境，优化模型表面的光影效果，比如一些凸起凹陷等细节；

### Height贴图
height贴图会给模型本身根据实际需要增加凸起或者凹陷的几何效果。比如木质模型，某处被敲打而导致的凹陷效果。

### linear模式
在PBR流程中，环境光线效果需要使用linear模式，原来的gamma模式不再适用。因为gamma模式本身是为了使其他技术渲染出来的模型具有更真实而效果，对环境光进行了修正。但是在pbr中，贴图在设计的时候，完全是按照真实的环境中的光照去计算的，所以此处不再需要修正。

这里补充一些色彩空间的知识：

### linear和srgb,HDR区别
人眼有光线自适应的特性，这样也是能让人在暗的场景里看到更多东西，在亮的场景里能分辨更多东西，这种效果一般从电影院这样昏暗的场所里走出来更能体会，眼睛会有一个慢慢适应的过程。现在摄像头一般也会自适应曝光度，但是工业需求的有些还是需要不自动的，三维渲染之中，如果把这种真实完全模拟的图像给人眼看到，会感觉比我们人眼看到灰暗的多，所以一般会在硬件方面做一个反曲线最后变成srgb的色彩空间让人看到。

游戏引擎里面最终效果给人看到，当然是这种强化过的适应人眼的色彩空间，但是我做光照计算和贴图就不行，因为会由于多次的色彩强化导致最终画面强烈失真，这时候就是需要linear的色彩空间，计算时候用真实色彩，直到输出这一步把色彩强化一遍以适合人眼。unity很早就是linear的色彩空间，但是由于后期最后一部的矫正方面很多从业人士的素养不足，或者完全没有这个意识，做出来的效果完全是不正确，或者缺乏调整弹性的。而unreal则在工作流上面几乎是整合和后期的色表，去解决这个问题。

而HDR是模拟人眼的过程,能看到更广范围的光（就是亮的时候能更亮瞎），HDR图像的一般色域也超过普通的RGB色域（但也可以不超过）。HDR在unity中需要勾选摄像机上的HDR选项和使用延迟渲染deferred才能（否则会有滤镜提示The camera is not HDR enabled 这是没使用延迟渲染），要看出效果还要加个bloom之类的

![image](https://github.com/Richbabe/Richbabe.github.io/blob/master/img/PBR/PBR%E7%9A%84%E4%B8%A4%E7%A7%8D%E5%B7%A5%E4%BD%9C%E6%B5%81%E7%A8%8B%E5%92%8CUnity%E4%B8%AD%E7%9A%84PBS/HDR.png?raw=true)
(对照图 在延迟渲染模式和HDR下加Tonemapping(effects里面的))




# Unity中的PBS
先来看看PBR和PBS的关系：

PBR（Physically Based Rendering）是一种渲染方式，它使用的材质是PBS（Physically Based Shader），中文名：基于物理的渲染技术。可以对光和材质之间的行为进行更加真实的建模。（之前我们已经聊过一种简单的建模，标准光照模型）PBS只考虑材质在真实物理环境下应该有的效果。PBR包围的范围会更广一些，比如GI/AO/SUN等复杂情况，这些东西加上PBS，才是PBR。

Unity中的PBS即是把PBR算法（具体算法实现请等待下回分解）封装起来，只用修改PBS的参数就能达到PBR的效果。

在Unity中，PBS分为两类，一个叫做Standard，一个叫做Standard（Specular Setup）,我们把它称为标准着色器的高光版，它们共同组成了一个完整的PBS光照明模型，而且非常易于使用。

## Standard(Specular Setup)
其使用流程为：
![image](https://github.com/Richbabe/Richbabe.github.io/blob/master/img/PBR/PBR%E7%9A%84%E4%B8%A4%E7%A7%8D%E5%B7%A5%E4%BD%9C%E6%B5%81%E7%A8%8B%E5%92%8CUnity%E4%B8%AD%E7%9A%84PBS/Specular%E6%B5%81%E7%A8%8B.png?raw=true)

Inspector面板为：
![image](https://github.com/Richbabe/Richbabe.github.io/blob/master/img/PBR/PBR%E7%9A%84%E4%B8%A4%E7%A7%8D%E5%B7%A5%E4%BD%9C%E6%B5%81%E7%A8%8B%E5%92%8CUnity%E4%B8%AD%E7%9A%84PBS/Specular%E9%9D%A2%E6%9D%BF.png?raw=true)
可以发现Standard(Specular setup)对应的即是我们上面讲的Specular-Glossiness流程。

其中Albedo贴图即是物体的BaseColor，Specular贴图反映了物体的Specular值，Smoothness反映了物体的光滑度（即Roughness）。

其他每个贴图代表的意义可以看[Unity官方文档](https://docs.unity3d.com/Manual/StandardShaderMaterialParameters.html)，我这里就不一一叙述了。

## Standard
其使用的流程为：
![image](https://github.com/Richbabe/Richbabe.github.io/blob/master/img/PBR/PBR%E7%9A%84%E4%B8%A4%E7%A7%8D%E5%B7%A5%E4%BD%9C%E6%B5%81%E7%A8%8B%E5%92%8CUnity%E4%B8%AD%E7%9A%84PBS/Metalic%E6%B5%81%E7%A8%8B.png?raw=true)

Inspector面板为：
![image](https://github.com/Richbabe/Richbabe.github.io/blob/master/img/PBR/PBR%E7%9A%84%E4%B8%A4%E7%A7%8D%E5%B7%A5%E4%BD%9C%E6%B5%81%E7%A8%8B%E5%92%8CUnity%E4%B8%AD%E7%9A%84PBS/Metal%E9%9D%A2%E6%9D%BF.png?raw=true)
可以发现其与Standard(Specular Setup)不同的是Specular贴图替换成了Metallic贴图。Metallic贴图对应的是材质的金属度，Standard所对应的即是我们上面提到的Metal-Roughness流程。

其他贴图的含义也可以在[Unity官方文档](https://docs.unity3d.com/Manual/StandardShaderMaterialParameters.html)中找到。

## Standard与Standard(Specular setup)比较
![image](https://github.com/Richbabe/Richbabe.github.io/blob/master/img/PBR/PBR%E7%9A%84%E4%B8%A4%E7%A7%8D%E5%B7%A5%E4%BD%9C%E6%B5%81%E7%A8%8B%E5%92%8CUnity%E4%B8%AD%E7%9A%84PBS/Specular%E5%92%8CMetal%E6%AF%94%E8%BE%83.png?raw=true)
这两者其实是可以混用的，没有绝对的使用选择，具体可以看[Unity的官方博客](https://blogs.unity3d.com/cn/2015/02/18/working-with-physically-based-shading-a-practical-approach/)和[使用substance来混合两种材质](https://blog.csdn.net/shenmifangke/article/details/51105092)

下面借用UWA的一张图，综述一下Unity中PBS的工作流程。如果把standard shader想成一个黑盒子，想要计算材质球在不同光照情况下的颜色。PBS最核心是BRDF函数，输入分为两类：第一类是材质的属性（几何信息、位置、法线等），材质的颜色信息（不管用金属工作流还是高光工作流，本质上都是为了得到漫反射颜色和高光颜色）。这样材质信息就准备好了。第二类输入是光照部分，光照部分分为两类，直接光照（平行光/点光/聚光灯的颜色和方向），间接光照。间接光照又分为两类，一类是漫反射光源（Unity的光照探针和lightmap），第二类是高光光源（Unity的反射探针），虽然这两个之间是有贡献的：
![image](https://github.com/Richbabe/Richbabe.github.io/blob/master/img/PBR/PBR%E7%9A%84%E4%B8%A4%E7%A7%8D%E5%B7%A5%E4%BD%9C%E6%B5%81%E7%A8%8B%E5%92%8CUnity%E4%B8%AD%E7%9A%84PBS/PBS%E5%B7%A5%E4%BD%9C%E6%B5%81%E7%A8%8B.PNG?raw=true)

# 结语
关于PBR的两种工作流程和Unity的两种PBS我就叙述到这里，如果可以的话我会接着更新PBR算法的具体实现（主要是BRDF）。