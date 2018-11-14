---
layout:     post
title:      The summary of silhouette drawing
subtitle:   轮廓线渲染方法总结
date:       2018-11-14
author:     Richbabe
header-img: img/okami.jpg
catalog: true
tags:
    - NPR实验室
---

> 中山大学 数据科学与计算机学院 软件工程数字媒体 洪鹏圳
---

# 前言
众所周知，在非真实感渲染领域的卡通渲染中，轮廓线的绘制起着至关重要的作用，只有加了轮廓的渲染才称得上是真正的卡通渲染。近20年来，有许多绘制模型轮廓线的方法被先后提出来。在《Real Time Rendering,third edition》[1]一书中，作者把这些方法分成了5种类型。

所用到的测试场景如下：
![image](https://github.com/Richbabe/NPR_Lab/blob/master/Image/%E7%AC%AC%E4%B8%89%E5%91%A8%E5%91%A8%E6%8A%A5/%E6%B5%8B%E8%AF%95%E5%9C%BA%E6%99%AF.png?raw=true)

其中，左右两边的模型是low polygon类型，即表面比较平坦；中间模型表面变化平缓

# Surface Angle Silhouette
## 介绍
基于观察角度和表面法线的轮廓线渲染。这种方法使用视角方向和表面法线的点乘结果来得到轮廓的信息。这种方法简单快速，可以在一个Pass中就得到渲染结果，但局限性很大，很多模型渲染出来的描边效果都不尽人意。

具体方法如下：

利用视角方向和表面法线的点乘结果得到轮廓线信息。这个值越接近0，说明离轮廓线越近。

这个技术相当于使用一个Spherical environment map（EM）来对整个surface进行渲染。如下图所示（来源：《Real-time Rendering, third edition》）：
![image](https://github.com/Richbabe/NPR_Lab/blob/master/Image/%E7%AC%AC%E4%B8%89%E5%91%A8%E5%91%A8%E6%8A%A5/p1.png?raw=true)

在实际应用中，我们通常使用一张一维纹理来模拟，即使用视角方向和顶点法向的点乘对该纹理进行采样。

## 实践
下面我将展示两种方法实现基于观察角度和表面法线的轮廓线渲染。一种方法是使用一个参数_Outline来控制轮廓线宽度；另一种方式是使用了一张一维纹理来控制轮廓线宽度。总的代码如下：

```
Shader "NPR_Lab/SurfaceAngleSilhouette" {
	Properties {
		_Color("Diffuse Color", Color) = (1, 1, 1, 1)
		_MainTex ("Albedo (RGB)", 2D) = "white" {}
		_Outline ("Outline", Range(0,1)) = 0.4
		_SilhouetteTex ("Silhouette Texture", 2D) = "White" {}
	}
	SubShader {
		Pass{
			Tags{ "RenderType" = "Opaque" }
			LOD 200

			CGPROGRAM
			#include "UnityCG.cginc"
			#include "Lighting.cginc"  
			#include "AutoLight.cginc" 

			#pragma vertex vert
			#pragma fragment frag

			fixed4 _Color;
			sampler2D _MainTex;
			float4 _MainTex_ST;
			float _Outline;
			sampler2D _SilhouetteTex;

			struct v2f {
				float4 pos : SV_POSITION;
				float2 uv : TEXCOORD0;
				fixed3 worldNormal : TEXCOORD1;
				float3 worldPos : TEXCOORD2;
			};

			//顶点着色器
			v2f vert(appdata_full v) {
				v2f o;

				o.pos = UnityObjectToClipPos(v.vertex);
				o.worldNormal = mul(v.normal, (float3x3)unity_WorldToObject);
				o.worldPos = mul(unity_ObjectToWorld, v.vertex).xyz;
				o.uv = TRANSFORM_TEX(v.texcoord, _MainTex);

				return o;
			}

			//通过一个参数_Outline作为阈值来控制轮廓线宽度
			fixed3 GetSilhouetteUseConstant(fixed3 normal, fixed3 viewDir) {
				fixed edge = saturate(dot(normal, viewDir));
				edge = edge < _Outline ? edge / 4 : 1;

				return fixed3(edge, edge, edge);
			}

			//通过一张一维纹理来控制轮廓线宽度
			fixed3 GetSilhouetteUseTexture(fixed3 normal, fixed3 viewDir) {
				fixed edge = dot(normal, viewDir);
				edge = edge * 0.5 + 0.5;//从[-1,1]转化到[0,1]作为纹理采样坐标
				return tex2D(_SilhouetteTex, fixed2(edge, edge)).rgb;
			}

			//片段着色器
			fixed4 frag(v2f i) : SV_Target{
				fixed3 worldNormal = normalize(i.worldNormal);
				fixed3 worldViewDir = UnityWorldSpaceViewDir(i.worldPos);

				fixed3 col = tex2D(_MainTex, i.uv).rgb;

				//通过一个参数_Outline作为阈值来控制轮廓线宽度
				//fixed3 silhouetteColor = GetSilhouetteUseConstant(worldNormal, worldViewDir);

				//通过一张一维纹理来控制轮廓线宽度
				fixed3 silhouetteColor = GetSilhouetteUseTexture(worldNormal, worldViewDir);

				fixed4 fragColor;
				fragColor.rgb = col * silhouetteColor;//混合模型颜色和边缘颜色
				fragColor.a = 1;

				return fragColor;
			}

			ENDCG
		}
	}
	FallBack "Diffuse"
}

```
代码分析：
* GetSilhouetteUseConstant函数对应了第一种方法，即使用一个参数_Outline来控制轮廓线宽度。这种方法是先得到当前片段的法线方向和观察方向的点乘结果。如果该结果小于阈值_Outline，则认为该片段是边缘点，让它除以4（一个实验值）来减少它的值得到黑色；否则让它等于1，即没有任何效果。
* GetSilhouetteUseTexture函数对应了第二种方法，即使用了一张一维纹理来控制轮廓线宽度。这种方法是先得到当前片段的法线方向和观察方向的点乘结果，并用该结果作为一维纹理的纹理采样坐标（需要转化到[0,1]范围内）进行采样，返回颜色值。
* 在片段着色器中将模型原来的颜色和边缘线颜色混合即可得到带轮廓的效果。

上面使用了纯色进行颜色渲染，没要考虑光照效果，使用的一维纹理如下：
![image](https://github.com/Richbabe/NPR_Lab/blob/master/Image/%E7%AC%AC%E4%B8%89%E5%91%A8%E5%91%A8%E6%8A%A5/p2.png?raw=true)

## 运行效果
运行效果如下：

![image](https://github.com/Richbabe/NPR_Lab/blob/master/Image/%E7%AC%AC%E4%B8%89%E5%91%A8%E5%91%A8%E6%8A%A5/p3.png?raw=true)

可以看出来对于左右模型这样的模型，这种方法的轮廓效果很难控制。有的地方轮廓很宽，有的地方却又捕捉不到。 

## 优缺点总结
基于观察角度和表面法线的轮廓线渲染的优点和缺点总结：
* 优点：非常简单快速，我们可以在一个pass里就得到结果，而且还可以使用texture filtering对轮廓线进行抗锯齿。 
* 缺点：存在局限性，只适用于某些模型，而对于像cube这样的平坦、棱角分明的模型就会有问题。虽然我们可以使用一些变量来控制轮廓线的宽度（如果使用纹理的话就是纹理中黑色的宽度），但实际的效果是依赖于表面的曲率（curvature）的。对于像cube这样表面非常平坦的物体，它的轮廓线会发生突变，要么没有，要么就全黑。

    ![image](https://github.com/Richbabe/NPR_Lab/blob/master/Image/%E7%AC%AC%E4%B8%89%E5%91%A8%E5%91%A8%E6%8A%A5/p4.png?raw=true)
    
    因为我们采用了顶点法向量来判断边界，所以对于正方体这种法线固定单一的情况，判断出来的边界要么基本不存在要么就大的离谱对于这样的对象，一个更好的方法是用Pixel&Fragment Shader、经过两个Pass渲染描边：第一个Pass，我们只渲染背面的网格，在它们的周围进行描边；第二个Pass中，再正常渲染正面的网格。其实，这是符合我们对于边界的认知的，我们看见的物体也都是看到了它们的正面而已。

游戏[《Cel Damage》](https://en.wikipedia.org/wiki/Cel_Damage)[2]的作者Wu发现，在他们的游戏中，这种技术只适用于1/4的模型。可以看出，这种技术局限性还是很大的。 

# Procedural Geometry Silhouette
## 介绍
过程式几何轮廓线渲染。这种方法的核心是使用两个Pass渲染。第一个Pass正常渲染正面的面片;第二个Pass渲染背面的面片，并使用某些技术让它的轮廓可见。

渲染背面的方法有很多，下面列举一些可以通过渲染背面来显示轮廓的方法：

1. 只渲染backfaces的edges（可以理解成把渲染模式设置为DRAW_EDGE），然后使用一些biasing等技术来保证这些线会在frontfaces的前面渲染。
2. Z-bias方法。把backfaces渲染成黑色，然后在屏幕空间的z方向上向前移动它们，使其可见。移动的距离可以是一个固定值，或其他适应后的值。 

    缺点：不能创建宽度相同的轮廓，因为frontface和backface的夹角不一样。可控性很弱。想象一个向里凹陷的物体，这种方法得到的背面将完全覆盖掉正面图形。
3. Triangle Fattening。也就是说，把每个backface triangle的edges都“变胖”一定程度，使其在视角空间中看起来宽度是一致的。

    缺点：对于一些瘦长的triangles来说，它的corner也会变得很细长。一种解决方法是可以把扩展后的edges链接在一起形成斜接在一起的corners。如下图（来源：《Real-time Rendering, third edition》）：
    
    ![image](https://github.com/Richbabe/NPR_Lab/blob/master/Image/%E7%AC%AC%E4%B8%89%E5%91%A8%E5%91%A8%E6%8A%A5/p5.png?raw=true)

    而且，这种方法无法应用在GPU生成的一些curved surfaces上（因为没有edges）。
4.  Shell or halo method(vertex normal)。把backface的顶点沿着顶点法线方向向外扩张。因为这种方法很像在模型外面又包裹了一层壳，所以叫做shell method。我们在[ToneBasedShading](http://richbabe.top/2018/10/21/ToneBasedShading/)中所用的就是这种方法。但是，正如在[ToneBasedShading](http://richbabe.top/2018/10/21/ToneBasedShading/)中提到的一样，仅仅这样做会根据轮廓线宽度造成模型穿透问题。为了解决这个问题，我们可以把背面扁平化。

    优点：很快速，可以在vertex shader中就完成，而且具有一定的健壮性。游戏[《Cel Damage》](https://en.wikipedia.org/wiki/Cel_Damage)[2]就是使用了这种技术。
    
    缺点：像cube这样的模型，它的同一个顶点在不同面上具有不同的顶点法向，所以向外扩张后会形成一个gaps。一种解决方法是，强迫同一个位置的顶点具有相同的法向（在建模时设置，把边设置成hard edge或soft edge）；另一种方法是在这些轮廓处创建额外的网格结构。

## 优缺点总结
上面所列出的所有方法都有一个共同的缺点，那就是对轮廓线的外观可控性很少，而且如果没有进行一些反锯齿操作，轮廓线看起来锯齿比较严重。

但它们一个非常吸引人的优点就是，不需要任何关于相邻顶点/边等信息，所有的处理都是独立的，因此从速度上来说很快。

## 实践
下面只给出z-bias和vertex normal的方法。

关于vertex normal的实现，网上有一个主流版本就是[Unity Wiki的Silhouette-Outlined_Diffuse Shader](http://wiki.unity3d.com/index.php/Silhouette-Outlined_Diffuse)。原Shader有两个地方需要注意，首先对顶点只在xy方向上扩张，这种操作在模型全是外凸的情况下基本没有什么问题，但是，如果一个模型有内凹的部分就有可能出现轮廓线挡住frontfaces的情况；另一点是，原Shader在扩张顶点时考虑了顶点在投影矩阵中的深度值，这意味着模型轮廓线的宽度会随着摄像机移动而改变。当然，不想这样的话可以去掉。

还有一些变种，例如如果我们想要实现宽度不一的轮廓线，可以利用顶点颜色作为一个参数来控制宽度。可以参见这篇博客：[在Unity中使用顶点颜色控制轮廓线厚度](http://yrkhnshk.hatenablog.com/entry/%E3%80%90Unity%E3%80%91%E9%A0%82%E7%82%B9%E3%82%AB%E3%83%A9%E3%83%BC%E3%81%A7%E8%BC%AA%E9%83%AD%E7%B7%9A%E3%81%AE%E5%A4%AA%E3%81%95%E3%82%92%E5%88%B6%E5%BE%A1%E3%81%99%E3%82%8B)。


```
Shader "Silhouette/Procedural Geometry Silhouette" {
    Properties {
        _MainTex ("Base (RGB)", 2D) = "white" {}
        _Outline ("Outline", Range(0,1)) = 0.1
    }
    SubShader {
        Tags { "RenderType"="Opaque" }
        LOD 200

        Pass {
            Tags { "LightMode"="ForwardBase" } 

            Cull Back   
            Lighting On  

            CGPROGRAM
            #include "UnityCG.cginc"
            #include "Lighting.cginc"  
            #include "AutoLight.cginc" 

            #pragma vertex vert
            #pragma fragment frag

            sampler2D _MainTex;

            struct v2f {
                float4 pos : SV_POSITION;   
                float2 uv : TEXCOORD0;
            };

            v2f vert(appdata_full i) {
                v2f o;
                o.pos= mul(UNITY_MATRIX_MVP, i.vertex);
                o.uv = i.texcoord;

                TRANSFER_VERTEX_TO_FRAGMENT(o); 

                return o;
            }

            fixed4 frag(v2f i) : COLOR {
                fixed3 col = tex2D(_MainTex, i.uv).rgb; 

                fixed4 fragColor;
                fragColor.rgb = col;
                fragColor.a = 1.0;

                return fragColor;
            }

            ENDCG
        }

        Pass {
            Tags { "LightMode"="ForwardBase" } 

            Cull Front   
            Lighting Off

            CGPROGRAM
            #include "UnityCG.cginc"
            #include "Lighting.cginc"  
            #include "AutoLight.cginc" 

            #pragma vertex vert
            #pragma fragment frag

            sampler2D _MainTex;
            float _Outline;

            struct v2f {
                float4 pos : SV_POSITION;   
                float2 uv : TEXCOORD0;
            };

            //z-bias方法
            void ZBiasMethod(appdata_full i, inout v2f o) {
                float4 viewPos = mul(UNITY_MATRIX_MV, i.vertex);
                viewPos.z += _Outline;

                o.pos = mul(UNITY_MATRIX_P, viewPos);
            }

            //vertexNormal方法1
            void VertexNormalMethod0(appdata_full i, inout v2f o) {
                o.pos = mul(UNITY_MATRIX_MVP, i.vertex);

                float3 normal = mul ((float3x3)UNITY_MATRIX_IT_MV, i.normal);
                float2 offset = TransformViewToProjection(normal.xy);

                // Only modify the xy components
                // Multiply o.pos.z, as a result the width of your outline will depend on the distance from viewer
                o.pos.xy += offset * o.pos.z * _Outline;
            }

            //vertexNormal方法2
            void VertexNormalMethod1(appdata_full i, inout v2f o) {
                float4 viewPos = mul(UNITY_MATRIX_MV, i.vertex);

                float3 normal = mul( (float3x3)UNITY_MATRIX_IT_MV, i.normal);
                // This is a tricky operation
                // The value of z avoid the expended backfaces to intersect with frontfaces
                // When z = 0.0,  it is equal to VertexNormalMethod0
                normal.z = -1.0;  
                viewPos = viewPos + float4(normalize(normal),0) * _Outline;  

                o.pos = mul(UNITY_MATRIX_P, viewPos);
            }

            v2f vert(appdata_full i) {
                v2f o;

                //ZBiasMethod(i, o);
                //VertexNormalMethod0(i, o);
                VertexNormalMethod1(i, o);

                o.uv = i.texcoord;

                TRANSFER_VERTEX_TO_FRAGMENT(o); 

                return o;
            }

            fixed4 frag(v2f i) : COLOR {
                return fixed4(0, 0, 0, 1);  
            }

            ENDCG
        }
    }
    FallBack "Diffuse"
}
```
代码分析：
* ZBiasMethod函数对应了z-bias方法，他是将顶点转换到视角空间后令坐标的z值偏移_Outline值，再转换到裁剪空间。
* VertexNormalMethod0函数对应了[Unity Wiki的Silhouette-Outlined_Diffuse Shader](http://wiki.unity3d.com/index.php/Silhouette-Outlined_Diffuse)的方法，其是在裁剪空间下只将顶点的xy方向沿着法线方向扩张，且扩张的值（即轮廓线的宽度）收到顶点深度值的影响。
* VertexNormalMethod1函数对应了我在[ToneBasedShading](http://richbabe.top/2018/10/21/ToneBasedShading/)中使用的方法。在顶点着色器中我们首先把顶点和法线变换到视角空间下，这是为了让描边可以在视角空间达到最好的效果。随后，我们设置法线的z分量，对其归一化后再将顶点沿其方向扩张（不同于VertexNormalMethod0，VertexNormalMethod1是xyz均扩张），得到扩张后的顶点坐标。对法线的处理是为了尽可能避免背面扩张后的顶点挡住正面的面片。最后，我们把顶点从视角空间转换到裁剪空间。

## 运行效果
使用了z-bias方法的运行效果如下:
![image](https://github.com/Richbabe/NPR_Lab/blob/master/Image/%E7%AC%AC%E4%B8%89%E5%91%A8%E5%91%A8%E6%8A%A5/p6.png?raw=true)

可以看出很多地方不是很理想，当然这里给出的是最简单的移动固定值的方法。有很多技术可以改善效果。

使用了vertex Normal（VertexNormalMethod1）的效果： 

![image](https://github.com/Richbabe/NPR_Lab/blob/master/Image/%E7%AC%AC%E4%B8%89%E5%91%A8%E5%91%A8%E6%8A%A5/P7.png?raw=true)

注意其中左边模型的头顶和中间模型的脚部，由于它们在同一个顶点处没有使用相同的法线所以出现了gaps。当调整VertexNormalMethod1中对normal.z的赋值时，可以发现，z越大，轮廓线越细，发生模型遮挡问题的可能性越小。

# Silhouetting by Image Processing
## 介绍
基于图像处理的轮廓线渲染。这种技术利用了G-buffer，在每个buffer中使用了图像处理的技术来检测轮廓信息。

G-buffer的提出是用于延迟渲染（deferred shading）的，而近年来被一些学者扩充到NPR的领域。

基本思想是，利用图像处理中的一些算法，在Z-buffer中找到不连续地方，就可以找到大部分轮廓线了。还可以在surface normal中找到不连续点，来找到更完整的轮廓线。最后还可以在ambient colors中，进一步完备前两步找到的轮廓线信息。

因此，基本步骤是： 
1. 使用vertex shader把world space normals和z-depths渲染到纹理中。 
2. 使用一些滤波算法来找到边缘信息。一种常见的滤波算子是Sobel边缘检测算子。 
3. 找到边缘后，我们还可以使用一些图像处理操作，例如腐蚀和膨胀，来扩展或者缩小轮廓线宽度。

## 实践
可以看我这篇博客：[基于图像处理的轮廓线渲染](http://richbabe.top/2018/11/07/%E5%9F%BA%E4%BA%8E%E5%9B%BE%E5%83%8F%E5%A4%84%E7%90%86%E7%9A%84%E8%BD%AE%E5%BB%93%E7%BA%BF%E6%B8%B2%E6%9F%93//)

也可以看看Unity官方给的边缘检测实现：[Edge Detect Effect Normals](http://docs.unity3d.com/Manual/script-EdgeDetectEffectNormals.html)，直接使用效果如下：

![image](https://github.com/Richbabe/NPR_Lab/blob/master/Image/%E7%AC%AC%E4%B8%89%E5%91%A8%E5%91%A8%E6%8A%A5/p8.png?raw=true)

可以看到锯齿还是比较严重的，如果想去除，可以考虑其他屏幕后处理效果中的反锯齿操作。

## 优缺点总结
这种方法的优点在于： 
* 适用于任何种类的模型 
* 而且不需要CPU参与创建和传递一些边的信息。

但也有它的缺点： 
* 首先，这种对z-depth比较来检测边界的方法，会受z变化范围的影响，一些z变化很小的轮廓就无法检测出来。例如，桌子上的纸张。 
* 同样，纸张的normal map filter同样会不起作用，因为纸张表面的normal都是一样的。

对上述问题的改进，一种解决方法是在物体颜色上再添加一层滤波。但是，也不是万能解决方法。例如，如果一张纸自己折叠了一下，我们仍然无法检测出来这个折痕。 

# Silhouette Edge Detection
## 介绍
基于轮廓边检测的轮廓线渲染。上面提到的各种方法，一个最大的问题是，无法控制轮廓线的风格渲染。对于一些情况，我们希望可以渲染出独特风格的轮廓线，例如水墨风格等等。

为此，我们希望可以检测出精确的轮廓边，然后直接渲染它们。检测一条边是否是轮廓边的公式很简单，我们只需要检查和这条边相邻的两个三角面片是否满足以下条件：

(n0⋅v > 0) != (n1⋅v > 0)

其中n0和n1表示三角面片的法向，v是从视角到该边上任意顶点的方向。即，检查它们是否一个朝正面，一个朝背面。

标准方法： 为了找到这些silhouette edges，标准方法是循环遍历模型所有的edges，然后进行上述检查。

优化：标准方法显然要求的工作量比较大，很多学者提出了优化的方法。优化的方法如下：
* 一种优化是，在同一个平面上的edges可以直接跳过检查。也就是说，如果这两个三角面片的法向相同，那么就不需要检查这条edge了。
* 还有学者提出，可以避免重复的点乘操作。即重用每个面片上点乘的结果。
* 由于silhouette是闭合的，所以一旦找到一条silhouette edge，就检查它的临边是否也是。直到找到整个轮廓。
* 对于动画来说，如果每一帧都进行一次搜索是很费事的。因此有学者提出，可以通过上一帧的silhouette edges来找到下一帧的silhouette edges。虽然性能可以提升，但这种方法可能会miss掉新的silhouette。

## 实践
可以参考这篇文章：[Silhouette Extraction](https://prideout.net/blog/old/blog/index.html@p=54.html)

## 优缺点总结
总体而言，这种技术的优点在于，一旦找到了这些silhouette edges，我们使用一些风格化算法进行渲染了。当然，还需要一些biasing操作来让它们可见。

当然也有缺点： 
* 这种方法得到的silhouette edge更加直线化，这是因为我们是以edge为单位渲染的。一些学者提出了方法可以绘制出曲线式的edges。
* 在每一帧需要大量CPU计算。如果使用geometry shader来实现的话，可以优化这一点。
* 会有Temporal Coherence的问题。解释一下就是说，由于是每一帧单独提取轮廓，所以在帧与帧之间会出现跳跃性。这也是图形学中的一个研究点。2014年SIGGRAPH有一篇论文（[Computing Smooth Surface Contours with Accurate Topology](http://www.labri.fr/perso/pbenard/publications/contours/)）[3]就讲了相关工作。

# Hybrid Silhouette
## 介绍
在一些对美观度要求比较高的应用里，会混合使用上述各种方法。例如，首先找到精确的轮廓边，把模型和轮廓边渲染到纹理中，再使用图像处理的方法识别出轮廓线，并在图像空间下进行风格化渲染。如下图所示（来源：《Real-time Rendering, third edition》）： 
![image](https://img-blog.csdn.net/20150508172712769)

# Reference
* [1] Tomas Akenine-Moller , Eric Haines , Naty Hoffman. 《Real-Time Rendering 3rd》
* [2] [《Cel Damage》 - Wikipedia](https://en.wikipedia.org/wiki/Cel_Damage)
* [3] Pierre Benard, Aaron Hertzmann, Michael Kass. Computing Smooth Surface Contours with Accurate Topology
* [4] [【NPR】漫谈轮廓线的渲染](https://blog.csdn.net/candycat1992/article/details/45577749)