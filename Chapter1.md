# Chapter 1 引言

图片显示速率使用frames per second（FPS）和Hertz（HZ）衡量。游戏的帧数一般是30，60，72，或者更高。

电影放映机的帧数是24，但是使用了快门系统来显示一帧2到4次，以避免闪烁。这叫做刷新率，使用Hz表示，跟显示率不同。一个照亮一帧3次的快门的拥有72Hz的刷新率。LCD显示器也区分刷新率和显示率。

15毫秒的延迟就会显著影响交互感【r1849】。例如，VR设备要求90FPS。

本书专注于提供方法来提升渲染速度和图像质量，也会涉及到加速算法和图形API的特性和限制。本书不会深入探讨每一个主题，而是展示核心概念和术语，解释相关领域内具有鲁棒性和实际性的算法，提供查找更多信息的方向。

## 1.1 内容梗概

* Chapter 1，引言。
* Chapter 2，图形渲染管线。实时渲染的核心就是一系列步骤，它们以场景描述为输入，输出我们看到的图像。
* Chapter 3，GPU。现代GPU使用固定函数和可编程单元的组合来实现渲染管线。
* Chapter 4，变换。变换用来操作位置，方位，大小，形状，以及摄像机的位置和视角
* Chapter 5，着色基础。包括材质和光的定义，如何实现期望的外观，如现实化和风格化。还包括抗锯齿，透明和伽马矫正。
* Chapter 6，纹理。
* Chapter 7，阴影。
* Chapter 8，光和颜色。在学习PBS之前，需要理解与光和颜色相关的量。在物理渲染过程结束后，我们还需要将结果转换到用来显示的量，这需要考虑屏幕属性和观察环境。
* Chapter 9，基于物理的着色。我们从开始构建对PBS的理解。包括物理现象，不同的渲染材质，材质混合，过滤。
* Chapter 10，局部光照。
* Chapter 11，全局光照。用算法模拟光和场景之间的交互。包括环境光遮挡，方向光遮挡，漫反射光和高光的全局光照效果。
* Chapter 12，图像空间效果。介绍图像过滤和二次投影技术，实现各种后效：lens flares（镜头光斑），motion blur（运动模糊）和depth of field（景深）。
* Chapter 13，多边形之外。三角形并非描述物件的唯一方法，其他方法还有：图像，点云，体素，等等。
* Chapter 14，体积和半透明渲染。这部分专注于体积材质的表示和光源的交互。模拟的效果包括大范围的大气效果和头发之间的光的散射。
* Chapter 15，非写实渲染。风格化渲染如卡通渲染，水彩渲染。还讨论了线条和文本生成技术。
* Chapter 16，多边形技术。包括多边形数据的表示和压缩。
* Chapter 17，曲线和曲面。
* Chapter 18，管线优化。探讨各种优化技术，也包括多进程。
* Chapter 19，加速算法。剔除，LOD。
* Chapter 20，高效着色。探索着色过程中的各种低效方法。
* Chapter 21，虚拟和增强现实。
* Chapter 22，相交测试。相交测试用于渲染，用户交互和碰撞检测。
* Chapter 23，图形硬件。包括颜色深度，帧缓冲，基本结构类型等等。还有一些代表性GPU的学习样例。
* Chapter 24，未来。

由于空间限制，碰撞检测，线性代数和三角学被放在了网站上。

## 1.2 符号和定义

### 1.2.1 数学符号

下表总结了数学符号

|类型|符号|样例|
|:-|:-|:-|
|角度|小写希腊符号|$\alpha_{i}, \phi, \rho, \eta, \gamma_{242}, \theta$|
|标量|小写斜体|$a, b, t, u_{k}, v, w_{i j}$|
|向量或点|小写粗体|$\mathbf{a}, \mathbf{u}, \mathbf{v}_{s} \mathbf{h}(\rho), \mathbf{h}_{z}$|
|矩阵|大写粗体|$\mathbf{T}(\mathbf{t}), \mathbf{X}, \mathbf{R}_{x}(\rho)$|
|平面|$\pi$：一个向量和一个标量|$\pi: \mathbf{n} \cdot \mathbf{x}+d=0$|
|三角形|$\triangle$ 3个点|$\triangle \mathbf{v}_{0} \mathbf{v}_{1} \mathbf{v}_{2}, \triangle \mathbf{c b} \mathbf{a}$|
|线段|2个点|$\mathbf{u} \mathbf{v}, \mathbf{a}_{i} \mathbf{b}_{j}$|
|几何体|大写斜体|$A_{O B B}, T, B_{A A B B}$|

一些例外包括：L表示辐射率（radiance），E表示辐照度（irradiance），$\sigma_{s}$表示散射系数。角度和标量都是实数。向量和点使用小写粗体表示：
$$\mathbf{v}=\left(\begin{array}{c}{v_{x}} \\ {v_{y}} \\ {v_{z}}\end{array}\right)$$
即，使用列向量的形式。我们有时也会写成$\left(v_{x}, v_{y}, v_{z}\right)$的形式，而非更加正确的$\left(\begin{array}{lll}{v_{x}} & {v_{y}} & {v_{z}}\end{array}\right)^{T}$形式，因为前者更好读。

使用齐次符号，坐标用4个值表示：$\mathbf{v}=\left(\begin{array}{llll}{v_{x}} & {v_{y}} & {v_{z}} & {v_{w}}\end{array}\right)^{T}$，其中向量是$\mathbf{v}=\left(\begin{array}{llll}{v_{x}} & {v_{y}} & {v_{z}} & {0}\end{array}\right)^{T}$，点是$\mathbf{v}=\left(\begin{array}{llll}{v_{x}} & {v_{y}} & {v_{z}} & {1}\end{array}\right)^{T}$。我们有时候也会使用3个变量来表示。对于向量个点使用相同的符号十分有利于矩阵运算，可以参考第4章的变化。在一些算法中，使用数字下标更方便，如：$\mathbf{v}=\left(\begin{array}{lll}{v_{0}} & {v_{1}} & {v_{2}}\end{array}\right)^{T}$。这些规则对2维向量同样适用。

一个3维矩阵M的元素表示为$m_{i j}, 0 \leq(i, j) \leq 2$，形式如下：
$$\mathbf{M}=\left(\begin{array}{lll}{m_{00}} & {m_{01}} & {m_{02}} \\ {m_{10}} & {m_{11}} & {m_{12}} \\ {m_{20}} & {m_{21}} & {m_{22}}\end{array}\right)$$
下面的符号用来分隔矩阵M的向量。$m_{,j}$表示第j个列向量，$m_{i,}$表示第i个行向量（以列向量的形式）。
$$\mathbf{M}=\left(\begin{array}{ccc}{\mathbf{m}, 0} & {\mathbf{m}, \mathbf{1}} & {\mathbf{m}, 2}\end{array}\right)=\left(\begin{array}{ccc}{\mathbf{m}_{x}} & {\mathbf{m}_{y}} & {\mathbf{m}_{z}}\end{array}\right)=\left(\begin{array}{c}{\mathbf{m}_{0}^{T}} \\ {\mathbf{m}_{1}^{T}} \\ {\mathbf{m}_{2}^{T}}\end{array}\right)$$

平面表示为：$\pi: \mathbf{n} \cdot \mathbf{x}+d=0$。法向量$\mathbf{n}$表示平面的方向。平面$\pi$将空间分为两部分：正半空间，满足$\mathbf{n} \cdot \mathbf{x}+d>0$，和负半空间，满足$\mathbf{n} \cdot \mathbf{x}+d<0$。其他的点位于平面上。

下表是一些其他的数学符号：
|符号|描述|
|:-|:-|
|.|点乘|
|$\times$|叉乘|
|$\mathbf{v}^{T}$|转置|
|$\perp$|一元垂直点乘|
|$\vert \cdot \vert$|矩阵的行列式|
|$\vert \cdot \vert$|标量的绝对值|
|$\|\cdot\|$|长度|
|$x^{+}$|将$x$截取到0|
|$x^{\mp}$|将$x$截取到0和1之间|
|$n !$|阶乘|
|$\left(\begin{array}{l}{n} \\ {k}\end{array}\right)$|二项式系数|

将$\perp$作用于一个向量$\mathbf{v}=\left(v_{x} \quad v_{y}\right)^{T}$，得到一个与$\mathbf{V}$垂直的向量$\mathbf{v}^{\perp}=\left(-v_{y} \quad v_{x}\right)^{T}$。我们用$|a|$表示标量$a$的绝对值，用$|\mathbf{A}|$表示矩阵$\mathbf{A}$的行列式。有时也会写成$|\mathbf{A}|=|\mathbf{a} \quad \mathbf{b} \quad \mathbf{c}|=\operatorname{det}(\mathbf{a}, \mathbf{b}, \mathbf{c})$，其中$\mathbf{a}, \mathbf{b},$ and $\mathbf{c}$是矩阵$\mathbf{A}$的列向量。

$x^{+}$和$x^{\mp}$是截断操作：
$$x^{+}=\left\{\begin{array}{ll}{x,} & {\text { if } x>0} \\ {0,} & {\text { otherwise }}\end{array}\right.$$
$$x^{\mp}=\left\{\begin{array}{ll}{1,} & {\text { if } x \geq 1} \\ {x,} & {\text { if } 0<x<1} \\ {0,} & {\text { otherwise }}\end{array}\right.$$
二项式系数$\left(\begin{array}{l}{n} \\ {k}\end{array}\right)$定义如下：
$$\left(\begin{array}{l}{n} \\ {k}\end{array}\right)=\frac{n !}{k !(n-k) !}$$

另外，我们把平面$x=0, y=0,$和$z=0$称作坐标平面或者轴对齐平面。轴$\mathbf{e}_{x}=\left(\begin{array}{lll}{1} & {0} & {0}\end{array}\right)^{T}$，$\mathbf{e}_{y}=\left(\begin{array}{lll}{0} & {1} & {0}\end{array}\right)^{T}$和$\mathbf{e}_{z}=\left(\begin{array}{lll}{0} & {0} & {1}\end{array}\right)^{T}$称为主轴或者主方向，分别叫做$x$轴，$y$轴和$z$轴。这一轴的集合叫做标准基，本书仅讨论正交基。

C函数$\operatorname{atan} 2(y, x)$是$\operatorname{atan} (y, x)$函数的拓展版。两者的区别是$-_{2}^{\pi}<\arctan (x)<_{2}^{\pi}$，$0 \leq \operatorname{atan} 2(\mathrm{y}, \mathrm{x})<2 \pi$（这里有点问题，我查阅资料显示$-\pi \leq \operatorname{atan} 2(\mathrm{y}, \mathrm{x})< \pi$）。$\operatorname{atan} 2(y, x)$函数会通过增加一个额外的参数来处理除数为0的情况。

符号$\log (n)$总是假定自然对数，即$\log _{e}(n)$。我们使用右手坐标系。颜色使用3个元素的向量表示，每个元素的范围是$[0,1]$。

### 1.2.2 几何定义

绝大多数图形硬件使用的渲染原型（也叫做绘画原型）是点，线和三角形。我们把几何实体称作模型，或者对象。一个场景就是模型的集合，还包括材质描述，光照，视角。一些对象可以有高级的几何表示，如贝塞尔曲线（Bézier curves）

### 1.2.3 着色

着色（shading），着色器（shader）和相关术语被用来指代两个不同的概念：计算机产生的可视化外观，或者是一个可编程组件（顶点着色器，着色语言）。实际含义依赖于具体的语境。

### 拓展阅读

本书的网站：realtimerendering.com