---
layout:     post
title:     RealTimeHatching算法实现
subtitle:   
date:       2019-02-13
author:     Richbabe
header-img: img/okami.png
catalog: true
tags:
    - NPR实验室
---

> 中山大学 数据科学与计算机学院 软件工程数字媒体 洪鹏圳

---

# 前言
在上一篇博客：[Real-Time Hatching](http://richbabe.top/2019/01/18/Real-Time-Hatching/)中我们介绍了微软研究院的Praun等人在2001年的SIGGRAPH上发表的一篇非常著名的论文[1]。在这篇文章中，他们使用了提前生成的素描纹理来实现实时的素描风格渲染，这些纹理组成了一个色调艺术映射（Tonal Art Map,TAM）,如图所示：

![image](https://github.com/Richbabe/NPR_Lab/blob/master/Image/%E7%AC%AC%E4%B9%9D%E5%91%A8%E5%91%A8%E6%8A%A5/%E8%AE%BA%E6%96%87%E6%88%AA%E5%9B%BE/TAM.png?raw=true)

从左到右纹理的笔触逐渐增多，用于模拟不同光照下的漫反射效果，从上到下则对应了每张纹理的多级渐远纹理（mipmaps）。这些多级渐远纹理的生成并不是简单的对上一层纹理进行降采样，而是需要保持笔触之间的间隔，以便更真实地模拟素描效果。

# 实现
在这篇博客中我们将会简化论文中提出的算法，不考虑多级渐远纹理地生成，而是直接使用上图地6张素描纹理进行渲染。在渲染阶段，我们首先在顶点着色阶段计算逐顶点光照（漫反射系数），根据光照结果（漫反射系数）来决定这六张纹理的混合权重，并传递给片元着色器。然后，在片元着色器中根据这些权重来混合6张纹理的采样结果。

（1）首先，我们需要声明渲染所需的各个属性：

```
Properties {
	_Color ("Color", Color) = (1,1,1,1)  // 控制模型颜色的属性
	_TileFactor ("Tile Factor", Float) = 1  // 纹理平铺系数，_TileFactor越大，模型上素描线条越密
	_Outline ("Outline", Range(0, 1)) = 0.1  // 轮廓宽度
	_OutlineColor("Outline Color", Color) = (0, 0, 0, 1)  // 轮廓颜色
	// 6张素描纹理，线条密度依次增大
	_Hatch0("Hatch 0", 2D) = "white" {}
	_Hatch1("Hatch 1", 2D) = "white" {}
	_Hatch2("Hatch 2", 2D) = "white" {}
	_Hatch3("Hatch 3", 2D) = "white" {}
	_Hatch4("Hatch 4", 2D) = "white" {}
	_Hatch5("Hatch 5", 2D) = "white" {}
}
```
其中，_TileFactor是纹理的平铺系数，_TileFactor越大，模型上的素描线条越密。_Hatch0至_Hatch5对应了渲染时使用的6张素描纹理，它们的线条密度依次增大。

（2）描边我们使用的是之前介绍过的过程式几何轮廓渲染方法：

```
UsePass "NPR_Lab/ToneBasedShading/OUTLINE"
```

（3）定义顶点着色器输入和顶点着色器输出：

```
// 定义顶点着色器输入
struct a2v {
	float4 vertex : POSITION;
	float4 tangent : TANGENT;
	float3 normal : NORMAL;
	float2 texcoord : TEXCOORD0;
};

// 定义顶点着色器输出
struct v2f {
	float4 pos : SV_POSITION;
	float2 uv : TEXCOORD0;
	// 把6张纹理的6个混合权重保存再两个fixed3类型的变量中
	fixed3 hatchWeights0 : TEXCOORD1;
	fixed3 hatchWeights1 : TEXCOORD2;
	float3 worldPos : TEXCOORD3;
	SHADOW_COORDS(4)
};
```
需要注意的是，因为我们是在顶点着色器中计算6张素描纹理贴图的混合权重，因此需要在顶点着色器输出中添加混合权重的变量。由于一共是6张素描纹理贴图，每张贴图对应一个混合权重，因此我们一共要输出6个混合权重。我们把它们保存在两个fixed3变量（hatchWeights0和hatchWeights1）中。为了添加阴影效果，我们还声明了worldPos变量，并使用SHADOWC_COORDS宏声明了阴影纹理的采样坐标。

（4）接下来我们要定义顶点着色器。首先需要将顶点从模型空间变换到裁剪空间：

```
// 定义顶点着色器
v2f vert(a2v v) {
	v2f o;

	// 模型空间到裁剪空间的坐标变化
	o.pos = UnityObjectToClipPos(v.vertex);
```

接着我们使用_TileFactor得到纹理采样坐标：

```
// 使用TileFactor得到纹理采样坐标
o.uv = v.texcoord.xy * _TileFactor;
```

然后计算逐顶点光照所需的方向向量并计算对应的漫反射系数diff:

```
// 计算逐顶点光照所需的方向向量
fixed3 worldLightDir = normalize(WorldSpaceLightDir(v.vertex));  // 计算世界坐标系下的光照方向
fixed3 worldNormal = UnityObjectToWorldNormal(v.normal);    // 计算世界坐标系下的法线方向

// 计算逐顶点光照
fixed diff = max(0, dot(worldLightDir, worldNormal));  // 漫反射系数
```

接着把6个素描纹理贴图的权重初始化为0，并把漫反射系数的范围从[0,1]扩大到[0.7]得到hatchFactor：

```
// 初始化6个纹理混合权重为0
o.hatchWeights0 = fixed3(0, 0, 0);
o.hatchWeights1 = fixed3(0, 0, 0);

// 把漫反射系数diff的范围从[0,1]扩大到[0.7]得到hatchFactor
float hatchFactor = diff * 7.0;
```

接下来进行最重要的一步：我们把[0,7]的区间划分为7个子区间，通过判断hatchFactor所处的子区间来计算对应的纹理混合权重：

```
// 把[0,7]的区间均匀划分为7个子区间
// 通过判断hatchFactor所处的子区间来计算对应的纹理混合权重
// 权重计算是先计算上一张的权重，用1-上一张的权重得到下一张的权重
if (hatchFactor > 6.0) {
	// (6.0, 7.0]
	// 纯白，无笔触, 每张TAM权重都为0
}
else if (hatchFactor > 5.0) {
	// (5.0, 6.0]
	o.hatchWeights0.x = hatchFactor - 5.0;  // 计算第一张TAM的权重
}
else if (hatchFactor > 4.0) {
	// (4.0, 5.0]
	o.hatchWeights0.x = hatchFactor - 4.0;  // 计算第一张TAM的权重
	o.hatchWeights0.y = 1.0 - o.hatchWeights0.x;  // 计算第二张TAM的权重
}
else if (hatchFactor > 3.0) {
	// (3.0, 4.0]
	o.hatchWeights0.y = hatchFactor - 3.0;  // 计算第二张TAM的权重
	o.hatchWeights0.z = 1.0 - o.hatchWeights0.y;  // 计算第三张TAM的权重
}
else if (hatchFactor > 2.0) {
	// (2.0, 3.0]
	o.hatchWeights0.z = hatchFactor - 2.0;  // 计算第三张TAM的权重
	o.hatchWeights1.x = 1.0 - o.hatchWeights0.z;  // 计算第四张TAM的权重
}
else if (hatchFactor > 1.0) {
	// (1.0, 2.0]
	o.hatchWeights1.x = hatchFactor - 1.0;  // 计算第四张TAM的权重
	o.hatchWeights1.y = 1.0 - o.hatchWeights1.x;  // 计算第五张TAM的权重
}
else {
	// [0.0, 1.0]
	o.hatchWeights1.y = hatchFactor;  // 计算第五章TAM的权重
	o.hatchWeights1.z = 1.0 - o.hatchWeights1.y;  // 计算第六张TAM的权重
}
```
可以观察到我们权重的计算方法是先计算上一张贴图的权重，然后用1减去上一张贴图的权重来得到下一张贴图的权重，这样做有利于保持模型表面笔触变化的连续性。

最后，我们计算顶点在世界坐标系下的坐标，通过顶点世界坐标和TRANSFER_SHADOW宏来计算阴影纹理的采样坐标：

```
    // 计算顶点的世界坐标
	o.worldPos = mul(unity_ObjectToWorld, v.vertex).xyz;

	// 根据顶点世界坐标计算阴影纹理采样坐标
	TRANSFER_SHADOW(o);

	return o;
}
```

(5)定义片段着色器

在得到6张纹理贴图的混合权重后，我们对每张纹理进行采样并和它们对应的权重值相乘得到每张纹理的采样颜色：

```
// 定义片段着色器
fixed4 frag(v2f i) : SV_Target{
	// 对每张纹理采样并乘上权重值得到当前片段每张纹理的采样颜色
	fixed4 hatchTex0 = tex2D(_Hatch0, i.uv) * i.hatchWeights0.x;
	fixed4 hatchTex1 = tex2D(_Hatch1, i.uv) * i.hatchWeights0.y;
	fixed4 hatchTex2 = tex2D(_Hatch2, i.uv) * i.hatchWeights0.z;
	fixed4 hatchTex3 = tex2D(_Hatch3, i.uv) * i.hatchWeights1.x;
	fixed4 hatchTex4 = tex2D(_Hatch4, i.uv) * i.hatchWeights1.y;
	fixed4 hatchTex5 = tex2D(_Hatch5, i.uv) * i.hatchWeights1.z;
```

因为素描中往往有留白的部分，因此我们希望在最后的渲染中光照最亮的部分（即hatchFactor在(6.0,7.0]范围内的片段）添加上纯白色。所以我们还要计算纯白部分的混合权重，这是通过1减去所有6张纹理的权重之和得到的，用该权重乘上白色得到留白部分：

```
// 通过纯白的权重计算留白部分
fixed4 whiteColor = fixed4(1, 1, 1, 1) * (1 - i.hatchWeights0.x - i.hatchWeights0.y - i.hatchWeights0.z -
	i.hatchWeights1.x - i.hatchWeights1.y - i.hatchWeights1.z);
```

最后，我们混合各个颜色值，并和阴影值atten，模型颜色_Color相乘后返回最终的渲染结果：

```
    // 混合各个颜色值
	fixed4 hatchColor = hatchTex0 + hatchTex1 + hatchTex2 + hatchTex3 + hatchTex4 + hatchTex5 + whiteColor;

	// 计算光照衰减和阴影
	UNITY_LIGHT_ATTENUATION(atten, i, i.worldPos);

	// 返回最终渲染结果
	return fixed4(hatchColor.rgb * _Color.rgb * atten, 1.0);
}
```

# 运行结果
![image](https://github.com/Richbabe/NPR_Lab/blob/master/Image/%E7%AC%AC%E4%B9%9D%E5%91%A8%E5%91%A8%E6%8A%A5/%E8%BF%90%E8%A1%8C%E6%95%88%E6%9E%9C%E6%88%AA%E5%9B%BE/result.png?raw=true)
从运行结果可以看到，我们成功混合了六张纹理贴图，并在最亮处留白（如球体的右上角部分）。

# Reference
* [1] Praun E,Hoppe H,Webb M,et al.Real-time hatching[C]//Proceeding of the 28th annual conference on Computer graphics and interactive techniques ACM,2001:581
* [2] Unity Shader入门精要

# 结语
本博客的代码和资源均可在我的[github](https://github.com/Richbabe/NPR_Lab)上下载，别忘了点颗Star哟！


