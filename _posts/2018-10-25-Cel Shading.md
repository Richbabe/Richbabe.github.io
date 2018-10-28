---
layout:     post
title:      Cel Shading
subtitle:   一种卡通着色技术
date:       2018-10-25
author:     Richbabe
header-img: img/okami.png
catalog: true
tags:
    - NPR实验室
---

> 中山大学 数据科学与计算机学院 软件工程数字媒体 洪鹏圳

---

# 前言
在非真实感渲染中，有两种经典的NPR着色方法，一种即是我们上次讲过的[Tone Based Shading](http://richbabe.top/2018/10/21/ToneBasedShading/)。另一种则是我们这次要讲的Cel Shading（[Wiki介绍](https://en.wikipedia.org/wiki/Cel_shading)）。由于Cel Shading对于美术来说更好控制渲染的效果，因此在现实中往往会采用Cel Shading来模拟卡通风格的渲染。

# 最原始的Cel Shading —— Antialiased Cel Shading

## 卡通渲染的特点
卡通渲染的特点通常有三个：
* 一般物体轮廓处有黑色描边
* 漫反射呈现明显的色块，而不是渐变
* 高光区域通常是一块突变的白色亮块

## 描边
在这里，我依旧使用过程式几何轮廓渲染方法来描边，即使用两个Pass，第一个Pass只渲染背面，把法线扁平化后再沿着法线方向扩张顶点，使得背部区域可见，再把这部分区域输出成轮廓线颜色即可。主要代码如下：

```
v2f vert (a2v v) {
    v2f o;

    float4 pos = mul(UNITY_MATRIX_MV, v.vertex); 
    float3 normal = mul((float3x3)UNITY_MATRIX_IT_MV, v.normal);  
    normal.z = -0.5;
    pos = pos + float4(normalize(normal), 0) * _Outline;
    o.pos = mul(UNITY_MATRIX_P, pos);

    return o;
}

float4 frag(v2f i) : SV_Target { 
    return float4(_OutlineColor.rgb, 1);               
}
```
第二个Pass即可以进行正常的渲染流程。这种方法简单而且对大部分模型都有比较好的健壮性，缺点是不适用于正方体这样扁平的表面和模型。 

## 漫反射部分
我们在上面提到过，卡通渲染的漫反射呈现明显的色块，而不是渐变[3]。这可以通过对法线和光源方向的点乘结果进行范围判断，使结果划分到固定的几个值（一般取三或四个值，模拟三层或者四层渐变）。例如我们可以在片元着色器中这样写：

```
fixed diff = dot(worldNormal, worldLightDir);
diff = diff * 0.5 + 0.5;
if (diff < _DiffuseSegment.x) {
    diff = _DiffuseSegment.x;
} 
else if (diff < _DiffuseSegment.y) {
    diff = _DiffuseSegment.y;
} 
else if (diff < _DiffuseSegment.z) {
    diff = _DiffuseSegment.z;
} 
else {
    diff = _DiffuseSegment.w;
}
```
其中，_DiffuseSegment为(0.1, 0.3, 0.6, 1.0)，表示漫反射系数的四个范围对应的四个值。在上面代码中，我们首先计算半兰伯特值diff，然后判断它的范围并进行修改，最后，整个模型表面的diff值实际只有4个不同的值，对应了_DiffuseSegment。然后，我们再根据这个diff值进行漫反射颜色的计算即可：

```
fixed3 texColor = tex2D(_MainTex, i.uv).rgb;
fixed3 diffuse = diff * _LightColor0.rgb * _DiffuseColor.rgb * texColor;
```
上面做法的问题在于，在分段的边界处会有明显的锯齿，这是因为从值_DiffuseSegment.x到_DiffuseSegment.y这样的变化是突变的。为了进行抗锯齿，我们可以使用fwidth函数：
```
//将漫反射系数划分到固定的4个值，实现卡通渲染中漫反射呈现色块
				fixed w = fwidth(diff) * 2.0;//计算领域内diff的梯度值w
				if (diff < _DiffuseSegment.x + w) {
					diff = lerp(_DiffuseSegment.x, _DiffuseSegment.y, smoothstep(_DiffuseSegment.x - w, _DiffuseSegment.x + w, diff));//根据diff在_DiffuseSegment.x+-w的范围内进行渐变混合
					//diff = lerp(_DiffuseSegment.x, _DiffuseSegment.y, clamp(0.5 * (diff - _DiffuseSegment.x) / w, 0, 1));
				}
				else if (diff < _DiffuseSegment.y + w) {
					diff = lerp(_DiffuseSegment.y, _DiffuseSegment.z, smoothstep(_DiffuseSegment.y - w, _DiffuseSegment.y + w, diff));//根据diff在_DiffuseSegment.y+-w的范围内进行渐变混合
					//diff = lerp(_DiffuseSegment.y, _DiffuseSegment.z, clamp(0.5 * (diff - _DiffuseSegment.y) / w, 0, 1));
				}
				else if (diff < _DiffuseSegment.z + w){
					diff = lerp(_DiffuseSegment.z, _DiffuseSegment.w, smoothstep(_DiffuseSegment.z - w, _DiffuseSegment.z + w, diff));//根据diff在_DiffuseSegment.z+-w的范围内进行渐变混合
					//diff = lerp(_DiffuseSegment.z, _DiffuseSegment.w, clamp(0.5 * (diff - _DiffuseSegment.z) / w, 0, 1));
				}
				else {
					diff = _DiffuseSegment.w;
				}
```
在上面的代码中，我们首先使用fwidth函数计算了邻域内diff的梯度值w，我们将据此在分段的边界处的+-w范围内进行渐变混合，这个混合值既可以使用smoothstep函数也可以通过clamp函数计算而得。

# 高光部分
对于高光区域的渲染也是类似的。我们首先计算得到高光反射因子，再判断它的范围，如果超过了某个值，对应了高光区域，否则值为0，没有任何高光。同样，为了进行抗锯齿，我们需要通过使用fwidth和smoothstep函数进行边界混合。主要代码如下：

```
fixed spec = max(0, dot(worldNormal, worldHalfDir));
spec = pow(spec, _Shininess);
w = fwidth(spec);
if (spec < _SpecularSegment + w) {
    spec = lerp(0, 1, smoothstep(_SpecularSegment - w, _SpecularSegment + w, spec));
} 
else {
    spec = 1;
}

fixed3 specular = spec * _LightColor0.rgb * _SpecularColor.rgb;

```

## 运行效果
将环境光，漫反射，高光混合，并加上阴影和光照衰减得到的结果如下：

![image](https://github.com/Richbabe/NPR_Lab/blob/master/Image/%E7%AC%AC%E4%BA%8C%E5%91%A8%E5%91%A8%E6%8A%A5/%E8%BF%90%E8%A1%8C%E6%95%88%E6%9E%9C%E6%88%AA%E5%9B%BE/AntialiasedCelShading.png?raw=true)

可以看到我们的效果图中有着明显的四条不同颜色的色带，这正是由于我们对漫反射结果进行范围划分得到的。

# 现代Cel Shading
由于上面讲的Antialiased Cel Shading需要我们在代码里自己修改范围值，所以很不方便。在现实中，我们往往会使用漫反射系数对一张一维纹理（通常是美术提供）进行采样，以控制漫反射的色调。这种技术在游戏《军团要塞2》（英文名:《TeamFortess 2》）中流行起来，它也是由Value公司（提出半兰伯特光照技术的公司）提出来的，他们使用这种技术来渲染游戏中具有卡通风格的角色。Value发表了一篇著名的论文来专门讲述在制作《军团要塞2》时使用的技术[1]。

## 主要方法
Cel Shading的基本思想是把色彩从多色阶降到低色阶，减少色阶的丰富程度，从而实现类似手工着色的效果，具体来说，可以使用如下的计算方法[4]：
![image](https://github.com/Richbabe/NPR_Lab/blob/master/Image/%E7%AC%AC%E4%BA%8C%E5%91%A8%E5%91%A8%E6%8A%A5/%E8%AE%BA%E6%96%87%E6%88%AA%E5%9B%BE/%E5%85%AC%E5%BC%8F.png?raw=true)

其中，Kd表示模型自身的贴图颜色，celCoord表示法线和光照方向的点积，用作一维色阶表的查找坐标，而palatteTex则是由美术绘制的一维色阶表，一般来说是由几个纯色色块组成的。如下图：
![image](https://github.com/Richbabe/NPR_Lab/blob/master/Image/%E7%AC%AC%E4%BA%8C%E5%91%A8%E5%91%A8%E6%8A%A5/%E8%AE%BA%E6%96%87%E6%88%AA%E5%9B%BE/pic1.png?raw=true)

## 漫反射部分
主要代码如下：

```
fixed diff = dot(worldNormal, worldLightDir);
diff = (diff * 0.5 + 0.5) * atten;

fixed3 diffuse = _LightColor0.rgb * albedo * tex2D(_Ramp, float2(diff, diff)).rgb;
```
diff为半兰伯特值，其范围为[0,1]，我们通过这个值构成一个纹理坐标，并用这个纹理坐标对渐变纹理_RampTex进行采样。 _RampTex纹理如下：
![image](https://github.com/Richbabe/NPR_Lab/blob/master/Image/%E7%AC%AC%E4%BA%8C%E5%91%A8%E5%91%A8%E6%8A%A5/%E8%AE%BA%E6%96%87%E6%88%AA%E5%9B%BE/Ramp_Texture0.png?raw=true)

由于_RampTex实际就是一个一维纹理（它在纵轴方向上颜色不变），因此纹理坐标的u和v方向我们都使用了halfLambert。然后把从渐变纹理采样得到的颜色和材质颜色、光的颜色相乘得到最终的漫反射颜色。

## 高光部分
对于卡通渲染需要的高光反射光照模型，我们同样需要计算normal和halfDir的点乘结果，但不同的是，我们把该值和一个阈值进行比较，如果小于该阈值，则高光反射系数为0，否则返回1。

```
float spec = dot(worldNormal,worldHalfDir);
spec = step(threshold,spec);
```
在上面的代码中，我们使用CG的step函数来实现和阈值比较的目的。step函数接受两个参数，第一个参数是参考值，第二个参数是待比较的数值。如果第二个参数大于等于第一个参数，则返回1，否则返回0。

但是，这种粗暴的判断方法会在高光区域的边界造成锯齿，如图14.3左图所示。
![image](https://github.com/Richbabe/NPR_Lab/blob/master/Image/%E7%AC%AC%E4%BA%8C%E5%91%A8%E5%91%A8%E6%8A%A5/%E8%AE%BA%E6%96%87%E6%88%AA%E5%9B%BE/pic3.png?raw=true)

出现这种问题的原因在于，高光区域的边缘不是平滑渐变的，而实由0突变到1。要想对其进行抗锯齿处理，我们可以在边界处很小的一块区域内，进行平滑处理。代码如下：

```
float spec = dot(worldNormal, worldHalfDir);
float w = fwidth(spec) * 2.0;
spec = lerp(0, 1, smoothstep(-w, w, spec + _SpecularScale - 1));
fixed3 specular = _Specular.rgb * spec * step(0.0001, _SpecularScale);
```
在上面的代码中，我们没有像之前一样直接使用step函数返回0或1。我们先用fwidth函数得到边界处领域像素之间的近似到数值。接着我们使用了CG的smoothstep函数。当spec + _SpecularScale - 1小于-w时，返回0，大于w时返回1，否则在0到1之间进行插值。这样的效果是，我们可以在[-w,w]区间内，即高光区域的边界处，得到一个从0到1平滑变化的spec值，从而实现抗锯齿的目的。

值得注意的是，我们在最后还乘了一个step(0.0001, _SpecularScale)，这是为了在_SpecularScale为9时，可以完全消除高光反射的光照。接着我们乘上高光颜色得到最终的镜面反射颜色。

## 运行效果
将环境光，漫反射，高光混合，并加上阴影和光照衰减得到的结果如下：

![image](https://github.com/Richbabe/NPR_Lab/blob/master/Image/%E7%AC%AC%E4%BA%8C%E5%91%A8%E5%91%A8%E6%8A%A5/%E8%BF%90%E8%A1%8C%E6%95%88%E6%9E%9C%E6%88%AA%E5%9B%BE/CelShading.png?raw=true)

可以看到我们的效果图中有着明显的三条不同颜色的色带，这正是由于我们对漫反射结果进行渐变纹理采样得到的。

# 对Cel Shading的改进
在上述的Cel Shading中，可以模拟卡通渲染的漫反射分量，却并没有考虑到视角相关的光照分量的模拟，因此很难实现类似菲涅尔效果的卡通渲染。实际上，也可以用类似的查找表的思路来实现视角相关光照分量的色阶离散化[5]，只需要将一维查找表扩展到二维即可：
![image](https://github.com/Richbabe/NPR_Lab/blob/master/Image/%E7%AC%AC%E4%BA%8C%E5%91%A8%E5%91%A8%E6%8A%A5/%E8%AE%BA%E6%96%87%E6%88%AA%E5%9B%BE/%E5%85%AC%E5%BC%8F2.png?raw=true)

相应的，查找坐标也扩展到了二维。
![image](https://github.com/Richbabe/NPR_Lab/blob/master/Image/%E7%AC%AC%E4%BA%8C%E5%91%A8%E5%91%A8%E6%8A%A5/%E8%AE%BA%E6%96%87%E6%88%AA%E5%9B%BE/%E6%94%B9%E8%BF%9B.png?raw=true)

# Reference
* [1] Jason Mitchell, Moby Francke, Dhabih Eng. Illustrative Rendering in Team Fortress 2
* [2] [Cel Shading - Wikipedia](https://en.wikipedia.org/wiki/Cel_shading)
* [3] [【NPR】卡通渲染](https://blog.csdn.net/candycat1992/article/details/50167285) 
* [4] [卡通渲染及其相关技术 - 知乎](https://zhuanlan.zhihu.com/p/26409746)
* [5] Pascal Barla, Joëlle Thollot, Lee Markosian. X-toon: an extended toon shader

# 结语
本博客的代码和资源均可在我的[github](https://github.com/Richbabe/NPR_Lab)上下载，别忘了点颗Star哟！
