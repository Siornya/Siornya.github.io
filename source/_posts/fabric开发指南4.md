---
title: fabric开发指南-#4物品提示和语言文件
date: 2022-11-26 23:01:11
categories: fabric
tags: 
- minecraft
- fabric
- 开发
---

# 添加物品提示

**以下内容以1.19及以后版本为准，1.18.2及以前版本可以移步[官网](https://fabricmc.net/wiki/zh_cn:tutorial:tooltip)**

想要添加物品提示，首先要先创建一个自己的物品类，然后重写`appendTooltip`

```Java
@Override
public void appendTooltip(ItemStack itemStack, World world, List<Text> tooltip, TooltipContext tooltipContext) {
    // 默认为白色文本
    tooltip.add(Text.translatable("item.tutorial.fabric_item.tooltip"));
    // 格式化为红色文本
    tooltip.add(Text.translatable("item.tutorial.fabric_item.tooltip").formatted(Formatting.RED));
}
```

也可以在方块类中通过重写类似方法来添加物品提示类

```java
@Override
public void appendTooltip(ItemStack itemStack, BlockView world, List<Text> tooltip, TooltipContext tooltipContext) {
    tooltip.add(Text.translatable("block.tutorial.fabric_block.tooltip") );
}
```

# 修改语言文件

# 传送门

[1. 开始](../fabric开发指南1)

[2. 添加物品与方块](../fabric开发指南2)
