---
title: 练习项目(六)：Back facing描边法
date: 2021-01-17 10:25:29
categories: 练习项目
tags: [Outline]
mathjax: true
---

# 概述

描边，在卡通渲染中是一个非常重要的主题。目前比较流行的描边方法有两种：一种是基于后处理的描边，这种方式相对不容易定制，适用于对复杂场景的描边；一种是过程式描边，通过两次绘制，一次绘制本体，一次绘制描边。<!--more-->本文主要介绍第二种描边方式，在《GUILTY GEAR Xrd》中称其为Back Facing法。

# 一、基本的实现描边

基本思路是通过两次绘制，一次绘制本体，一次绘制描边。

这里就有个问题，两次绘制的顺序怎么处理呢？

经过试验可以发现，两种顺序可以得到相同的结果。

本文使用的顺序是先绘制本体，再绘制描边。参考下图，在片元着色器之前，有个Depth Test操作，这样，在后绘制描边的时候可以通过深度检测过滤掉本体覆盖的像素，效率更高。

![](https://cdn.jsdelivr.net/gh/bzyzhang/ImgHosting//img/2021-01-17/20210117104333.png)

在URP中，如果没有设置LightMode，那么URP默认使用SRPDefaultUnlit。所以，可以将绘制本体Pass的LightMode设为SRPDefaultUnlit，而将绘制描边Pass的LightMode设为UniversalForward。这样，就可以实现先绘制本体，再绘制描边的功能了。主要的代码如下：

```
            Varyings vert(Attributes input)
            {
                Varyings output = (Varyings)0;
                
                UNITY_SETUP_INSTANCE_ID(input);
                UNITY_TRANSFER_INSTANCE_ID(input, output);
                UNITY_INITIALIZE_VERTEX_OUTPUT_STEREO(output);
                
                output.vertex = TransformObjectToHClip(input.positionOS.xyz);
                
                float3 normal = TransformObjectToWorldNormal(input.normalOS);
                float2 offset = TransformWorldToHClipDir(normal).xy;
                output.vertex.xy += offset * _Outline;
                
                return output;
            }
```

此时，可以得到如下的结果：

![](https://cdn.jsdelivr.net/gh/bzyzhang/ImgHosting//img/2021-01-17/20210117111142.png)

查看Frame Debugger，可以发现，确实是先绘制本体，再绘制描边。

![](https://cdn.jsdelivr.net/gh/bzyzhang/ImgHosting//img/2021-01-17/20210117111538.png)

# 二、到相机距离造成的描边粗细问题

上面的步骤，得到了一个基本的描边效果。但是当物体远离相机时，可以发现，描边会变细。我们希望得到的，是描边宽度不随物体距离相机远近而变化的效果。

![](https://cdn.jsdelivr.net/gh/bzyzhang/ImgHosting//img/2021-01-17/20210117111857.png)

这里就需要多提一个知识点。物体变换到投影空间后，x、y代表投影空间下的横纵坐标，z代表投影空间下的深度，w等于-z，w用于后面的齐次除法。我们希望得到的，是在屏幕上显示固定宽度的描边，那么顶点向外延伸的距离就应该是NDC空间下的固定距离，而不是投影空间下的固定距离。于是，在投影空间下计算向外延伸的距离的时候，乘上w的值，这样，在之后的齐次除法中会将坐标值除以w，得到的就是不会随距离相机远近不同的描边宽度了。代码如下：

```
                output.vertex = TransformObjectToHClip(input.positionOS.xyz);
                
                float3 normal = TransformObjectToWorldNormal(input.normalOS);
                float2 offset = TransformWorldToHClipDir(normal).xy;
                output.vertex.xy += offset * output.vertex.w * _Outline;
```

![](https://cdn.jsdelivr.net/gh/bzyzhang/ImgHosting//img/2021-01-17/20210117133827.png)

# 三、屏幕分辨率造成的非等比缩放问题

上面两个部分，得到的都是宽度一致的描边。但是，当试着对得到的offset进行归一化时，就会出现下面这种问题。代码如下：

```
                output.vertex = TransformObjectToHClip(input.positionOS.xyz);
                
                float3 normal = TransformObjectToWorldNormal(input.normalOS);
                float2 offset = normalize(TransformWorldToHClipDir(normal).xy);
                output.vertex.xy += offset * output.vertex.w * _Outline;
```

![](https://cdn.jsdelivr.net/gh/bzyzhang/ImgHosting//img/2021-01-17/20210117135246.png)

这是因为，观察空间变换到投影空间xy会非等比缩放，所以正常的法线在投影空间就是非归一化的。在投影空间下，output.vertex.xy未归一化，如果法线offset归一化了，最后计算得到的偏移值在xy方向上的拉伸程度就会不同。所以，这里不需要对offset归一化。

当然，也可以对offset归一化后，再根据屏幕的宽高比计算出一个系数，将offset.y乘以这个系数，得到一个新的法线，这样也可以解决上面的问题。

```
                output.vertex = TransformObjectToHClip(input.positionOS.xyz);
                
                float3 normal = TransformObjectToWorldNormal(input.normalOS);
                float2 offset = normalize(TransformWorldToHClipDir(normal).xy);

                //将近裁剪面右上角位置的顶点变换到观察空间
                float4 nearUpperRight = mul(unity_CameraInvProjection, float4(1, 1, UNITY_NEAR_CLIP_VALUE, _ProjectionParams.y));
                //求得屏幕宽高比
                float aspect = abs(nearUpperRight.x / nearUpperRight.y);
                offset.y *= aspect;

                output.vertex.xy += offset * output.vertex.w * _Outline;
```

![](https://cdn.jsdelivr.net/gh/bzyzhang/ImgHosting//img/2021-01-17/20210117142510.png)

当然，这种方式相对上一种方式有点麻烦，只是提供一种思路。

[完整代码](https://github.com/bzyzhang/RoadOfShader/blob/main/Assets/1.5-Outline/Shader/1.5.5-Outline.shader)

# 四、其它的一些技巧

在《罪恶装备-Xrd》的分享中，也提到了在卡通渲染中，其他的一些提升描边质量的方法。比如，使用顶点色存储描边粗细、颜色等，可以更加精细地控制描边。最近在实际的工作中，也遇到一种提升描边效果的方式：根据顶点距离相机的距离，计算出一个参数，在代表描边宽度的渐变贴图中采样，这样，可以定制各种不同距离的描边宽度。

# 参考

- [1] [URP ShaderLab Pass tags](https://docs.unity3d.com/Packages/com.unity.render-pipelines.universal@10.2/manual/urp-shaders/urp-shaderlab-pass-tags.html)
- [2] [OpenGL Projection Matrix](http://www.songho.ca/opengl/gl_projectionmatrix.html)
- [3] [【02】卡通渲染基本光照模型的实现](https://zhuanlan.zhihu.com/p/95986273)
- [4] [【01】从零开始的卡通渲染-描边篇](https://zhuanlan.zhihu.com/p/109101851)