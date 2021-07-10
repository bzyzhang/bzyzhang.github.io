---
title: 【躬行】-Assembly definitions的相关实验
date: 2021-07-10 10:01:38
categories: 躬行
tags: [Untiy]
mathjax: true
---

# 概述

大概从Unity 2017.3开始，添加了assembly definition相关的功能。为了更深入的了解，进行了多次打包和对比。本文主要是对这一过程进行记录。<!--more-->

# 文档解释

## 1、编译脚本

Untiy在默认情况下，根据脚本在项目中的文件夹，会分成四个阶段编译脚本。

当脚本引用在其它阶段(即位于不同程序集中)编译的类时，编译顺序非常重要。基本规则是，在当前编译阶段之后的任何编译阶段都不能被引用。在当前阶段或更早阶段编译的任何内容都是完全可用的。

编译的各个阶段如下：

| 阶段 | 程序集名                         | 脚本文件                                                     |
| :--- | :------------------------------- | :----------------------------------------------------------- |
| 1    | Assembly-CSharp-firstpass        | Standard Assets, Pro Standard Assets和Plugins文件夹下面的运行时脚本 |
| 2    | Assembly-CSharp-Editor-firstpass | Standard Assets, Pro Standard Assets和Plugins文件夹下面的Editor文件夹下面的Editor脚本 |
| 3    | Assembly-CSharp                  | 其它不在Editor文件夹下面的脚本                               |
| 4    | Assembly-CSharp-Editor           | 所有剩下的脚本（Editor文件夹下面的脚本）                     |

## 2、Assembly definitions

程序集是一个C#代码库，它包含由脚本定义的已编译类和结构，还定义了对其他程序集的引用。

默认情况下，Unity将几乎所有的游戏脚本编译到预定义的程序集中（Assembly-CSharp.dll）。

这种安排对于小型项目来说是可以接受的，但是当你向项目中添加更多代码时，会有一些缺点：

- 每当你改变一个脚本时，Unity就必须重新编译所有其他脚本，这增加了迭代代码更改的整体编译时间。
- 任何脚本都可以直接访问任何其他脚本中定义的类型，这使得重构和改进代码变得更加困难。
- 所有脚本都是为所有平台编译的。

通过定义程序集，你可以组织代码以促进模块化和可重用性。你为项目定义的程序集中的脚本将不再添加到默认程序集中，并且只能访问你指定的其他程序集中的脚本。

![](https://docs.unity3d.com/uploads/Main/ScriptCompilation.png)

上面的图表说明了如何将项目中的代码拆分为多个程序集。因为Main引用Stuff而不是相反，你知道任何对Main中的代码的更改都不会影响Stuff中的代码。类似地，因为Library不依赖于任何其他程序集，所以可以更容易地在另一个项目中重用Library中的代码。

默认情况下，预定义程序集引用所有其他程序集，包括使用Assembly Definition(1)创建的程序集和作为plugin添加到项目中的预编译程序集(2)。此外，使用Assembly Definition创建的程序集自动引用所有预编译程序集(3)：

![](https://docs.unity3d.com/uploads/Main/AssemblyDependencies.png)

要将项目代码组织成程序集，请为每个所需程序集创建一个文件夹，并将应该属于每个程序集的脚本移动到相关文件夹中。然后创建Assembly Definition资产以指定程序集属性。

Unity在一个包含Assembly Definition资产的文件夹中获取所有脚本，并使用该资产定义的名称和其他设置将它们编译为一个程序集。Unity还将任意子文件夹中的脚本包含到同一程序集中，除非子文件夹有自己的Assembly Definition或Assembly Definition Reference资产。

要在现有程序集中包含来自非子文件夹的脚本，请在非子文件夹中创建Assembly Definition Reference资产，并将其设置为引用定义目标程序集的Assembly Definition资产。例如，你可以将项目中所有Editor文件夹中的脚本组合到它们自己的程序集中，而不管这些文件夹位于何处。

Assembly Definition Reference在下面这种情况下可以解决问题：在Unity的Packages中，有一些访问级别为internal的类。如果我们在Assets下面创建脚本，是不能访问Packages中的internal类。有了Assembly Definition Reference就可以了。可以在创建的脚本的同级目录中创建一个Assembly Definition Reference，设置引用包含internal类的Package即可。

# 一些实验

Unity版本：2020.3.13f1

ILSpy版本：7.1.0.6543

创建一个空的URP工程，设置到Android平台，Player Setting中的Scripting Backend设置为“Mono”。默认情况下，项目中包含的脚本如下所示：

此处有图

每次打完包后，对apk进行解压即可。解压后，程序集的位置在“{解压的文件}\assets\bin\Data\Managed\”下面：

此处有图

## 1、默认打包

我们需要关心的是“Assembly-CSharp.dll”程序集。使用ILSpy打开Assembly-CSharp.dll，可以看到：

此处有图

项目中的两个运行时脚本Readme和SimpleCameraController都被编译到程序集Assembly-CSharp.dll中。

## 2、使用Assembly Definition

在SimpleCameraController的同级目标创建Assembly Definition，Assembly Definition的设置保持默认即可。观察SimpleCameraController，可以发现SimpleCameraController的程序集信息是新创建的Assembly Definition：

此处有图。

打包，解压，可以看到，刚才新创建的Assembly Definition产生的程序集：

此处有图。

使用ILSpy分别打开Assembly-CSharp.dll和NewAssembly.dll，结果如下所示：

此处有图

此处有图

可以发现，SimpleCameraController被编译进新的程序集NewAssembly.dll中，而原本的程序集Assembly-CSharp.dll中没有SimpleCameraController了。

### 3、使用Assembly Definition Reference

删除上面创建的Assembly Definition，在SimpleCameraController的同级目标创建Assembly Definition Reference。假设，我们想访问的Unity的Package是Unity UI，设置Assembly Definition Reference上的Assembly Definition为UnityEngine.UI即可，如下图所示：

此处有图。

打包，解压，可以看到，上面新创建的程序集NewAssembly.dll没了，而使用ILSpy打开Assembly-CSharp.dll，也并没有看到SimpleCameraController。那么，SimpleCameraController去哪儿了呢？使用ILSpy打开UnityEngine.UI.dll程序集：

此处有图。

可以发现，SimpleCameraController被包含到了UnityEngine.UI.dll程序集中，也就是说，SimpleCameraController可以访问程序集中的internal级别的类了。

# 一些使用场景

1. 对于Assembly Definition的用处，网上很多的介绍是可以加快代码编译速度。这个也很容易理解，使用Assembly Definition对项目中的脚本进行细分后，每次修改只需要编译脚本所在的程序集，因此减少了编译的时间。
2. 如果想访问Untiy的Package中的internal类的话，由于可以获得Package的源码，当然可以直接在Package中添加修改。当现在如果不修改Package的话，就可以使用Assembly Definition Reference，将自己的代码包含到Package中，这样，就会编译到同一个程序集中，也就可以访问internal类了。

# 参考

- [1] [Unity 2017.3](https://unity3d.com/unity/whats-new/unity-2017.3.0)
- [2] [Special folders and script compilation order](https://docs.unity3d.com/Manual/ScriptCompileOrderFolders.html)
- [3] [Assembly definitions](https://docs.unity3d.com/Manual/ScriptCompilationAssemblyDefinitionFiles.html)
- [4] [Unity3D研究院新方法加快代码编译速度（九十六）](https://www.xuanyusong.com/archives/4474)
- [5] [提升Unity编辑器中代码的编译速度](https://answer.uwa4d.com/question/58d2829a9ad5c0094f461e30)