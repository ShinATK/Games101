# 作业 2：Triangle and Z-Buffering

#cg #games101

## 问题

> 自行填写并调用函数 `rasterize_triangle(const Triangle& t)` 在屏幕上栅格化一个实心三角形。

函数 `rasterize_triangle(const Triangle& t)` 的内部工作流程如下：

- 创建三角形的2维 bounding box
- 遍历 bounding box 内所有像素（整数索引）。然后使用像素中心的屏幕空间坐标来检查中心点是否在三角形内
- 如果在内部，则将其位置处的 **插值深度值（interpolated depth value）** 与深度缓冲区（depth buffer）中的相应值进行比较
- 如果当前点更靠近相机，设置像素颜色并更新深度缓冲区（depth buffer）

需要修改的函数如下：

- `rasterize_triangle()`: 执行三角形栅格化算法
- `static bool insideTriangle()`: 测试点是否在三角形内

正确的输出结果如图：

<img src=./img/hw2_output.png>

## 确定思路

### 判断测试点是否在三角形内

利用测试点与三角形三个顶点之间构成的向量计算叉积。最后得到的三个叉积结果应该为同方向

### 三角形栅格化算法

给定三角形位置，对整个显示范围内的像素进行扫描。利用编写好的函数判断像素是否在三角形内部。

如果在，则将该位置处的插值深度值与深度缓冲区存储的深度值进行比较，如果插值深度小于深度缓冲区深度，则更新缓冲区的深度值，并将像素颜色设置为对应三角形的颜色。


## 开始编写

### 两个函数

`insideTriangle()` ：判断测试点是否在三角形内

```cpp
// 判断测试点是否在三角形内
static bool insideTriangle(float x, float y, const Vector3f* _v)
{
    //测试点的坐标为(x, y)
    //三角形三点的坐标分别为_v[0], _v[1], _v[2]
    //叉乘公式为(x1, y1)X(x2, y2) = x1*y2 - y1*x2

// STEP01
// 准备三角形各边的的向量
    Eigen::Vector2f side1;
    side1 << _v[1].x() - _v[0].x(), _v[1].y() - _v[0].y();
    Eigen::Vector2f side2;
    side2 << _v[2].x() - _v[1].x(), _v[2].y() - _v[1].y();
    Eigen::Vector2f side3;
    side3 << _v[0].x() - _v[2].x(), _v[0].y() - _v[2].y();

// STEP02
// 准备测量点和三角形各点连线的向量
    Eigen::Vector2f v1;
    v1 << x - _v[0].x(), y - _v[0].y();
    Eigen::Vector2f v2;
    v2 << x - _v[1].x(), y - _v[1].y();
    Eigen::Vector2f v3;
    v3 << x - _v[2].x(), y - _v[2].y();

// STEP03
// 三角形各边的的向量叉乘测量点和三角形各点连线的向量
    float z1 = side1.x() * v1.y() - side1.y() * v1.x();
    float z2 = side2.x() * v2.y() - side2.y() * v2.x();
    float z3 = side3.x() * v3.y() - side3.y() * v3.x();

// STEP04
// 判断叉乘结果是否有相同的符号
    if ((z1 > 0 && z2 > 0 && z3 > 0) || (z1 < 0 && z2 < 0 && z3 < 0))
    {
        return true;
    }
    else
    {
        return false;
    }
}
```

`rasterize_triangle()` ：栅格化三角形

```cpp
//执行三角形栅格化算法
void rst::rasterizer::rasterize_triangle(const Triangle& t)
{
    auto v = t.toVector4();

// STEP01
// 寻找 bounding box
    // 用矩形将三角形包围起来,找到矩形的四个顶点，构建三角形包围盒
    float min_x = std::min(v[0].x(), std::min(v[1].x(), v[2].x()));
    float max_x = std::max(v[0].x(), std::max(v[1].x(), v[2].x()));
    float min_y = std::min(v[0].y(), std::min(v[1].y(), v[2].y()));
    float max_y = std::max(v[0].y(), std::max(v[1].y(), v[2].y()));

// STEP02
// 遍历 bounding box 中所有测试点
    // 两层 for 循环遍历
    for (int x = min_x; x <= max_x; x++)
    {
        for (int y = min_y; y <= max_y; y++)
        {
// STEP03
// 判断像素是否在三角形内部
            float min_depth = FLT_MAX; //最小深度，默认是无穷远
            if (insideTriangle(x, y, t.v))
            {
                // 如果在内部，则将其位置处的插值深度值 (interpolated depth value) 与深度缓冲区 (depth buffer) 中的相应值进行比较
                auto tup = computeBarycentric2D(x, y, t.v);
                float alpha, beta, gamma;

                std::tie(alpha, beta, gamma) = tup;
                float w_reciprocal = 1.0 / (alpha / v[0].w() + beta / v[1].w() + gamma / v[2].w());
                float z_interpolated = alpha * v[0].z() / v[0].w() + beta * v[1].z() / v[1].w() + gamma * v[2].z() / v[2].w();
                z_interpolated *= w_reciprocal;
                // 更新最小深度值
                min_depth = std::min(z_interpolated, min_depth);
                if (depth_buf[get_index(x, y)] > min_depth)
                {//如果x,y所在点的深度小于z-buffer的深度
                    Vector3f color = t.getColor();// 获得最上层应该更新的颜色
                    Vector3f point;
                    point << x, y, min_depth;
                    //更新深度
                    depth_buf[get_index(x, y)] = min_depth;
                    //更新所在点的颜色
                    set_pixel(point, color);
                }
            }
        }
    }
}
```

### 运行结果

<img src=./img/hw2_rasterize_triangle_output.png width=50%>

## 提高项

> 用 super-sampling 处理 Anti-aliasing : 注意到当放大图像时，图片边缘会有锯齿感，可以使用 super-sampling 来解决这个问题。
> 方法：对每个像素进行 2*2 采样，并比较前后的结果（这里不考虑像素与像素之间的样本复用）。需要注意的是，对于像素内的每一个样本都需要维护它自己的深度值，即每一个像素都需要维护一个 `sample_list` 。最后，如果实现正确的话，得到的三角形不应该有不正常的黑边

### MSAA 算法

多重采样抗锯齿 Multisampling Anti-Aliasing，MSAA，最常见的反锯齿算法，首先来自于OpenGL

只对z缓冲（z-buffer）和模板缓存（stencil buffer）中的数据进行超级采样抗锯齿的处理。可以简单理解为只对多边形的边缘进行抗锯齿处理。从而相比SSAA对画面中的所有数据进行处理，MSAA对资源的消耗需求大幅度减少，不过在画纸上可能稍有不如SSAA。

### 算法实现

```cpp
void rst::rasterizer::rasterize_triangle(const Triangle& t)

{//执行三角形栅格化算法

   auto v = t.toVector4();

   // 相邻两个像素之间差值为1
   // 这里将每个像素分成4份，即进行 2*2 采样
   // x，y分别取值
   // +0.25, +0.25; +0.75, +0.25; +0.75, +0.75; +0.25, +0.75;
   std::vector<float> _msaa{ 0.25,0.25,0.75,0.75,0.25 };

   float min_x = std::min(v[0].x(), std::min(v[1].x(), v[2].x()));
   float max_x = std::max(v[0].x(), std::max(v[1].x(), v[2].x()));
   float min_y = std::min(v[0].y(), std::min(v[1].y(), v[2].y()));
   float max_y = std::max(v[0].y(), std::max(v[1].y(), v[2].y()));

   for (int x = min_x; x <= max_x; x++)
   {
       for (int y = min_y; y <= max_y; y++)
       {
           float min_depth = FLT_MAX;
           float count = 0.0;
           for (int k = 0; k < 4; k++)
           {// 遍历采样的四个点
               if (insideTriangle(x + _msaa[k + 1], y + _msaa[k], t.v))
               {// 寻找四个采样点中处于三角形内部的点
                   auto tup = computeBarycentric2D(x + _msaa[k + 1], y + _msaa[k], t.v);
                   float alpha, beta, gamma;
                   std::tie(alpha, beta, gamma) = tup;
                   float w_reciprocal = 1.0 / (alpha / v[0].w() + beta / v[1].w() + gamma / v[2].w());
                   float z_interpolated = alpha * v[0].z() / v[0].w() + beta * v[1].z() / v[1].w() + gamma * v[2].z() / v[2].w();
                   z_interpolated *= w_reciprocal;
                   count += 0.25; // 每个采样点占比 1/4
                   min_depth = std::min(z_interpolated, min_depth);
               }
           }
           
           if (count > 0 && depth_buf[get_index(x, y)] > min_depth)
           {// count不为0，就说明该像素至少有 1/4 部分在三角形内
               Vector3f color = t.getColor() * count;// 给颜色加上采样点权重
               Vector3f point;
               point << x, y, min_depth;
               depth_buf[get_index(x, y)] = min_depth;
               set_pixel(point, color);
           }
       }
   }
}
```
### 运行结果

<img src=./img/hw2_msaa_output.png width=50%>

与之前的运行结果进行对比可以发现，先前蓝色三角形边缘处明显的锯齿感在此时已经几乎感觉不到了。

### 黑边分析

仔细观察运行结果，发现在绿色三角形和蓝色三角形重叠部分的边缘上出现了很窄的黑边。

修改代码，让蓝色三角形处于绿色三角形上方，进行对比。

<img src=./img/hw2_msaa_output_blueUp.png width=50%>

为了对比，这里再给出不使用 MSAA 算法时，蓝色三角形在上方时的情况：

<img src=./img/hw2_output_blueUp.png width=50%>

可以发现，黑色描边只在使用了 MSAA 采样算法时出现。

因为在 MSAA 中通过计算图形对某个像素的覆盖率来调整位于图形边缘位置的像素颜色，当覆盖率过低时，接近于 0 的权重值会使得原本的 rgb 颜色数值同样接近于 0，进而导致显示出的颜色接近于黑色。同时蓝色三角形又位于绿色三角形的下方，这就导致重合部分的蓝色完全不会写入像素。

### 解决办法

对每个像素的四个采样点用深度和颜色表进行维护，最后设置像素颜色时，根据四个采样点的颜色之和进行计算。

```cpp
void rst::rasterizer::rasterize_triangle(const Triangle& t) {//执行三角形栅格化算法
    auto v = t.toVector4();
    std::vector<float> msaa{ 0.25,0.25,0.75,0.75,0.25 };

    // 用矩形将三角形包围起来,找到矩形的四个顶点，构建三角形包围盒
    float min_x = std::min(v[0].x(), std::min(v[1].x(), v[2].x()));
    float max_x = std::max(v[0].x(), std::max(v[1].x(), v[2].x()));
    float min_y = std::min(v[0].y(), std::min(v[1].y(), v[2].y()));
    float max_y = std::max(v[0].y(), std::max(v[1].y(), v[2].y()));

    // 遍历三角形包围盒中的所有测试点
    for (int x = min_x; x <= max_x; x++)
    {
        for (int y = min_y; y <= max_y; y++)
        {
            float min_depth = FLT_MAX; //最小深度，默认是无穷远
            int eid = get_index(x, y) * 4;
            for (int k = 0; k < 4; k++)
            {
                if (insideTriangle(x + msaa[k + 1], y + msaa[k], t.v))
                {
                    //如果在三角形内部，计算当前深度,得到当前最小深度
                    auto tup = computeBarycentric2D(x, y, t.v);
                    float alpha, beta, gamma;
                    std::tie(alpha, beta, gamma) = tup;
                    float w_reciprocal = 1.0 / (alpha / v[0].w() + beta / v[1].w() + gamma / v[2].w());
                    float z_interpolated = alpha * v[0].z() / v[0].w() + beta * v[1].z() / v[1].w() + gamma * v[2].z() / v[2].w();
                    z_interpolated *= w_reciprocal;

// depth_sample 维护采样点深度值
// frame_sample 维护采样点颜色，/4 为了保证求和后其整体亮度不变
                    if (z_interpolated < depth_sample[eid + k]) {// 根据插值更新深度列表和颜色
                        depth_sample[eid + k] = z_interpolated;
                        frame_sample[eid + k] = t.getColor() / 4;
                    }

                    min_depth = std::min(depth_sample[eid+k], z_interpolated);
                }
            }
            Vector3f color = frame_sample[eid + 0] + frame_sample[eid + 1] + frame_sample[eid + 2] + frame_sample[eid + 3];
            Vector3f point;
            point << x, y, min_depth;
            set_pixel(point, color);
            depth_buf[get_index(x, y)] = std::min(min_depth, depth_buf[get_index(x,y)]);
        }
    }
}
```

头文件中声明

```cpp
namespace rst{
    class rasterizer{
        ...
        private:
            std::vector<Eigen::Vector3f> frame_sample;
            std::vector<float> depth_sample;
    };
}
```

构造函数初始化：

```cpp
rst::rasterizer::rasterizer(int w, int h) : width(w), height(h)
{
    frame_buf.resize(w * h);
    depth_buf.resize(w * h);
// 因为每个像素以 2*2 采样
// 所以需要维护的列表是之前的 4 倍
    frame_sample.resize(w * h * 4);
    depth_sample.resize(w * h * 4);
}
```

```cpp
void rst::rasterizer::clear(rst::Buffers buff)
{
    if ((buff & rst::Buffers::Color) == rst::Buffers::Color)
    {
        std::fill(frame_buf.begin(), frame_buf.end(), Eigen::Vector3f{0, 0, 0});
        std::fill(frame_sample.begin(), frame_sample.end(), Eigen::Vector3f{ 0,0,0 });
    }
    if ((buff & rst::Buffers::Depth) == rst::Buffers::Depth)
    {
        std::fill(depth_buf.begin(), depth_buf.end(), std::numeric_limits<float>::infinity());
        std::fill(depth_sample.begin(), depth_sample.end(), std::numeric_limits<float>::infinity());
    }
}
```

### 运行结果

<img src=./img/hw2_msaa_noEdge.png width=50%>

<img src=./img/hw2_msaa_noEdge_blueUp.png width=50%>


## 遗留问题

黑边，背景色混入问题

# 问题记录

## 问题：修改hw2代码完成后运行，全黑，看不到三角形

使用 WinMerge 与正确代码反复对比发现是在 main.cpp 中，
计算空间范围时出错

具体情况如下：

在错误的代码中，根据 `eye_fov` 和 `z` 轴坐标计算 `top` 位置时，使用了 `far`，即选取了远平面的 `z` 轴数据进行了计算，如下：

```cpp
float half_eye_fov = 0.5 * eye_fov / 180.0 * MY_PI;
float top = zFar * tan(half_eye_fov);
```

正确的写法应该是：

```cpp
float half_eye_fov = 0.5 * eye_fov / 180.0 * MY_PI;
float top = zNear * tan(half_eye_fov);
```

使用 `zNear` 与 `tan(half_eye_fov)` 进行计算 `top`，修改后图像正常出现

## 为什么不能使用 `zFar` 而是要使用 `zNear` 计算呢？

重温视图变换章节内容，当我们进行透视投影变换操作时，可以将透视投影分解为多个步骤，数学矩阵形式如下：

$$M_{persp} = M_{ortho_scale} \cdot M_{ortho_translate} \cdot M_{persp_to_ortho}$$

注意这里的变换步骤，在上方出错的代码位置，这里计算的内容是正交变换操作，此时应当是已经完成了变换矩阵 $M_{persp_to_ortho}$ 所代表的操作。

故，当运算进行到代码位置时，物体已经被压缩，只剩下通过正交投影将物体投影成标准小立方体，其前后平面的高和宽已经一致。

并且在该代码所在的函数提供的形参 `eye_fov` 就是指的从相机/原点出发的视线与近平面最上最下位置的夹角大小，所以这里应该是用近平面的 `zNear` 进行计算 `top`。

通过修改代码为 `float top = (zFar * TIMES + zNear) * tan(half_eye_fov);` 修改 `TIMES` 分别为：1，0.5，0.1，0.05，0。观察运行结果，发现随着 `TIMES` 的减小，绘制的两个三角形逐渐靠近。说明之前的错误代码的运行结果，是三角形的绘制距离过远导致的黑屏。

