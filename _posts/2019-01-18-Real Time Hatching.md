---
layout:     post
title:      Real-Time Hatching
subtitle:   
date:       2019-01-18
author:     Richbabe
header-img: img/okami.png
catalog: true
tags:
    - NPR实验室
---

> 中山大学 数据科学与计算机学院 软件工程数字媒体 洪鹏圳

---

# 参考论文
> Real-Time Hatching[1]

# Abstract
我们提出了一种实时非真实感素描风格渲染算法，该算法通过将素描笔调绘制于任意物体表面上，从而传达了该物体的材质、色调和形状等信息。

在算法的自动预处理过程中，我们创造了一系列多级渐远素描纹理（mip-mapped hatch image）来对应不同的色调，统称这些纹理为色调艺术映射（Tonal Art Map,TAM）。在这些素描纹理里，笔触通过缩放从而得到在不同分辨率下合适的笔触大小和笔触密度，使其看起来更加协调。

在算法运行的时候，硬件将多张素描纹理进行混合来渲染物体的表面，从而在保持空间和时间连贯性的同时改变物体表面的色调。

为了能在任意表面渲染笔触，我们通过建立重叠纹理参数化来使叠加后的纹理对齐基于表面曲率的方向场。我们将会展示在复杂的物体表面将会有不同形式的笔触。

![image](https://github.com/Richbabe/NPR_Lab/blob/master/Image/%E7%AC%AC%E5%85%AB%E5%91%A8%E5%91%A8%E6%8A%A5/%E8%AE%BA%E6%96%87%E6%88%AA%E5%9B%BE/Figure1.png?raw=true)

# 1 Introduction
在绘画中，素描笔触可以同时传递光照信息、物体属性并揭示物体的形状。素描通常是指具有空间连贯方向和质量的笔触组。其中，局部笔触的密度控制了着色色调，而组成的聚合则表现了物体表面纹理。最后，物体表面上的笔触通常遵循曲率原则或其他自然现象，从而揭示了物体的体积和形状。

该论文介绍了一种实时素描风格渲染的方法，该方法可以用于在一些需要非真实感渲染（NPR）风格的可交互应用上，比如游戏，拥有可交互技术的插图和艺术动画电影中。

在这些可交互的应用中，用户决定了摄像机、光照和场景的变化，因此需要物体表面上的素描笔触能够动态的重新排列。如果素描纹理序列中每张纹理的笔触是互相独立的，那么将会导致缺乏时间连续性，使得画面出现闪烁跳帧的现象，其中某些纹理将不会显示。

为了实现笔触之间的时间连续性，我们首先必须在图像空间（image-space）和物体空间（object-space）中做出选择。图像空间的一致性使得更容易维护人们期望的绘图的恒定笔触宽度。但是，选择图像空间可能会导致动画序列出现"shower-door effect"现象——即用户看到笔触会产生一种通过一片半透明玻璃观察场景的错觉（笔触让用户误以为物体是玻璃）。同时，使用图像空间会使得在揭示物体形状时通过物体表面某些参数对齐笔触的方向这一操作变得困难。因此，我们选择了物体空间。选择物体空间的困难点在于当物体朝向或者远离摄像机移动时，物体表面的笔触的宽度和密度必须进行调整来保持应有的色调。

简而言之，要进行可交互的实时素描风格渲染有三个主要的挑战：

（1）在运行时进行有限的计算

（2）在每一帧切换时保持笔触连续性

（3）在动态切换视角时控制好笔触的大小和密度。

我们利用现代硬件的纹理映射来解决这些挑战。

我们的方法是将素描笔触提前渲染到一系列的多级渐远纹理中，分别对应不同的色调，该方法法统称为色调艺术映射（Tonal Art Map,TAM）。在运行的时候，场景中物体的表面将通过这些TAM纹理进行纹理采样。此时关键问题就在于如何在改变表面色调的同时保持空间和时间的连续性。我们解决该问题的方法分为两部分：

1. 在离线创建TAM纹理的同时，我们在笔触、色调和mip-maps级别之间建立一个嵌套结构。我们通过使那些在更亮的图片中的笔触成为更暗图片中的笔触的子集来实现色调的连续性。同理，我们通过使较低的mip-map级别中的笔触成为较高的mip-map级别中的笔触的子集来实现分辨率的连续性。
2. 在渲染每个三角形网格时我们根据逐顶点光照计算来获取每张纹理的权重，通过硬件多纹理映射来混合多张TAM纹理。因为混合过的TAM纹理共享许多笔触，就会产生以下效果：在离摄像机更近或者变暗的表面，在保持原有笔触的同时会有一些新的笔触但如。

Girshick等人[3]观察得出，物体表面笔触的方向可以清晰展示物体的形状。对于带有自然参数的表面，TAMs可以通过以下方式进行应用：通过让渲染的笔触遵循表面曲线参数。但是，为了能在任意形状物体的表面应用实时素描风格渲染，我们为给定的模型建立了一个搭接纹理参数化（lapped texture parametrization[17]），然后通过结果参数集合渲染TAMs。lapped texture将会根据物体表面方向场来构造与之对齐的参数。物体表面方向场可以让用户知道物体的表面特征，或者通过和Hertzmann和Zorin[7]等人的方法来根据物体表面方向自动对齐参数。

该论文具体贡献在于：
* 介绍了通过色调艺术纹理映射（TAMs）来使得硬件能够渲染笔触（第三节）。
* 一种自动创建TAMs笔触算法，并适应笔触在不同尺寸和色调的连续性（第四节）。
* 一个多纹理映射算法高效渲染TAMs,并遵循空间和时间连续性。（第五节）
* 将lapped textures和基于曲率的方向场整合到TAMs中形成一个可以应用在任意模型上的实时素描风格渲染算法（第六节）。

# 2 Previous work
许多非真实感渲染工作主要关注的是帮助画家（或者技术人员）来生成图像的工具。在许多传统的媒介或者风格比如印象派[5,12,14]，技术插画[4,19,20]，铅笔和油画[1,21,22,26,27]，雕刻风格[16]等不同非真实感渲染风格上得到应用。我们的工作适用于那些要求可以看清笔触的多种不同风格的非真实感渲染。

对NPR方法进行分类的一种依据是通过该方法使用的输入形式进行分类。基于素描风格的NPR渲染的一个分支是使用一张具有颜色、色调或方向的图片作为输入，比如[5,21,22]。对于视频的绘画处理，Litwinowitcz[12]和之后的Hertzmann、Perlin[6]等人通过可见流方法来近似物体空间中笔触的连续性来解决帧到帧连续性的挑战。我们的方法基于上面提到的研究并将3D模型作为输入。

大部分应用于3D的工作都是关注创建一个3D场景的静态图像[1,2,20,24,25,26,27]。有几个算法解决了离线动画[1,14]，其中物体空间的笔触连续性比他应用于视频处理更容易处理。许多研究人员已经介绍了许多应用于渲染非真实感风格的3D场景的方法，包括技术插画[4]，“graftals”（对于抽象纹理和树叶特别有效的集合纹理）[8,10]和实时轮廓绘制[2,4,7,13,15,18]。然而这些都不是我们研究关注的重点，我们提取并渲染轮廓因为它们经常在绘画中起着至关重要的作用。

在该论文中，我们介绍了色调艺术纹理映射，它基于之前两个技术：“prioritized stroke textures”和“art maps”，他们分别由Salisbury等人[12]和Winkenbach等人[26]提出。优先笔画纹理（prioritized stroke texture）是笔触的集合，其同时传递了物体材质和连续的色调的信息。优先笔画纹理目的是创建静态的图片，因此其没有解决高效渲染和时间连续性的问题。你可以认为TAMs是存储一组离散的优先笔触纹理和采样的整合。它可以应用于实时的，时间连续的渲染。TAMS也基于Klein等人[9]提出的“art maps”技术，该技术通过基于硬件的纹理映射缓慢加入或减少笔触来实时调整屏幕空间笔触的密度。TAMs和art maps不同点有两个。第一点，art maps包含每一张图片的不同色调，而TAMs则是由一系列的art maps组织而成，每一张art maps代表一个单一色调。第二点，TAMs提供了在形状和色调上的笔触连贯性来提高最终渲染结果的时间连续性。

在此之前已经有三个算法实现了实时渲染素描风格的3D场景。Markosian[13]等人介绍了一种简单的素描风格表示一个光源离摄像机很近，方法是散射一些笔触于接近或者平行于轮廓的表面。Elber等人[2]展示了如何实时渲染艺术线条于带有参数的表面。他通过不管观察距离并选择一个固定纹理密度来规避时间连续性的问题。最后，Lake等人[11]介绍了一种可交互素描风格渲染方法，其在图片空间上是线条连续的（我们的方法是在物体空间）。

# 3 Tonal Art Maps
即使是使用现代图形硬件，逐片元绘制素描笔触的开销也很大。在真实感渲染领域，绘制复杂细节的传统方法是通过纹理映射来获取它。我们使用同样的方法来渲染素描笔触从而实现非真实感渲染。素描笔触被提前渲染到一个纹理图片集合中，并且在运行的时候由这些纹理混合得到物体表面的颜色。

传统的纹理映射往往只传递了物体材质细节，而我们绘制素描笔触的技术不仅传递了物体材质细节，还传递了物体的着色。我们离散化了色调值的范围，并用一系列的素描纹理表示这些离散色调。为了渲染一个片面，我们先通过顶点光照计算得到目标颜色值，然后通过近似的颜色值使用纹理渲染该顶点。通过选择多张纹理并混合它们，我们可以在保持素描笔触细节的连续性的同时很好的控制表面色调。我们将会在第五部分展示完整的算法。

当使用传统纹理映射来表示非真实感细节时，另一个挑战就是当缩放的时候效果不是很好。比如说当你靠近物体或者远离物体时，笔触的密度是一成不变的。所以，当放大一个物体的时候我们期望去看到更多的笔触细节出现（与此同时保持物体在屏幕空间的整体色调不变），而传统的纹理映射方法只是让笔触更粗而已（通过mip-maps技术）。Klenin[9]等人提出的art maps方法重新定义了传统mip-maps技术从而解决了该问题。在这篇论文中，我们使用同样的方法来处理笔触的宽度。我们通过设计mip-map等级并让所有等级的纹理中笔触的宽度是一样的。高等级的纹理是通过加入新的笔触来填充低等级纹理中笔触之间的间隙而得到的。我们将这些mip-map笔触纹理与每个色调值关联起来，并将这些纹理称为色调艺术映射（Tonal Art Map,TAM）。


![image](https://github.com/Richbabe/NPR_Lab/blob/master/Image/%E7%AC%AC%E5%85%AB%E5%91%A8%E5%91%A8%E6%8A%A5/%E8%AE%BA%E6%96%87%E6%88%AA%E5%9B%BE/Figure2.png?raw=true)

正如Figure2所示，一个TAM包含一个由纹理组成的2D网格。令(l,t)为网格的点的下标，行下标l表示了mip-map的等级(l = 0表示最低级)，列下标t表示色调值（t = 0时为白色）。

因为我们是根据不同的色调值和不同的分辨率（mip-map级别）来混合纹理，所以这些纹理必须要有高度连续性。论文[9]中的art maps因为每个级别的mipmap由NPR图像处理工具"off-the-shelf"独立创建，因此其连续性很差。当不同mipmap级别的纹理笔触具有不连续性时，当你靠近或者原理物体表面将会出现"swimming strokes"的现象（即笔触像是漂浮在物体上面一样）。

我们解决该问题的方法是给笔触加上一个嵌套属性：在2D网格中，纹理(l,t)的所有笔触均在比他更暗（黑）且分辨率相同的纹理中，也在一样的色调且分辨率更高的纹理中。即：纹理（l,t）的所有笔触均在纹理(l',t')中，其中l' >= l,t' >= t。举个例子，在Figure2中第四行的第1个纹理加上第二个纹理就可以得到第三个纹理。因此，当混合2D纹理网格中两个相邻的纹理时，因为只有一部分像素不同，导致混合结果只是多了一些灰色的笔触，达到最小化混合效果。

![image](https://github.com/Richbabe/NPR_Lab/blob/master/Image/%E7%AC%AC%E5%85%AB%E5%91%A8%E5%91%A8%E6%8A%A5/%E8%AE%BA%E6%96%87%E6%88%AA%E5%9B%BE/Figure3.png?raw=true)

Figure3展示了使用TAMs渲染的效果。其中Figure3(a)展示了mip-maps在球的极点被压缩。Figure3(b)展示了TAMs在不同色调间平滑转换。Figure3(c)展示了这两种效果的结合。第五部分将会讲诉如何使用物体表面的光照参数来混和这些纹理并应用到模型上。

# 4 Automatic generation of line-art TAMs
TAMs的概念非常普遍，可以用来表示各种美学（例如铅笔画、蜡笔画、点画和木炭画）。TAM的纹理网格既可以手画，也可以自动生成。在这一章，我们将展示我们自动生成素描TAM纹理的方法。

我们在上一节提到过TAM纹理中的笔触必须满足嵌套属性，因此我们的方法是按照从上到下，从左到右的顺序生成TAM网格纹理。对于每一列，我们先复制其左边一列的全部纹理贴图（如果是最左边一列则令全部纹理贴图为全白）。这确保了亮色调纹理中的所有笔触都会在暗色调纹理中出现。在每一列中，我们接着考虑从上到下（mip-map级别从低到高）的顺序。我们重复地添加笔触到纹理中直到纹理的平均色调值达到该列的色调值。当前行的所有笔触会加到下一行（mip-map级别更高）的纹理中。这确保了低mipmap级别纹理中的笔触一定会出现在高mipmap级别纹理中。

![image](https://github.com/Richbabe/NPR_Lab/blob/master/Image/%E7%AC%AC%E5%85%AB%E5%91%A8%E5%91%A8%E6%8A%A5/%E8%AE%BA%E6%96%87%E6%88%AA%E5%9B%BE/fg.png?raw=true)

如上图所示，随机选取笔触会导致笔触的非均匀分布。通常画家会将笔触放置的更均匀一点。为了实现均匀分布，我们生成多张随机笔触纹理然后选择最适合的一张。在Figure2这个例子中，我们生成了1000张亮的（最左边）的纹理贴图，然后逐渐减少到100张使笔触叠加起来生成暗的纹理贴图。这些笔触的长度在贴图宽度的0.3倍到1.0倍这个区间随机取值，然后方向取均值方向的一个随机扰动。Figure5中展示了不同参数生成的TAMs。

我们由色调和笔触分布度来决定那张贴图是最适合的（best-fitting）。对于每条笔触si我们按如下方法来计算：
对于TAM中第l个mip-map级别的纹理贴图，如果它接收si，我们会先分别找到在加入该笔触之前的平均色调Tl和加入该笔触之后的平均色调Til。Σl(Til-Tl)表示将该笔触加入到当前列的所有纹理后总的变黑值。该值是一个衡量TAM中所有级别会朝目标色调前进多少的标准。

为了实现更均匀的笔触分布，我们为TAM每一列中每张未填充的TAM纹理图维持一个纹理渐增系数，然后找到当加入笔触si时所有等级p的纹理渐增系数。即使si可能加入，但不会重叠。一条位于高分辨率的轮廓会在低分辨率中与其他轮廓重叠。因此我们会找到不带si时的平均色调Tpl和带si时的平均色调Tipl，计算总和Σp,l(Tipl - Tpl)。

最后，我们将该笔触进行单位化：

![image](https://github.com/Richbabe/NPR_Lab/blob/master/Image/%E7%AC%AC%E5%85%AB%E5%91%A8%E5%91%A8%E6%8A%A5/%E8%AE%BA%E6%96%87%E6%88%AA%E5%9B%BE/%E5%85%AC%E5%BC%8F1.png?raw=true)

在执行这个处理之后，素描纹理的笔触将会像Figure2中那样分布均匀。

画家经常用平行的笔触来表示亮色调，用交叉的笔触来表示暗色调。我们的TAM支持构造交叉笔触，就像Figure2展示的那样。对于Figure2中一些左边的列（比如第3列，我们只加入水平方向的笔触），但是从一些中间列（比如说第4列），我们只添加垂直方向的笔触来实现交叉笔触的效果。最后，一些右边的列（比如第6列）我们既添加水平方向的笔触也添加垂直方向的笔触来使得笔触间的间隔达到一个最合适的效果。

在Figure5中，不同的TAMs因为角度、笔触交叉成都、笔触长度和笔触的选择导致了不同的风格。目前我们只使用灰色调的笔触，最高分辨率的笔触纹理贴图的分辨率是256 x 256。

# 5 Rendering with TAMs
TAM纹理网格的嵌套结构保证了帧与帧之间的连续性。mip-map技术保证了能在不同分辨率下自动混合贴图。但是，我们的TAM仍需要在色调上能够平滑过渡。我们通过一种6路混合的方式来实现色调平滑过渡。但在讲述6路混合之前我们先来讲述3种其他近似方式。

第一种，如果对于每个片面我们直接选择TAM中代表一个色调的某一列而不进行混合（相当于用一个色调板来进行表面着色），我们的渲染结果将会出现空间不连续性（在笔触交错位置）和时间不连续性，如下图所示：

![image](https://github.com/Richbabe/NPR_Lab/blob/master/Image/%E7%AC%AC%E5%85%AB%E5%91%A8%E5%91%A8%E6%8A%A5/%E8%AE%BA%E6%96%87%E6%88%AA%E5%9B%BE/fg2.png?raw=true)

第二种，对于一个片面上的所有像素我们选择TAM中两个相邻的列进行插值。该方法可以实现时间连续性，但是看起来效果很差。我们需要通过顶点的属性来进行混合从而实现空间连续性。于是，第三种近似方式是每个顶点选择一列然后逐片面进行混合。该方法在空间上是连续的但是在时间上是离散的，因为一些色调值的丢失。（第三种方式相当于逐顶点进行光照模型计算，即Gouraud-shading，会导致马赫带现象等）。

为了能同时保持空间和时间的连续性，对于每个顶点我们使用两列进行TAM种两列的混合，然后对于每个三角形片面，我们将3个顶点的纹理进行混合然后由每个像素离三角形重心的距离来决定该像素最终的颜色（就像Gouraud Shading那样）。因为2路色调混合和3路空间混合都是线性组合，这就相当于将6张纹理贴图线性混合来计算一个表面。这个过程在Figure4中展示了出来：
![image](https://github.com/Richbabe/NPR_Lab/blob/master/Image/%E7%AC%AC%E5%85%AB%E5%91%A8%E5%91%A8%E6%8A%A5/%E8%AE%BA%E6%96%87%E6%88%AA%E5%9B%BE/Figure4.png?raw=true)

虽然这种复杂的混合方式让人第一眼看上去就被劝退，但是目前商业显卡已经由几种方法能高效实现它。接下来我们将介绍其中两种方法。

## Single-pass-6-way-blend
6路混合可以在一个渲染pass里完成。目前已经有一些显卡能够支持在光栅化的时候同时读取两张纹理（这个数字很快会变成四张）。因为我们TAM纹理都是灰度图，所以我们可以把6张连续的素描纹理保存到两张纹理的RGB通道中，然后用片段的漫反射和镜面反射的光照颜色值来决定混合的比例。我们在顶点设置这些颜色，然后让硬件自动插值到像素上。通过取两张纹理与片段颜色点积的和来进行混合。第一个阶段执行两次点积，第二阶段把第一阶段的两个结果加起来。

Hertzmann和Zorin[7]提出可以通过一个有限颜色的调色板来进行高效绘制，他们将色调值分为4个等级。对于我们的例子，我们选择将色调值分为6个等级。所以TAM纹理可以被打包进两张纹理并渲染所有三角形片面。因为纹理的状态是不变的，为了高效我们选择三角形条带作为图元。我们在顶点数组里存储顶点的几何信息和纹理坐标，然后在每一帧从主存中复制顶点的漫反射和镜面反射的光照颜色。使用现代的显卡，这些都可以在顶点着色器通过GPU进行计算。

我们6张纹理里面并没有包含白色（纸的颜色）。但是，我们的插值方法提供了白色的获取。具体为：因为混合系数的总和是1.0，通过指定6个系数我们实际上混合了7张纹理图——第七张纹理图的色调为0.0（黑色）。因为我们想要获得白色，所以我们在进行点积之前将输入的纹理颜色取反，则最终的点积和也被取反。于是，最终渲染效果是由6张纹理图和一张白色图混合而成。

当在任意多边形的表面上使用素描风格渲染时，需要通过轮廓的alpha mask来修改纹理，这看起来很复杂，我们将在第6节讲述。

## Triple-pass 2-way blend
如果TAM纹理包含带有颜色的笔触（非灰色），或者一次需要超过6张以上的TAM纹理，一个可选的方案是在每个片面上通过3个pass来累积纹理值。具体做法为：首先将整个片面绘制成黑色。然后接下来对每个三角片面进行3次循环，每次循环通过混合计算结果并逐顶点保存到帧缓冲中。对于每个顶点我们在通过对两列TAM列进行插值混合。一个简单的优化是组织每个顶点所在的三角形片面(没看懂...原文是： A
simple optimization is to organize the faces in triangle fans around
each vertex)。

## Tone thresholding
6路混合的方案虽然保持了连续性，但是会导致一些灰色笔触的出现。灰色的笔触在一些艺术风格中看起来是自然的，比如说铅笔画和木炭画，但是对于水墨画来说并不自然。为了更接近水墨画的效果，我们通过阈值来将色调颜色写入到帧缓冲中。对于一个渲染pass,我们可以配置多纹理寄存器，通过转换函数(8t-3.5)进行组合，来近似模拟阶梯函数。这个转换函数使新的笔画变得更长更黑而不是灰色。不幸的是，如附带的视频所示，它具有副作用：在笔触的边缘会出现锯齿，这可以通过超分辨采样来解决。

# Applying TAMs to arbitrary surfaces
对于参数化的表面，我们可以直接应用我们在第五章所提到的TAM渲染算法。比如，Figure3的球体和圆柱体使用一个环形的TAM进行平铺映射。但是，对于那些缺乏自然参数的表面，或者是太过扭曲的表面，我们使用lapped textures。

先复习一下，lapped textures由Praun等人[17]提出来，其不用获得表面的参数就能对任意形状的表面进行纹理映射。关键点就在于任何表面都可以被一组重叠的片面所覆盖。所有片面都被构造成带参数的，并且失真很小。对于许多纹理，包括我们的TAMs,不同片面之间的边界是看不到的，其被高频率出现的细节所覆盖。我们可以利用视觉遮蔽，构建带有不规则形状的边界的片面来使这些边界可见。

素描画很适合使用lapped textures，因为它缺少一些低频出现的细节（空白处会揭示片面的边界）。为了使用TAMs和lapped texture来渲染一个表面，我们将第五章的算法应用到每一个片面上，并让他们进行重叠，如下图：

![image](https://github.com/Richbabe/NPR_Lab/blob/master/Image/%E7%AC%AC%E5%85%AB%E5%91%A8%E5%91%A8%E6%8A%A5/%E8%AE%BA%E6%96%87%E6%88%AA%E5%9B%BE/fg3.png?raw=true)

我们修改了原来的lapped texture渲染算法，使其可以适应对于分开存放的素描纹理能够渲染他们之间的轮廓。当渲染一个片面时，我们首先渲染这个片面的轮廓，目的是把这个轮廓值存储到alpha通道中。接着，当使用TAM纹理对三角形片面进行渲染时，混合操作设置为加上已经渲染过的像素值并乘上存储在帧缓冲中的alpha值。（在这个过程中，帧缓冲中只有rgb通道被写入）

从素描纹理中得到片面的轮廓这一做法有很多优点：

（1）他可以减少内存消耗，因为我们只用储存一次轮廓，并且不用TAM每张纹理都复制。

（2）我们可以在一次渲染循环中对一个片面中的笔触进行旋转和缩放，而不用重新计算重叠参数。

（3）笔画纹理可以进行平铺来覆盖大面积的片面。由于轮廓和笔触的大小不一样，我们可以放置不同大小的片面在物体表面的不同部分上（小的片面放置在曲面处，大的片面放置在平面处）从而在只使用一个片面轮廓上达到更好的效果。在原lapped textures方法中，要有不同大小的片面必须要使用多个轮廓。

各项异性的重叠纹理需要一个方向场来进行参数化。在我们的项目中，方向场管理了表面笔触的方向。Girshick等人[3]观察得出笔触方向增强了观察者对于物体形状的感知。Hertzmann和Zorin[7]指出在许多绘画中素描笔触的方向是和表面曲率有关的，而细节是通过色调变化来表达的。因此，我们会计算简化版的网格表面的曲率，接着平滑方向场的结果并在原始网格上进行重新采样。对于粗糙网格，我们首先获得每个面相交的顶点的集合，然后我们通过这些顶点来拟合二次曲面，并最终计算得到二次曲面的曲率（即方向）。拟合的误差和曲率的比值也为计算方向提供了方法。现在我们已经为每个表面定义了方向，然后我们平滑向量场，将high-confidence区域的方向混合到low-confidence的区域。接着我们使用一个全局的非线性优化，正如Hertzmann和Zorin[7]提到那样，不过调整成180度对称而不是90度对称。

# Result and Discussion
![image](https://github.com/Richbabe/NPR_Lab/blob/master/Image/%E7%AC%AC%E5%85%AB%E5%91%A8%E5%91%A8%E6%8A%A5/%E8%AE%BA%E6%96%87%E6%88%AA%E5%9B%BE/Figure5.png?raw=true)

Figure5展示了使用我们算法渲染的6个模型，每个模型使用的TAM风格都不同。如下图所示：

![image](https://github.com/Richbabe/NPR_Lab/blob/master/Image/%E7%AC%AC%E5%85%AB%E5%91%A8%E5%91%A8%E6%8A%A5/%E8%AE%BA%E6%96%87%E6%88%AA%E5%9B%BE/fg4.png?raw=true)

通过简单修改TAM每一列的色调值并调整下色调转换函数，我们可以渲染出像图上上半部分的黑白蜡笔风格，或者渲染出下半部分的黑色刮板白色笔触风格。原始模型有着7500到15000个面，当使用lapped textures时会增加大概50%个面（因为 覆盖），我们的算法在跑这些模型的时候每秒大概20到40帧。为了让算法更高效我们将TAMS的贴图限定为灰度图。但是目前高性能显卡可以同时进行更多贴图映射所以可以将TAMS的贴图换成彩色图。

# 8 Summary and future work
我们介绍了TAM技术并展示了它可以实时渲染素描风格的场景。虽然我们的算法可以实时应用在许多复杂的模型上，但仍有一些可以改进的地方：
* Stroke growth

目前，TAM中的色调都是通过调整笔触数量来调节的，我们也可以考虑通过调整笔触长度和宽度来调节。

* Automatic indication

画家通常在一些选定区域绘制笔调来指示一些东西，如果加上这项技术那么会加强实时素描渲染的效果。

* More general TAMs

虽然这篇论文都在叙述素描画风格渲染，但是我们仍希望我们的框架能够渲染更多风格的NPR。除此之外，除了色调的其他轴也可以用来参数化art maps。一个有趣的例子就是可以构造依赖光照的贴图，比如通过相对光照的位置来调整的砖块（像[26]展示的那样）。

* View-dependent stroke direction

目前笔触的方向都是由TAMs过程中的参数决定，我们可以将笔触的方向设置为在运行时候随着观察点的移动而改变。

# Reference
[1] Praun E,Hoppe H,Webb M,et al.Real-time hatching[C]//Proceeding of the 28th annual conference on Computer graphics and interactive techniques ACM,2001:581