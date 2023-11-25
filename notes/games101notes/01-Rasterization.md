> 阅读材料：[Fundamentals of Computer Graphics, Fourth Edition](Fundamentals%20of%20Computer%20Graphics,%20Fourth%20Edition.pdf)
> 第10章 surface shading 10.1、10.2，第17章 Using Graphics Hardware 17.1

---

# Shading 着色

shading: the process of applying a material to an object

## 着色模型

**A simple shading model: Blinn-Phong Reflectance Model**

明确三个主要事情:

- **镜面高光** Specular hightlights
- **漫反射** Diffuse reflection
- **环境照明** Ambient lighting

**Compute light reflected toward camera at a specific shading point**

先给出一些定义：

![](img/Pasted image 20231122160830.png)

*Shading is local, no shadows will be generated!(shading != shadows)*

- 如何描述一个表面接收了多少 light/engergy？
  - 计算表面法线和光源方向向量的点积
  - ![](./img/Pasted%20image%2020231122161044.png)

光源随着距离的一个衰减：![](./img/Pasted%20image%2020231122161144.png)

**Lambertian (Diffuse) Shading**

### 漫反射项 Diffuse

shading independent of view direction

![](./img/Pasted%20image%2020231122161258.png)

- kd 表示漫反射系数，kd=1表示完全不吸收，写成三维向量还可以分别描述RGB三通道各自的吸收情况
- 看到公式中，漫反射项和view的方向无关

kd系数变化导致的结果变化：![img](./img/Pasted%20image%2020231122161446.png)

### 高光项 Specular Term（Blinn-Phong）

- ![](./img/Pasted%20image%2020231122163300.png)
- 观测方向和镜面反射方向越接近，高光越亮
- *但反射方向不好计算*
- Blinn-Phong模型对上的修改

  - ![](./img/Pasted%20image%2020231122164115.png)
  - 将判断观测方向和镜面反射方向是否接近转换为判断*法线方向和半程方向是否接近*
  - 衡量两个方向是否接近：点乘，接近->结果接近1，远离->结果接近0
  - ks 镜面反射系数
  - 简化掉了是否吸收光线的部分即n和l的点乘
  - 计算n和h的点乘上的指数系数p
    - 观察余弦的变化，发现余弦结果对各个角度的区别不大，但按照现实中的观察，高光一定是在某些角度会特别强，即需要让这个点乘项对某个范围内敏感，而对其他大部分角度较弱
    - ![](./img/Pasted%20image%2020231122164511.png)
- ks和p的变化对结果的影响示意图：![](./img/Pasted%20image%2020231122164636.png)

### 环境光照项 ambient term

- 假设：任何一个点接收到环境的光都是相同的

![](./img/Pasted%20image%2020231122164735.png)

**环境光项+漫反射项+高光项=Blinn-Phong反射模型**

![](./img/Blinn-Phong反射模型.png)

**着色模型**：目前只是针对一个点进行的

对多个点则涉及到*着色频率*问题

## 着色频率 **Shading Frequencies**

![](./img/Pasted%20image%2020231122165248.png)

- 考虑着色要在哪些点上进行？
  - 左：对整个平面；
  - 中：对每个顶点；
  - 右：对每个像素。

### 逐面着色 Flat Shading: Shade each triangle

对每个三角形进行着色

![](./img/Pasted%20image%2020231122165545.png)

### 逐顶点着色 Gouraud Shading: Shade each vertex

对每个顶点着色

![](./img/Pasted %20image%2020231122165611.png)

### 逐像素着色 Phong Shading：Shade each pixel

对每个像素着色

![](./img/Pasted%20image%2020231122165655.png)

### 三种着色的区别

![](./img/Pasted%20image%2020231122165905.png)

观察发现，几何形体同样会对最后的着色结果起到影响

### 思考问题

#### 如何计算逐顶点计算法线？

通过对某个顶点周围几个面的法线计算平均，或各个面法向量的加权平均

![](./img/Pasted%20image%2020231122170302.png)

*要注意计算向量要进行归一化*

#### 如何得到内部平滑过渡的法线？

![](./img/Pasted%20image%2020231122170448.png)

**通过重心坐标 Barycentric Interpolation**

## Graphics（Real-time Rendering Pipeline）

图形/实时渲染管线

从开始的场景到最后的一张图，这中间经历的一系列过程

![](./img/Pasted%20image%2020231122170735.png)

---

> 阅读材料：[Fundamentals of Computer Graphics, Fourth Edition](Fundamentals%20of%20Computer%20Graphics,%20Fourth%20Edition.pdf)
> - 第11章 Texture Mapping 
>   - 11.1、11.2

# 纹理映射 Texture Mapping

定义一个点的基本属性

## 纹理坐标 (u,v)

![](./img/Pasted%20image%2020231122191556.png)
- 红色：u值大
- 绿色：v值大
- **(u,v) 坐标范围认为都在 (0,1) 之内**

纹理坐标的可视化：![](./img/Pasted%20image%2020231122191707.png)

可以发现纹理是一块一块拼接起来的，但在实际场景中看不到缝

*纹理经过特殊设计从而实现无缝衔接*

纹理无缝衔接的合成的研究
- tileable texture

## 重心坐标 Barycentric Coordinates

> 三角形的三个顶点有各自的属性，那么如何将这个属性在三角形内部平滑过渡？

*为什么要进行插值？*
- 顶点上操作
- 获取在三角形内部平滑过渡的性质

*通过插值想要获得什么？*
- 纹理坐标、颜色、法向量等等

*如何插值？*
- 重心坐标 Barycentric Coordinates $(\alpha, \beta, \gamma)$

### 重心坐标

![](./img/Pasted%20image%2020231122192504.png)

三角形内部的任意一个点，都可以表示为三角形三个顶点的线性组合（*系数都为非负数*）
（三个系数之和为1只能表示这个点是和三角形共平面的）
- A的重心坐标 (1,0,0)

*如何计算任意一个点的坐标？*
- 重心坐标可以通过面积比计算得到
	- ![](./img/Pasted%20image%2020231122192834.png)
- 一个特殊的点：**三角形自己的质心**
	- ![](./img/Pasted%20image%2020231122193019.png)

**任意一个点的重心坐标计算公式：**
- ![](./img/Pasted%20image%2020231122193044.png)

### 使用重心坐标进行插值

![](./img/Pasted%20image%2020231122193241.png)

*需要注意*：
- 重心坐标没有投影不变性
	- 三维空间属性要在三维空间中进行插值，再投影到二维空间中

## 如何将纹理应用在渲染之中

### 简单的纹理映射：Diffuse Color

![](./img/Pasted%20image%2020231122193626.png)

大致思路：
- 对于任意一个光栅化的样本坐标 (x,y)
- 找到其对应的纹理坐标 (u,v)
- 再从纹理样本中找到对应位置的属性 texture.sample(u,v);
- 将光栅化的样本性质设置为对应纹理的属性即可

*这么简单吗？*
- 存在一些问题
- 纹理太小
- 纹理太大

### 纹理放大 Texture Magnification（simple case） 

*如果纹理太小？* 纹理放大问题

下面是一个简单的例子

![](./img/Pasted%20image%2020231122193951.png)

图中给出了三种解决*纹理太小*问题的方法：
- Nearest
- Bilinear
- Bicubic
#### 近邻插值 Nearest

对于sample上任意一个点都可以在纹理上找到对应的一个位置，但这个位置可能不是整数位置
- 解决办法：*round成整数*
- 即，找相邻的纹理，从而使得多个pixel都会映射到同一个texel上
- **Nearest***

#### 双线性插值 Bilinear

思考：如果想要找到红点位置所在的纹理数值？

![](./img/Pasted%20image%2020231122194329.png)
- 找到临近的四个点![](./img/Pasted%20image%2020231122194403.png)
- 定义线性插值操作：![](./img/Pasted%20image%2020231122194501.png)
- 找到水平的两个点的位置![](./img/Pasted%20image%2020231122194540.png)
	- 进行水平插值操作![](./img/Pasted%20image%2020231122194617.png)
- 再进行纵向插值![](./img/Pasted%20image%2020231122194636.png)

#### 双三次插值 Bicubic


### 纹理缩小 Texture Magnification（hard case） 

*如果纹理太大？* 纹理缩小问题

**问题反而会更严重**

如果继续使用之前的纹理映射方法，一个简单的例子：

![](./img/Pasted%20image%2020231122195241.png)

- 远处出现摩尔纹
- 近处出现锯齿

出现了*走样*问题

#### 问题分析

![](./img/Pasted%20image%2020231122195309.png)

可以发现，在近处，pixel所覆盖的纹理范围相对较小。但是*随着距离拉远，一个pixel所对应的texel范围会变得很大*。

思考之前应对走样的方法，MSAA。使用更多的样本点来进行采样

![](./img/Pasted%20image%2020231122195515.png)

可以看到结果还算不错，但问题是*开销会非常大*，越远的位置signal frequency越高，就越需要更高的sampling frequency

另一个思路，*如果不进行采样如何？*
- *获取一个range内的average*
- 算法问题：*点查询问题 point query* 和 *范围查询问题 range query*

#### 范围查询方法：MipMap 

Allowing (*fast, approx, square*) range queries

- 只能进行近似的、正方形查询

Mipmap是什么？
- ![](./img/Pasted%20image%2020231122200104.png)
- 从一张图生成一系列图
- 如上图所示，后一张图总是前一张图分辨率的一半
- 将这些图进行堆叠成不同的层 (D)![](img/Pasted image 20231122200230.png)
	- **思考问题：这么引入了多大的额外存储量？**
		- 假设初始为1，每次边长变1/2，总信息量变为1/4，即计算 1+1/4+1/16+1/64+...=4/3
		- 最后只多了1/3原存储量大小
- 使用Mipmap来进行区域查询

*问题：如何知道需要查询的范围有多大？或者说如何得到像素对应的纹理范围？*
- 将pixel映射到texel上![](./img/Pasted%20image%2020231122201134.png)
- 我们知道在屏幕上，相邻两个pixel之间距离是固定的，而映射到纹理上像素与像素之间的距离会被改变
	- ![](./img/Pasted%20image%2020231122201203.png)
	- 再使用一个正方形的框来近似这个纹理空间上被像素所覆盖的区域![](img/Pasted image 20231122201319.png)（简单方法：正方形的边长取像素距离较大的那个值）
	- 到此，剩下的就是需要查询这个正方形内的平均值是多少
		- 根据mipmap，边长为L的正方形一定会在 D（log2 L）层的层级上变为一个像素
		- 从而可以直接到对应的层级上查询并获得平均值

- 远近位置的Mipmap层级可视化![](./img/Pasted%20image%2020231122201734.png)

##### 三线性插值 Trilinear Interpolation

- *问题：这种查询方式仍然存在一些明显的边界，有没有办法构造出例如1层与2层之间的1.8层，从而使得这种查询的层级连续呢？*
	- **使用插值**
	- 先在目标层级两侧的层级上，通过双线性插值得到对应的数值![](./img/Pasted%20image%2020231122201914.png)
	- 之后将得到的这两个双线性插值结果，再进行层与层之间的双线性插值
	- *通过这种方法，就可以进行一个连续层级上的查询了*
	- 通过三线性插值后，就可以在一个浮点型的连续层级上进行查询![](./img/Pasted%20image%2020231122202357.png)

#### Mipmap的一些问题

然而实际应用了Mipmap后，远处的东西变得模糊，像是“融为一体”![](./img/Pasted%20image%2020231122202508.png)
- 这种现象叫做*overblur*
- *思考：出现这种现象的原因是什么呢？*
	- 因为Mipmap只能查询方形区域
	- 如果对应的区域并不是方形，就无能为力了
	- **解决办法：各向异性过滤**

#### 各向异性过滤 Anisotropic Filtering

![](./img/Pasted%20image%2020231122202748.png)

**思考：那么为什么各向异性过滤可以解决这个问题呢？各向异性过滤又做了什么呢？**

![](./img/Pasted%20image%2020231122202859.png)

可以看到，这种方式多了一些横向和纵向压缩拉伸的变化（Ripmap）。*从而使得这种查询方式不被局限在方形查询上*

（各向异性过滤 2x，4x，8x的区别，表示的是计算到多少层）

通过将屏幕空间映射到纹理空间上，可以发现，很多屏幕上的方形区域或者说pixel映射到文理上，并不都是对应着一个方形区域，也有可能是*被拉伸变形的矩形区域*。所以在Mipmap中简单的方形近似会使得远处出现overblur的问题。

![](./img/Pasted%20image%2020231122203029.png)

但各向异性过滤，仍然存在问题。比如图中左侧斜向的矩形，如果直接使用一个矩形去描述，仍然会存在一些近似问题。

所以，各向异性过滤目前只是补全了Mipmap中方形区域查询的短板，但是对于斜着的区域，还没有完全结局。

*那么如何解决这种斜着的区域呢？*

**EWA 过滤![](./img/Pasted%20image%2020231122203526.png)**
- 将不规则的区域拆解为多个圆形
- 多次查询这些分解出来的圆形区域
- 问题就是，开销很大

## 纹理应用

- **什么是纹理？**
- 对于现代GPU，texture = memory + range query (filtering)
	- 纹理就是一块可以做存储和查询的数据区域
- *我们可以将纹理这个概念抽象化...*

### 环境光照、环境映射 Environment Map

![](./img/Pasted%20image%2020231122205049.png)

**如何描述来自不同方向的光照信息呢？**
- *用纹理描述环境光*

#### Sphercial Environment Map

这里假设环境光源无限远，从而我们可以只记录*方向信息*不需要考虑深度信息

![](./img/Pasted%20image%2020231122205447.png)

- 可以将环境光信息存储在球面的纹理当中


将记录在球面上的环境光信息展开：![](./img/Pasted%20image%2020231122205552.png)

*可以发现存在问题，在顶部与底部出现了扭曲*

#### Cube Map

![](./img/Pasted%20image%2020231122205650.png)

将光照信息转换到立方体的表面上

![](./img/Pasted%20image%2020231122205712.png)

### 凹凸贴图/法线贴图

纹理还能影响阴影（shading）

![](./img/Pasted%20image%2020231122205924.png)

通过纹理记录表面的相对高度信息，使法线发生变化，进而使得shading变化，从而实现不把几何形体变复杂的情况下来实现相对高度的观感改变

#### Bump Mapping

**那么法线贴图到底做了什么？**

*通过定义复杂的纹理，但不改变任何几何信息，进而产生物体表面的高度变化*
- 对每个像素的法线，在进行阴影计算（shading computations）时进行一个扰动，将p改变到了n
- 从而实现通过纹理 texture 来定义每一个texel之间的高度变化height shift
- ![](./img/Pasted%20image%2020231122210617.png)

**问题：如何去对法线进行perturb 微扰呢？**

简单假设：flatland case 一个一维上的变化![](./img/Pasted%20image%2020231122210927.png)
- 首先知道原本点的坐标n(p)
- 然后计算这一点的切线，这个切线向量坐标就可以根据差分计算出来
- 因为法线和切线垂直，从而直接通过矩阵变换就可以计算得到法线的向量坐标

*那么，在实际的3D情况下呢？*

![](./img/Pasted%20image%2020231122211203.png)

**这里是在局部坐标系下，从而认为法线永远为(0,0,1)，经过上边的计算后再经过变换再变换为世界坐标系的坐标**

凹凸贴图只是改变了法线，使得纹理上出现一个凹凸的观感，并不是实际上的相对高度差

### 位移贴图 Displacement Mapping

和凹凸贴图一样，都是要用纹理去定义一个点之间的相对差别，*但是，位移贴图是真的会对顶点进行一个位置的移动*，具体效果可以参考下方对比图

![](./img/Pasted%20image%2020231122211553.png)

要求几何模型内部的三角形足够细，从而跟得上纹理的频率变化

*DirectX上可以动态曲面细分，从而根据需要来改变几何形体上三角形的细分*

### 3D Procedural Noise + Solid Modeling

三维的纹理

![](./img/Pasted%20image%2020231122211947.png)

通过特定的公式计算噪声，在经过一定计算可以转换为所需要的一些特定纹理，参考：*Perlin noise, Ken Perlin*

### Provide Precomputed Shading

纹理还可以存储一些提前计算好的结果，比如下图中的一些阴影

![](./img/Pasted%20image%2020231122212107.png)

### 3D Texture and Volume Rendering

三维贴图广泛应用在体渲染之中

![](./img/Pasted%20image%2020231122212231.png)


