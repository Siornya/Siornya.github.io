---
title: fabric开发指南#1 环境配置
date: 2023-02-05 18:21:37
categories: fabric
tags: 
- minecraft
- fabric
- 开发
---

# 旧版

**这是2023年版本的前言部分**

这个系列创作于2021年6月，不过在2022年11月已经完整更新过啦！所以可以放心食用，~~至少可以用几年吧？~~

这个系列的内容相比网上的其他教程而言更专注于**官方wiki没有的，网上找不到的**~~**，大家不一定用得到的**~~内容

这个系列的内容默认你**写过**代码，所以会一定程度上省略极其基础的内容

希望这个系列的内容可以在你突然想做一些网上没有教程的内容时给你提供帮助~~祝你灵感不断，开发顺利！

# 新版

## 开始

首先需要配置JDK和IDE。JDK方面官方推荐的是JDK 17（2022-11-12），你可以在[这个网站](http://jdk.java.net/)获取更多JDK版本。IDE官方推荐的是[Intellij IDEA](https://www.jetbrains.com/idea/)，选择社区版即可（免费），愿意买专业版当然也行啦。VSC等编辑器也是可以用的，不过还是推荐使用IDEA。

你可以手动下载 `fabric-example-mod` 来新建项目，如果你使用IDEA进行开发，也可以安装**Minecraft DEV插件**，用起来很方便。

生成Minecraft源代码是非常有必要的，你需要自己生成 Minecraft 源代码。

运行 gradle 任务 `genSources`。如果你的 IDE 没有嵌入 gradle，在终端内运行以下命令：`./gradlew genSources`。**IDEA使用MDEV插件会自动运行**
