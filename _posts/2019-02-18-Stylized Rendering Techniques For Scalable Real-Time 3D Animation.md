---
layout:     post
title:      Stylized Rendering Techniques For Scalable Real-Time 3D Animation
subtitle:   
date:       2019-02-18
author:     Richbabe
header-img: img/okami.png
catalog: true
tags:
    - NPR实验室
---

> 中山大学 数据科学与计算机学院 软件工程数字媒体 洪鹏圳

---

# 参考论文
> Stylized Rendering Techniques For Scalable Real-Time 3D Animation[1]

# Abstract
非真实感渲染（NPR）的研究人员已经研究了许多技术来模拟不同风格的艺术作品。最近的成果包括了模拟钢笔画、铅笔素描、水彩画、雕刻和轮廓线渲染的方法。本文介绍了实时卡通渲染的方法。我们还提出了使用纹理映射技术（texture mapping）来实现实时铅笔素描渲染。我们展示了我们如何使用笔触来描边，绘制物体轮廓和折痕。除此之外，我们通过一些几何信息来强调物体的运动。该渲染系统是由动画系统和实时多分辨率网格（MRM）系统集成得到，具有可扩展性，确保任何平台上的实时性能。这样的解决方案使我们利用不断发展的硬件来制作非真实感动画并在低端和高端用户平台上都能实现渲染。所有提出的技术都可以适用于由标准建模工具创建的模型，并且不需要建模者提供额外的标记信息。

![image](https://github.com/Richbabe/NPR_Lab/blob/master/Image/%E7%AC%AC10%E5%91%A8%E5%91%A8%E6%8A%A5/%E8%AE%BA%E6%96%87%E6%88%AA%E5%9B%BE/Figure1.png?raw=true)

# Introduction
由于图形显卡的飞速发展，我们再也不需要用图形处理器来进行光栅化多变边形操作，从而可以用它来实现其他效果，比如细分曲面的计算、实时物理、布料模拟、逼真动画和逆向运动学等。图形处理器和CPU的另外一个用途是创建一个风格化的渲染。接下来我们将介绍我们的钢笔画系统，它可以在当前PC平台上进行实时非真实感渲染。除此之外，其还集成了一个多分辨率网格系统来保证帧率以及处理器内存。因为日益增长的对更高视觉效果的需求，所以把这些方法集成一套实时图形系统应用到不同性能水平上的电脑是十分重要的。

Result：在这篇论文中我们将会介绍一个新的实时卡通着色算法、铅笔画和轮廓检测和渲染。我们也会提出一个新的生成运动线条的技术来强调3D卡通渲染中的运动。需要注意的是，该系统不需要建模者提供额外的信息。这些着色技术只需要每个顶点的位置、法线、材质属性、纹理坐标和连接信息。

Organization：我们的实时渲染架构由两个部分组成。Section2将会介绍之前这两部分研究过的工作。Section3将会介绍决定着色信息的要素。Section4展示了我们边缘检测的方法并给出一些优化的细节。Section5将会提出一种展现物体运动状态的方法。

# 2 Previous Work
## 2.1 Previous Work In Stylistic Rendering
近年来已经又很多实时NPR方法的研究。我们实现卡通着色的标准渲染方程和[Gooch98]中一样，也是创建一个暖色调到冷色调的变化实现插画效果。[Rade99]通过将Gooch的warm-to-cool着色方法应用到每个表面上实现了一系列的NPR效果。我们的卡通着色也在光照方程上使用同样的修改，还利用硬件来加快纹理映射从而模拟画家的调色板。[Deca96]使用多pass渲染和图像处理的混合来创建卡通风格的图片。我们的方法和Decaidin的通过n·l的值作为阈值决定表面颜色的方法很像。

## 2.2 Previous Work In Silhouette Edge Detection(SED)
SED是一项NPR领域用了很多年的技术。[Mark97]提供了一种成为Apple's hiddenline算法来补充实时SED。Markosian的算法为了快速找到轮廓牺牲了精确度，但是他没有利用深度缓冲来解决轮廓渲染的问题。

[Gooch99]展示了一些找到模型轮廓的方法。它们一共提出了三种方法：
1. 使用Gauss map
2. 使用带有缩放系数的glPolygonOffset
3. 使用直接测试每条边的方法。

Gauss map虽然会带来细轮廓不连续的现象，但其效果却十分有趣。glPolygonOffset将模型网格沿着视线方向向后平移一小段距离来显示轮廓，但是他需要渲染模型两次，这会给用户的硬件带来严重的性能消耗。

我们的方法支持实时快速并准确找到模型的轮廓。我们使用的是直接找到每条边的轮廓，既模型的每条边都检查是否轮廓的方法。

# Stylistic Shading
通过变换阴影和高光参数、纹理的数量和风格以及权重，我们可以产生不同风格的卡通渲染。

## 3.1 Cartoon Shading
卡通人物基本上都是二维的。动画师故意减少人物表面的细节来让观众更加关注故事本身。[McC193]演示了通过减少人物的细节（通常是面部的）来让观众更好的了解人物。和三维人物的渲染不同，卡通画家通常用大片纯色来绘制人物。他们经常使用暗色调的颜色来绘制物体的阴影部分，这有助于传递灯光和角色形状的信息。阴影部分和光照部分的边界是遵循物体轮廓的硬边缘。我们将这种技术称为hard shading，Figure3，16，17和18就是例子。

我们的算法支持3D模型的卡通渲染。该技术基于纹理映射和传统漫反射光照。

和[Gour71]中Gouraud着色的平滑插值着色方式不同，我们的着色技术是找到一个转换边界，并让边界两边的区域用一块纯色来进行着色。公式1展示了我们的漫反射光照方程：

![image](https://github.com/Richbabe/NPR_Lab/blob/master/Image/%E7%AC%AC10%E5%91%A8%E5%91%A8%E6%8A%A5/%E8%AE%BA%E6%96%87%E6%88%AA%E5%9B%BE/Equation1.png?raw=true)

其中，Ci是顶点颜色，ag是全局环境光系数,al和dl是光源的环境光和漫反射系数，am和dm是物体材质的环境光和漫反射系数。L是光源指向顶点的单位向量，n是顶点法向量，L·n是两个向量夹角的余弦值。

我们卡通着色技术的数学原理跟上面讲的是一样的。不同的是，上面计算的是每个顶点的颜色，而我们创建了一张由最少数目颜色（大部分情况是两种颜色，一种是明亮处的颜色，一种是阴影处的颜色）组成的纹理贴图。然后通过将方程1中点积替换成1来计算纹理贴图的明亮颜色（相当于n和L的夹角为0度，此时光线垂直射向顶点，顶点是明亮的）。将方程1中的点积替换成0来计算纹理贴图中的阴影颜色（此时颜色值完全由环境光决定）。最后得到的纹理是张一维的纹理贴图，该纹理每个材质计算依次并提前储存起来。

光照部分和阴影部分之间的边界，由光线向量和法线向量之间的夹角的余弦值决定。在每一帧，我们逐顶点计算Max{L·n，0}。并利用该值来作为纹理采样坐标进行纹理映射。

![image](https://github.com/Richbabe/NPR_Lab/blob/master/Image/%E7%AC%AC10%E5%91%A8%E5%91%A8%E6%8A%A5/%E8%AE%BA%E6%96%87%E6%88%AA%E5%9B%BE/Figure2&3.png?raw=true)

Figure2演示了灯光位置和法线方向决定了一维纹理贴图的纹理坐标。注意到当Max{L·n，0}经过0.5时，纹理的索引采样了不同的颜色。如果近距离观察，光照区域和阴影区域之间的过渡可能会出现锯齿。3D图形的API提供了texture-filtering模式来解决这种问题。该问题的出现与模型和观察位置有关。一种模式是选择最近的纹素(texel)作为像素(pixel)另一种是把一个纹素集合线性插值成一个像素。我们发现使用线性过滤器可以平滑颜色过渡带的锯齿。但是如果多边形在屏幕空间上过于大时会造成较宽的过渡区域。解析来我们将展示我们卡通着色的预处理和实时渲染部分的算法：

![image](https://github.com/Richbabe/NPR_Lab/blob/master/Image/%E7%AC%AC10%E5%91%A8%E5%91%A8%E6%8A%A5/%E8%AE%BA%E6%96%87%E6%88%AA%E5%9B%BE/CartoonShade%20ALGORITHM.png?raw=true)

注意到我们需要每一帧都计算每一个顶点的一个点积。我们纹理映射的方法等价于逐像素通过阈值计算颜色。如果不用纹理映射，那么我们需要通过硬件进行加速来计算Gouraud Shading才能达到同样的运行速度。

## 3.2 Variations On Cartoon Shading
section 3.1中的卡通着色技术可以用来渲染多种卡通风格。颜色的选择很大程度决定了渲染的卡通风格。我们的算法支持用户自己修改纹理的颜色来改变渲染效果。通过使用高分辨率的纹理贴图，可以创造出额外的效果。如果使用两色的贴图，使用高分辨率的贴图可以提供阴影边界区域的灵活性。比如，使用8纹素的纹理，可以令其中7个纹素设置为阴影颜色，另外1个纹素设置为光照颜色。大部分角色都是位于阴影处，只有小部分是照亮的。一种黑暗漫画风格可以方式实现：把阴影色设置为黑色或者背景色，然后把光照颜色设为亮一点的颜色，Figure16展示了这个例子（出自[Hoga81]）。Hogarth也提出了一种近似double-source lighting的方法：将纹理贴图的两端设置为光照颜色，将纹理贴图的中间设置为阴影颜色。

另一种效果是shadow-and-highlight风格，它的做法和标准hard shading一样，但使用3种颜色而不是2种颜色来构建一维纹理。我们使用高分辨率的纹理，带有8或7种纹素，其中一半的纹素设置为阴影色。在光照处末端，一到两个纹素设置为高光颜色，其余纹素设置为光照材质颜色。高光颜色可以通过材质颜色乘上高光系数自动选择。Shadow-and-highlight shading在Figure16和17展示了效果。这些高光是视角无光的，通过漫反射生成的高光。为了渲染镜面反射的卡通高光，我们需要添加第二张纹理并和第一张纹理进行混合，且第二张纹理的采样坐标是通过视角相关镜面反射计算而来（而不是漫反射）。现如今的显卡已经支持在一个pass里进行多纹理采样。

我们的算法也支持使用颜色渐变纹理，用户可以简单指定纹理分辨率和开始和结束的颜色。然后算法会自动计算中间的渐变颜色来填充纹理。这项技术可以使用Phong Shading进行着色，通过是指定高分辨率纹理（128或256个线性插值的纹素），一端是光照色另一端是阴影色。实际上，纹理映射着色是对漫反射光照的一种近似。我们的算法通过让用户提供少量的参数，便能渲染出一系列不同风格的卡通渲染效果。

## 3.3 Pencil Sketch Shading
![image](https://github.com/Richbabe/NPR_Lab/blob/master/Image/%E7%AC%AC10%E5%91%A8%E5%91%A8%E6%8A%A5/%E8%AE%BA%E6%96%87%E6%88%AA%E5%9B%BE/Figure4.png?raw=true)

在这项技术中，我们将卡通着色算法中一维纹理映射扩展为二维纹理映射。我们通过逐顶点计算L·n，并用这个值近似对二维纹理进行采样。纹理是通过随机选取Figure6中的铅笔笔触，将他们随机放置在纹理中（FIgure7）。那些接受较少光照的区域会有更高密度的铅笔笔触。在那些笔触密度最高的区域（最少光照），我们将铅笔笔触在垂直和水平方向上进行混合。

![image](https://github.com/Richbabe/NPR_Lab/blob/master/Image/%E7%AC%AC10%E5%91%A8%E5%91%A8%E6%8A%A5/%E8%AE%BA%E6%96%87%E6%88%AA%E5%9B%BE/Figure5.png?raw=true)

我们使用多纹理映射来渲染纸的纹理到模型上，使其看起来像是在纸上用铅笔画出来一样。如果模型被缩放，则纹理也要被缩放来保持其在背景纸纹理的连续性。在卡通着色中，可能会在L·n的值和被选中纹理之间进行非线性映射。比如L·n的值在[0.0,0.5]范围内时映射到一张纹理，在[0.5,1.0]范围内时映射到其他剩余的纹理。

因为L·n的值在这里选择的是纹理而不是纹素。所以我们必须生成一个纹理网格的坐标轴。一种方法是将一张纹理“投影”到模型上，将视口的(x,y)坐标作为模型的(u,v)坐标，正如Figure5中所展示的。这步操作可以逐帧执行也可以在预处理中只执行一次。

![image](https://github.com/Richbabe/NPR_Lab/blob/master/Image/%E7%AC%AC10%E5%91%A8%E5%91%A8%E6%8A%A5/%E8%AE%BA%E6%96%87%E6%88%AA%E5%9B%BE/Figure6&7.png?raw=true)

铅笔画算法如下：
![image](https://github.com/Richbabe/NPR_Lab/blob/master/Image/%E7%AC%AC10%E5%91%A8%E5%91%A8%E6%8A%A5/%E8%AE%BA%E6%96%87%E6%88%AA%E5%9B%BE/PencilSketchAlgorithm.png?raw=true)

当一个面上的顶点属于不同区间时，我们将这个面分成离散的三角形。为了实现这个方法，我们计算那些L·n的值时边界的顶点。比如v1是范围i，v2是范围i+j,我们沿着边v1v2对顶点进行插值得到j。对于每个新的顶点，都会存在另一个顶点使这两个顶点的边能使这两个顶点划分到不同范围。通过连接对应的顶点，我们会将三角形分到不同的子区间里，每个三角形对应不同的纹理。

这种视口映射技术保持了纹理的手绘风格，可以适用于静态图片。但是，在动画上，这种映射技术会导致"shower door effect"现象，正如[Meie96]中出现那样。如果出现这种情况，则需要在预处理期间修改纹理坐标从而使动画序列看起来更加平滑。

# 4 Stylistic Inking
Inker主要的技术是轮廓边缘检测（SED）。一条轮廓由一个前向面和一个后向面共享。一个物体或角色的轮廓可以使外形在场景中更加清晰地显示。在我们的算法中，我们不仅找到物体的轮廓，还找到了一些其他线条特征，比如折痕或边界。当两个面之间的二面角大于给定的阈值时我们检测出一条折痕。边缘是单面体的边缘或者两个不同材质的单面体共享的边。

## 4.1 Silhouette Edge Detection
![image](https://github.com/Richbabe/NPR_Lab/blob/master/Image/%E7%AC%AC10%E5%91%A8%E5%91%A8%E6%8A%A5/%E8%AE%BA%E6%96%87%E6%88%AA%E5%9B%BE/Figure8.png?raw=true)

正如Figure8所示，一条边被标记为轮廓边，当一个前向片面和一个后向片面共享它的时候。在每一帧我们通过将两个片面的法线点乘视线方向，再把结果相乘（见Equation2），比较最后乘积与0的大小。如果最后的结果小于等于0，则该边为轮廓边并标记起来进行渲染。

![image](https://github.com/Richbabe/NPR_Lab/blob/master/Image/%E7%AC%AC10%E5%91%A8%E5%91%A8%E6%8A%A5/%E8%AE%BA%E6%96%87%E6%88%AA%E5%9B%BE/Equation2.png?raw=true)

具体的轮廓边检测和绘制算法如下：
![image](https://github.com/Richbabe/NPR_Lab/blob/master/Image/%E7%AC%AC10%E5%91%A8%E5%91%A8%E6%8A%A5/%E8%AE%BA%E6%96%87%E6%88%AA%E5%9B%BE/SED.png?raw=true)

上述算法的计算可以通过z-buffer和图形API视线。如果是静态的话轮廓边可以在加载的时候计算，如果是动态的话则需要逐帧检测轮廓边。

![image](https://github.com/Richbabe/NPR_Lab/blob/master/Image/%E7%AC%AC10%E5%91%A8%E5%91%A8%E6%8A%A5/%E8%AE%BA%E6%96%87%E6%88%AA%E5%9B%BE/Figure15-18.png?raw=true)

![image](https://github.com/Richbabe/NPR_Lab/blob/master/Image/%E7%AC%AC10%E5%91%A8%E5%91%A8%E6%8A%A5/%E8%AE%BA%E6%96%87%E6%88%AA%E5%9B%BE/Figure20.png?raw=true)

# Reference
[1] Lake A, Marshall C, Harris M, et al. Stylized rendering techniques for scalable real-time 3D animation[C]//Proceedings of the 1st international symposium on Non-photorealistic animation and rendering. ACM, 2000: 13-2
