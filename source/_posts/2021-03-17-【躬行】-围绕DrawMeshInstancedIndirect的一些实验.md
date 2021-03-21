---
title: 【躬行】-围绕DrawMeshInstancedIndirect的一些实验
date: 2021-03-17 21:29:18
categories: 躬行
tags: [图形学]
mathjax: true
---

# 概述

项目中使用了Graphics.DrawMeshInstancedIndirect()方法来实现草地的渲染。最近，需要测试在真机上的兼容性。<!--more-->在测试的过程中，遇到了一些问题，也打了包，在真机上进行了各种实验。这里记录一下。

# 文档解释

在官方文档的[GPU instancing](https://docs.unity3d.com/Manual/GPUInstancing.html)中提到：

> You can also use the calls Graphics.DrawMeshInstanced and Graphics.DrawMeshInstancedIndirect to perform GPU Instancing from your scripts.

所以，Graphics.DrawMeshInstancedIndirect()对平台和图形API的要求，与GPU instancing的要求是一致的。在上面的文档中，提到了GPU instancing对平台和图形API的要求：

> GPU Instancing is available on the following platforms and APIs:
>
> - **DirectX 11** and **DirectX 12** on Windows
>
> - **OpenGL Core 4.1+/ES3.0+** on Windows, macOS, Linux, and Android
> - **Metal** on macOS and iOS
> - **Vulkan** on Windows, Linux and Android
> - **PlayStation 4** and **Xbox One**
> - **WebGL** (requires WebGL 2.0 API)

先考虑Android平台的情况，从上面可以看出，需要支持OpenGL ES 3.0+或者是Vulkan。

但是，在另一份[文档](https://unity3d.com/unity/beta/unity5.5.0b6)中提到，由于驱动问题，对于仅具有OpenGL ES 3.0的Adreno GPU的设备，禁用了GPU instancing：

> Graphics: GPU Instancing: Added support for Android with OpenGL ES 3.0 or newer. Note however that GPU instancing support is disabled for Android devices that have the Adreno GPU with only OpenGL ES 3.0, because of driver issues.

暂且先认为，OpenGL ES 3.1之后开始完全支持GPU instancing。

那么，在哪里配置Unity对图像API的支持呢？答案是Player Settings->Other Settings下面的Graphics API部分。

首先，看一下官方手册中的解释：

> Disable this option to manually pick and reorder the graphics APIs. (OpenGL). By default this option is enabled, and Unity tries GLES3.2. If the device doesn’t support GLES3.2, Unity falls back to GLES3.1, GLES3 or GLES2. If only GLES3 is in the list, additional checkboxes appear: **Require ES3.1**, **Require ES3.1+AEP** and **Require ES3.2**. These allow you to force the corresponding graphics API.
> **Important**: Unity adds the GLES3/GLES3.1/AEP/3.2 requirement to your Android manifest only if GLES2 is not in the list and the Minimum API Level is set to JellyBean (API level 18) or higher. In this case only, your application does not appear on unsupported devices in the Google Play Store.

我的理解是，当选择了Auto Graphics API时，Unity会按照GLES3.2->GLES3.1->GLES3->GLES2的顺序，设置设备支持的图像API；当没有选择Auto Graphics API时，可以手动控制顺序，Unity会按照列表，从上至下的顺序，设置设备支持的图像API。

下面，会进行几个实验，来测试Graphics API的配置在真机上的表现，顺便测试了Graphics.DrawMeshInstancedIndirect()方法在真机上的兼容性。

# 真机测试

测试手机：华为 Nova 3、魅蓝 E3

Unity版本：2019.4.22f1c1

代码：参考[Graphics.DrawMeshInstancedIndirect](https://docs.unity3d.com/ScriptReference/Graphics.DrawMeshInstancedIndirect.html)

下面在真机上的图片，会按照华为 Nova 3在上，魅蓝 E3在下的顺序排序，不再赘述。

## 1、实验一

![](https://cdn.jsdelivr.net/gh/bzyzhang/ImgHosting1//img/2021-3-19/20210319122642.png)

![](https://cdn.jsdelivr.net/gh/bzyzhang/ImgHosting1//img/2021-3-19/20210321103207.jpg)

![](https://cdn.jsdelivr.net/gh/bzyzhang/ImgHosting1//img/2021-3-19/20210321103227.jpg)

## 2、实验二

![](https://cdn.jsdelivr.net/gh/bzyzhang/ImgHosting1//img/2021-3-19/20210319123142.png)

![](https://cdn.jsdelivr.net/gh/bzyzhang/ImgHosting1//img/2021-3-19/20210321103300.jpg)

![](https://cdn.jsdelivr.net/gh/bzyzhang/ImgHosting1//img/2021-3-19/20210321103320.jpg)

## 3、实验三

![](https://cdn.jsdelivr.net/gh/bzyzhang/ImgHosting1//img/2021-3-19/20210319123242.png)

![](https://cdn.jsdelivr.net/gh/bzyzhang/ImgHosting1//img/2021-3-19/20210321103344.jpg)

![](https://cdn.jsdelivr.net/gh/bzyzhang/ImgHosting1//img/2021-3-19/20210321103406.jpg)

## 4、实验四

![](https://cdn.jsdelivr.net/gh/bzyzhang/ImgHosting1//img/2021-3-19/20210319123340.png)

![](https://cdn.jsdelivr.net/gh/bzyzhang/ImgHosting1//img/2021-3-19/20210321103430.jpg)

![](https://cdn.jsdelivr.net/gh/bzyzhang/ImgHosting1//img/2021-3-19/20210321103448.jpg)

# 实验结论

根据实验的结果，同时参考官方手册中的内容，可以得出如下结论：

1. 根据实验一的结果，当启用了Auto Graphics API的时候，Unity会尝试支持GLES3.2。但是对比两台手机的表现，可以发现，华为 Nova 3上面的显示是有问题的。目前没有找到出现问题的原因。
2. 根据实验二、三和四的结果，当没有启用Auto Graphics API的时候，可以手动选择、排序Graphics APIs。同时，可以发现，硬件设备支持多个图形API。根据实验结果，可以得出：Unity会根据Graphics APIs在列表中的排序，从上至下，检测硬件设备是否支持相应的图形API。Vulkan支持Graphics.DrawMeshInstancedIndirect()，GLES 2.0不支持Graphics.DrawMeshInstancedIndirect()。而GLES 3可能需要某个版本之后才完全支持Graphics.DrawMeshInstancedIndirect()。

# 项目中的做法

目前的做法是，选择手动排序Graphics APIs，按照顺序添加了Vulkan、OpenGLES3。但根据官方的统计[数据](https://developer.android.com/about/dashboards)，支持Vulkan的的设备只有53%左右。后面可能需要进行更多的验证，有了新的结果也会更新本文。

# 参考

- [1] [Android Player settings](https://docs.unity3d.com/Manual/class-PlayerSettingsAndroid.html#Other)
- [2] [GPU instancing](https://docs.unity3d.com/Manual/GPUInstancing.html)
- [3] [5.5.0 Beta 6](https://unity3d.com/unity/beta/unity5.5.0b6)
- [4] [GPU Instancing手机兼容性报告](https://zhuanlan.zhihu.com/p/72717290)
- [5] [Graphics.DrawMeshInstancedIndirect](https://docs.unity3d.com/ScriptReference/Graphics.DrawMeshInstancedIndirect.html)
- [6] [分发信息中心](https://developer.android.com/about/dashboards)