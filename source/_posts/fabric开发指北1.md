---
title: fabric开发指南#1 环境配置
date: 2026-06-09 20:38:49
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

## 配置环境

首先需要配置JDK和IDE。JDK方面需要的版本**取决于你要开发的Minecraft版本**：从 1.20.5（2024年4月）起 Minecraft 已经迁移到 **JDK 21**，因此现在开发新版本请直接安装 JDK 21；如果你还在维护 1.18 ~ 1.20.4 的老版本，则继续使用 JDK 17 即可。你可以在[这个网站](https://jdk.java.net/)获取各个版本的 OpenJDK，。IDE官方推荐的是[Intellij IDEA](https://www.jetbrains.com/idea/)，选择社区版即可（免费），愿意买专业版当然也行啦。VSC等编辑器也是可以用的，不过还是推荐使用IDEA。

## 创建项目

新建项目最方便的方式是使用官方的[模板生成器](https://fabricmc.net/develop/template/)：在网页上填好 Mod 名称、包名、目标 Minecraft 版本等信息，下载生成的项目压缩包即可，省去了手动改 `gradle.properties` 的麻烦。当然你也可以照旧手动 clone [`fabric-example-mod`](https://github.com/FabricMC/fabric-example-mod) 来新建项目。如果你使用IDEA进行开发，建议安装 **Minecraft Development 插件**（即原来的 MDEV/Minecraft DEV 插件），它对运行配置、混入（Mixin）等都有很好的支持，用起来很方便。

生成Minecraft源代码是非常有必要的——只有这样你才能在 IDE 里查看、跳转游戏本体的（已反混淆的）源码。

运行 gradle 任务 `genSources`。如果你的 IDE 没有嵌入 gradle，在终端内运行以下命令：`./gradlew genSources`（Windows 下使用 `gradlew genSources`）。**IDEA 安装 Minecraft Development 插件后，导入项目时通常会自动完成这一步**。