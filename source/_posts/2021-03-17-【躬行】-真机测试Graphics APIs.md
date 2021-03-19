---
title: 【躬行】-真机测试Graphics APIs
date: 2021-03-17 21:29:18
categories: 躬行
tags: [GPU]
mathjax: true
---

# 概述

在Unity的Player Settings的Other Settings部分，有个Graphics APIs设置部分。之前一直没有深入的去研究。<!--more-->最近在学习Graphics.DrawMeshInstancedIndirect()接口的过程中，牵涉到了Graphics APIs的设置。下面，会通过一系列的实验，来探究Graphics APIs的作用。

# 文档解释

首先，看一下官方手册中的解释：

> Disable this option to manually pick and reorder the graphics APIs. (OpenGL). By default this option is enabled, and Unity tries GLES3.2. If the device doesn’t support GLES3.2, Unity falls back to GLES3.1, GLES3 or GLES2. If only GLES3 is in the list, additional checkboxes appear: **Require ES3.1**, **Require ES3.1+AEP** and **Require ES3.2**. These allow you to force the corresponding graphics API.
> **Important**: Unity adds the GLES3/GLES3.1/AEP/3.2 requirement to your Android manifest only if GLES2 is not in the list and the Minimum API Level is set to JellyBean (API level 18) or higher. In this case only, your application does not appear on unsupported devices in the Google Play Store.

# 真机测试

测试手机：华为 Nova 3

Unity版本：2019.4.22f1c1

## 1、实验一

![](https://cdn.jsdelivr.net/gh/bzyzhang/ImgHosting1//img/2021-3-19/20210319122642.png)

![](https://cdn.jsdelivr.net/gh/bzyzhang/ImgHosting1//img/2021-3-19/20210319123106.png)

## 2、实验二

![](https://cdn.jsdelivr.net/gh/bzyzhang/ImgHosting1//img/2021-3-19/20210319123142.png)

![](https://cdn.jsdelivr.net/gh/bzyzhang/ImgHosting1//img/2021-3-19/20210319123212.png)

## 3、实验三

![](https://cdn.jsdelivr.net/gh/bzyzhang/ImgHosting1//img/2021-3-19/20210319123242.png)

![](https://cdn.jsdelivr.net/gh/bzyzhang/ImgHosting1//img/2021-3-19/20210319123307.png)

## 4、实验四

![](https://cdn.jsdelivr.net/gh/bzyzhang/ImgHosting1//img/2021-3-19/20210319123340.png)

![](https://cdn.jsdelivr.net/gh/bzyzhang/ImgHosting1//img/2021-3-19/20210319123401.png)

## 5、实验五

![](https://cdn.jsdelivr.net/gh/bzyzhang/ImgHosting1//img/2021-3-19/20210319123426.png)

![](https://cdn.jsdelivr.net/gh/bzyzhang/ImgHosting1//img/2021-3-19/20210319123448.png)

# 实验结论

根据实验的结果，同时参考官方手册中的内容，可以得出如下结论：

1. 根据实验一的结果，当启用了Auto Graphics API的时候，Unity会尝试支持GLES3.2。
2. 根据实验二、三和四的结果，当没有启用Auto Graphics API的时候，可以手动选择、排序Graphics APIs。同时，可以发现，硬件设备支持多个图形API。根据实验结果，可以得出：Unity会根据Graphics APIs在列表中的排序，从上之下，检测硬件设备是否支持相应的图形API。
3. 根据实验五，发现与官方手册中的说法不太一致。实验发现，只要列表中没有GLES2的话，就会出现额外的选项。

# 参考

- [1] [Android Player settings](https://docs.unity3d.com/Manual/class-PlayerSettingsAndroid.html#Other)