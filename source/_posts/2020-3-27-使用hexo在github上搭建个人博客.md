---
title: 使用hexo在github上搭建个人博客
date: 2020-03-27 15:09:03
tags: [hexo,github,博客]
categories: 工具
---

# 一、 软件的安装

## 1. 在Github创建个人仓库

在Github上Create a new repository，需要注意的是，Repository name应该为：`xxx.github.io`，其中`xxx`是你的用户名。<!--more-->

## 2.安装Git

这里推荐廖雪峰的官方网站的[Git教程](https://www.liaoxuefeng.com/wiki/896043488029600/896067074338496)

这里要特别说一下，按照教程操作完成后，还要配置一下ssh：

```bash
ssh-keygen -t rsa -C "你的GitHub注册邮箱"
```



然后按照提示回车即可，默认不需要设置密码。

然后找到生成的.ssh文件夹（我的在C:\Users\xxxx\\.ssh）下的id_rsa.pub密钥，将内容全部复制。

打开[Github_settings_keys](https://github.com/settings/keys)，新建New SSH keys。Titile随便取，Key输入上面复制的id_rsa.pub密钥即可。

完成后，在Git Bash中检测Github公钥是否设置成功。

```bash
ssh git@github.com
```

根据提示，判断设置是否成功。

## 3.安装Node.js

Node.js的[下载地址](https://nodejs.org/en/download/)。安装完成后，在CMD命令行输入以下命令检测是否安装成功：

```base
node -v
npm -v
```

## 4.安装Hexo

Hexo就是我们的个人博客网站的框架。首先先建一个文件夹，可以，命名为Hexo，Hexo框架和你以后发布的网页都在这个文件夹。创建好文件后，进入Hexo文件夹，在这里右键，点击Git Bash Here选项。

首先安装Hexo，输入：

```bash
npm install -g hexo-cli 
```

估计安装时间较长，请耐心等待。安装完成后，初始化博客：

```bash
hexo init blog
```

以上两个命令，都是在Hexo文件夹操作的。

现在Hexo文件夹下应该有一个blog文件夹，是上面初始化博客的时候生成的。关闭Git Bash，进入blog文件夹，右键打开Git Bash。现在检测我们的博客雏形了。分别输入以下三条命令：

```bash
hexo new "第一个博客"
hexo g
hexo s
```

没有错误的话，现在在浏览器中输入网址：

```
localhost:4000
```

顺利的话，会看到第一个博客。

## 5.推送网站

上面的操作，只是在本地预览，下面要做的，就是将博客推送到Github远端，从而可以通过互联网访问我们的博客。在进行下一步之前，有一个概念要说一下。

在blog根目录下面，有一个**_config.yml**文件，被称为站点配置文件。

进入根目录的themes文件夹，里面也有个**_config.yml**文件，被称为**主题**配置文件。

下面，我们需要将Hexo与Github关联起来。打开站点配置文件_config.yml，搜索到下面的对应部分进行修改：

```
deploy:
  type: git
  repo: 这里填入你之前在GitHub上创建仓库的完整路径
  branch: master
```

完成之后，保存，在blog文件下加执行以下命令：

```bash
npm install hexo-deployer-git --save
```

然后，分别输入以下命令：

```bash
hexo clean
hexo g
hexo d
```

完成之后，打开浏览器，输入类型`xxx.github.io`的地址，没有问题的话，可以打开博客了。

# 二、更换主题

可以在blog目录下的themes文件夹里面查看自己的主题是什么。[这里](https://hexo.io/themes/)是Hexo的主题合集，一般在主题的Github主页中都会有安装方法的。下面介绍一下更换NexT的主题。在blog目录中执行以下命令：

```bash
git clone https://github.com/theme-next/hexo-theme-next themes/next
```

下载完成后，themes文件夹下应该有next文件夹。打开站点配置文件_config_yml，修改主题为NexT：

```
# Extensions
## Plugins: https://hexo.io/plugins/
## Themes: https://hexo.io/themes/
theme: next
```

然后，打开next的主题配置文件_config.yml。这里可以选择显示方案：

```
# Schemes
#scheme: Muse
#scheme: Mist
#scheme: Pisces
scheme: Gemini
```

只需要取消#，即可选择相应的主题（在其他的方案前面要加#）。

选择好相应的主题之后，可以查找相关资料，装扮我们的博客。

# 三、数学工具的安装

目前，使用的主题是Hexo的NexT主题。

NexT 内部提供数学公式渲染的引擎，这样你就不需要自己手动在模板中引入 JS 或者 CSS； 只需要选择对应的渲染引擎，并在 next/_config.yml 中将其 enable 选项改为 true 即可。

需要注意的是，仅仅将 enable 打开并不能让你看到数学公式，你还需要使用对应的 Hexo 渲染器(Renderer) 才能真正在博客页面中显示出数学公式。

1. 需要安装[pandoc](https://pandoc.org/installing.html)。（version >= 2.0）

2. 在blog文件夹，卸载原有的渲染器==hexo-renderer-marked==，再安装==hexo-renderer-pandoc==:

   ```bash
   npm uninstall hexo-renderer-marked
   npm install hexo-renderer-pandoc
   ```

3. 在`next/_config.yml`中将mathjax的enable打开：

   ```
   math:
   ...
     mathjax:
         enable: true
   ```

4. 在需要显示公式的文章开头，还需要打开`mathjax`开关，如下：

   ```
   ---
   title: index.html
   date: 2018-07-05 12:01:30
   mathjax: true
   --
   ```

   

# 四、源码的管理

上面使用`hexo d`命令，只是把hexo生成的博客文件上传到了Github，但是博客的源文件并没有上传上去。下一步，我们将要把博客的源文件上传到Github。

在浏览器中，打开上面新建的Github仓库。目前，只有一个master分支，我们需要新建一个hexo分支来存储我们的博客源文件。在左侧Branch:master的位置，输入分支的名称，下面会有提示创建新的分支。

然后，在settings->Branches->Default branch中，将hexo设为默认分支。

在新的文件夹中，使用Git Bash将仓库克隆到本地。

显示隐藏文件，将除了.git文件夹以外的都删除。

把我们之前的博客源文件全复制过来，除了`.deploy_git`。这里注意一下，复制过来的源文件应该有一个`.gitignore`，用来忽略一些不需要的文件，如果没有的话，自己新建一个，内容如下：

```
.DS_Store
Thumbs.db
db.json
*.log
node_modules/
public/
.deploy*/
```

还要注意一下，如果之前克隆过theme中的主题文件，那么也应该把主题文件中的`.git`文件夹删除。

而后，将博客源文件上传到hexo分支。

这样，更换电脑之后，只要将环境配置好，然后把仓库克隆下来，进入克隆下来的文件夹，先运行`hexo clean`试试。根据提示进行一些操作即可。[这里](https://zhuanlan.zhihu.com/p/44213627)有比较详细的步骤可以参考一下，之前做的时候在这一步也耽误了好久。

# 五、Markdown的编辑器

测试了markdownpad2、vscode等，最终选择了Typora编辑器。但要注意的是，Typora有一些markdown的扩展功能，需要在文件->偏好设置里面设置一下，才会有效果，例如==高亮==等。

# 六、一些有帮助的工具软件

1. ScreenToGit：截屏，生成动态图的软件。

2. GeoGebra：动态画图软件，可以很方便的得到各种函数的几何图案。

# 七、一些有用的包

1. serve

   hexo g生成的文件在public文件夹，与网页真正使用的资源文件一致。可以通过打开public里面的网页判断发布的网站是否有问题。

   安装serve：

   > npm install -g serve
   
   开启serve：
   
   > serve

# 参考

- [1] [GitHub+Hexo 搭建个人网站详细教程](https://zhuanlan.zhihu.com/p/26625249)
- [2] [hexo-renderer-pandoc](https://github.com/wzpan/hexo-renderer-pandoc)
- [3] [hexo-theme-next](https://github.com/theme-next/hexo-theme-next)
- [4] [hexo的next主题个性化配置教程](https://segmentfault.com/a/1190000009544924)
- [5] [使用Typora添加数学公式](https://blog.csdn.net/mingzhuo_126/article/details/82722455)
- [6] [hexo超完整的搭建教程，让你拥有一个专属个人博客](https://zhuanlan.zhihu.com/p/44213627)
- [7] [serve](https://github.com/vercel/serve)