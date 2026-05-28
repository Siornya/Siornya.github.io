---
title: fabric开发指南-#1开始
date: 2023-02-05 18:21:37
categories: fabric
tags: 
- minecraft
- fabric
- 开发
---

# 前言

这个系列创作于2021年6月，不过在2022年11月已经完整更新过啦！所以可以放心食用，~~至少可以用几年吧？~~

这个系列的内容相比网上的其他教程而言更专注于**官方wiki没有的，网上找不到的**~~**，大家不一定用得到的**~~内容

这个系列的内容默认你有**写过**代码（成功运行过自己写的代码即可），所以会一定程度上省略极其基础的内容

希望这个系列的内容可以在你突然想做一些网上没有教程的内容时给你提供帮助~~祝你灵感不断，开发顺利！

**2023-02-05：无语，1.19.3又改东西了，所以又更新了一遍**

# 开始

首先需要配置JDK和IDE。JDK方面官方推荐的是JDK 17（2022-11-12），你可以在[这个网站](http://jdk.java.net/)获取更多JDK版本。IDE官方推荐的是[Intellij IDEA](https://www.jetbrains.com/idea/)，选择社区版即可（免费），愿意买专业版当然也行啦。VSC等编辑器也是可以用的，不过还是推荐使用IDEA。

你可以手动下载 `fabric-example-mod` 来新建项目，如果你使用IDEA进行开发，也可以安装**Minecraft DEV插件**，用起来很方便。

> 以下是官网上的手动配置教程
>
> 1. 复制 [fabric-example-mod](https://github.com/FabricMC/fabric-example-mod/) 中的初始文件，并删除 `LICENSE`（许可证）及 `README.md`（简介）文件，因为它们只应用于模板自身，而非你的模组。
> 2. 编辑 `gradle.properties`:
>    - 确保将 `archives_base_name` 和 `maven_group` 设为你喜欢的值。
>    - 确保更新 Minecraft、映射、加载器和 loom 的版本——可以在[官网](https://fabricmc.net/develop/)查询。
>    - 添加你需要在 `build.gradle` 中使用的其他依赖。
> 3. 将 `build.gradle` 导入到你的 IDE 中

> 以下是官网上针对IDEA给出的建议
>
> *可选，但推荐做的一件事*： IDEA 默认使用 Gradle 来构建你的项目，而这在 Fabric 是不必要的，而且会导致构建时间变长以及热交换（hotswapping）相关的种种问题。以下是让 IDEA 使用默认编译器的步骤：
>
> 1. 在 Gradle 页面里打开“Gradle 设置（Gradle Settings）”
> 2. 将“使用此工具构建和运行（Build and run using）”和“使用此工具运行测试（Run tests using）”选项改成“IntelliJ IDEA”。
> 3. 进入 文件（File）→ 项目结构…（Project Structure…）→ 项目（Project）然后将模块编译输出路径（Project compiler output）改成 `$PROJECT_DIR$/out`。
>
> 不幸的是，目前还不能给“使用此工具构建和运行”和“使用此工具运行测试”设置一个全 IDE 内的默认值，所以这些每创建一个新项目都得重复上述步骤。
>
> **注：\*千万\*** 不要运行 `idea` 的 gradle 任务，已知它会破坏开发环境。

其他IDE的配置可以前往[官网](https://fabricmc.net/wiki/zh_cn:tutorial:setup)自行查阅

生成Minecraft源代码是非常有必要的，你需要自己生成 Minecraft 源代码。

运行 gradle 任务 `genSources`。如果你的 IDE 没有嵌入 gradle，在终端内运行以下命令：`./gradlew genSources`。**IDEA使用MDEV插件会自动运行**

## 官网上的一些建议

> - 虽然 Fabric API 并不是必需的，但其最首要的目标是提供游戏引擎所不提供的跨模组兼容性和接口，所以我们**强烈**推荐多使用 Fabric API。本 wiki 上的许多教程也会默认使用 Fabric API。
>
> - 随着 fabric-loom（我们的Gradle构建插件）的开发和改动，有些时候你可能会遇上一些通过重置 Gradle 缓存才能解决的问题。使用 `gradlew cleanloom` 便能清理缓存，而 `gradlew --stop` 则能帮助你解决一些其他疑难杂症。
>
> - 问问题不要犹豫，有问题就问，总有人会帮你解决的。（笑
>
> - 故障诊断
>
>   ### 缺少声音
>
>   有时当 IDE 在导入 Gradle 项目的时候有些游戏素材不会正常下载。如果遇到这种情况则要手动运行 `downloadAssets` 任务——既可以使用 IDE 的自带菜单也可以直接执行 `gradlew downloadAssets`。
>
>   ### java.lang.ClassNotFoundException: net.fabricmc.loader.impl.launch.knot.KnotClient
>
>   这可能是因为项目路径有中文字符或其他可能造成编码不兼容的字符造成的。可以尝试将项目移到不含中文的路径中，或者在启动参数中（编辑配置），将“Minecraft Client”和“Minecraft Server”启动配置的“缩短命令行”（Shorten command line）设为“无”。
>
>   ## 应用更改而无需重新启动 Minecraft
>
>   重新启动 Minecraft 可能会花费大量时间。不过还好，有一些工具可以让您在游戏运行时应用某些更改。
>
>   ### 重新加载更改的类
>
>   在 Eclipse 或 Intellij IDEA 中，以调试模式运行 Minecraft。要应用代码更改，在 Intellij 中点按“构建”按钮或在 Eclipse 中保存。注意：这仅允许您更改方法主体。如果进行任何其他类型的更改，则必须重新启动。但是，如果使用 [DCEVM](http://dcevm.github.io/)，则可以进行大多数更改，包括添加和删除方法和类。
>
>   ### 重新加载资源
>
>   更改资源（如纹理或者方块/物体模型）后，可以重建项目并按 `F3 + T` 以应用更改，而无需重新启动Minecraft。这实际上就是重新加载模组作为资源包提供的任何内容的方法。
>
>   ### 重新加载数据
>
>   你可以应用在 `data/` 目录中做出的任何更改（如配方、战利品表和标签），方法就是重建项目并使用游戏命令 `/reload`。这实际上就是重新加载模组作为数据包提供的任何内容的方法。

# 传送门

[2. 添加物品](../fabric开发指南2)

