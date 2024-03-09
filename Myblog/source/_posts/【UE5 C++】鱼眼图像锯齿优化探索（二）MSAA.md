---
title: 【UE5 C++】鱼眼图像锯齿优化探索（二）MSAA
tags: [UE5, C++]
cover: /res/img/post/【UE5 C++】鱼眼图像锯齿优化探索（二）MSAA/cover.jpg
top_img: /res/img/post/【UE5 C++】鱼眼图像锯齿优化探索（二）MSAA/cover.jpg
date: 2024-03-09 21:55:00
---
上一篇讲到使用高斯模糊后处理的方式来一定程度上消除鱼眼图像的边缘锯齿，但是会导致一定的模糊和重影。

高斯模糊并不是主流抗锯齿方案，本文介绍一些主流的抗锯齿方案，以及如何在 UE5 中实现 **MSAA** 后处理材质来优化鱼眼图像边缘锯齿。

# 锯齿产生的原因

我们都知道，显示器屏幕是由一个一个的像素组成的，所以渲染的过程需要将屏幕中的像素进行填充。

图形渲染的流程，就是一个一个的三角形，通过顶点着色器->光栅化->像素着色器这样的顺序进行渲染。

如下图所示，是光栅化时一个三角形的上边界。如果三角形覆盖了像素的中心点，则该像素点被着色。如果三角形没有覆盖到该像素的中心点，该像素不会被着色。可以看出三角形在光栅化时，覆盖到的像素点是离散的、断断续续的，这样就形成了 **锯齿/aliasing**。

这样的斜线形成的锯齿，是阶梯状的，因此也被称为 **stair-stepping**。

<img src="/res/img/post/【UE5 C++】鱼眼图像锯齿优化探索（二）MSAA/1_边界锯齿的形成.jpg" width = 800px>

而实际情况下，我们期望得到的结果应该是如下图所示这样的。每个像素点，根据三角形对当前像素正方形覆盖的面积，按比例贡献颜色值。这就是我们的抗锯齿技术想要达到的目标。

<img src="/res/img/post/【UE5 C++】鱼眼图像锯齿优化探索（二）MSAA/2_理想的抗锯齿边界.jpg" width = 800px>

# 实现抗锯齿的两种思路

知道了锯齿产生的原因和期望的抗锯齿的目标，现在我们有两种思路去实现抗锯齿效果。

第一种思路自然就是在每个像素中进行多次采样，然后根据多次采样的结果综合来计算像素的颜色值。使用这种方式来实现的抗锯齿技术有 **MSAA，TAA**。

第二种思路是通过后处理的方式，寻找屏幕中的像素块边界，然后根据边界的信息，将两侧的像素点颜色进行插值，这样就会得到平滑过渡的边缘，实现抗锯齿的效果。使用这种方式来实现的抗锯齿技术有 **FXAA，SMAA**。

# UE5 实现 MSAA 后处理材质

## 魔改 shadertoy MSAA 代码

这里我魔改的样例是：[shadertoy MSAA](https://www.shadertoy.com/view/3tdGD7)

<img src="/res/img/post/【UE5 C++】鱼眼图像锯齿优化探索（二）MSAA/3_shaertoy_msaa.png" width = 800px>

魔改代码：

```glsl
float checkerboard(vec2 uv) {
    vec2 adjusted = floor(uv);
 	bool inside = mod(adjusted.y, 2.0) == 1.0
		? mod(adjusted.x, 2.0) == 0.0
        : mod(adjusted.x, 2.0) == 1.0;
    return inside ? 1.0 : 0.0;
}

struct HitResult { bool hit; vec3 point; };
struct Ray { vec3 origin; vec3 direction; };
struct Plane { vec3 origin; vec3 normal; };

HitResult rayPlaneIntersect(Ray ray, Plane plane) {
 	float horizonDot = dot(ray.direction, plane.normal);
    if (horizonDot < 0.00001) {
		vec3 difference = plane.origin - ray.origin;
        float time = dot(difference, plane.normal) / horizonDot;
        
        bool hit = time > 0.0;
        vec3 point = hit
            ? ray.origin + ray.direction * time
            : vec3(0.0);
        
        return HitResult(hit, point);
    }
    
    return HitResult(false, vec3(0.0));
}

vec3 scene(vec2 uv) {
    const vec3 up = vec3(0.0, 1.0, 0.0);
    
    vec3 origin = vec3(0.0, 2.0, 0.0);
    vec3 rayDirection = normalize(vec3(uv.x, uv.y, 0.9));
    
    Ray ray = Ray(origin, rayDirection);
    Plane plane = Plane(vec3(0.0), up);  
    
    HitResult result = rayPlaneIntersect(ray, plane);

    vec3 color = result.hit
        ? vec3(checkerboard(result.point.xz))
        : vec3(rayDirection.x, rayDirection.y, rayDirection.z);

   	return color;
}

vec2 coordToUv(vec2 coord) {
 	return (coord - iResolution.xy * 0.5) / iResolution.y;
}

void mainImage( out vec4 fragColor, in vec2 fragCoord ) {
    float a = (3.0 / 8.0);
    float b = (1.0 / 8.0);
    vec3 acc = vec3(0.0);
    acc += scene(coordToUv(fragCoord + vec2(-a, b)));
    acc += scene(coordToUv(fragCoord + vec2(-b, -a)));
    acc += scene(coordToUv(fragCoord + vec2(a, -b)));
    acc += scene(coordToUv(fragCoord + vec2(b, a)));
    acc /= 4.0;

    vec3 color = acc;
    
    //color = pow(color, vec3(1.0 / 2.2));
    
    fragColor = vec4(color, 1.0);
}
```

这里魔改代码是因为 **shadertoy** 上的作者写的代码会有很多专门用于 **shadertoy** 的，对于移植到 **UE5** 上没有任何作用（ 对于我们移植代码来说相当于是废代码，可以删除 ）

将该代码粘贴至 Image 下编译运行：

<img src="/res/img/post/【UE5 C++】鱼眼图像锯齿优化探索（二）MSAA/4_shaertoy_msaa_change.png" width = 800px>

## 将魔改的代码移植到 UE5

1. 新建材质函数 `MF_Scene` 用于更新场景，这里根据 Uv 来改变场景像素位置

<img src="/res/img/post/【UE5 C++】鱼眼图像锯齿优化探索（二）MSAA/5.png" width = 800px>

2. 新建后处理材质 `PP_MSAA`

<img src="/res/img/post/【UE5 C++】鱼眼图像锯齿优化探索（二）MSAA/6.png" width = 800px>

> 这里的名称例如：aa_picsize（这是图片大小），是通过脱出节点然后输入 **name** 设置的类似变量的节点，方便后续使用，个人习惯，这样更容易管理

<img src="/res/img/post/【UE5 C++】鱼眼图像锯齿优化探索（二）MSAA/7.png" width = 800px>

<img src="/res/img/post/【UE5 C++】鱼眼图像锯齿优化探索（二）MSAA/8.png" width = 800px>

<img src="/res/img/post/【UE5 C++】鱼眼图像锯齿优化探索（二）MSAA/9.png" width = 800px>

3. 最后将 **acc** 输出

<img src="/res/img/post/【UE5 C++】鱼眼图像锯齿优化探索（二）MSAA/10.png" width = 800px>

# 最终效果

测试场景：Living_Dining_Room-1

## 默认场景光照下（静止）

1. 项目抗锯齿 TAA + 鱼眼畸变：

<img src="/res/img/post/【UE5 C++】鱼眼图像锯齿优化探索（二）MSAA/11_1_before.png" width = 800px>

<img src="/res/img/post/【UE5 C++】鱼眼图像锯齿优化探索（二）MSAA/11_2_before.png" width = 800px>

2. 项目抗锯齿 TAA + 鱼眼畸变 + MSAA：

<img src="/res/img/post/【UE5 C++】鱼眼图像锯齿优化探索（二）MSAA/11_3_after.png" width = 800px>

<img src="/res/img/post/【UE5 C++】鱼眼图像锯齿优化探索（二）MSAA/11_4_after.png" width = 800px>

3. 再对比一下使用高斯模糊的效果：

<img src="/res/img/post/【UE5 C++】鱼眼图像锯齿优化探索（二）MSAA/11_5_after.png" width = 800px>

<img src="/res/img/post/【UE5 C++】鱼眼图像锯齿优化探索（二）MSAA/11_6_after.png" width = 800px>

明显是 MSAA 更胜一筹！

## 降低场景亮度（静止）

1. 项目抗锯齿 TAA + 鱼眼畸变：

<img src="/res/img/post/【UE5 C++】鱼眼图像锯齿优化探索（二）MSAA/12_1_before.png" width = 800px>

<img src="/res/img/post/【UE5 C++】鱼眼图像锯齿优化探索（二）MSAA/12_2_before.png" width = 800px>

2. 项目抗锯齿 TAA + 鱼眼畸变 + MSAA：

<img src="/res/img/post/【UE5 C++】鱼眼图像锯齿优化探索（二）MSAA/12_3_after.png" width = 800px>

<img src="/res/img/post/【UE5 C++】鱼眼图像锯齿优化探索（二）MSAA/12_4_after.png" width = 800px>

## 默认场景光照下（运动-已关闭运动模糊）

1. 项目抗锯齿 TAA + 鱼眼畸变：

<img src="/res/img/post/【UE5 C++】鱼眼图像锯齿优化探索（二）MSAA/13_before.gif" width = 800px>

2. 项目抗锯齿 TAA + 鱼眼畸变 + MSAA：

<img src="/res/img/post/【UE5 C++】鱼眼图像锯齿优化探索（二）MSAA/13_after.gif" width = 800px>

## 降低场景亮度（运动-已关闭运动模糊）

1. 项目抗锯齿 TAA + 鱼眼畸变：

<img src="/res/img/post/【UE5 C++】鱼眼图像锯齿优化探索（二）MSAA/14_before.gif" width = 800px>

2. 项目抗锯齿 TAA + 鱼眼畸变 + MSAA：

<img src="/res/img/post/【UE5 C++】鱼眼图像锯齿优化探索（二）MSAA/14_after.gif" width = 800px>

# 参考

主流抗锯齿方案详解（一）：https://zhuanlan.zhihu.com/p/415087003

抗锯齿相关技术介绍：https://www.cnblogs.com/somedayLi/p/12445939.html

图形学基础-抗锯齿：https://www.liaomz.top/2022/02/27/tu-xing-xue-ji-chu-kang-ju-chi/

MSAA-GLSL 代码参考：https://www.shadertoy.com/view/3tdGD7

The End.