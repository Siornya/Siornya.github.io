---
title: fabric开发指北#3 特殊种类物品与容器
date: 2026-06-14 01:57:40
tags: 
- minecraft
- fabric
- 开发
---

# 前言

第 2 篇里我们注册的都是「什么都不会做」的普通物品。这一篇来看几类**有特殊行为的物品**：能吃的食物、能挖能打的工具武器、能穿的护甲、能烧的燃料，最后再做一个能装东西的容器物品。

本文沿用第 2 篇里 `ModItems` 的 `register` 辅助方法，所有物品都加在那个类里。`register` 的第二个参数是物品的构造函数，第三个参数是 `Item.Properties`——特殊物品的「特殊」，要么体现在换一个物品类（工具、护甲），要么体现在给 `Properties` 挂上对应的组件（食物、容器）。回忆一下它的样子：

~~~java
public static Item register(String name, Function<Item.Properties, Item> factory, Item.Properties properties) {
    ResourceKey<Item> key = ResourceKey.create(Registries.ITEM,
            ResourceLocation.fromNamespaceAndPath(ExampleMod.MOD_ID, name));
    Item item = factory.apply(properties.setId(key));
    return Registry.register(BuiltInRegistries.ITEM, key, item);
}
~~~

下面每一节给出的，基本就是往 `register(...)` 里填的那一行，以及需要配套的资源文件。

## 食物

食物的本质是给普通物品挂一个「食物属性」组件，玩家右键就能吃。用 `Item.Properties` 的 `.food(...)` 方法，传入一个 `FoodProperties`：

~~~java
public static final Item TASTY_BERRY = register("tasty_berry", Item::new,
        new Item.Properties().food(
                new FoodProperties.Builder()
                        .nutrition(4)             // 回复的饥饿值（半个鸡腿为 1）
                        .saturationModifier(0.3f) // 饱和度系数
                        .alwaysEdible()           // 饥饿值满时也能吃，可选
                        .build()
        ));
~~~

如果想让食物吃下去附带状态效果（比如金苹果的抗性提升），可以在 `Builder` 上链式添加效果。不同版本里效果相关的写法有差异，具体参数请对照你手上的 API；思路都是「在 `FoodProperties.Builder` 里描述吃下去会发生什么」。

食物物品的纹理、模型、翻译和普通物品完全一样，按第 2 篇的方式补齐即可，这里不再重复。

## 工具与武器

工具（镐、斧、锹、锄）和剑都有各自的物品类：`PickaxeItem`、`AxeItem`、`ShovelItem`、`HoeItem`、`SwordItem`。它们都需要一个**工具材料**（`ToolMaterial`）来决定挖掘等级、耐久、挖掘速度、附魔能力等。

原版自带一套材料（木、石、铁、金、钻石、下界合金），直接拿来用最省事。下面注册一把「铁镐」和一把「铁剑」：

~~~java
public static final Item CUSTOM_PICKAXE = register("custom_pickaxe",
        properties -> new PickaxeItem(ToolMaterial.IRON, properties),
        new Item.Properties().attributes(
                PickaxeItem.createAttributes(ToolMaterial.IRON, 1.0f, -2.8f)));

public static final Item CUSTOM_SWORD = register("custom_sword",
        properties -> new SwordItem(ToolMaterial.IRON, properties),
        new Item.Properties().attributes(
                SwordItem.createAttributes(ToolMaterial.IRON, 3, -2.4f)));
~~~

这里有两个要点：

- 工具的攻击力、攻击速度等数值是通过 `Properties` 上的**属性组件**（`.attributes(...)`）来设置的，每个工具类一般都提供了 `createAttributes(...)` 帮你算好。
- 两个数字分别是「攻击力加成」和「攻击速度」，攻击速度为负值，绝对值越大挥得越慢。

如果你想要一套自己的材料（比如自定义挖掘等级和耐久），就自己实现一个 `ToolMaterial`。它需要描述：挖掘等级（能挖哪些方块）、耐久、挖掘速度倍率、攻击力加成、附魔能力，以及修复材料。把你定义好的材料替换上面的 `ToolMaterial.IRON` 即可。由于这部分接口在不同版本间字段名变动较频繁，建议直接参考你工程里 `ToolMaterial` 的源码或官方文档对照填写。

工具的纹理用「手持工具」的父模型 `minecraft:item/handheld`（而不是普通物品的 `item/generated`），这样握持姿势才正确：

~~~json
{
  "parent": "minecraft:item/handheld",
  "textures": {
    "layer0": "tutorial:item/custom_pickaxe"
  }
}
~~~

别忘了第 2 篇讲的物品模型定义文件 `assets/tutorial/items/custom_pickaxe.json` 也要一并创建。

## 护甲

护甲使用 `ArmorItem` 类，需要一个**护甲材料**（`ArmorMaterial`）和一个**装备槽位**（头、胸、腿、脚）。原版同样自带几套材料。

注册一套护甲就是把同一个材料注册到四个槽位：

~~~java
public static final Item CUSTOM_HELMET = register("custom_helmet",
        properties -> new ArmorItem(ArmorMaterials.IRON, ArmorType.HELMET, properties),
        new Item.Properties().durability(ArmorType.HELMET.getDurability(15)));

public static final Item CUSTOM_CHESTPLATE = register("custom_chestplate",
        properties -> new ArmorItem(ArmorMaterials.IRON, ArmorType.CHESTPLATE, properties),
        new Item.Properties().durability(ArmorType.CHESTPLATE.getDurability(15)));

// 护腿 LEGGINGS、靴子 BOOTS 同理
~~~

护甲比普通物品多一项资源：**穿在身上时的贴图**（玩家模型上渲染的那层）。除了背包里显示的物品纹理外，你还需要提供一套「装备贴图」，并在材料里指定它对应的资源 ID。这套贴图放在 `assets/<命名空间>/textures/entity/equipment/...` 下，路径和分层（外层/内层）的具体要求请对照官方文档，因为这块在版本间调整过几次。

自定义护甲材料则需要描述：每个槽位的防御点数、韧性、击退抗性、附魔能力、穿戴音效、修复材料，以及上面说的装备贴图 ID。

## 燃料

让物品能在熔炉里当燃料烧，不需要新建物品类，只要把它注册进**燃料表**。Fabric 提供了对应的事件，在 `onInitialize` 里注册即可：

~~~java
FuelRegistryEvents.BUILD.register((builder, context) -> {
    // 数字是燃烧时间，单位 tick（一次熔炼 = 200 tick）
    builder.add(ModItems.TASTY_BERRY, 100);
});
~~~

`200` tick 正好能烧一个物品，所以上面这个莓果烧半个东西。你也可以把整组物品（用物品标签）一次性加进去。

> 注意：不要直接修改原版的全局燃料常量，统一走这个事件，既安全又能和别的模组兼容。

## 容器物品

最后做一个能「装东西」的容器物品——类似一个储物袋，能把物品塞进去、取出来。在新版本里，物品的数据是通过**数据组件（Data Components）**存储的，容器物品靠的就是「容器组件」：物品实例上挂一个 `ContainerComponent`，用来保存它内部的物品列表。

最简单的做法是给物品一个空的默认容器组件，让它出生时就带一个空背包：

~~~java
public static final Item POUCH = register("pouch", Item::new,
        new Item.Properties().component(DataComponents.CONTAINER, ContainerComponent.EMPTY));
~~~

只挂组件还不会有交互——光有「存储格子」，玩家还点不动它。要让物品真正能用，有两条路：

1. **复用原版的捆绑包行为**：如果你要的就是「像收纳袋一样塞一堆物品」，可以参考原版 `BundleItem` 的实现，它已经处理好了右键塞取、容量上限、tooltip 预览等逻辑，照着写一个自己的变体最省力。
2. **打开一个自定义界面**：如果你想要一个真正像箱子那样、带格子能拖拽的界面，那就需要 `Screen` + `MenuType`（容器界面）这一套 GUI 机制。这部分内容较多，留到后面专门讲容器界面的篇章。

换句话说，**「数据怎么存」由容器组件解决，「玩家怎么操作」由物品交互或 GUI 解决**，两者是分开的。本篇先让你知道容器物品的数据落点在哪儿，交互部分按需求选上面任一条路。

到这里，你已经掌握了食物、工具、武器、护甲、燃料和容器这几类常见的特殊物品。它们的共同套路是：**普通能力靠换物品类，附加数据靠挂组件**。理解了这一点，再看官方文档里其它种类的物品就会顺很多。

# 传送门

[2 添加物品与方块](../fabric开发指北2)

[目录](../fabric开发指北-目录)
