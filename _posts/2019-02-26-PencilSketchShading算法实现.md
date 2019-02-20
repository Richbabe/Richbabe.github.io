---
layout:     post
title:      PencilSketchShading算法实现
subtitle:   
date:       2019-02-26
author:     Richbabe
header-img: img/okami.png
catalog: true
tags:
    - NPR实验室
---

> 中山大学 数据科学与计算机学院 软件工程数字媒体 洪鹏圳

---

# 前言
在上一篇博客：[Stylized Rendering Techniques For Scalable Real-Time 3D Animation](http://richbabe.top/2019/02/18/Stylized-Rendering-Techniques-For-Scalable-Real-Time-3D-Animation/)中，我们展示了Lake等人提出的卡通渲染算法。其中，Pencil Sketch Shading是他们提出的铅笔画渲染算法。这篇博客将会简单实现论文中的铅笔画算法，并与之前的[Real-Time Hatching](http://richbabe.top/2019/02/13/RealTimeHatching%E7%AE%97%E6%B3%95%E5%AE%9E%E7%8E%B0/)作比较。

# 实现
在实现之前，我们先看看论文中PencilSketchShading算法的具体步骤：
![image](https://github.com/Richbabe/NPR_Lab/blob/master/Image/%E7%AC%AC10%E5%91%A8%E5%91%A8%E6%8A%A5/%E8%AE%BA%E6%96%87%E6%88%AA%E5%9B%BE/PencilSketchAlgorithm.png?raw=true)

在预处理阶段要求我们读入一个由铅笔笔触纹理组成的集合，并要求按照笔触密度的增加，纹理等级降低。在这里我还是使用之前[Real-Time Hatching](http://richbabe.top/2019/02/13/RealTimeHatching%E7%AE%97%E6%B3%95%E5%AE%9E%E7%8E%B0/)的6张笔触纹理图：


```
Properties {
	_Color("Diffuse Color", Color) = (1, 1, 1, 1)
	_Outline("Outline", Range(0.001, 1)) = 0.1
	_OutlineColor("Outline Color", Color) = (0, 0, 0, 1)
	_TileFactor("Tile Factor", Range(1, 10)) = 5 // 纹理平铺系数，_TileFactor越大，模型上素描线条越密
	// 6张素描纹理，线条密度依次减小
	_Level1("Level 1 (Darkest)", 2D) = "white" {}
	_Level2("Level 2 ", 2D) = "white" {}
	_Level3("Level 3 ", 2D) = "white" {}
	_Level4("Level 4 ", 2D) = "white" {}
	_Level5("Level 5 ", 2D) = "white" {}
	_Level6("Level 6 ", 2D) = "white" {}
}
```

![image](https://github.com/Richbabe/NPR_Lab/blob/master/Image/%E7%AC%AC11%E5%91%A8%E5%91%A8%E6%8A%A5/%E8%BF%90%E8%A1%8C%E7%BB%93%E6%9E%9C%E6%88%AA%E5%9B%BE/6%E5%BC%A0%E7%BA%B9%E7%90%86.png?raw=true)

接着要求我们用正交投影来对背景纸纹理进行纹理映射，具体的顶点着色器和像素着色器代码如下：

```
// 定义顶点着色器
v2f vert(a2v v) {
	v2f o;

	o.pos = UnityObjectToClipPos(v.vertex);
	o.scrPos = ComputeScreenPos(o.pos);  // 计算屏幕空间坐标

	return o;
}

// 定义像素着色器
float4 frag(v2f i) : COLOR{
	fixed2 scrPos = i.scrPos.xy / i.scrPos.w;  // 获取屏幕空间uv坐标
	fixed3 fragColor = tex2D(_MainTex, scrPos);  // 用屏幕空间uv坐标进行采样

	fragColor *= _Color.rgb;

	return fixed4(fragColor, 1.0);
}
```
可以看到我们是根据屏幕空间的uv坐标对纸张纹理进行采样的。

接着我们就要进行最重要的步骤：根据漫反射系数选择对应的笔画纹理进行映射。

因此我们需要先计算漫反射系数，片段着色器代码如下：

```
UNITY_LIGHT_ATTENUATION(atten, i, i.worldPos);  // 计算阴影

fixed diff = (dot(worldNormal, worldLightDir) * 0.5 + 0.5) * atten * 6.0;  // 计算漫反射系数
```
可以看到与前[Real-Time Hatching](http://richbabe.top/2019/02/13/RealTimeHatching%E7%AE%97%E6%B3%95%E5%AE%9E%E7%8E%B0/)不同的是我们在漫反射系数这里乘多了个阴影系数，这是因为PencilSketchShading的阴影是用笔触纹理渲染的。如果该区域是阴影，则漫反射系数为0，此时会选择笔触密度最大的纹理进行纹理映射。而在[Real-Time Hatching](http://richbabe.top/2019/02/13/RealTimeHatching%E7%AE%97%E6%B3%95%E5%AE%9E%E7%8E%B0/)中阴影是在最后输出颜色那里乘上的，这就导致[Real-Time Hatching](http://richbabe.top/2019/02/13/RealTimeHatching%E7%AE%97%E6%B3%95%E5%AE%9E%E7%8E%B0/)的阴影是纯黑的。简而言之，PencilSketchShading算法比[Real-Time Hatching](http://richbabe.top/2019/02/13/RealTimeHatching%E7%AE%97%E6%B3%95%E5%AE%9E%E7%8E%B0/)在阴影部分多了阴影的铅笔画风格。

接着，我们用计算得到的漫反射系数来划分区间，这一步和[Real-Time Hatching](http://richbabe.top/2019/02/13/RealTimeHatching%E7%AE%97%E6%B3%95%E5%AE%9E%E7%8E%B0/)是一样的，然后根据选择区间对应的纹理用屏幕空间uv坐标值来进行纹理映射，最后输出纹理采样结果和物体本身颜色的乘积，具体片段着色器代码如下：

```
fixed3 fragColor;
// 根据屏幕空间uv坐标进行纹理采样
if (diff < 1.0) {
	fragColor = tex2D(_Level1, scrPos).rgb;
}
else if (diff < 2.0) {
	fragColor = tex2D(_Level2, scrPos).rgb;
}
else if (diff < 3.0) {
	fragColor = tex2D(_Level3, scrPos).rgb;
}
else if (diff < 4.0) {
	fragColor = tex2D(_Level4, scrPos).rgb;
}
else if (diff < 5.0) {
	fragColor = tex2D(_Level5, scrPos).rgb;
}
else {
	fragColor = tex2D(_Level6, scrPos).rgb;
}

fragColor *= _Color.rgb * _LightColor0.rgb;

return fixed4(fragColor, 1.0);
```

可以看到我们是用屏幕空间的uv坐标来对纹理进行采样的，具体的原理如下：

![image](https://github.com/Richbabe/NPR_Lab/blob/master/Image/%E7%AC%AC10%E5%91%A8%E5%91%A8%E6%8A%A5/%E8%AE%BA%E6%96%87%E6%88%AA%E5%9B%BE/Figure5.png?raw=true)

看起来我们是直接将一张纹理“投影”到模型上。

而我们之前的[Real-Time Hatching](http://richbabe.top/2019/02/13/RealTimeHatching%E7%AE%97%E6%B3%95%E5%AE%9E%E7%8E%B0/)是使用纹理坐标进行纹理映射，采样结果通过纹理混合进行叠加，在计算上，PencilSketchShading的开销显然要更小一点。

# 运行结果
使用PencilSketchShading算法的运行结果如下：
![image](https://github.com/Richbabe/NPR_Lab/blob/master/Image/%E7%AC%AC11%E5%91%A8%E5%91%A8%E6%8A%A5/%E8%BF%90%E8%A1%8C%E7%BB%93%E6%9E%9C%E6%88%AA%E5%9B%BE/%E8%BF%90%E8%A1%8C%E6%95%88%E6%9E%9C.png?raw=true)

而如果使用Real-Time Hatching算法，运行的效果是这样的：
![image](https://github.com/Richbabe/NPR_Lab/blob/master/Image/%E7%AC%AC11%E5%91%A8%E5%91%A8%E6%8A%A5/%E8%BF%90%E8%A1%8C%E7%BB%93%E6%9E%9C%E6%88%AA%E5%9B%BE/Hatching.png?raw=true)

可以看到，在阴影部分Real-Time Hatching算法是完全黑色的，而PencilSketchShading算法则是用较密笔触进行绘制。在高光部分Real-Time Hatching算法是留白的，而PencilSketchShading算法则是用较少的线条进行绘制。

Real-Time Hatching是模拟素描风格的非真实感渲染，PencilSketchShading是模拟铅笔画风格的非真实感渲染，至于哪种效果好还是就见仁见智了~

# Reference
[1] Lake A, Marshall C, Harris M, et al. Stylized rendering techniques for scalable real-time 3D animation[C]//Proceedings of the 1st international symposium on Non-photorealistic animation and rendering. ACM, 2000: 13-2

# 结语
本博客的代码和资源均可在我的[github](https://github.com/Richbabe/NPR_Lab)上下载，别忘了点颗Star哟！