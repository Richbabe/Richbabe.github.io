---
layout:     post
title:      Bresenham算法与三角形光栅化
subtitle:   
date:       2018-06-22
author:     Richbabe
header-img: img/u3d技术博客背景.jpg
catalog: true
tags:
    - 计算机图形学
    - OpenGL
---
# Bresenham算法
Bresenham算法是计算机图形学中为了“显示器（屏幕或打印机）系由像素构成”的这个特性而设计出来的算法，使得在求直线各点的过程中全部以整数来运算，因而大幅度提升计算速度。

Bresenham输入为两个点的坐标，输出为这两个点连成的直线上每一个点的坐标，具体算法如下：
![image](https://github.com/Richbabe/Richbabe.github.io/blob/master/img/Bresenham/Bresenham.png?raw=true)
可见其是对横坐标进行整数采样（每次递增加1），对纵坐标进行量化（根据Pi的正负性来决定取值）。需要注意的是：其中输入的两个顶点构成的斜率m需要满足0 <= m <= 1。

先将Bresenham算法转换成代码，我们可以使用递推来实现该算法（第一个点和最后一个点为我们输入的直线的两个顶点）：

```
//Bresenham算法，前提是斜率m满足-1<=m<=1
void bresenham(int y[], int p, int i, int size, int dx, int dy) {
	if (i == size - 1) {
		return;
	}
	int pnext;
	if (p <= 0) {
		y[i + 1] = y[i];
		pnext = p + 2 * dy;
	}
	if (p > 0) {
		y[i + 1] = y[i] + 1;
		pnext = p + 2 * dy - 2 * dx;
	}
	bresenham(y, pnext, i + 1, size, dx, dy);
}
```
可以看到我们通过递推求出每个点的Pi，接着通过Pi是否大于0求出当前点纵坐标值以及求出下一个点的Pi。

由于该算法只有在斜率m满足0 <= m <= 1的情况下成立，而直线的形状有多种方式，因此我们需要考虑以下几种情况：

（1）斜率m不存在的情况，即直线与y轴平行或者重合。

此时要求出直线上的每一个点，只需将每个点的横坐标都设成直线起点的横坐标，纵坐标每次加1递增到直线终点（默认起点的纵坐标小于终点的纵坐标）：

```
//斜率不存在的情况，即直线与y轴平行或者重合
	if (x0 == x1) {
		//保证y1始终大于或等于y0
		if (y0 > y1) {
			int temp = y0;
			y0 = y1;
			y1 = temp;
		}
		size = y1 - y0 + 1;//计算直线总点数
		//计算直线上每个点的横纵坐标
		for (int i = 0; i < size; i++) {
			x[i] = x0;
			y[i] = y0 + i;
		}
	}
```

（2）斜率m满足0 <= m <= 1的情况。

此时可以直接使用Bresenham算法，默认起点的横坐标值x0小于终点的横坐标x1。先通过x1 - x0 + 1求出直线上的总点数，对于直线上的每一点，横坐标的值每次递增加1。通过bresenham算法求出直线上除了起点和终点之外其他点的纵坐标：

```
float m = float(y1 - y0) / float(x1 - x0);
		//斜率m满足-1 <= m <= 1
		if (fabs(m) <= 1) {
			//默认x1大于x0
			if (x0 > x1) {
				int temp = x0;
				x0 = x1;
				x1 = temp;
				temp = y0;
				y0 = y1;
				y1 = temp;
			}
			size = x1 - x0 + 1;//计算直线总点数
			//计算直线上每点的横坐标
			for (int i = 0; i < size; i++) {
				x[i] = x0 + i;
			}
			//如果斜率m满足0 <= m <= 1
			if (m >= 0 && m <= 1) {
				y[0] = y0;
				y[size - 1] = y1;
				dx = x1 - x0;
				dy = y1 - y0;
				int p0 = 2 * dy - dx;
				bresenham(y, p0, 0, size, dx, dy);
			}
```

（3）斜率m满足-1 <= m < 0的情况。

此时只需要把m看成0<=m<=1（先将直线关于x轴对称，把直线的起点纵坐标y0和直线终点纵坐标y1取反），使用Bresenham求出所要求的直线关于x轴对称的直线l’，再做一次关于x轴对称即可得到所要求的直线（将l’上所有点的纵坐标取反）:

```
//如果斜率m满足-1 <= m < 0,只需要把m看成0 <= m <= 1再关于x轴对称即可
			else {
				y[0] = -1 * y0;
				y[size - 1] = -1 * y1;
				dx = x1 - x0;
				dy = -(y1 - y0);
				int p0 = 2 * dy - dx;
				bresenham(y, p0, 0, size, dx, dy);
				//关于x轴对称
				for (int i = 0; i < size; i++) {
					y[i] *= -1;
				}
			}
```

（4）斜率m满足m > 1的情况。

因为|m|>1，y轴上的变化从尺度要比x轴上的大，所以应该在y轴上进行整数取样，在x轴上进行量化。此时我们只需要把x看成y，把y看成x，直线上的纵坐标每次递增加1，将横坐标数组作为Bresenham函数第一个参数传入即可求出直线上所有点的横坐标：

```
//斜率m满足m > 1 或 m < -1，此时把x看成y，y看成x即可
		else {
			//默认y1大于y0
			if (y0 > y1) {
				int temp = y0;
				y0 = y1;
				y1 = temp;
				temp = x0;
				x0 = x1;
				x1 = temp;
			}
			size = y1 - y0 + 1;
			//计算直线上每点的纵坐标
			for (int i = 0; i < size; i++) {
				y[i] = y0 + i;
			}
			//如果斜率m满足m > 1
			if (m >= 1) {
				x[0] = x0;
				x[size - 1] = x1;
				dx = x1 - x0;
				dy = y1 - y0;
				int p0 = 2 * dx - dy;
				bresenham(x, p0, 0, size, dy, dx);
			}
```

（5）斜率m满足m < -1的情况，此时和情况（3）一样将直线关于y轴对称即可得到情况（4），求出直线上每个点的横坐标后再关于y轴对称即可求出直线上每一点的横纵坐标：

```
//如果斜率m满足m < -1,只需要把m看成m > 1再关于y轴对称即可
			else {
				x[0] = -1 * x0;
				x[size - 1] = -1 * x1;
				dx = -(x1 - x0);
				dy = y1 - y0;
				int p0 = 2 * dx - dy;
				bresenham(x, p0, 0, size, dy, dx);
				//关于y轴对称
				for (int i = 0; i < size; i++) {
					x[i] *= -1;
				}
			}
```

此时我们已经把全部情况讨论完毕，只需要将直线起点横纵坐标和直线终点横纵坐标输入便可求出直线上每一点的坐标。

# 利用Bresenham算法和OpenGL画直线
因为我们输入的点的坐标值为整数，通过bresenham算法求出来的直线上的点坐标值也为整数，而OpenGL默认显示坐标范围是[-1,1]区间内的浮点数。因此我们需要把顶点横纵坐标归一化到[-1,1]。我假设输入的顶点坐标范围为[-height,height]（height为窗口的宽和高），因此可以通过将坐标值除height得到[-1,1]范围内的浮点数坐标，归一化函数如下：

```
//归一化函数，将坐标的x值和y值归一化到[-1,1]
float normalize(int input) {
	return float(input) / height;
}
```
接着将直线上每个点的坐标数据绑定VBO和VAO，再通过OpenGL的DrawArray(GL_POINTS，0，1)即可画出直线，具体代码这里就不放出来了。

# 使用Bresenham算法画圆
假设圆的圆心位于坐标原点（如果圆心不在原点则可以通过坐标平移平移至原点），半径为R。以原点为圆心的圆C有4条对称轴:x = 0,y = 0,x = y,x = -y。如果已知圆弧上一点P1=C(x,y)，利用对称性便可以得到关于四条对称轴的其他7个点，即：
![image](https://github.com/Richbabe/Richbabe.github.io/blob/master/img/Bresenham/Bresenham%E7%94%BB%E5%9C%86.png?raw=true)

这种性质成为八对称性。

因此只需要扫描转换八分之一圆弧，就可以通过圆弧的八对称性得到整个圆。
先算出从(0, r)到(r/√2, r/√2)之间的1/8圆弧的点的坐标值，其他部分直接用轴对称和中心对称变换，最后画出完整的圆形。

也就是计算出那特定的1/8圆弧上的点的坐标(x,y)，再绘制出相应的(x, -y), (-x, y), (-x, -y), (y, x), (-y, x), (y, -x), (-y, -x)。

而那1/8圆弧上因为x上的变化比y大，所以在x方向上取样，在y方向上量化。递推思想就是：

d0=1-r

if d<0　　d=d+2*x+3

if d>=0　　d=d+2*(x-y)+5, y=y-1

x=x+1

需要注意的两点是：

（1）需要把坐标值归一化到[-1,1]才能进行点的绘制。

（2）如果圆心不在原点，需要把圆心平移到原点。

实现的Bresenham画圆的算法代码如下：

```
//画圆函数
void drawCircle(int r, float cx, float cy, Shader shader) {
	//归一化
	float fcx = normalize(cx);
	float fcy = normalize(cy);

	int x = 0, y = r;
	int d = 1 - r;//起点(0,R),下一中点(1,R - 0.5）,d=1*1+(R-0.5)*(R-0.5)-R*R=1.25-R,d只参与整数运算，所以小数部分可省略
	
	while (y >= x) {
		float fx = normalize(x);
		float fy = normalize(y);

		//绘制点(x,y) (-x,-y) (-x,y) (x,-y)
		drawPoints(fcx + fx, fcy + fy,shader);
		drawPoints(fcx - fx, fcy - fy, shader);
		drawPoints(fcx - fx, fcy + fy, shader);
		drawPoints(fcx + fx, fcy - fy, shader);

		//绘制点(x,y) (-x,-y) (-x,y) (x,-y)
		drawPoints(fcx + fy, fcy + fx, shader);
		drawPoints(fcx - fy, fcy - fx, shader);
		drawPoints(fcx - fy, fcy + fx, shader);
		drawPoints(fcx + fy, fcy - fx, shader);

		if (d < 0) {
			d = d + 2 * x + 3;
		}
		else {
			d = d + 2 * (x - y) + 5;
			--y;
		}
		++x;
	}
}
```

# 三角形的光栅化
三角形的光栅化，目的即是判断一个在视口坐标系的点是否在三角形内。所以我们首先要做的事是判断一个点是否在三角形内。

## 如何判断一个点是否在三角形内呢?
在这里我罗列了以下方法，由浅入深：

### 面积法
![image](https://github.com/Richbabe/Richbabe.github.io/blob/master/img/%E5%85%89%E6%A0%85%E5%8C%96/%E9%9D%A2%E7%A7%AF%E6%B3%95.png?raw=true)
如果一个点在三角形内，其与三角形的三个点构成的三个子三角形的面积等于大三角形的面积。否则，大于大三角形的面积。

我们可以通过向量法求得这个三角形的对应的平行四边形的面积。然后这个面积的1/2就是三角形的面积了。具体方法为：

先随意选择两个点，如B、C通过其坐标相减得向量（B，C）。记得谁减另一个就是指向谁。然后求出其中一个点和剩下一个点的向量。这两个向量的叉乘的便是平行四边形的面积。除以2就是三角形的面积。（注意这里是叉乘 (cross product)，而非内积（dot product））

此法直观，但效率低下。

### 内角和法
连接点P和三角形的三个顶点得到三条线段PA，PB和PC，求出这三条线段与三角形各边的夹角（可以用单位化的向量求点积再求arc），如果所有夹角之和为180度，那么点P在三角形内，否则不在，此法直观，但效率低下。
![image](https://github.com/Richbabe/Richbabe.github.io/blob/master/img/%E5%85%89%E6%A0%85%E5%8C%96/%E5%86%85%E8%A7%92%E5%92%8C.png?raw=true)

### 同向法
假设点P位于三角形内，会有这样一个规律，当我们沿着ABCA的方向在三条边上行走时，会发现点P始终位于边AB，BC和CA的右侧。我们就利用这一点来判断点P的位置。

当选定线段AB时，点C位于AB的右侧，同理选定BC时，点A位于BC的右侧，最后选定CA时，点B位于CA的右侧，所以当选择某一条边时，我们只需验证点P与该边所对的点在同一侧即可。 

![image](https://github.com/Richbabe/Richbabe.github.io/blob/master/img/%E5%85%89%E6%A0%85%E5%8C%96/%E5%90%8C%E5%90%91%E6%B3%95.png?raw=true)

要判断两个点是否在某条线段的同一侧可以通过叉积（得到的是一个向量）来实现。连接PA，将PA和AB做叉积，再将CA和AB做叉积，如果两个叉积（向量）的结果方向一致，那么两个点在同一侧。

判断两个向量是否同向可以用点积实现，如果点积大于0，则两向量夹角是锐角，否则是钝角。

还有一种稍显复杂的方法，需要先求出三条边的方程，然后利用同向法来判断。 
例如，三个顶点为A（a1,a2）,B(b1,b2),C(c1,c2). P点坐标为(p1，p2)。 
记三条边方程为：

```
BC: fa(x,y)=0
AC: fb(x,y)=0
AB: fc(x,y)=0
```
以AB为例，在三角形内的点P必须与点C在AB的同侧，即fc(p1,p2)*fc(c1,c2)>0 
记u、v、w为下式：

```
u = fa(x,y)*fa(a1,a2)
v = fb(x,y)*fb(b1,b2)
w = fc(x,y)*fc(c1,c2)
```
则由u、v、w三个数的符号可以确定任意点P(x,y)与三角形ABC的位置关系：

```
三个数都是正数：P在三角形内；
至少有一个负数：P在三角形外；
有且只有一个0，另两个为正数：P在三角形边上；
有二个0：在三角形的顶点上。
```

### 重心法
上面这个方法简单易懂，速度也快，下面这个方法速度更快，只是稍微多了一点数学而已

三角形的三个点在同一个平面上，如果选中其中一个点，其他两个点不过是相对该点的位移而已，比如选择点A作为起点，那么点B相当于在AB方向移动一段距离得到，而点C相当于在AC方向移动一段距离得到。
![image](https://github.com/Richbabe/Richbabe.github.io/blob/master/img/%E5%85%89%E6%A0%85%E5%8C%96/%E9%87%8D%E5%BF%83%E6%B3%95.png?raw=true)
所以对于平面内任意一点，都可以由如下方程来表示：

```
AP = u*AC + v*AB （1）
其中，两个大写字母表示向量，小写字母表示标量。
```
由该方程中u、v的符号可以判断出点P与三角形ABC的位置关系。

（1）P位于三角形内

```
u > 0
v > 0
u + v < 1
```

（2）P位于三角形边上

```
u = 0 且 0 < v < 1 //P在AB边上
v = 0 且 0 < u < 1 //P在AC边上
u > 0 且 v > 0 且 u+v < 1 //P在AB边上
```

（3）P位于三角形顶点上

```
u = 0 且 v = 0     //P位于A点
u = 0 且 v = 1     //P位于B点
u = 1 且 v = 0     //P位于C点
```

（4）P位于三角形外

```
u<0 或 v<0 或 n+v>1
```

u,v 的求解方法如下：

记AP=(x1,y1),AC=(x2,y2),AB=(x3,y3),则代入方程（1）解得：

```
u = (x1*y3-x3*y1)/(x2*y3-x3*y2)
v = (x1*y2-x2*y1)/(x3*y2-x2*y3)
```
或者用向量形式进行计算(以下默认两个大写字母表示一个向量)

```
分别在方程（1）的左右两端同时点乘AC、AB，得到下式：
AP·AC = u*AC·AC + v*AB·AC
AP·AB = u*AC·AB + v*AB·AB
```
因为两个向量点乘后得到一个数，所以上式解得：

```
u = [（AP·AC）(AB·AB)-(AP·AB)(AB·AC)]/[(AC·AC)(AB·AB)-(AC·AB)(AB·AC)]
v = [（AP·AC）(AC·AB)-(AP·AB)(AC·AC)]/[(AB·AC)(AC·AB)-(AB·AB)(AC·AC)]
```

## 三角形的光栅化
在这里我使用同向法来判断点是否在三角形内，（1）我们可以先求出三角形三条边的方程，然后利用同向法来判断。边的方程我记为:
```
kx - y + b = 0。
```
k即直线的斜率，假设直线两个顶点分别为A(x1,y1),B(x2,y2)，因此有:
```
k = (y2 - y1)/(x2 - x1)
b = y2 - kx2
```
直线AB的方程fc(x,y)即为
```
(y2 - y1)/(x2 - x1) * x - y + y2 - kx2 = 0
```
求直线方程的代码如下：

```
//直线方程为kx - y + b = 0
	float k1 = (float)(point3[1] - point2[1]) / (float)(point3[0] - point2[0]);
	float k2 = (float)(point1[1] - point3[1]) / (float)(point1[0] - point3[0]);
	float k3 = (float)(point2[1] - point1[1]) / (float)(point2[0] - point1[0]);

	float b1 = (float)point2[1] - k1 * point2[0];
	float b2 = (float)point3[1] - k2 * point3[0];
	float b3 = (float)point1[1] - k3 * point1[0];
```
为了加快点的遍历，首先我们要求出包围三角形的最小矩形，对这个矩形里的点遍历并判断是否出现在三角形内。
![image](https://github.com/Richbabe/Richbabe.github.io/blob/master/img/%E5%85%89%E6%A0%85%E5%8C%96/AABB.png?raw=true)
如图，我们只需求出三个顶点最小横纵坐标和最大横纵坐标便可得到最小包围矩形的四个顶点：

```
//找出包围三角形的最小矩形
	int xMIN, xMAX, yMIN, yMAX;//最小矩形的四个顶点
	xMIN = getMin(point1[0], point2[0], point3[0]);
	xMAX = getMax(point1[0], point2[0], point3[0]);
	yMIN = getMin(point1[1], point2[1], point3[1]);
	yMAX = getMax(point1[1], point2[1], point3[1]);
```
最后我们只需要通过同向法判断最小包围矩形中的每个点是否在三角形内，如果是则绘制该点：

```
//遍历矩形中的每个点，如果点在三角形中则填充颜色
	for (int i = xMIN; i <= xMAX; i++) {
		for (int j = yMIN; j <= yMAX; j++) {
			float u = (k1 * i - j + b1) * (k1 * point1[0] - point1[1] + b1);
			float v = (k2 * i - j + b2) * (k2 * point2[0] - point2[1] + b2);
			float w = (k3 * i - j + b3) * (k3 * point3[0] - point3[1] + b3);
			//在三角形中
			if (!(u < 0.0f || v < 0.0f || w < 0.0f)) {
				//归一化
				float fx = normalize(i);
				float fy = normalize(j);

				//填充颜色

				float vertices[] = {
					fx, fy, 0.0f,   color[0], color[1], color[2]
				};
				unsigned int points_VBO;//顶点缓冲对象
				unsigned int points_VAO;//顶点数组对象
				glGenVertexArrays(1, &points_VAO);//生成一个VAO对象
				glGenBuffers(1, &points_VBO);//生成一个VBO对象
				glBindVertexArray(points_VAO);//绑定VAO
				//把顶点数组复制到缓冲中供OpengGL使用
				glBindBuffer(GL_ARRAY_BUFFER, points_VBO);//把新创建的缓冲VBO绑定到GL_ARRAY_BUFFER目标上
				glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);//把之前定义的顶点数据points_vertices复制到缓冲的内存中

				//链接顶点属性
				//位置属性，值为0
				glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 6 * sizeof(float), (void*)0);//解析顶点数据
				glEnableVertexAttribArray(0);
				//颜色属性，值为1
				glVertexAttribPointer(1, 3, GL_FLOAT, GL_FALSE, 6 * sizeof(float), (void*)(3 * sizeof(float)));//解析顶点数据
				glEnableVertexAttribArray(1);
				shader.use();//激活着色器程序对象
				glBindVertexArray(points_VAO);//绑定VAO
				glDrawArrays(GL_POINTS, 0, 1);//绘制图元
				glBindVertexArray(0);
				glDeleteVertexArrays(1, &points_VAO);
				glDeleteBuffers(1, &points_VBO);
			}
		}
	}
```

# 运行效果
通过Bresenham算法和同向法算法实现的光栅化三角形：
![image](https://github.com/Richbabe/Richbabe.github.io/blob/master/img/%E5%85%89%E6%A0%85%E5%8C%96/%E4%B8%89%E8%A7%92%E5%BD%A2.png?raw=true)

通过Bresenham算法绘制圆：
![image](https://github.com/Richbabe/Richbabe.github.io/blob/master/img/%E5%85%89%E6%A0%85%E5%8C%96/%E5%9C%86.png?raw=true)

# 结语
本博客的代码和资源均可在我的[github](https://github.com/Richbabe/Bresenham-TriangelRasterization)上下载，别忘了点颗Star哟！












