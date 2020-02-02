# Chapter 5 着色基础

渲染分写实化和风格化。本节介绍的着色概念同时适用于这两种渲染方法。

## 5.1 着色模型

决定物体外观的第一步就是选择一个着色模型（*shading model*）来描述物体的颜色如何根据表面方向，视方向，光方向之类的因素而变化。

下面以*Gooch shading model*的一个变种为例进行介绍。这是一种非写实化渲染，被设计用来在技术示例中提升细节的可识别性。其基本思想是比较表面法向量与光的位置。如果法向量指向光，使用暖色调着色；如果法向量指离光，使用冷色调着色。中间角度的颜色插值获得。我们还会增加一个高光效果。结果如下图：
![](f5.2a.png)

着色模型通常有属性来控制外观的变化。我们的示例模型只有一个属性，表面颜色。如下图所示：
![](f5.2b.png)

与绝大多数着色模型一样，我们的模型受表面相对于视方向和光方向的方位的影响。这些方向通常表示被单位向量，如下图所示：
![](f5.3.png)

模型的数学定义如下：
$$
\mathbf{c}_{\text {shaded }}=s \mathbf{c}_{\text {highlight }}+(1-s)\left(t \mathbf{c}_{\text {warm }}+(1-t) \mathbf{c}_{\text {cool }}\right)
$$
方程中，我们使用了如下的中间计算：
$$
\begin{aligned}
\mathbf{c}_{\text {cool }} &=(0,0,0.55)+0.25 \mathbf{c}_{\text {surface }} \\
\mathbf{c}_{\text {warm }} &=(0.3,0.3,0)+0.25 \mathbf{c}_{\text {surface }} \\
\mathbf{c}_{\text {highlight }} &=(1,1,1) \\
t &=\frac{(\mathbf{n} \cdot 1)+1}{2} \\
\mathbf{r} &=2(\mathbf{n} \cdot \mathbf{l}) \mathbf{n}-1 \\
s &=(100(\mathbf{r} \cdot \mathbf{v})-97)^{\mp}
\end{aligned}
$$
截断操作很常见。我们使用$x^{\mp}$将结果截取到0和1之间。另一个常见的操作是线性插值。我们在模型中先在$\mathbf{c}_{\text {warm }}$和$\mathbf{C}_{\text {cool }}$之间插值，然后在得到的插值结果和$\mathbf{C}_{\text {highlight }}$之间插值。$\mathbf{r}=2(\mathbf{n} \cdot \mathbf{l}) \mathbf{n}-1$计算反射向量，将$\mathbf{l}$关于$\mathbf{n}$反射。

## 5.2 光源

我们的模型很简单，但现实中的光很复杂。光源可以有大小，形状，颜色，强度。间接光也会影响着色。基于物理的写实化着色模型需要把这些因素都考虑进去。相反的，风格化着色模型会根据需要以不同方式使用光照。一些模型甚至没有光的概念。增加光照复杂度的另一个因素是着色模型如何对待有光和无光这两种不同的情况。这样的着色模型在有光照的时候是一种外观，在无光照的时候是另一种外观。区分有光照和无光照有多种准则：距离光源的距离，阴影，法向量与光向量之间的角度是否大于90度，或者是这些因素的组合。可以将这种有光和无光的二分性拓展到光强度的连续性变化。这可以通过在无光和有光之间插值来实现。这意味着将光强度限制在一个范围内，可能是0到1。也可以用一个无范围限制的量来以另一种方式影响着色。后者的一种常规做法是将着色模型分解为有光和无光两部分，并使用光强度$k_{\text {light }}$来线性放缩有光部分：
$$
\mathbf{c}_{\text {shaded }}=f_{\text {unlit }}(\mathbf{n}, \mathbf{v})+k_{\text {light }} f_{\text {lit }}(\mathbf{l}, \mathbf{n}, \mathbf{v})
$$
这可以拓展到RGB光颜色：
$$
\mathbf{c}_{\text {shaded }}=f_{\text {unlit }}(\mathbf{n}, \mathbf{v})+\mathbf{c}_{\text {light }} f_{\text {lit }}(\mathbf{l}, \mathbf{n}, \mathbf{v})
$$
再拓展到多光源：
$$
\mathbf{c}_{\text {shaded }}=f_{\text {unlit }}(\mathbf{n}, \mathbf{v})+\sum_{i=1}^{n} \mathbf{c}_{\text {light }_{i}} f_{\text {lit }}\left(\mathbf{l}_{i}, \mathbf{n}, \mathbf{v}\right)
$$
无光的部分$f_{\text {unlit }}(\mathbf{n}, \mathbf{v})$对应于不受光影响的外观。它依据需要的效果可以有多种形式。例如，可以使用纯黑来表达无光，也可以使用风格化的外观。此外，无光部分通常用来表达一些其他类型的光，这些光并非来自于场景中显示摆放的光源。例如，天空的光和周围物体反射的光，都使用无光部分来表达。

光的照射效果可以使用射线可视化。击中表面的射线的密度对应于光的强度。下图是光照射的横截面示意图：
![](f5.4.png)
击中表面的光线之间的空间大小反比于$\mathbf{l}$和$\mathbf{n}$之间夹角的余弦值。因此，击中表面的射线密度正比于$\mathbf{l}$和$\mathbf{n}$之间夹角的余弦值。将光方向定义为光照射方向取反是为了计算方便。还要注意背面照射的光不应该贡献亮度，因此需要一步截断操作。有：
$$
\mathbf{c}_{\text {shaded }}=f_{\text {unlit }}(\mathbf{n}, \mathbf{v})+\sum_{i=1}^{n}\left(\mathbf{l}_{i} \cdot \mathbf{n}\right)^{+} \mathbf{c}_{\text {light }_{i}} f_{\text {lit }}\left(\mathbf{l}_{i}, \mathbf{n}, \mathbf{v}\right)
$$
该公式是对上一个公式的特殊化。它可用于PBS，也可用于风格化渲染以保证光照的整体一致性。

$f_{\mathrm{lit}}()$的最简单的形式就是一个恒定的颜色：
$$
f_{\mathrm{lit}}()=\mathbf{c}_{\mathrm{surface}}
$$
这将得到如下的着色模型：
$$
\mathbf{c}_{\text {shaded }}=f_{\text {unlit }}(\mathbf{n}, \mathbf{v})+\sum_{i=1}^{n}\left(\mathbf{l}_{i} \cdot \mathbf{n}\right)^{+} \mathbf{c}_{\text {light }_{i}} \mathbf{c}_{\text {surface }}
$$
改模型的光照部分对应于*Lambertian*光照模型。改模型适用于完美漫反射表面，即完美粗糙的表面。本节的介绍只是兰伯特模型的一个简化的阐释。兰伯特模型本身就可以用于简单的着色，并且也是许多着色模型的关键组件。

光源于着色模型的交互主要受2个参数控制：指向光源的向量$\mathbf{l}$和光的颜色$\mathbf{c}_{light}$。不同光源的主要区别就在于这2个参数如何变化。光源被近似视为无穷小的点。

### 5.2.1 方向光

方向光是最简单的光源模型，其$\mathbf{l}$和$\mathbf{c}_{light}$恒定不变，例如太阳。也可以将方向光的概念进行拓展，允许$\mathbf{c}_{light}$变化而保持$\mathbf{l}$不变。

### 5.2.2 精确光

*punctual lights*不是指光源很“守时”，而是指光源有一个确定的位置。这样的光源也没有维度，形状和大小。*punctual*一词来自于拉丁文*punctus*，意思是“点”。术语“点光”指代一种特殊的光源，该光源向所有方向均匀发射光线。因此，点光和聚光是两种不同的精确光。光方向$\mathbf{l}$依赖于当前着色点$\mathbf{p}_0$相对于精确光$\mathbf{p}_{light}$的位置：
$$
\mathrm{I}=\frac{\mathbf{p}_{\text {light }}-\mathbf{p}_{0}}{\left\|\mathbf{p}_{\text {light }}-\mathbf{p}_{0}\right\|}
$$
该方程是一个向量标准化操作。但有时候我们需要该操作的中间结果：
$$
\begin{aligned}
&\mathbf{d}=\mathbf{p}_{\text {light }}-\mathbf{p}_{0}\\
&r=\sqrt{\mathbf{d} \cdot \mathbf{d}}\\
&1=\frac{\mathrm{d}}{r}
\end{aligned}
$$
$r$被用来计算光的衰减。

#### 点光/泛光

朝所有方向均匀发射光线的精确光被称为点光（*point lights*）或者泛光（*omni lights*）。对于点光，$\mathbf{c}_{light}$随距离$r$变化，也就是衰减。下图显示了衰减的几何解释：
![](f5.5.png)
如图所示，在一个给定的表面，光线之间的距离正比于平面到点光的距离。由于光线之间的距离增长发生在平面的两个维度，因此光线的密度（也即$\mathbf{c}_{light}$）正比于平面到点光的距离的反平方$1 / r^{2}$。令$\mathbf{c}_{light_0}$表示$\mathbf{c}_{light}$在参考距离$r_0$的值，有：
$$
\mathbf{c}_{\mathrm{light}}(r)=\mathbf{c}_{\mathrm{light}_{0}}\left(\frac{r_{0}}{r}\right)^{2}
$$
上式通常被称为光的平方反比衰减（*inverse-square light attenuation*）。尽管该公式技术上讲是正确的，但它还有一些问题。因此不能直接用在实际应用中。

第一个问题发生在距离相对较小时。当$r$趋近于0，$\mathbf{c}_{light}$会无限制增长。当$r$等于0，会发生除0错误。一种解决办法【r861】是在分布上加一个很小的值$\epsilon$：
$$
\mathbf{c}_{\mathrm{light}}(r)=\mathbf{c}_{\mathrm{light}_{0}} \frac{r_{0}^{2}}{r^{2}+\epsilon}
$$
$\epsilon$的值依赖于具体的应用。例如，Unreal引擎使用$\epsilon=1$cm【r861】。CryEngine【r1591】和Frostbite【r960】使用了另一种方法，将$r$截断到一个最小值$r_{\mathrm{min}}$：
$$
\mathbf{c}_{\mathrm{light}}(r)=\mathbf{c}_{\operatorname{light}_{0}}\left(\frac{r_{0}}{\max \left(r, r_{\min }\right)}\right)^{2}
$$
不像上一个方法中的$\epsilon$可以比较自由地设置，这里的$r_{\mathrm{min}}$有物理解释：它代表物理光源的半径。小于$r_{\mathrm{min}}$的$r$值表示着色表面穿插到了物理光源的内部，这显然是不可能的。

第二个问题发生在非常远的距离。该问题不是视觉上的问题，而是性能上的问题。尽管光强度随着距离衰减，但永远不会到0。为了高效渲染，我们需要光的强度在某个有限距离处减小到0。有许多种修改方法可以实现该效果，但引入的修改要越小越好。为了避免在光影响范围的边界处出现突变，要求修改后的函数的值和导数要在相同的距离处达到0。一种解决办法是用一个窗口函数（*windowing function*）去乘以原来的方程。Unreal Engine【r861】和Frostbite【r960】使用的一个窗口函数是【r860】：
$$
f_{\mathrm{win}}(r)=\left(1-\left(\frac{r}{r_{\mathrm{max}}}\right)^{4}\right)^{+2}
$$
其中$+2$表示先截断再平方。下图分别显示了原始的平方反比曲线，前述的窗口函数，以及这两者的乘积：
![](f5.6.png)

应用的实际需要也会影响方法的选择。例如，当距离衰减函数以一个很低的空间频率（光照贴图或者逐顶点）进行采样时，要求导数在$r_{\mathrm{max}}$处等于0就非常重要。CryEngine不适用光照贴图和逐顶点光照，所以采用了一种很简单的做法：在$0.8r_{\mathrm{max}}$到$r_{\mathrm{max}}$范围内切换到线性衰减【r1591】。

对于一些应用来说，符合平方反比曲线并不是要务，它们会使用全新的函数，形式如下：
$$
\mathbf{c}_{\operatorname{light}}(r)=\mathbf{c}_{\operatorname{light}_{0}} f_{\text {dist }}(r)
$$
其中$f_{\text {dist }}(r)$是距离的函数，被称作距离衰减函数（*distance falloff functions*）。一些应用可能出于性能原因不使用平方反比衰减函数，例如正当防卫2【r1379】：
$$
f_{\text {dist }}(r)=\left(1-\left(\frac{r}{r_{\text {max }}}\right)^{2}\right)^{+2}
$$
在其他应用中，衰减函数的选择会有更多考虑。例如，在Unreal Engine中，不管是写实化还是风格化渲染，都有两种光衰减模式：一种是前述的平方反比模式，另一种是指数衰减模式。该模式可以通过微调来获得各种各样的衰减曲线【r1802】。古墓丽影（2013）使用样条编辑工具来设置衰减曲线，这可以对曲线的形状进行更加精细的控制。

#### 聚光

现实世界中的光源往往具有方向性，这种方向性使用方向衰减函数$f_{\operatorname{dir}}(\mathbf{l})$来表达。将其与距离衰减函数结合得到：
$$
\mathbf{c}_{\mathrm{light}}=\mathbf{c}_{\mathrm{light}_{0}} f_{\mathrm{dist}}(r) f_{\mathrm{dir}}(\mathbf{l})
$$
不同的$f_{\operatorname{dir}}(\mathbf{l})$可以产生不同的光照效果。一种重要的类型是聚光，其光的范围是一个圆锥体。聚光的方向衰减函数绕聚光的方向$s$圆形对称，因此可以表达为$s$和$-\mathrm{l}$之间的角度$\theta_{s}$的函数。注意需要对光向量取反。

聚光通常有一个本影角$\theta_{u}$，满足当$\theta_{s} \geq \theta_{u}$时有$f_{\operatorname{dir}}(\mathbf{l})=0$。这个角类似于最大衰减距离$r_{max}$，可以用于剔除。聚光还有一个半影角$\theta_{p}$，这定义了一个内部的圆锥体，其中的光不受方向衰减的影响。如下图所示：
![](f5.7.png)

聚光有多种不同的方向衰减函数，但都大同小异。例如，Frostbite【r960】使用的函数$f_{\operatorname{dir}_{\mathrm{F}}}(1)$和three.js浏览器图形库【r218】使用的函数$f_{\operatorname{dir}_{\mathrm{T}}}(1)$分别如下：
$$
\begin{aligned}
t &=\left(\frac{\cos \theta_{s}-\cos \theta_{u}}{\cos \theta_{p}-\cos \theta_{u}}\right)^{\mp} \\
f_{\text {dir }_{F}}(1) &=t^{2} \\
f_{\text {dir }_{T}}(1) &=\text { smoothstep }(t)=t^{2}(3-2 t)
\end{aligned}
$$
smoothstep是一个3次多项式，常用于平滑插值。下图显示了不同光的效果：
![](f5.8.png)
图中从左到右分别是方向光，无衰减的点光和平滑变换的聚光。注意到点光虽然没有衰减，但是边缘仍然会变暗，这是因为光向量和法向量之间角度的变化。

#### 其他精确光

方向衰减函数并不局限于前述的聚光衰减函数，它可以表达任意的方向变化，包括复杂的表格模式。IES（illuminating Engineering Society）针对这种模型定义了一个标准的文件格式。IES的相关资料被用于游戏《杀戮地带：暗影堕落》【r379，380】，以及Unreal【r861】和Frostbite【r960】。【r961】总结了如何解析并使用这种格式的文件。游戏《古墓丽影》（2013）【r953】使用的精确光在世界坐标的$x$，$y$，$z$轴方向有独立的距离衰减函数。光强度随时间的变化也是用曲线来表达，以实现闪烁的火把的效果。

### 5.2.3 其他类型的光

方向光和精确光主要通过光方向$\mathbf{l}$来刻画。不同类型的光可以通过使用其他计算光方向的方法来定义。例如，《古墓丽影》有一种胶囊光，它使用线段来代替点。对于每一个着色像素，使用到线段上最近点的方向作为光方向。现实中，光源可以有大小和形状，并从多个方向照射表面。在渲染中，这种光叫做区域光（*area lights*）。它在实时渲染中变得越来越流行。区域光渲染技术主要分为两类：模拟区域光因被部分遮挡而造成的阴影边缘的软化，和区域光在表面上的着色效果。

## 5.3 实现着色模型

本节使用代码实现上述着色方程。

### 5.3.1 评估频率

当设计着色实现时，需要根据评估频率（*frequency of evaluation*）来划分计算。首先，判断一个给定计算的结果在一整个*draw call*中是否不变。在这种情况下，计算可由应用进行，结果通过着色去输入传递给图形API。即使在这一类别中也存在不同的评估频率。最简单的形式即使着色方程中的恒定子表达式，或者是任何基于不变量（例如硬件参数和配置选项）的计算。这种着色计算可能在编译的时候就已经计算完毕，因此不需要使用一致着色器输入。此外，该类计算也可以在离线情况下预先计算，并在运行时载入。

另一种情况是计算结果虽然不断变化，但是速度很慢，以至于没必要每帧更新，例如游戏世界中的太阳光。如果这种计算的消耗很大，我们可以把它分散在多个帧中。

其他情况包括每帧一次的计算，如连结是矩阵和投影矩阵；每个模型计算一次，如根据位置更新光照参数；每个*draw call*计算一次，如更新材质的参数。根据评估频率划分一致着色器输入有利于提升应用的性能【r1165】。

如果一个着色计算的结果在一个*draw call*中会发生变化，那么它就不能通过一致着色器输入传递到着色器中，而是必须在可编程着色器阶段被计算，或者通过可变着色器输入传递到其他阶段。理论上，着色器计算可以发生在任何可编程阶段，不同阶段有不同的评估频率：
* 顶点着色器：曲面细分之前每个顶点评估一次
* 壳着色器：每个面片评估一次
* 域着色器：曲面细分后每个顶点评估一次
* 几何着色器：每个原型评估一次
* 像素着色器：每个像素评估一次

在实际中，绝大多数着色计算每个像素进行一次，主要用像素着色器实现，计算着色器也越来越受到重视。其他阶段主要用于几何操作，如变换和形变。其原因可以通过比较逐顶点着色和逐像素着色的效果来理解。在以前，这两者分别被叫做*Gouraud shading*【r578】和*Phong shading*【r1414】。这里使用的着色模型与前述的模型（$\mathbf{c}_{\text {shaded }}=s \mathbf{c}_{\text {highlight }}+(1-s)\left(t \mathbf{c}_{\text {warm }}+(1-t) \mathbf{c}_{\text {cool }}\right)$）类似，但作了修改以适用于多光源。下图显示了结果：
![](f5.9.png)
可以看到，对于龙这个模型，其顶点数很密，因此逐顶点和逐像素着色的差别很小。但是对于茶壶，逐顶点着色会产生一些视觉上的错误，如高光的棱角。而对于有两个三角形组成的面片，逐顶点着色是完全错误的。错误的原因是着色器算中有一部分计算（如高光）在网格表面上非线性变化，而顶点着色器的输出在三角形上会被线性插值。

原则上，可以只在像素种色器中计算高光，而把其他计算放在顶点着色器中。理论上这会产生相同的结果，并且节省计算。但在实际中，这种混合计算往往不是最佳的。线性变化的部分计算量非常小，以这种方式分割着色计算往往会增加开销，例如重复的计算和额外的可变输入。这反而会降低性能。

顶点着色器主要负责变换。其输出的表面属性需要被变换到合适的坐标系。这些属性包括位置，法向量，切向量等。在被通过可变着色器输入传递给像素着色器之前，这些属性会在三角形上被线性插值。尽管顶点着色器通常输出单位法向量，插值会改变法向量的长度，如下图所示：
![](f5.10.png)
因此像素着色器需要重新单位化法向量。上图还展示了另一个问题：如果法向量在不同顶点间变化巨大（可能是顶点混合的副作用），那么插值结果会朝更长的法向量倾斜。因此，对于需要被插值的向量，通常的做法是在插值前后分别进行一次单位化。与法向量不同，指向特定位置的向量（如视向量和光向量）通常不需要插值。这些向量在像素着色器中使用插值得到的位置来计算。如果由于特殊原因需要插值这些向量，则在插值前不要进行单位化，否则会产生错误的结果，如下图所示：
![](f5.11.png)

之前我们提到过，顶点着色器需要将表面属性变换到合适的坐标系。摄像机和光的位置通常由应用负责变换到相同的坐标系。但是，哪一个坐标系才是合适的坐标系？可能的选择有世界坐标，或者摄像机的局部坐标，甚至是模型的坐标系。具体的选择需要考虑具体的应用特性，如灵活性，性能，简单性。例如，如果场景中包含大量的灯光，那么可以选择世界坐标系以避免变换光的位置。也可以选择摄像机空间，以优化像素着色器中关联视向量的计算并提高精度。

也有一些特殊的着色模型，如平面着色（*flat shading*），如下图所示：
![](f5.12.png)
原则上平面着色可以使用几何着色器实现，但最近的实现通常使用顶点着色器。方法是将每个原型的属性与其第一个顶点关联，并关闭顶点属性插值。当关闭顶点属性插值后（每个顶点属性可以分别设置是否关闭插值），第一个顶点的值会被传递到该原型的所有像素。

### 5.3.2 实现样例

下面介绍一个实现着色模型的例子。使用的着色方程如下：
$$
\mathbf{c}_{\text {shaded }}=\frac{1}{2} \mathbf{c}_{\text {cool }}+\sum_{i=1}^{n}\left(\mathbf{l}_{i} \cdot \mathbf{n}\right)^{+} \mathbf{c}_{\text {light }_{i}}\left(s_{i} \mathbf{c}_{\text {highlight }}+\left(1-s_{i}\right) \mathbf{c}_{\text {warm }}\right)
$$
中间计算如下：
$$
\begin{aligned}
\mathbf{c}_{\text {cool }} &=(0,0,0.55)+0.25 \mathbf{c}_{\text {surface }} \\
\mathbf{c}_{\text {warm }} &=(0.3,0.3,0)+0.25 \mathbf{c}_{\text {surface }} \\
\mathbf{c}_{\text {highlight }} &=(2,2,2) \\
\mathbf{r}_{i} &=2\left(\mathbf{n} \cdot \mathbf{l}_{i}\right) \mathbf{n}-1_{i} \\
s_{i} &=\left(100\left(\mathbf{r}_{i} \cdot \mathbf{v}\right)-97\right)^{\mp}
\end{aligned}
$$
该光源遵循之前介绍的多光源着色方程，即：
$$
\mathbf{c}_{\text {shaded }}=f_{\text {unlit }}(\mathbf{n}, \mathbf{v})+\sum_{i=1}^{n}\left(\mathbf{l}_{i} \cdot \mathbf{n}\right)^{+} \mathbf{c}_{\text {light }_{i}} f_{\text {lit }}\left(\mathbf{l}_{i}, \mathbf{n}, \mathbf{v}\right)
$$
其中的有光项和无光项分别是：
$$
\begin{array}{l}
{f_{\text {unlit }}(\mathbf{n}, \mathbf{v})=\frac{1}{2} \mathbf{c}_{\text {cool }}} \\
{f_{\text {lit }}\left(\mathbf{l}_{i}, \mathbf{n}, \mathbf{v}\right)=s_{i} \mathbf{c}_{\text {highlight }}+\left(1-s_{i}\right) \mathbf{c}_{\text {warm }}}
\end{array}
$$
在绝大多数的应用中，材质属性中可变值（如$\mathbf{c}_{surface}$）被存储在顶点数据中，或者更一般的，存储在纹理中。然而，为了保持示例的简单性，我们假定$\mathbf{c}_{surface}$在模型层面是恒定的。

实现使用动态分支迭代所有光源。但是，渲染大量的光源需要其他的方法。另外，为了实现的简单，我们限制光源类型为点光源。

着色器模型不是孤立的，而是实现在一个更大的渲染框架中。我们的样例基于WebGL 2。我们会按照由内向外的顺序实现，即先是像素着色器，然后顶点着色器，最后是应用端的图形API调用。

首先是着色器输入输出的定义。使用GLSL的术语，着色器输入包含两类：一致输入和可变输入。下面是像素着色器可变输入和输出的定义：
```GLSL
in vec3 vPos ;
in vec3 vNormal ;
out vec4 outColor ;
```
像素着色器只有一个输出，就是最终的颜色。像素着色器的输入要与顶点着色器的输出相匹配。像素着色器有2个可变输入：位置和法向量，都位于世界坐标中。一致输入的数量要多得多，这里为了简单，只列出与光源相关的2个定义：
```GLSL
struct Light {
    vec4 position ;
    vec4 color ;
};

uniform LightUBlock {
    Light uLights [ MAXLIGHTS ];
};

uniform uint uLightCount ;
```
由于是点光，因此需要位置和颜色。使用vec4而不是vec3是为了遵循GLSL std140数据分布标准的限制。尽管这会浪费空间，但它保证了CPU和GPU之间数据分布的一致性。Light结构的数组被定义在一个命名的uniform块中，这是GLSL的一个特性，通过将一组一致变量绑定到一个缓冲对象上以加速数据传输。应用端会在编译着色器之前使用正确的值（本示例中是10）来替换MAXLIGHTS字符串。整数uLightCount是光的实际数量。

下面是像素着色器的代码：
```GLSL
vec3 lit( vec3 l, vec3 n, vec3 v) {
    vec3 r_l = reflect (-l, n);
    float s = clamp (100.0 * dot(r_l , v) - 97.0 , 0.0 , 1.0) ;
    vec3 highlightColor = vec3 (2 ,2 ,2);
    return mix( uWarmColor , highlightColor , s);
}

void main () {
    vec3 n = normalize ( vNormal );
    vec3 v = normalize ( uEyePosition .xyz - vPos );
    outColor = vec4 ( uFUnlit , 1.0) ;
    for ( uint i = 0u; i < uLightCount ; i++) {
        vec3 l = normalize ( uLights [i]. position .xyz - vPos );
        float NdL = clamp (dot(n, l), 0.0 , 1.0) ;
        outColor .rgb += NdL * uLights [i]. color .rgb * lit(l,n,v);
    }
}
```
注意$f_{\text {unlit }}()$和$\mathbf{c}_{warm }$的值以一致变量的形式传入。mix()函数执行插值操作。

下面关注顶点着色器。由于已经看过像素着色器的一致变量定义的例子，这里省略一致变量的定义。下面是可变输入和输出的定义：
```GLSL
layout ( location =0) in vec4 position ;
layout ( location =1) in vec4 normal ;
out vec3 vPos ;
out vec3 vNormal ;
```
输入定义包含指定数据在顶点数组中如何分布的指令。下面是顶点着色器代码：
```GLSL
void main () {
    vec4 worldPosition = uModel * position ;
    vPos = worldPosition .xyz;
    vNormal = ( uModel * normal ).xyz;
    gl_Position = viewProj * worldPosition ;
}
```
顶点着色器中有一些通用的操作。位置和法向量会被变换到世界空间并传递给像素着色器。最后，位置被变换到齐次剪裁空间并传递给gl_Position。该变量是光栅化需要使用的一个系统定义变量。任何顶点着色器都需要输出gl_Position变量。

注意法向量并没有做单位化操作。因为在原始的网格数据中它们的长度就是1，并且我们也没有做任何以非均匀方式修改它们长度的操作，如定点混合或者非均匀放缩。模型矩阵可能包含均匀放缩项，但那会成比例地修改所有法向量的长度，因此不会产生之前所提到的法向量偏移问题。

使用WebGL API设定像素着色器的代码如下：
```GLSL
var fSource = document . getElementById (" fragment "). text . trim ();

var maxLights = 10;
fSource = fSource . replace (/ MAXLIGHTS /g, maxLights . toString ());

var fragmentShader = gl. createShader (gl. FRAGMENT_SHADER );
gl. shaderSource ( fragmentShader , fSource );
gl. compileShader ( fragmentShader );
```
这里把MAXLIGHTS替换为了合适的数值。绝大多数渲染框架提供类似的预编译控制。其他应用端代码被省略，如设置一致变量，初始化顶点数组，清空，绘制，等等。

### 5.3.3 材质系统

