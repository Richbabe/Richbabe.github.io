---
layout:     post
title:      Phong&BlinnPhong光照模型实现
subtitle:   Unity3D Shader学习日记（3）
date:       2018-06-18
author:     Richbabe
header-img: img/u3d技术博客背景.jpg
catalog: true
tags:
    - 计算机图形学
    - Unity
---
# 引言
在上一篇我们已经实现了[Lambert光照模型](http://richbabe.top/2018/06/13/Lambert&HalfLambert%E5%85%89%E7%85%A7%E6%A8%A1%E5%9E%8B%E5%AE%9E%E7%8E%B0/)。

现在，让我们再来实现一个更加高级的光照模型：Phong光照模型。

光除了漫反射，还有镜面反射。一些金属类型的材质，往往表现出一种高光效果，用兰伯特模型是模拟不出来的，所以就有了Phong模型。Phong模型主要有三部分构成，第一部分是上一篇中介绍了的Diffuse，也就是漫反射，第二部分是环境光，在非全局光照的情况下，我们一般是通过一个环境光来模拟物体的间接照明，这个值在shader中可以通过一个宏来直接获取，而第三部分Specular，也就是高光部分的计算，是一种模拟镜面反射的效果，也是本篇文章重点介绍的内容。

在现实世界中，粗糙的物体一般会是漫反射，而光滑的物体呈现得较多的就是镜面反射，最明显的现象就是光线照射的反射方向有一个亮斑。再来复习一下镜面反射的概念：当平行入射的光线射到这个反射面时，仍会平行地向一个方向反射出来，这种反射就属于镜面反射，其反射波的方向与反射平面的法线夹角（反射角），与入射波方向与该反射平面法线的夹角（入射角）相等，且入射波、反射波，及平面法线同处于一个平面内。反射光的亮度不仅与光线的入射角有关，还与观察者视线和物体表面之间的角度有关。镜面反射通常会造成物体表面上的“闪烁”和“高光”现象，镜面反射的强度也与物体的材质有关，无光泽的木材很少会有镜面反射发生，而高光泽的金属则会有大量镜面反射。

# Phong光照明模型
Phong光照模型中主要的部分就是对高光（镜面反射）的计算，首先看看这张图片：
![image](https://github.com/Richbabe/Richbabe.github.io/blob/master/img/U3d%20Shader/Phong%E7%A4%BA%E6%84%8F%E5%9B%BE.png?raw=true)
理想情况下，光源射出的光线，通过镜面反射，正好在反射光方向观察，观察者可以接受到的反射光最多，那么观察者与反射方向之间的夹角就决定了能够观察到高光的多少。夹角越大，高光越小，夹角越小，高光越大。而另一个影响高光大小的因素是表面的光滑程度，表面越光滑，高光越强，表面月粗糙，高光越弱。L代表光源方向，N代表顶点法线方向，V代表观察者方向，R代表反射光方向。首先需要计算反射光的方向R，反射光方向R可以通过入射光方向和法向量求出，R + L = 2dot(N,L)N，进而推出R = 2dot(N,L)N - L。关于R计算的推导，可以看下面这张图：
![image](https://github.com/Richbabe/Richbabe.github.io/blob/master/img/U3d%20Shader/%E8%AE%A1%E7%AE%97%E5%8F%8D%E5%B0%84%E5%90%91%E9%87%8F.png?raw=true)
不过在cg中，我们不用这么麻烦（其实OpengGL也有提供），cg为我们提供了一个计算反射光方向的函数reflect函数，我们只需要传入入射光方向（光源方向的反方向）和表面法线方向，就可以计算得出反射光的方向。然后，我们通过dot（R,V）就可以得到反射光方向和观察者方向之间的夹角余弦值了。下面给出冯氏反射模型公式：
> I(specular) = I * k * pow(max(0,dot(R,V)), gloss)

其中I为入射光颜色向量，k为镜面反射强度系数，gloss为光滑程度（反光度）。

通过上面的公式，我们可以看出，镜面反射强度跟反射向量与观察向量的余弦值呈指数关系，指数为gloss，该系数反映了物体表面的光滑程度，该值越大，表示物体越光滑，反射光越集中，当偏离反射方向时，光线衰减程度越大，只有当视线方向与反射光方向非常接近时才能看到高光现象，镜面反射光形成的光斑较亮并且较小；该值越小，表示物体越粗糙，反射光越分散，可以观察到光斑的区域越广，光斑大并且强度较弱：
![image](https://github.com/Richbabe/Richbabe.github.io/blob/master/img/U3d%20Shader/%E9%95%9C%E9%9D%A2%E5%8F%8D%E5%B0%84%E5%85%89%E6%BB%91%E7%A8%8B%E5%BA%A6%E5%8F%98%E5%8C%96%E5%9B%BE.png?raw=true)
可以看到一个物体的反光度越高，反射光的能力越强，散射得越少，高光点就会越小。

Phong光照模型在Unity中实现：
下面看一下冯氏光照模型在Unity中的实现，由于有高光，为了更好的效果，我们将主要的计算放到了fragment shader中进行：

```
Shader "Custom/Phong" {
	Properties{
		//材质的颜色
		_MaterialColor("MaterialColor",Color) = (1,1,1,1)
		//镜面反射光强系数
		_SpecularStrength("Specular",Range(0,1)) = 0.5
		//光滑程度(反光度)
		_Gloss("Gloss",Range(1.0,255)) = 20
	}
	SubShader{
		Pass{
			Tags{ "LightingType" = "ForwardBase" }
			LOD 200

			//******开始CG着色器语言编写模块******
			CGPROGRAM
			//引入头文件
			#include "Lighting.cginc"

			//定义vertexShader函数和fragmentShader函数
			#pragma vertex vertexShader
			#pragma fragment fragmentShader

			//定义Properties中的变量
			fixed4 _MaterialColor;
			float _SpecularStrength;
			float _Gloss;


			//定义结构体：顶点着色器阶段输入的数据
			struct vertexShaderInput {
				float4 vertex : POSITION;//顶点坐标
				float3 normal : NORMAL;//法向量
			};

			//定义结构体: 顶点着色器阶段输出的内容
			struct vertexShaderOutput {
				float4 pos : SV_POSITION;//顶点视口坐标系的坐标
				float3 worldNormal : NORMAL;//世界坐标系中的法向量
				float3 worldPos : TEXCOORD1;//世界坐标系的坐标
			};

			//定义顶点着色器
			vertexShaderOutput vertexShader(vertexShaderInput v) {
				vertexShaderOutput o;//顶点着色器的输出

				//把顶点从局部坐标系转到世界坐标系再转到视口坐标系
				o.pos = UnityObjectToClipPos(v.vertex);

				//把法线转换到世界空间
				o.worldNormal = mul(v.normal, (float3x3)unity_WorldToObject);

				//把顶点从局部坐标系转到世界坐标系
				o.worldPos = mul(unity_ObjectToWorld, v.vertex).xyz;

				return o;
			}

			//定义片段着色器
			fixed4 fragmentShader(vertexShaderOutput i) : SV_Target{
				//环境光
				fixed3 ambient = UNITY_LIGHTMODEL_AMBIENT.xyz * _MaterialColor;

				/* 归一化法线，不能在顶点着色器中归一化后传进来，
				因为从顶点着色器到片段着色器有差值处理，
				传入的归一化法线并不是从顶点着色器直接传出的 */
				fixed3 worldNormal = normalize(i.worldNormal);

				//把光照方向归一化
				fixed3 worldLightDir = normalize(_WorldSpaceLightPos0.xyz);

				//计算漫反射光照强度
				/* 半兰伯特光照：将原来[-1,1]区间的光照条件转化到[0,1]区间，既保证
				了结果的正确，又整体提高了亮度，保证非受光面也能有光，而不是全黑 */
				fixed3 lambert = 0.5 *  dot(worldNormal, worldLightDir) + 0.5;
				//漫反射光照强度为lambert光强 * 材质Diffuse颜色 * 光颜色
				fixed3 diffuse = lambert * _MaterialColor.xyz * _LightColor0.xyz;

				//计算镜面反射光照强度
				/* 光的反射方向reflectDir，worldLightDir表示光源方向（指向光源），光入射方向
				   为-worldLightDir，通过reflect函数(入射方向,法线方向)获得反射方向
				*/
				fixed3 reflectDir = normalize(reflect(-worldLightDir, worldNormal));
				//计算该像素对应位置（顶点计算过后传给像素经过插值后）的观察向量v，相机坐标 - 像素位置
				fixed3 viewDir = normalize(_WorldSpaceCameraPos.xyz - i.worldPos.xyz);
				/* 
					计算镜面反射光照强度，其与反射光方向和观察方向的夹角有关，
					夹角为dot(reflectDir,viewDir)，最后根据镜面反射光强系数计算反射值为
					_SpecularStrength * pow(dot(reflectDir,viewDir),Gloss)
				*/
				fixed3 specular = _LightColor0.rgb * _MaterialColor.rgb *_SpecularStrength * pow(max(0.0, dot(reflectDir, viewDir)), _Gloss);

				//计算Phong光照模型:Ambient + Diffuse + Specular
				fixed3 color = ambient + diffuse + specular;

				return fixed4(color, 1.0);

			}

			//*****结束CG着色器语言编写模块******
			ENDCG
		}
	}
	//前面的Shader失效的话，使用默认的Diffuse
	FallBack "Diffuse"
}
```
我们新建一个球体来看看我们shader的效果：
![image](https://github.com/Richbabe/Richbabe.github.io/blob/master/img/U3d%20Shader/Phong.png?raw=true)
可以看到有亮斑出现！

# Blinn-Phong光照模型
Phong光照模型能够很好地表现高光效果，不过Phong光照的缺点就是计算量较大，所以，在1977年，Jim Blinn对Phong光照进行了改进，称之为Blinn-Phong光照模型。

关于Blinn-Phong和Phong光照模型的对比，可以参照这张图片：
![image](https://github.com/Richbabe/Richbabe.github.io/blob/master/img/U3d%20Shader/BlinnPhong%E7%A4%BA%E6%84%8F%E5%9B%BE.png?raw=true)
Blinn-Phong光照引入了一个概念，半角向量，用H表示。半角向量计算简单，通过将光源方向L和视线方向V相加后归一化即可得到半角向量。Phong光照是比较反射方向R和视线方向V之间的夹角，而Blinn-Phong改为比较半角向量H和法线方向N之间的夹角。半角向量的计算复杂程度要比计算反射光线简单得多，所以Blinn-Phong的性能要高得多，效果比Phong光照相差不多，所以OpenGL中固定管线的光照模型就是Blinn-Phong光照模型。

BlinnPhong光照模型如下：
> I(specular) = I * k * pow(max(0,dot(N,H)), gloss) 

其中I为入射光颜色向量，k为镜面反射强度系数，gloss为光滑程度。

# Blinn-Phong光照在Unity中的实现

```
Shader "Custom/BlinnPhong" {
	Properties{
		//材质的颜色
		_MaterialColor("MaterialColor",Color) = (1,1,1,1)
		//镜面反射光强系数
		_SpecularStrength("Specular",Range(0,1)) = 0.5
		//光滑程度(反光度)
		_Gloss("Gloss",Range(1.0,255)) = 20
	}
	SubShader{
		Pass{
			Tags{ "LightingType" = "ForwardBase" }
			LOD 200

			//******开始CG着色器语言编写模块******
			CGPROGRAM
			//引入头文件
			#include "Lighting.cginc"

			//定义vertexShader函数和fragmentShader函数
			#pragma vertex vertexShader
			#pragma fragment fragmentShader

			//定义Properties中的变量
			fixed4 _MaterialColor;
			float _SpecularStrength;
			float _Gloss;


			//定义结构体：顶点着色器阶段输入的数据
			struct vertexShaderInput {
				float4 vertex : POSITION;//顶点坐标
				float3 normal : NORMAL;//法向量
			};

			//定义结构体: 顶点着色器阶段输出的内容
			struct vertexShaderOutput {
				float4 pos : SV_POSITION;//顶点视口坐标系的坐标
				float3 worldNormal : NORMAL;//世界坐标系中的法向量
				float3 worldPos : TEXCOORD1;//世界坐标系的坐标
			};

			//定义顶点着色器
			vertexShaderOutput vertexShader(vertexShaderInput v) {
				vertexShaderOutput o;//顶点着色器的输出

				//把顶点从局部坐标系转到世界坐标系再转到视口坐标系
				o.pos = UnityObjectToClipPos(v.vertex);

				//把法线转换到世界空间
				o.worldNormal = mul(v.normal, (float3x3)unity_WorldToObject);

				//把顶点从局部坐标系转到世界坐标系
				o.worldPos = mul(unity_ObjectToWorld, v.vertex).xyz;

				return o;
			}

			//定义片段着色器
			fixed4 fragmentShader(vertexShaderOutput i) : SV_Target{
				//环境光
				fixed3 ambient = UNITY_LIGHTMODEL_AMBIENT.xyz * _MaterialColor;

				/* 归一化法线，不能在顶点着色器中归一化后传进来，
				因为从顶点着色器到片段着色器有差值处理，
				传入的归一化法线并不是从顶点着色器直接传出的 */
				fixed3 worldNormal = normalize(i.worldNormal);

				//把光照方向归一化
				fixed3 worldLightDir = normalize(_WorldSpaceLightPos0.xyz);

				//计算漫反射光照强度
				/* 半兰伯特光照：将原来[-1,1]区间的光照条件转化到[0,1]区间，既保证
				了结果的正确，又整体提高了亮度，保证非受光面也能有光，而不是全黑 */
				fixed3 lambert = 0.5 *  dot(worldNormal, worldLightDir) + 0.5;
				//漫反射光照强度为lambert光强 * 材质Diffuse颜色 * 光颜色
				fixed3 diffuse = lambert * _MaterialColor.xyz * _LightColor0.xyz;

				//计算镜面反射光照强度
				//计算该像素对应位置（顶点计算过后传给像素经过插值后）的观察向量v，相机坐标 - 像素位置
				fixed3 viewDir = normalize(_WorldSpaceCameraPos.xyz - i.worldPos.xyz);
				//计算半角向量（光线方向 + 视线方向，结果归一化）
				fixed3 halfDir = normalize(worldLightDir + viewDir);
				/*
				计算BlinnPhong镜面反射光照强度，其与半角向量方向和法线方向的夹角有关，
				夹角为dot(halfDir,worldNormal)，最后根据镜面反射光强系数计算反射值为
				_SpecularStrength * pow(dot(halfDir,worldNormal),Gloss)
				*/
				fixed3 specular = _LightColor0.rgb * _MaterialColor.rgb *_SpecularStrength * pow(max(0.0, dot(halfDir, worldNormal)), _Gloss);

				//计算BlinnPhong光照模型:Ambient + Diffuse + Specular
				fixed3 color = ambient + diffuse + specular;

				return fixed4(color, 1.0);

			}

			//*****结束CG着色器语言编写模块******
			ENDCG
		}
	}
	//前面的Shader失效的话，使用默认的Diffuse
	FallBack "Diffuse"
}
```
可以看到我们在Phong光照明模型中计算镜面反射强度时计算的余弦值是反射光方向和观察方向夹角的余弦值：

```
/* 
					计算Phong镜面反射光照强度，其与反射光方向和观察方向的夹角有关，
					夹角为dot(reflectDir,viewDir)，最后根据镜面反射光强系数计算反射值为
					_SpecularStrength * pow(dot(reflectDir,viewDir),Gloss)
				*/
				fixed3 specular = _LightColor0.rgb * _MaterialColor.rgb *_SpecularStrength * pow(max(0.0, dot(reflectDir, viewDir)), _Gloss);
```
而在BlinnPhong光照明模型中计算镜面反射强度时计算的余弦值是半角向量和法向量的夹角的余弦值：

```
/*
				计算BlinnPhong镜面反射光照强度，其与半角向量方向和法线方向的夹角有关，
				夹角为dot(halfDir,worldNormal)，最后根据镜面反射光强系数计算反射值为
				_SpecularStrength * pow(dot(halfDir,worldNormal),Gloss)
				*/
				fixed3 specular = _LightColor0.rgb * _MaterialColor.rgb *_SpecularStrength * pow(max(0.0, dot(halfDir, worldNormal)), _Gloss);
```
其他计算步骤是一样的，让我们来看看Phong和BlinnPhong的对比：
![image](https://github.com/Richbabe/Richbabe.github.io/blob/master/img/U3d%20Shader/Phong&BlinnPhong.png?raw=true)
左边是Phong，右边是BlinnPhong，emmmmm虽然效果好像差了一点点，但是至少减少了很多复杂的计算！！

# 带纹理的BlinnPhong光照Shader
只有两个没有纹理的球，是说明不了问题的，下面来看一下带有纹理的Phong光照Shader。首先，我们要思考一个问题，如果我们要使用Specular类型的Shader，那么这个物体一般是金属类型的，这样，这个物体就会呈现金属特有的高光属性。然而，实际上完全是金属的物体并不是很多，现实世界中漫反射和镜面反射是共存的，拿一把刀来说，刀身是金属，刀柄是木头，那么，只有刀身适合这种类型的shader。可能我们最简单的想法是把刀拆成两个部分，刀身用的是Specular，刀柄用Diffuse；但是这种做法很麻烦，而且一个物体通过了两个drall call才能渲染出来。所以，聪明的前辈们总是能想到好的办法，次时代类型游戏中最简单的一种贴图就诞生了---高光贴图（通道）。

所谓高光贴图，或者说成高光通道，就是通过在制作贴图时，把图片的高光信息存储在一个灰度图或者直接存储在贴图的通道内，如果不需要Alpha Test的话，可以直接把高光通道放在Diffuse贴图的Alpha通道。而我们在shader中计算时，通过采样，就可以获得这个贴图中每个像素对应的位置是否是有高光的。这样，在Fragment Shader中可以直接通过这个Mask值乘以高光从而通过一个材质渲染出同一个模型上的不同质地。比如在通道内，0表示无高光，1（255）表示高光最强，那么，不需要高光的地方我们就可以在制作这张图的时候给0，需要高光的，刷上颜色，颜色越接近白色，高光越强。

知道了原理之后，我们就可以找一个人物模型贴图，然后在PhotoShop中给RGB格式的贴图增加一个通道，作为高光通道，然后把需要高光的部分抠出来，其他部分置为黑色（这部分让你的美工同事做，他们是专业的，你只要把Shader写好就行，在这里我直接在asset store下载了一把带有高光通道贴图的刀模型）

shader如下：

```
Shader "Custom/BlinnPhongWithTex" {
	Properties{
		//材质的颜色
		_MaterialColor("MaterialColor",Color) = (1,1,1,1)
		//镜面反射光强系数
		_SpecularStrength("Specular",Range(0.0,5.0)) = 1.0
		//光滑程度(反光度)
		_Gloss("Gloss",Range(1.0,255)) = 20
		//主贴图
		_MainTex("Main Texture",2D) = "white"{}
	}
	SubShader{
		Pass{
			Tags{ "LightingType" = "ForwardBase" }
			LOD 200

			//******开始CG着色器语言编写模块******
			CGPROGRAM
			//引入头文件
			#include "Lighting.cginc"

			//定义vertexShader函数和fragmentShader函数
			#pragma vertex vertexShader
			#pragma fragment fragmentShader

			//定义Properties中的变量
			fixed4 _MaterialColor;
			float _SpecularStrength;
			float _Gloss;
			sampler2D _MainTex;

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
				float3 worldNormal : NORMAL;//世界坐标系中的法向量
				float3 worldPos : TEXCOORD0;//世界坐标系的坐标
				float2 uv : TECOORD1;//转换后（缩放+采样偏移）的纹理坐标
			};

			//定义顶点着色器
			vertexShaderOutput vertexShader(vertexShaderInput v) {
				vertexShaderOutput o;//顶点着色器的输出

				//把顶点从局部坐标系转到世界坐标系再转到视口坐标系
				o.pos = UnityObjectToClipPos(v.vertex);

				//把法线转换到世界空间
				o.worldNormal = mul(v.normal, (float3x3)unity_WorldToObject);

				//把顶点从局部坐标系转到世界坐标系
				o.worldPos = mul(unity_ObjectToWorld, v.vertex).xyz;

				//转换uv
				o.uv = TRANSFORM_TEX(v.texcoord, _MainTex);

				return o;
			}

			//定义片段着色器
			fixed4 fragmentShader(vertexShaderOutput i) : SV_Target{
				//环境光
				fixed3 ambient = UNITY_LIGHTMODEL_AMBIENT.xyz;

				/* 归一化法线，不能在顶点着色器中归一化后传进来，
				因为从顶点着色器到片段着色器有差值处理，
				传入的归一化法线并不是从顶点着色器直接传出的 */
				fixed3 worldNormal = normalize(i.worldNormal);

				//把光照方向归一化
				fixed3 worldLightDir = normalize(_WorldSpaceLightPos0.xyz);

				//计算漫反射光照强度
				/* 半兰伯特光照：将原来[-1,1]区间的光照条件转化到[0,1]区间，既保证
				了结果的正确，又整体提高了亮度，保证非受光面也能有光，而不是全黑 */
				fixed3 lambert = 0.5 *  dot(worldNormal, worldLightDir) + 0.5;
				//漫反射光照强度为lambert光强 * 光颜色
				fixed3 diffuse = lambert * _LightColor0.xyz;

				//计算镜面反射光照强度
				//计算该像素对应位置（顶点计算过后传给像素经过插值后）的观察向量v，相机坐标 - 像素位置
				fixed3 viewDir = normalize(_WorldSpaceCameraPos.xyz - i.worldPos.xyz);
				//计算半角向量（光线方向 + 视线方向，结果归一化）
				fixed3 halfDir = normalize(worldLightDir + viewDir);
				/*
				计算BlinnPhong镜面反射光照强度，其与半角向量方向和法线方向的夹角有关，
				夹角为dot(halfDir,worldNormal)，最后根据镜面反射光强系数计算反射值为
				_SpecularStrength * pow(dot(halfDir,worldNormal),Gloss)
				*/
				fixed3 specular = _LightColor0.rgb *_SpecularStrength * pow(max(0.0, dot(halfDir, worldNormal)), _Gloss);

				//纹理采样
				fixed4 tex = tex2D(_MainTex, i.uv);

				//纹理中rgb为正常颜色，a为一个高光的mask图，非高光部分a值为0，高光部分根据a的值控制高光强度
				fixed3 color = (ambient * _MaterialColor.rgb + diffuse * _MaterialColor.rgb + specular * _MaterialColor.rgb * tex.a) * tex.rgb;

				return fixed4(color, 1.0);

			}

			//*****结束CG着色器语言编写模块******
			ENDCG
		}
	}
	//前面的Shader失效的话，使用默认的Diffuse
	FallBack "Diffuse"
}

```
可以看到我们在应用高光（镜面反射）时到贴图上时，是否应用由该贴图所在像素的高光通道的值决定，如果是1则应用，如果是0则不应用。

来看看加了高光效果和普通diffuse效果的刀的对比：
![image](https://github.com/Richbabe/Richbabe.github.io/blob/master/img/U3d%20Shader/BlinnPhong%E5%AE%9E%E7%8E%B0%E6%98%8E%E6%99%83%E6%99%83%E7%9A%84%E5%88%80.gif?raw=true)
上面是Diffuse,下面是BlinnPhong

可以看到在刀柄等不是会反光的材质两者相同（其实是因为在刀柄的贴图的高光通道值为0），但是在刀头加了BlinnPhong的刀会随着视角变换出现了明晃晃的效果，让人看起来更加真实，这是因为刀头的贴图的高光通道值为1。

# 优化BlinnShader
一般情况下，fragment shader是性能的瓶颈，所以优化Shader的重要思路之一就是减少逐像素计算，将计算挪到vertex shader部分，然后通过vertex shader向fragment shader中传递参数。正常情况下，一个物体在屏幕上，逐顶点计算的量级要远远小于物体在屏幕上逐像素计算的量（当然如果物体离相机特别远，光栅化之后在屏幕上只占了很小的一部分时，有可能有反过来的情况，但是有LOD之类的技术的话，远了之后，更换为低模，也会降低顶点数，所以还是逐像素计算的比较可怕，尤其是分辨率大了之后）。当然，我们也不能把所有计算都放在vertex shader中，上一篇文章中说过，如果将高光计算放在vertex shader中，效果很差，下面就来看一下，效果有多差：
![image](https://github.com/Richbabe/Richbabe.github.io/blob/master/img/U3d%20Shader/%E9%AB%98%E5%85%89%E5%9C%A8%E9%A1%B6%E7%82%B9%E8%AE%A1%E7%AE%97%E6%95%88%E6%9E%9C%E5%B7%AE.png?raw=true)
为什么会有这样的结果呢，主要是顶点计算的结果是通过顶点传递好的颜色进行高洛德着色，只是一个颜色的插值。而放在像素着色阶段，是通过顶点传递过来的参数，并且传递到像素阶段时经过了插值计算得到的信息，逐像素计算光照效果得到最终结果。更加详细的解释可以参照上一篇文章。2001年左右第三代modern GPU开始支持vertex shader，而在2003年左右，NVIDIA的GeForce FX和ATI Radeon 9700开始，GPU才开始支持fragment shader，也就是说fragment更先进，可以得到更好的效果。所以，我们只是将一些不会影响效果的计算放在vertex shader中即可。

上面的blinn-phong shader中，我们在fragment shader中计算了世界空间下的ViewDir，我们可以把这个计算移到vertex shader中进行来实现优化：

```
Shader "Custom/BetterBlinnPhongWithTex" {
	Properties{
		//材质的颜色
		_MaterialColor("MaterialColor",Color) = (1,1,1,1)
		//镜面反射光强系数
		_SpecularStrength("Specular",Range(0.0,5.0)) = 1.0
		//光滑程度(反光度)
		_Gloss("Gloss",Range(1.0,255)) = 20
		//主贴图
		_MainTex("Main Texture",2D) = "white"{}
	}
	SubShader{
		Pass{
			Tags{ "LightingType" = "ForwardBase" }
			LOD 200

			//******开始CG着色器语言编写模块******
			CGPROGRAM
			//引入头文件
			#include "Lighting.cginc"

			//定义vertexShader函数和fragmentShader函数
			#pragma vertex vertexShader
			#pragma fragment fragmentShader

			//定义Properties中的变量
			fixed4 _MaterialColor;
			float _SpecularStrength;
			float _Gloss;
			sampler2D _MainTex;

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
				float3 worldNormal : NORMAL;//世界坐标系中的法向量
				float3 viewDir : TEXCOORD0;//世界坐标系下的观察向量
				float2 uv : TECOORD1;//转换后（缩放+采样偏移）的纹理坐标
			};

			//定义顶点着色器
			vertexShaderOutput vertexShader(vertexShaderInput v) {
				vertexShaderOutput o;//顶点着色器的输出

				//把顶点从局部坐标系转到世界坐标系再转到视口坐标系
				o.pos = UnityObjectToClipPos(v.vertex);

				//把法线转换到世界空间
				o.worldNormal = mul(v.normal, (float3x3)unity_WorldToObject);

				//把顶点从局部坐标系转到世界坐标系
				float worldPos = mul(unity_ObjectToWorld, v.vertex).xyz;

				//计算观察方向
				o.viewDir = _WorldSpaceCameraPos - worldPos;

				//转换uv
				o.uv = TRANSFORM_TEX(v.texcoord, _MainTex);

				return o;
			}

			//定义片段着色器
			fixed4 fragmentShader(vertexShaderOutput i) : SV_Target{
				//环境光
				fixed3 ambient = UNITY_LIGHTMODEL_AMBIENT.xyz;

				/* 归一化法线，不能在顶点着色器中归一化后传进来，
				因为从顶点着色器到片段着色器有差值处理，
				传入的归一化法线并不是从顶点着色器直接传出的 */
				fixed3 worldNormal = normalize(i.worldNormal);

				//把光照方向归一化
				fixed3 worldLightDir = normalize(_WorldSpaceLightPos0.xyz);

				//计算漫反射光照强度
				/* 半兰伯特光照：将原来[-1,1]区间的光照条件转化到[0,1]区间，既保证
				了结果的正确，又整体提高了亮度，保证非受光面也能有光，而不是全黑 */
				fixed3 lambert = 0.5 *  dot(worldNormal, worldLightDir) + 0.5;
				//漫反射光照强度为lambert光强 * 光颜色
				fixed3 diffuse = lambert * _LightColor0.xyz;

				//计算镜面反射光照强度
				//直接通过归一化从顶点着色器传来的观察向量，减轻计算量!
				fixed3 viewDir = normalize(i.viewDir);
				//计算半角向量（光线方向 + 视线方向，结果归一化）
				fixed3 halfDir = normalize(worldLightDir + viewDir);
				/*
				计算BlinnPhong镜面反射光照强度，其与半角向量方向和法线方向的夹角有关，
				夹角为dot(halfDir,worldNormal)，最后根据镜面反射光强系数计算反射值为
				_SpecularStrength * pow(dot(halfDir,worldNormal),Gloss)
				*/
				fixed3 specular = _LightColor0.rgb *_SpecularStrength * pow(max(0.0, dot(halfDir, worldNormal)), _Gloss);

				//纹理采样
				fixed4 tex = tex2D(_MainTex, i.uv);

				//纹理中rgb为正常颜色，a为一个高光的mask图，非高光部分a值为0，高光部分根据a的值控制高光强度
				fixed3 color = (ambient * _MaterialColor.rgb + diffuse * _MaterialColor.rgb + specular * _MaterialColor.rgb * tex.a) * tex.rgb;

				return fixed4(color, 1.0);

			}

			//*****结束CG着色器语言编写模块******
			ENDCG
		}
	}
	//前面的Shader失效的话，使用默认的Diffuse
	FallBack "Diffuse"
}


```

看一下优化后的效果对比：
![image](https://github.com/Richbabe/Richbabe.github.io/blob/master/img/U3d%20Shader/%E4%BC%98%E5%8C%96%E5%90%8E%E7%9A%84BlinnPhong.gif?raw=true)
如果我不说哪个是经过优化，你肯定分辨不出来。说明优化后的效果没有变差，而且计算的量更少了，那肯定要选择优化的Shader呀！（追求极致画面除外。。。）

# 结语
经过这几天的学习，我已经初步掌握unity shader并实现了一些基础的光照明模型，希望我以后能坚持下去，实现更多高阶、炫酷的渲染，要时刻记住自己是要成为一个优秀的游戏开发者呀~

本博客的代码和资源均可在[我的github](https://github.com/Richbabe/Phong-BlinnPhong_Unity)上下载，别忘了点颗Star哟！
