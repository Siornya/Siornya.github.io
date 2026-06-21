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

## Fabric 的组成部分

- Fabric Loader：模组加载器，用于在游戏启动时加载模组
- Fabric API：一套 API 接口，封装了很多游戏内功能，本质上是一个基础模组
- Fabric Loom：一款 Gradle 插件，用于开发和调试模组，并且完成编译和打包任务
- Yarn：Yarn 是 Fabric 生态中的混淆映射表，用于反混淆

*在Minecraft 26.1版本以后，mojang官方取消了混淆，Yarn已经不再为新版本提供支持，需要迁移到 Mojang 的官方映射*

## 配置 JDK

首先需要配置JDK(Java Development Kit)，用于编译和运行 Java 代码——Minecraft 本体以及 Fabric 都是基于 Java 的，所以这是开发环境的基础。

对于 Minecraft 1.20.5 版本需要使用 JDK 21，而在更新的 26.1 及更新版本，需要使用 JDK 25。大部分情况下安装更新版本的 JDK 不会有问题，所以你可以直接选择最新的 LTS（长期支持）版本。

一般来说 IDE 中提供了一键下载的方式，但是你也可以选择手动安装。可以选择[Temurin JDK](https://adoptium.net/zh-CN/temurin/releases)，由社区驱动更新，完全免费，这也是 Fabric 官方推荐的版本。或者也可以选择[Oracle JDK](https://www.oracle.com/cn/java/technologies/downloads/)，个人非商用免费，但商业需付费。

安装完成后，记得检查环境变量 `JAVA_HOME` 是否指向了正确的 JDK 目录，并在终端运行 `java -version` 确认版本无误。

## 搭建 IDE

在开发模组之前，你还需要搭建开发环境，可以选择 [IntelliJ IDEA（推荐）](https://www.jetbrains.com/idea/download/)或者 [Visual Studio Code](https://code.visualstudio.com/)

**开发 26.1 版本的模组需要IntelliJ IDEA 2025.3或更高版本。**

如果你选择 VSC，那还需要安装扩展Extension Pack for Java，安装扩展后你可以使用命令`Java: Install New JDK`快速安装JDK。

对于 IDEA，在创建项目后会提示安装 JDK。另外，你可以安装由社区开发的插件[Minecraft Development](https://plugins.jetbrains.com/plugin/8327-minecraft-development)，当然这不是必要的。

## 创建项目

你可以选择使用Minecraft Development插件创建项目，不过官方提供了更好更通用的方法：使用[Fabric模板模组生成器](https://fabricmc.net/develop/template/)

因为所有模组最终会加载到一起，所以请填写一个不会和别人重复的 Package Name。另外，你可以启用 Advanced Options 中的 Data Generation，可以用来帮你快速生成配方、进度、战利品表等任何 JSON 格式的文件

下载生成的模板以后打开项目。如果你使用 IDEA 此时顶端会提示你进行下载。