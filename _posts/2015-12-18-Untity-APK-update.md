---
layout: post
title:  "Unity3d在安卓android的更新(APK覆盖)"
author: "Zongquan"
comments: true
---

其实这并没什么技术难点，也不是完美的热更新方案，只能说是退而求其次的一个方法。

起因主要是因为公司几个U3D项目在立项之初都没有能做好热更新的规化，导致现在要去做U3D的热更新非常难，并且项目已处于中后期，大部分的方案不管是用反射，还是用Lua，或是jsbinding，都需要把项目大部分代码结构推倒重来，这是非常不现实的。于是退而求其次，选择还是直接用最小APK来更新游戏。

也许很多人也是这么做的，但未见人分享，写这篇Blog的目的主要是在网上很难搜到相关的整理，并且大部分游戏制作者仅仅是对开发游戏和用Unity会比较熟悉，对Andriod和iOS并不熟悉，所以有些方法可能想到了，但如果没有人去验证过，总还是不放心的，于是就打算记录下来分享给大家。

**方案重点就是：用户在第一次安装游戏的时候可以用完整的APK包来进行安装，在之后如果存在逻辑代码需要更新时，仅需帮用户下载7MB左右的一个最小APK来把游戏覆盖安装即可。（虽然这个7MB左右还是比较大）**

我想强调一下，这种方式是100%可行的，只适合于安卓，已验证，并且正在使用，没有更好的方案前会一直用，有好的更新方案后也会在Blog更新，并且也希望大家能分享。

U3D的资源更新的方式有两种，这两种方式能保证游戏的资源更新，具体就不详细说明了：

1. WWW.LoadFromCacheOrDownload，这种方式主要是只支持AssetBundle，所以你需要将所有文件打包成AssetBundle。

2. WWW之后用File来写到磁盘，这种方式就得自己来控制版本了。

**用最小APK来更新游戏逻辑的具体步骤：**

**第一步：**拷贝APK安装路径Application.dataPath的asset目录文件到可读写目录Application.persistentDataPath

在第一次初始化游戏的时候，需要通过WWW.LoadFromCacheOrDownload将所有游戏里用到的资源加载一次（或者WWW+File，在打包APK时将所有asset做成一个zip包，第一次启动游戏时解压到Application.persistentDataPath），让他们放进缓存（这个过程视游戏安装包大小而定，当然，你也可以在其它你觉得合适的时候才做这件事件）。

**第二步：**制作最小安装包

将工程中StreamingAssets这个目录下的文件全部清空，然后再用Unity打包一个APK，这个APK大概有7MB左右。

**第三步：**更新逻辑

将你制作的最小的APK放到网站服务器上，通过WWW.bytes来下载，然后用FileStream来写到到Application.persistentDataPath。

**第四步：**调用Andriod的系统API，来执行APK的安装（怎么调用可以去百度搜“Unity Andriod交互”，"Andriod 安装APK 代码"）。

这个安装会覆盖掉之前安装的APK，由于这个APK里asset目录是空的，所以覆盖之后也就没有asset了，这也是为什么要做第一步的原因。另外，这个Application.persistentDataPath+filename 在Java端调用的时候，需要在路径最前面添加"file://"。

这里强调一下，重新安装APK后，Application.persistentDataPath和WWW.LoadFromCacheOrDownload缓存的文件，是不会被覆盖的，所以请放心覆盖原来的APK。

虽然说这种方法的确不怎么人性化，每次要用户下载的更新包最小也是7MB+，另外每次还要弹出那个Andriod上安装APK的对话框，但是，谁叫我们在最初用U3D的时候不好好规划下热更新的问题呢。

同时也在此呼吁Unity3D官方能给出热更新方案，至少给出Andriod的逻辑热更方案，明明只需要把*.so的加载路径改为可配置，大家就可以方便地实现逻辑的热更新，但确没有提供这样的方法。我只想说，Unreal4都开源了，Unity3D还拿什么竞争。