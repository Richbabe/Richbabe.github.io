---
layout:     post
title:      Lambert&HalfLambert光照模型实现
subtitle:   Unity3D Shader学习日记（2）
date:       2018-06-13
author:     Richbabe
header-img: img/u3d技术博客背景.jpg
catalog: true
tags:
    - 计算机图形学
    - Unity
---
# 引言
[上篇博客](http://richbabe.top/2018/06/13/Unity3D-Shader-note01/)我们已经简单介绍了Unity Shader编程，现在我们来实现兰伯特和半兰伯特光照模型

# 漫反射和镜面反射
我们观察世界是因为有光进入我们的眼睛，光在世界中主要有反射和折射两种属性，当光照在某种介质表面时，一部分光发生反射，另一部分光进入介质，发生折射，也有转化为其他能量的光。本篇文章只讨论反射，折射等其他现象以后再学习。

光的反射分为两种，漫反射和镜面反射。

* 漫反射

是投射在粗糙表面上的光向各个方向反射的现象。当一束平行的入射光线射到粗糙的表面时，表面会把光线向着四面八方反射，所以入射线虽然互相平行，由于各点的法线方向不一致，造成反射光线向不同的方向无规则地反射，这种反射称之为“漫反射”或“漫射”。这种反射的光称为漫射光。很多物体，如植物、墙壁、衣服等，其表面粗看起来似乎是平滑，但用放大镜仔细观察，就会看到其表面是凹凸不平的，所以本来是平行的太阳光被这些表面反射后，弥漫地射向不同方向。

* 镜面反射

是指若反射面比较光滑，当平行入射的光线射到这个反射面时，仍会平行地向一个方向反射出来，这种反射就属于镜面反射，其反射波的方向与反射平面的法线夹角（反射角），与入射波方向与该反射平面法线的夹角（入射角）相等，且入射波、反射波，及平面法线同处于一个平面内。

# 兰伯特光照模型
兰伯特光照模型是目前最简单通用的模拟漫反射的光照模型。

兰伯特光照模型定义如下：
模型表面的明亮度直接取决于光线向量（light vector）和表面法线（normal）两个向量将夹角的余弦值。光线向量是指这个点到光从哪个方向射入，表面法线则定义了这个表面的朝向。

漫反射示意图：
![image](https://github.com/Richbabe/Richbabe.github.io/blob/master/img/U3d%20Shader/%E6%BC%AB%E5%8F%8D%E5%B0%84%E5%9B%BE%E7%A4%BA.png?raw=true)

如果漫反射光强设置为Diffuse，入射光光强为I，光方向和法线夹角为θ，那么兰伯特光照模型可以用下面的公式表示：
> Diffuse = I * cosθ

进一步地，我们可以通过点乘来求得两个方向向量之间的夹角，入射光方向设置为L，法线方向设置为N，如果光方向向量和法线方向向量都为单位向量（这就是为什么我们在写shader的时候需要normalize操作的原因），那么它们之间的夹角余弦值就可以表示为：cosθ = dot（L，N），最终漫反射光强公式，也就是兰伯特光照模型可以表示为：Diffuse = I  * dot（L，N）

注意：为了得到两个向量夹角的余弦值，我们需要将这两个向量单位化，否则点乘返回的就不是余弦值了，而是余弦值乘上这两个向量的长度。

## 逐顶点计算着色shader实现Lambert
我们在shader中需要计算输出的颜色，逐顶点着色也就是说我们的计算主要放在了vertex shader中，根据顶点来计算，每个顶点中计算出了该点的颜色，直接作为vertex shader的输出，pixel（fragment） shader的输入，当到达pixel阶段时，直接输出顶点shader的结果。比如一个三角形面片，在vertex阶段，分别计算了每个顶点的颜色值，在pixel阶段时，这个面片经过投影，最终显示在屏幕上的像素，会根据该像素周围的顶点来插值计算像素的最终颜色，这种着色方式也叫做[Gouraud Shading](https://en.wikipedia.org/wiki/Gouraud_shading)。

接下来看看如何用Unity Shader实现逐顶点计算着色shader实现Lambert：

```
// Upgrade NOTE: replaced '_World2Object' with 'unity_WorldToObject'
// Upgrade NOTE: replaced 'mul(UNITY_MATRIX_MVP,*)' with 'UnityObjectToClipPos(*)'

Shader "Custom/DiffusePerVertex" {
	Properties{
		//材质的颜色
		_Diffuse("Diffuse",Color) = (1,1,1,1)
	}
	SubShader{
		Pass{
			Tags{ "RenderType" = "Opaque" }
			LOD 200

			//******开始CG着色器语言编写模块******
			CGPROGRAM
			//引入头文件
			#include "Lighting.cginc"

			//定义Properties中的变量
			fixed4 _Diffuse;

			//定义结构体：顶点着色器阶段输入的数据
			struct vertexShaderInput {
				float4 vertex : POSITION;//顶点坐标
				float3 normal : NORMAL;//法向量
			};

			//定义结构体: 顶点着色器阶段输出的内容
			struct vertexShaderOutput {
				float4 pos : SV_POSITION;
				fixed4 color : COLOR;
			};

			//定义顶点着色器
			vertexShaderOutput vertexShader(vertexShaderInput v) {
				vertexShaderOutput o;//顶点着色器的输出

				//把顶点从局部坐标系转到世界坐标系再转到视口坐标系
				o.pos = UnityObjectToClipPos(v.vertex);

				//把法线转换到世界空间
				float3 worldNormal = mul(v.normal, (float3x3)unity_WorldToObject);

				//归一化法线
				worldNormal = normalize(worldNormal);

				//把光照方向归一化
				fixed3 worldLightDir = normalize(_WorldSpaceLightPos0.xyz);

				//根据兰伯特模型计算顶点的光照信息,dot为负值时取0
				fixed3 lambert = max(0.0, dot(worldNormal, worldLightDir));

				//最终输出颜色为lambert光强 * 材质Diffuse颜色 * 光颜色
				o.color = fixed4(lambert * _Diffuse.xyz * _LightColor0.xyz, 1.0);

				return o;
			}

			//定义片段着色器
			fixed4 fragmentShader(vertexShaderOutput i) : SV_Target{
				return i.color;
			}

			//使用vertexShader函数和fragmentShader函数
			#pragma vertex vertexShader
			#pragma fragment fragmentShader

			//*****结束CG着色器语言编写模块******
			ENDCG
		}
	}
	//前面的Shader失效的话，使用默认的Diffuse
	FallBack "Diffuse"
}
```
将该shader应用到Material再绑定到一个圆柱体上看看效果：


## 逐像素计算着色shader实现Lambert
逐像素计算时，我们的主要计算放到了pixel shader里，在vertex shader阶段只是进行了基本的顶点变换操作，以及顶点的法线转化到世界空间的操作，然后将转化后的法线作为参数传递给pixel shader。其他的计算都放到了pixel shader阶段，这样，针对每个像素，我们都可以来计算这个像素的光照情况，而不是像逐顶点计算时，先计算好顶点的颜色，然后差值得到中间的像素颜色。这种逐像素着色的方式也叫作[Phong Shading](https://en.wikipedia.org/wiki/Phong_shading)（注意不是冯氏光照模型，不要搞混呦）。

接下来看看如何用Unity Shader实现逐片段计算着色shader实现Lambert：

```
// Upgrade NOTE: replaced '_World2Object' with 'unity_WorldToObject'
// Upgrade NOTE: replaced 'mul(UNITY_MATRIX_MVP,*)' with 'UnityObjectToClipPos(*)'

Shader "Custom/DiffusePerPixel" {
	Properties {
		//材质的颜色
		_Diffuse("Diffuse",Color) = (1,1,1,1)
	}
	SubShader{
		Pass{
			Tags{ "RenderType" = "Opaque" }
			LOD 200

			//******开始CG着色器语言编写模块******
			CGPROGRAM
			//引入头文件
			#include "Lighting.cginc"

			//定义Properties中的变量
			fixed4 _Diffuse;

			//定义结构体：顶点着色器阶段输入的数据
			struct vertexShaderInput {
				float4 vertex : POSITION;//顶点坐标
				float3 normal : NORMAL;//法向量
			};

			//定义结构体: 顶点着色器阶段输出的内容
			struct vertexShaderOutput {
				float4 pos : SV_POSITION;//顶点视口坐标系的坐标
				float3 worldNormal : TEXCOORD0;//世界坐标系中的法向量
			};

			//定义顶点着色器
			vertexShaderOutput vertexShader(vertexShaderInput v) {
				vertexShaderOutput o;//顶点着色器的输出

				//把顶点从局部坐标系转到世界坐标系再转到视口坐标系
				o.pos = UnityObjectToClipPos(v.vertex);

				//把法线转换到世界空间
				o.worldNormal = mul(v.normal, (float3x3)unity_WorldToObject);

				return o;
			}

			//定义片段着色器
			fixed4 fragmentShader(vertexShaderOutput i) : SV_Target{
				/* 归一化法线，不能在顶点着色器中归一化后传进来，
				因为从顶点着色器到片段着色器有差值处理，
				传入的归一化法线并不是从顶点着色器直接传出的 */
				fixed3 worldNormal = normalize(i.worldNormal);

				//把光照方向归一化
				fixed3 worldLightDir = normalize(_WorldSpaceLightPos0.xyz);

				//根据兰伯特模型计算顶点的光照信息,dot为负值时取0
				fixed3 lambert = max(0.0, dot(worldNormal, worldLightDir));

				//最终输出颜色为lambert光强 * 材质Diffuse颜色 * 光颜色
				fixed3 diffuse = lambert * _Diffuse.xyz * _LightColor0.xyz;
				return fixed4(diffuse, 1.0);

			}

			//使用vertexShader函数和fragmentShader函数
			#pragma vertex vertexShader
			#pragma fragment fragmentShader

			//*****结束CG着色器语言编写模块******
			ENDCG
		}
	}
	//前面的Shader失效的话，使用默认的Diffuse
	FallBack "Diffuse"
}
```
可以看到我们把颜色计算放在了片段着色器中，来看看与在顶点着色器计算有什么差别：
![image](https://github.com/Richbabe/Richbabe.github.io/blob/master/img/U3d%20Shader/%E9%80%90%E9%A1%B6%E7%82%B9&%E9%80%90%E7%89%87%E6%AE%B5%E5%AE%9E%E7%8E%B0Lambert.png?raw=true)
可以看到GouraudShading(左)的圆柱体可以看出线条状的轮廓，其实每个线条都是由两个三角形面片组成的长方形面片。

这其实就是马赫带效应，借用我之前用OpengGL实现的较为明显的效果：
Gouraud Shading：
![image](https://github.com/Richbabe/Richbabe.github.io/blob/master/img/U3d%20Shader/%E9%A9%AC%E8%B5%AB%E5%B8%A6%E6%95%88%E5%BA%941.png?raw=true)
Phong Shading：
![image](https://github.com/Richbabe/Richbabe.github.io/blob/master/img/U3d%20Shader/%E9%A9%AC%E8%B5%AB%E5%B8%A6%E6%95%88%E5%BA%942.png?raw=true)
可以看到因为Gouraud的马赫带效应，在两个三角形的交界处有一条明显的亮边，当使用更复杂的形状时，这种效果会变得更加明显。其原因是因为对于立方体正面的两个三角形右上方顶点用镜面高光照明，而这两个三角形的另外两个顶点没有。而立方体正面的中间片段的颜色不是直接来自光源，而是来自顶点的插值，因此在两个三角形之间会产生一条亮边。

为什么逐像素计算会得到更好的效果？

因为我们逐像素取的光照的方向是一致的，法线方向也是通过上一步的vertex shader传递过来的，如果像素和顶点对应了的话，那不是每个像素的计算结果都会一样呢？然而，其实像素和顶点是不对应的，这个就是传说中的渲染流水线了，在顶点阶段计算的结果，并不是直接传递给像素着色器的，而是经过了一系列的插值计算，我们从vertex shader传递过来的法线方向，只代表了这一个顶点的顶点法线方向，而到了pixel阶段，这个像素所对应的法线等参数相当于其周围几个顶点进行插值后的结果。我们用这一个像素点对应的法线方向与光照方向进行计算，就可以获得该像素点在光照条件下的颜色值，而不是先计算好颜色再插值得到结果。
![image](https://github.com/Richbabe/Richbabe.github.io/blob/master/img/U3d%20Shader/%E9%A1%B6%E7%82%B9%E7%9D%80%E8%89%B2%E5%99%A8%E6%8F%92%E5%80%BC%E5%88%B0%E7%89%87%E6%AE%B5%E7%9D%80%E8%89%B2%E5%99%A8.png?raw=true)

总结一下Gouraud Shading和Phong Shading的区别：
* Gouraud Shading是在顶点着色器实现冯氏光照模型，而Phong Shadinng是在片段着色器实现冯氏光照模型。相对于片段来说，顶点要少得多，因此Gouraud Shading比Phong Shading的计算量要少很多，但是光照看起来不会非常真实：
![image](https://github.com/Richbabe/Richbabe.github.io/blob/master/img/U3d%20Shader/Gouraud&PhongShading.png?raw=true)
* Gouraud有可能会出现马赫带效应

# 半兰伯特光照模型
实现了逐顶点和逐像素的兰伯特光照模型，我们再来看一下兰伯特光照模型的变种--半兰伯特光照。经过上面的对比，逐像素光照计算会获得更好的效果，所以我们下面就采用逐像素的方式来实现半兰伯特光照模型。

上面的shader计算光照的时候，我们计算法线方向和光方向的点乘值时，得到的结果有可能是负数，而兰伯特光照模型对于该情况的处理是，dot值为负数，说明该点不会受到光的照射，所以对于该光源，该点无光，直接使用max（0，diffuse）来将不应该受光的位置全都置为黑色。虽然听起来很有道理的样子，然而这种并不好看。
![image](https://github.com/Richbabe/Richbabe.github.io/blob/master/img/U3d%20Shader/%E9%9D%9E%E5%8F%97%E5%85%89%E9%9D%A2%E5%85%A8%E9%BB%91.png?raw=true)
可以明显看到非受光面都是黑色的。

然而，实际上，我们在现实世界中经常会发现，即使我们让一个物体不被光直接照射，我们也可能会看到物体，虽然亮度不是很高，这其实是由于物体之间光的反射造成的，也就是间接光照，间接光照是更高级的渲染，比如光线追踪算法等。但是在实时图形学，我们大部分情况是通过一个环境光（Ambient Light）统一代表了间接光，这样，即使在没有光的时候，我们也可以看见物体。

兰伯特光照出来的时候，貌似还没有这么高科技的技术，所以呢，有人就想到了一个取巧的技术（据说是《半条命》），既保证了兰伯特模型计算出来的光照结果大于0，又整体提升了亮度，使非直接受光面不是单纯的置为黑色。这是一个在图形学领域经常有的变换，区间转化，从（-1,1）转化到（0,1），如果不考虑无意义的负值，也可以说成从（0,1）转化到了（0.5,1）。方法很简单，乘以0.5再加上0.5。这样，原本亮度为1的地方，乘以0.5变成了0.5，加上0.5就又成了1，而原本光照强度为0的地方，就变成了0.5，原本为负数的地方，也能保证为大于0了。半兰伯特光照这种区间转化的原理图如下所示：
![image](https://github.com/Richbabe/Richbabe.github.io/blob/master/img/U3d%20Shader/HalfLambert%E5%8C%BA%E9%97%B4%E8%BD%AC%E5%8C%96.png?raw=true)

下面看一下逐像素计算的半兰伯特光照shader，比兰伯特光照的只是将法线向量与光方向向量的点乘结果用一种更好的方式区间转化到了（0,1）区间：

```
// Upgrade NOTE: replaced '_World2Object' with 'unity_WorldToObject'
// Upgrade NOTE: replaced 'mul(UNITY_MATRIX_MVP,*)' with 'UnityObjectToClipPos(*)'

Shader "Custom/HalfLambert" {
	Properties{
		//材质的颜色
		_Diffuse("Diffuse",Color) = (1,1,1,1)
	}
	SubShader{
		Pass{
			Tags{ "RenderType" = "Opaque" }
			LOD 200

			//******开始CG着色器语言编写模块******
			CGPROGRAM
			//引入头文件
			#include "Lighting.cginc"

			//定义Properties中的变量
			fixed4 _Diffuse;

			//定义结构体：顶点着色器阶段输入的数据
			struct vertexShaderInput {
				float4 vertex : POSITION;//顶点坐标
				float3 normal : NORMAL;//法向量
			};

			//定义结构体: 顶点着色器阶段输出的内容
			struct vertexShaderOutput {
				float4 pos : SV_POSITION;//顶点视口坐标系的坐标
				float3 worldNormal : TEXCOORD0;//世界坐标系中的法向量
			};

			//定义顶点着色器
			vertexShaderOutput vertexShader(vertexShaderInput v) {
				vertexShaderOutput o;//顶点着色器的输出

				//把顶点从局部坐标系转到世界坐标系再转到视口坐标系
				o.pos = UnityObjectToClipPos(v.vertex);

				//把法线转换到世界空间
				o.worldNormal = mul(v.normal, (float3x3)unity_WorldToObject);

				return o;
			}

			//定义片段着色器
			fixed4 fragmentShader(vertexShaderOutput i) : SV_Target{
				/* 归一化法线，不能在顶点着色器中归一化后传进来，
				因为从顶点着色器到片段着色器有差值处理，
				传入的归一化法线并不是从顶点着色器直接传出的 */
				fixed3 worldNormal = normalize(i.worldNormal);

				//把光照方向归一化
				fixed3 worldLightDir = normalize(_WorldSpaceLightPos0.xyz);

				/* 半兰伯特光照：将原来[-1,1]区间的光照条件转化到[0,1]区间，既保证
				了结果的正确，又整体提高了亮度，保证非受光面也能有光，而不是全黑 */
				fixed3 lambert = 0.5 *  dot(worldNormal, worldLightDir) + 0.5;

				//最终输出颜色为lambert光强 * 材质Diffuse颜色 * 光颜色
				fixed3 diffuse = lambert * _Diffuse.xyz * _LightColor0.xyz;
				return fixed4(diffuse, 1.0);

			}

			//使用vertexShader函数和fragmentShader函数
			#pragma vertex vertexShader
			#pragma fragment fragmentShader

			//*****结束CG着色器语言编写模块******
			ENDCG
		}
	}
	//前面的Shader失效的话，使用默认的Diffuse
	FallBack "Diffuse"
}

```
看一下兰伯特光照模型和半兰伯特光照模型的对比：
![image](https://github.com/Richbabe/Richbabe.github.io/blob/master/img/U3d%20Shader/Lambert&HalfLambert.png?raw=true)

# 带有纹理的半兰伯特光照shader
我们修改一下带上纹理的半兰伯特光照模型
（1）Gouraud Shading

```
Shader "Custom/DiffuseWithTex_HLV" {
	Properties{
		//材质的颜色
		_Diffuse("Diffuse",Color) = (1,1,1,1)
		//主贴图
		_MainTex("Texture",2D) = "white"{}
	}
	SubShader{
		Pass{
			Tags{ "RenderType" = "Opaque" }
			LOD 200

			//******开始CG着色器语言编写模块******
			CGPROGRAM
			//引入头文件
			#include "Lighting.cginc"

			//定义Properties中的变量
			fixed4 _Diffuse;//材质颜色
			sampler2D _MainTex;//主贴图

			//使用TRANSFROM_TEX宏就需要定义XXX_ST
			float4 _MainTex_ST;

			//定义结构体：顶点着色器阶段输入的数据
			struct vertexShaderInput {
				float4 vertex : POSITION;//顶点坐标
				float3 normal : NORMAL;//法向量
				float4 texcoord : TEXCOORD0;//纹理坐标
			};

			//定义结构体: 顶点着色器阶段输出的内容
			struct vertexShaderOutput {
				float4 pos : SV_POSITION;//顶点视口坐标系的坐标
				fixed4 color : COLOR;//输出颜色
				float2 uv : TEXCOORD1;//转换后的纹理坐标
			};

			//定义顶点着色器
			vertexShaderOutput vertexShader(vertexShaderInput v) {
				vertexShaderOutput o;//顶点着色器的输出

				 //把顶点从局部坐标系转到世界坐标系再转到视口坐标系
				o.pos = UnityObjectToClipPos(v.vertex);

				//Unity自身的diffuse带了环境光，在这里我们增强一下环境光
				fixed3 ambient = UNITY_LIGHTMODEL_AMBIENT.xyz * _Diffuse.xyz;//增强后的环境光

				//把法线转换到世界空间
				float3 worldNormal = mul(v.normal, (float3x3)unity_WorldToObject);

				//归一化法线
				worldNormal = normalize(worldNormal);

				// 把光照方向归一化
				fixed3 worldLightDir = normalize(_WorldSpaceLightPos0.xyz);

				/* 半兰伯特光照：将原来[-1,1]区间的光照条件转化到[0,1]区间，既保证
				了结果的正确，又整体提高了亮度，保证非受光面也能有光，而不是全黑 */
				fixed3 lambert = 0.5 *  dot(worldNormal, worldLightDir) + 0.5;

				//最终输出颜色为lambert光强 * 材质Diffuse颜色 * 光颜色 + 环境光
				fixed3 diffuse = lambert * _Diffuse.xyz * _LightColor0.xyz + ambient;
				o.color = fixed4(diffuse, 1.0);

				/*
					通过TRANSFORM_TEX宏转化纹理坐标，主要处理了Offset和Tiling的改变，默认
					时等同于o.uv = v.texcoord.xy
				*/
				o.uv = TRANSFORM_TEX(v.texcoord, _MainTex);

				return o;
			}

			//定义片段着色器
			fixed4 fragmentShader(vertexShaderOutput i) : SV_Target{
				//进行纹理采样
				fixed4	texColor = tex2D(_MainTex, i.uv);
				return i.color * texColor;
			}

			//使用vertexShader函数和fragmentShader函数
			#pragma vertex vertexShader
			#pragma fragment fragmentShader

			//*****结束CG着色器语言编写模块******
			ENDCG
		}
	}
	//前面的Shader失效的话，使用默认的Diffuse
	FallBack "Diffuse"
}


```
（2）Phong Shading

```
Shader "Custom/DiffuseWithTex_HLP" {
	Properties {
		//材质的颜色
		_Diffuse("Diffuse",Color) = (1,1,1,1)
		//主贴图
		_MainTex("Texture",2D) = "white"{}
	}
	SubShader{
		Pass{
			Tags{ "RenderType" = "Opaque" }
			LOD 200

			//******开始CG着色器语言编写模块******
			CGPROGRAM
			//引入头文件
			#include "Lighting.cginc"

			//定义Properties中的变量
			fixed4 _Diffuse;//材质颜色
			sampler2D _MainTex;//主贴图

			//使用TRANSFROM_TEX宏就需要定义XXX_ST
			float4 _MainTex_ST;

			//定义结构体：顶点着色器阶段输入的数据
			struct vertexShaderInput {
				float4 vertex : POSITION;//顶点坐标
				float3 normal : NORMAL;//法向量
				float4 texcoord : TEXCOORD0;//纹理坐标
			};

			//定义结构体: 顶点着色器阶段输出的内容
			struct vertexShaderOutput {
				float4 pos : SV_POSITION;//顶点视口坐标系的坐标
				float3 worldNormal : TEXCOORD0;//世界坐标系中的法向量
				float2 uv : TEXCOORD1;//转换后的纹理坐标
			};

			//定义顶点着色器
			vertexShaderOutput vertexShader(vertexShaderInput v) {
				vertexShaderOutput o;//顶点着色器的输出

				//把顶点从局部坐标系转到世界坐标系再转到视口坐标系
				o.pos = UnityObjectToClipPos(v.vertex);

				//把法线转换到世界空间
				o.worldNormal = mul(v.normal, (float3x3)unity_WorldToObject);

				/*
					通过TRANSFORM_TEX宏转化纹理坐标，主要处理了Offset和Tiling的改变，默认
					时等同于o.uv = v.texcoord.xy
				*/
				o.uv = TRANSFORM_TEX(v.texcoord, _MainTex);

				return o;
			}

			//定义片段着色器
			fixed4 fragmentShader(vertexShaderOutput i) : SV_Target{
				//Unity自身的diffuse也带了环境光，在这里我们增强一下环境光
				fixed3 ambient = UNITY_LIGHTMODEL_AMBIENT.xyz * _Diffuse.xyz;//增强后的环境光

				/* 归一化法线，不能在顶点着色器中归一化后传进来，
				因为从顶点着色器到片段着色器有差值处理，
				传入的归一化法线并不是从顶点着色器直接传出的 */
				fixed3 worldNormal = normalize(i.worldNormal);

				//把光照方向归一化
				fixed3 worldLightDir = normalize(_WorldSpaceLightPos0.xyz);

				/* 半兰伯特光照：将原来[-1,1]区间的光照条件转化到[0,1]区间，既保证
				了结果的正确，又整体提高了亮度，保证非受光面也能有光，而不是全黑 */
				fixed3 lambert = 0.5 *  dot(worldNormal, worldLightDir) + 0.5;

				//最终输出颜色为lambert光强 * 材质Diffuse颜色 * 光颜色 + 环境光
				fixed3 diffuse = lambert * _Diffuse.xyz * _LightColor0.xyz + ambient;

				//进行纹理采样
				fixed4 color = tex2D(_MainTex, i.uv);

				return fixed4(diffuse * color.rgb, 1.0);

			}

			//使用vertexShader函数和fragmentShader函数
			#pragma vertex vertexShader
			#pragma fragment fragmentShader

			//*****结束CG着色器语言编写模块******
			ENDCG
		}
	}
	//前面的Shader失效的话，使用默认的Diffuse
	FallBack "Diffuse"
}

```
![image](https://github.com/Richbabe/Richbabe.github.io/blob/master/img/U3d%20Shader/HalfLambertModel.png?raw=true)
左边是Gouraud Shading，右边是Phong Shading

可以看出，如果模型比较细致，其实在diffuse情况下，是没有特别明显的区别的，而大部分计算放在vertex shader中，对于效率更有益处，vertex shader一般不是GPU的瓶颈，逐顶点计算可以比逐像素计算省很多，所以将尽可能多的计算放在vertex阶段而不是fragment阶段是一个很好的优化shader的策略。但是，注意！是在diffuse的情况，如果我们的shader中有高光specular，那么，用逐顶点计算高光就会出现特别难看的光斑，这个下篇文章再进行介绍。

由于unity shader中diffuse是带有环境光的，所以我们也在shader中计算了环境光。由于没有全局光照，所以间接光照就通过UNITY_LIGHTMODEL_AMBIENT这个宏进行访问

# TRANSFORM_TEX宏
在添加了纹理之后，主要使用了一个宏和一个采样函数。


```
/*
通过TRANSFORM_TEX宏转化纹理坐标，主要处理了Offset和Tiling的改变，默认
时等同于o.uv = v.texcoord.xy
*/
o.uv = TRANSFORM_TEX(v.texcoord, _MainTex);
```


```
//进行纹理采样
fixed4 color = tex2D(_MainTex, i.uv);
```


采样函数顾名思义，tex2D，就是通过传入的纹理坐标，来获得纹理采样点所对应的颜色值。下面重点看一下Unity为我们提供的TRANSFORM_TEX宏，我们从UnityCG.cginc中可以找到这个宏的定义如下：

```
// Transforms 2D UV by scale/bias property  
#define TRANSFORM_TEX(tex,name) (tex.xy * name##_ST.xy + name##_ST.zw) 
```
如果我们使用了这个宏，就需要在shader中定义我们要采样的纹理的一个系数，命名方式为 纹理名_ST，float4类型。那么这个值是什么呢？
![image](https://github.com/Richbabe/Richbabe.github.io/blob/master/img/U3d%20Shader/TRANSFORM_TEX%E5%AE%8F.png?raw=true)

就是这个啦！我们在使用纹理时，unity会为我们提供两个参数，一个是Tiling，一个是Offset。简单来说，Tiling表示纹理的缩放比例，Offset表示了纹理使用时采样的偏移值。关于Tiling和Offset的介绍，可以参考[这篇文章](https://blog.csdn.net/puppet_master/article/details/50358612)。

知道了这个，也就好理解TRANSFORM_TEX宏所做的事情了，在采样时，将材质面板上设置的Tiling值通过XXX_ST.xy传递进来，用于和采样的坐标相乘，进行采样的缩放，将Offset值通过XXX_ST.zw传递进来，作为纹理采样的偏移。TRANSFORM_TEX返回值即为经过缩放和采样偏移后的纹理坐标，

# 结语
我们可以看到在Unity Shader（CG/HLSL）实现光照模型和我们在OpenGL中实现原理是一样的，只不过具体的实现方式有点不同，比如说一些定义好的宏以及着色器的链接方式等，接下来我们将会实现Phong和BlinnPhong光照明模型。

本博客的代码和资源均可在[我的github](https://github.com/Richbabe/Lambert-HalfLambert)上下载，别忘了点颗Star哟！


