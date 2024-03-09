---
title: 【UE5 C++】鱼眼图像锯齿优化探索（一）高斯模糊
tags: [UE5, C++]
cover: /res/img/post/【UE5 C++】鱼眼图像锯齿优化探索（一）高斯模糊/cover.jpg
top_img: /res/img/post/【UE5 C++】鱼眼图像锯齿优化探索（一）高斯模糊/cover.jpg
date: 2024-03-09 21:54:00
---
# 鱼眼锯齿产生原因

虽然在引擎的项目设置中已经开启了抗锯齿的选项，但由于鱼眼畸变的影响，又使图像产生了边缘锯齿。

因为需要实时处理图像，所以打算在当前已有的鱼眼后处理效果上再加一层抗锯齿或者模糊。

下面介绍如何在 UE5 中使用 **高斯模糊后处理** 来一定程度的消除锯齿。

# 材质自定义表达式中引用 USF

## Generate Visual Studio project files

<img src="/res/img/post/【UE5 C++】鱼眼图像锯齿优化探索（一）高斯模糊/2_1.png" width = 800px>

## 添加（自定义模块的）文件映射路径

<img src="/res/img/post/【UE5 C++】鱼眼图像锯齿优化探索（一）高斯模糊/2_2.png" width = 800px>

<img src="/res/img/post/【UE5 C++】鱼眼图像锯齿优化探索（一）高斯模糊/2_3.png" width = 800px>

<img src="/res/img/post/【UE5 C++】鱼眼图像锯齿优化探索（一）高斯模糊/2_4.png" width = 800px>

## 重新生成项目

<font color=red>为了防止文件冲突，删除掉 Binaries 和 Intermediate 文件夹之后，再次点击 Uproject 右键 Generate Visual Studio project files

重新生成之后，进入解决方案 Build 成功之后，在材质表达式中引用 USF 文件了</font>

<img src="/res/img/post/【UE5 C++】鱼眼图像锯齿优化探索（一）高斯模糊/2_5.png" width = 800px>

# 创建 Gaussian Blur 后处理材质

## 确定好高斯模糊公式

曲线接受大约从 -1 到 1 的输入。然后，它将输出一个从 0 到 1 的值。

[一维简化高斯函数](https://www.desmos.com/calculator/2rpzsgh5ol?lang=zh-CN)：

<img src="/res/img/post/【UE5 C++】鱼眼图像锯齿优化探索（一）高斯模糊/3_1_GaussianFunc.png" width = 300px>

## 完整代码

```hlsl
struct FunctionStruct
{
    //计算一维高斯模糊
    float Cal_1DGaussian(float x)
    {
        return exp(-0.5f * pow(3.141 * x, 2));
    }
};

FunctionStruct FS;

//需要获得的场景贴图index
static const int SceneTextureID = 14;
//纹素大小，比如一张512 X 512大小的纹理，那么纹素大小为（1/512）
//用于UV的偏移
float2 TexelSize = View.ViewSizeAndInvSize.zw;
//获取当前像素的UV
float2 UV = GetDefaultSceneTextureUV(Parameters, SceneTextureID);
//用于存储累积的颜色
float3 PixelSum = float3(0, 0, 0);
//累积权重值
float WeightSum = 0;

//水平与垂直模糊
for (int x = -BlurRadius;x<=BlurRadius;x++)
{
    for (int y = -BlurRadius;y<=BlurRadius;y++)
    {
        //计算偏移的UV
        float2 offsetUV = UV + float2(x,y)*TexelSize;
        //采样偏移后的贴图颜色
        float3 PixelColor = SceneTextureLookup(offsetUV,SceneTextureID,false).rgb;
        //计算采样像素的权重，/Raduis的原因是为了限制输入范围为-1到1
        float weight = FS.Cal_1DGaussian(x / BlurRadius) * FS.Cal_1DGaussian(y / BlurRadius);
        //累加颜色
        PixelSum += PixelColor*weight;
        //累加权重值
        WeightSum += weight;
    }
}

//返回加权平均值
return PixelSum / WeightSum;
```

在项目目录创建 `Shaders` 文件夹，并在 `Shaders` 文件夹下新建 `Gaussian.usf` 文件

将代码写入该文件，供后续使用

材质节点参考：

<img src="/res/img/post/【UE5 C++】鱼眼图像锯齿优化探索（一）高斯模糊/3_2_GaussianBlur.png" width = 800px>

# 添加后处理材质

<img src="/res/img/post/【UE5 C++】鱼眼图像锯齿优化探索（一）高斯模糊/4_1_AddPostProcessMaterial.png" width = 800px>

# 最终效果

1. 项目抗锯齿 TAA + 鱼眼畸变：

<img src="/res/img/post/【UE5 C++】鱼眼图像锯齿优化探索（一）高斯模糊/5_1_before.png" width = 800px>

<img src="/res/img/post/【UE5 C++】鱼眼图像锯齿优化探索（一）高斯模糊/5_2_before.png" width = 800px>

2. 项目抗锯齿 TAA + 鱼眼畸变 + 高斯模糊：

> **【BlurRadius参数 = 1.5】**（ 默认为 1.5 ）

<img src="/res/img/post/【UE5 C++】鱼眼图像锯齿优化探索（一）高斯模糊/5_3_after.png" width = 800px>

<img src="/res/img/post/【UE5 C++】鱼眼图像锯齿优化探索（一）高斯模糊/5_4_after.png" width = 800px>

# 参考

自定义着色器教程：https://www.kodeco.com/57-unreal-engine-4-custom-shaders-tutorial

Gaussian Smoothing：https://homepages.inf.ed.ac.uk/rbf/HIPR2/gsmooth.htm

UE4[C++]添加自定义模块：https://zhuanlan.zhihu.com/p/101179587

The End.