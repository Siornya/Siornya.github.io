---
title: fabric开发指南#2 添加物品与方块
date: 2023-02-05 18:22:17
categories: fabric
tags: 
- minecraft
- fabric
- 开发
---

# 添加物品

## 介绍

创建一个`Item`对象，注册它，并赋予它一个纹理。然后自定义的`Item`类，向物品添加其他行为。与官网相同，在本系列的所有内容中，`tutorial`命名空间均用作模组的ID的占位符。如果你有单独的模组ID，则应当使用它。在这部分内容中会先简单说明怎么注册物品，然后详细看一个例子 (☆▽☆)

## 注册物品

首先，创建一个`Item`的实例。我们将其存放在模组主类的顶部。`Item`的构造方法接受一个`Item.Settings`（或`FabricItemSettings`，`FabricItemSettings`提供了更多设置，可以优先选择这个）对象，该对象用于设置物品属性，例如创造模式物品栏中的分类、耐久和堆叠数量。在[官网](https://fabricmc.net/wiki/zh_cn:tutorial:items_docs)可以查到所有的设置。

~~~Java
public class ExampleMod implements ModInitializer {
    public static final Item FABRIC_ITEM = new Item(new FabricItemSettings());
    // public static final Item FABRIC_ITEM = new Item(new FabricItemSettings().group(ItemGroup.MISC));
    //                                                                         ^^^^^^^^^^^^^^^^^^^^^^
    // 以前可以这样指定物品组
    @Override
    public void onInitialize() {
        Registry.register(Registries.ITEM, new Identifier("tutorial", "fabric_item"), FABRIC_ITEM);
        //                ^^^^^^^^^^                      ^^^^^^^^^^^^^^^^^^^^^^^^^
        // 1.19.3为Registries，以前版本为Registry
        // 此处的tutorial为namespace，fabric_item为path
        // 你可以通过指令/give @s namespace:path来获取物品，例如这个物品可以通过/give @s tutorial:fabric_item得到
    } 
}
~~~

## 通过单独文件注册物品

如果我们注册了很多物品，而且把所有代码都放在同一个文件中代码就会显得非常复杂。加上把物品添加到物品组中的代码以后，这个文件就会显得极其复杂，所以我们可以单独创建一个文件来注册物品。

而为了方便我们后期添加物品提示或其它操作，我们把这个文件再放到一个文件夹中。以下是参考结构树。

```
├── ExampleMod.java
└── NewFolder
    ├── A.java
    ├── B.java
    └── ···.java
```

接下来在新创建的文件夹下创建一个新文件`AddItem.java`，文件名可以随意更改。然后我们将文章上方的注册物品部分的代码移动到这个新文件中，补全缺失的类。

```Java
// 把此处的"implements ModInitializer"去掉
public class AddItem {
    public static final Item FABRIC_ITEM = new Item(new FabricItemSettings());
    // 把此处的"@Override"去掉
    // 创建一个RegistryItem方法来注册物品，方法名可更改
    public static void RegisterItem(){
        Registry.register(Registry.ITEM, new Identifier("tutorial", "fabric_item"), FABRIC_ITEM);
        //                ^^^^^^^^^
        // 1.19.3为Registries，以前版本为Registry
    }
}
```

然后在主文件中调用这个方法，记得`import`新的文件。

```Java
import [package路径].[文件夹名].[文件名];

public class ExampleMod implements ModInitializer {
    @Override
    public void onInitialize() {
        AddItem.RegisterItem();
    }
}
```

# 添加方块

## 介绍

# 传送门

[1. 开始](../fabric开发指南1)

[3. 物品提示和语言文件](../fabric开发指南3)
