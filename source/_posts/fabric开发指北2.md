---
title: fabric开发指南#2 添加物品与方块
date: 2023-02-05 18:22:17
categories: fabric
tags: 
- minecraft
- fabric
- 开发
---

# 前言

我选择了官方推荐的模板生成器，如果你选择使用插件生成也完全没有问题，因为差别基本只在于插件生成的包结构更加复杂。你几乎不可能在教程里遇到包路径错误的问题，如果遇到了也只要稍微调整一下就好。

在本篇中，我们会添加物品、方块，让方块可以作为一个物品。然后创建一个物品组，让你把所有模组物品放到一个标签页中。最后添加对应的翻译、物品提示和配方。

和之前一样，本文中所有的 `tutorial` 都是模组 ID（命名空间）的占位符。如果你有自己的模组 ID，请替换成你自己的。代码里的 `MOD_ID` 常量一般在模板生成的主类里已经帮你定义好了。

# 添加物品与方块

## 添加物品

### 创建物品类

游戏里每一个物品，本质上都是 `Item` 类（或它的子类）的一个实例。所以「添加物品」要先想清楚一件事：你要的这个物品，**有没有需要写代码实现的特殊行为**。

- 如果只是一个普普通通、放在背包里看看、顶多能合成或当材料用的物品，那你**不需要写任何新类**，直接用原版的 `Item` 就够了——本节后面的例子都属于这种。
- 如果你想让物品有自己的逻辑，比如右键触发一个效果、悬停时显示提示文字、被使用时消耗等等，那就需要写一个类继承 `Item`，重写对应的方法：

~~~java
public class FabricItem extends Item {
    public FabricItem(Properties properties) {
        super(properties);
    }

    // 在这里重写需要的方法，比如 use(...)、appendHoverText(...) 等
}
~~~

继承 `Item` 时，构造函数必须接收一个 `Item.Properties` 并把它交给 `super(...)`，这是后面注册时统一传参的约定。具体能重写哪些方法、怎么实现自定义行为（物品提示、右键交互、食物、工具等），会在本篇后面的[物品提示](#添加物品提示)和[第三篇](../fabric开发指北3)里展开。

本节我们先用最简单的普通 `Item` 走通「创建 → 注册 → 给纹理」的完整流程，把基础打牢。

### 注册物品

添加一个物品本质上只有三步：**创建一个 `Item` 对象 → 把它注册进物品注册表 → 给它一个纹理和模型**。

物品在创建时必须知道自己的注册 ID（也就是要先把 ID「塞」进 `Item.Properties` 里），所以我们要先构造一个 `ResourceKey`，再用它来创建物品。

我们先看最直接的写法，把它放在模组主类里：

~~~java
public class ExampleMod implements ModInitializer {
    public static final String MOD_ID = "tutorial";

    // 1. 先为物品创建一个注册键（ResourceKey）
    public static final ResourceKey<Item> FABRIC_ITEM_KEY =
            ResourceKey.create(Registries.ITEM,
                    ResourceLocation.fromNamespaceAndPath(MOD_ID, "fabric_item"));

    // 2. 用这个键创建物品实例
    public static final Item FABRIC_ITEM =
            new Item(new Item.Properties().setId(FABRIC_ITEM_KEY));

    @Override
    public void onInitialize() {
        // 3. 注册物品
        Registry.register(BuiltInRegistries.ITEM, FABRIC_ITEM_KEY, FABRIC_ITEM);
    }
}
~~~

注册完成后，你就可以在游戏里用 `/give @s tutorial:fabric_item` 拿到这个物品了。此时它还没有纹理，会显示成紫黑色的方块，这是正常的，我们马上来补上。

`Item.Properties` 用来设置物品的各种属性，比如最大堆叠数 `stacksTo(16)`、耐久 `durability(100)`、稀有度 `rarity(...)` 等。你可以在[官方文档](https://docs.fabricmc.net/)查到全部设置，这里先不展开。

> 注意：创建物品时不能直接指定创造模式物品组，物品组要单独处理，我们在后面[创建物品组](#创建物品组)一节再说。

### 用单独的类来注册物品

如果只注册一两个物品，写在主类里没什么问题。但物品一多，主类就会变得又长又乱。一个常见的做法是把所有物品的注册集中到一个单独的类里，并且封装一个 `register` 辅助方法，避免每次都重复写一长串模板代码。

新建一个 `item` 包，在里面创建 `ModItems.java`：

~~~java
public class ModItems {
    // 一个通用的注册辅助方法：传入名字、构造函数和属性，返回注册好的物品
    public static Item register(String name, Function<Item.Properties, Item> factory, Item.Properties properties) {
        ResourceKey<Item> key = ResourceKey.create(Registries.ITEM,
                ResourceLocation.fromNamespaceAndPath(ExampleMod.MOD_ID, name));
        Item item = factory.apply(properties.setId(key));
        return Registry.register(BuiltInRegistries.ITEM, key, item);
    }

    // 之后所有物品都写在这里
    public static final Item FABRIC_ITEM = register("fabric_item", Item::new, new Item.Properties());

    // 这个方法本身可以是空的，作用只是确保类被加载、上面的静态字段被初始化
    public static void initialize() {}
}
~~~

然后在主类的 `onInitialize` 里调用一次 `ModItems.initialize()` 即可：

~~~java
public class ExampleMod implements ModInitializer {
    public static final String MOD_ID = "tutorial";

    @Override
    public void onInitialize() {
        ModItems.initialize();
    }
}
~~~

这样以后每加一个物品，只要在 `ModItems` 里加一行 `register(...)`，主类完全不用动，干净很多。本文后面的内容都默认你采用这种方式。

### 物品的纹理和模型

光注册还不够，物品需要纹理（一张图）和模型（描述怎么渲染这张图）。这部分是纯资源文件，和 Java 代码无关。

先把你的纹理图放到：

```
src/main/resources/assets/tutorial/textures/item/fabric_item.png
```

普通物品的纹理一般是 16×16 的 PNG。

然后是模型。物品模型分两层，先有一个「物品模型定义」文件，用来把物品和具体模型关联起来。创建：

```
src/main/resources/assets/tutorial/items/fabric_item.json
```

~~~json
{
  "model": {
    "type": "minecraft:model",
    "model": "tutorial:item/fabric_item"
  }
}
~~~

它指向真正的模型文件，再创建：

```
src/main/resources/assets/tutorial/models/item/fabric_item.json
```

~~~json
{
  "parent": "minecraft:item/generated",
  "textures": {
    "layer0": "tutorial:item/fabric_item"
  }
}
~~~

`minecraft:item/generated` 是最常见的「平面物品」父模型，`layer0` 指向你刚放进去的纹理。保存后重新进游戏，物品就有图了。

> 这些 JSON 文件大多是高度模板化的，正好适合用[第一篇](../fabric开发指北1)里提到的 Data Generation 自动生成，省去手写一堆重复文件的麻烦。本文为了讲清楚结构先手写，等你熟悉了强烈建议改用数据生成。

## 添加方块

### 注册方块

方块的注册和物品几乎一模一样，只是把注册表换成方块注册表，属性换成 `BlockBehaviour.Properties`。在 `block` 包下新建 `ModBlocks.java`：

~~~java
public class ModBlocks {
    public static Block register(String name, Function<BlockBehaviour.Properties, Block> factory,
                                 BlockBehaviour.Properties properties) {
        ResourceKey<Block> key = ResourceKey.create(Registries.BLOCK,
                ResourceLocation.fromNamespaceAndPath(ExampleMod.MOD_ID, name));
        Block block = factory.apply(properties.setId(key));
        return Registry.register(BuiltInRegistries.BLOCK, key, block);
    }

    public static final Block FABRIC_BLOCK = register("fabric_block", Block::new,
            BlockBehaviour.Properties.of().strength(4.0f));

    public static void initialize() {}
}
~~~

`BlockBehaviour.Properties.of()` 用来设置方块的硬度、爆炸抗性、音效、是否需要工具等。例如 `strength(4.0f)` 同时设置了挖掘硬度和爆炸抗性；你也可以用 `requiresCorrectToolForDrops()`、`sound(SoundType.STONE)` 等。

别忘了在主类里调用 `ModBlocks.initialize()`。

### 让方块成为一个物品（BlockItem）

到这里方块在世界里已经存在了，但它**没有对应的物品**——你没法拿在手上，也没法放进物品栏。要让方块能被拿取和放置，需要为它注册一个 `BlockItem`。

最简单的方式是在注册方块后顺手注册一个对应的物品。我们可以在 `ModBlocks.register` 里直接做掉，或者单独写。这里用直观一点的写法，在注册方块的辅助方法里加一个「同时注册方块物品」的版本：

~~~java
public class ModBlocks {
    public static Block register(String name, Function<BlockBehaviour.Properties, Block> factory,
                                 BlockBehaviour.Properties properties, boolean registerItem) {
        ResourceKey<Block> key = ResourceKey.create(Registries.BLOCK,
                ResourceLocation.fromNamespaceAndPath(ExampleMod.MOD_ID, name));
        Block block = factory.apply(properties.setId(key));

        if (registerItem) {
            // 方块物品复用同一个名字
            ResourceKey<Item> itemKey = ResourceKey.create(Registries.ITEM,
                    ResourceLocation.fromNamespaceAndPath(ExampleMod.MOD_ID, name));
            BlockItem blockItem = new BlockItem(block, new Item.Properties().setId(itemKey));
            Registry.register(BuiltInRegistries.ITEM, itemKey, blockItem);
        }

        return Registry.register(BuiltInRegistries.BLOCK, key, block);
    }

    public static final Block FABRIC_BLOCK = register("fabric_block", Block::new,
            BlockBehaviour.Properties.of().strength(4.0f), true);

    public static void initialize() {}
}
~~~

现在 `/give @s tutorial:fabric_block` 就能拿到方块物品了。

### 方块的模型、方块状态和纹理

方块需要的资源文件比物品多一点，因为它既要在世界里渲染（方块状态 + 方块模型），又要在物品栏里渲染（物品模型）。

先放纹理：

```
src/main/resources/assets/tutorial/textures/block/fabric_block.png
```

**1. 方块状态文件**，告诉游戏每种状态用哪个模型。我们的方块没有额外状态，所以只有一个默认项：

```
src/main/resources/assets/tutorial/blockstates/fabric_block.json
```

~~~json
{
  "variants": {
    "": { "model": "tutorial:block/fabric_block" }
  }
}
~~~

**2. 方块模型**，这里用最常见的「六面相同」的立方体父模型：

```
src/main/resources/assets/tutorial/models/block/fabric_block.json
```

~~~json
{
  "parent": "minecraft:block/cube_all",
  "textures": {
    "all": "tutorial:block/fabric_block"
  }
}
~~~

**3. 物品模型**，方块物品在手上和物品栏里的样子，直接复用方块模型即可。同样用物品模型定义文件：

```
src/main/resources/assets/tutorial/items/fabric_block.json
```

~~~json
{
  "model": {
    "type": "minecraft:model",
    "model": "tutorial:block/fabric_block"
  }
}
~~~

**4. 战利品表**（可选但通常需要），否则方块被破坏后不会掉落自己。注意目录名是单数 `loot_table`：

```
src/main/resources/data/tutorial/loot_table/blocks/fabric_block.json
```

~~~json
{
  "type": "minecraft:block",
  "pools": [
    {
      "rolls": 1,
      "entries": [
        { "type": "minecraft:item", "name": "tutorial:fabric_block" }
      ],
      "conditions": [
        { "condition": "minecraft:survives_explosion" }
      ]
    }
  ]
}
~~~

进游戏后，方块应该能正常放置、破坏并掉落自己了。

## 创建物品组

模组物品多了以后，散落在原版各个创造栏里很难找，给自己的模组建一个专属物品组（也就是创造模式背包里的一个标签页）会方便很多。物品组对应的类型是 `CreativeModeTab`。

Fabric 提供了 `FabricItemGroup.builder()` 来方便地创建。我们新建一个 `ModItemGroups.java`：

~~~java
public class ModItemGroups {
    public static final ResourceKey<CreativeModeTab> TUTORIAL_GROUP_KEY =
            ResourceKey.create(Registries.CREATIVE_MODE_TAB,
                    ResourceLocation.fromNamespaceAndPath(ExampleMod.MOD_ID, "general"));

    public static final CreativeModeTab TUTORIAL_GROUP = Registry.register(
            BuiltInRegistries.CREATIVE_MODE_TAB,
            TUTORIAL_GROUP_KEY,
            FabricItemGroup.builder()
                    // 标签页的图标
                    .icon(() -> new ItemStack(ModItems.FABRIC_ITEM))
                    // 标签页的标题（翻译键，稍后在语言文件里填）
                    .title(Component.translatable("itemGroup.tutorial.general"))
                    // 直接在这里加入要显示的物品
                    .displayItems((params, output) -> {
                        output.accept(ModItems.FABRIC_ITEM);
                        output.accept(ModBlocks.FABRIC_BLOCK);
                    })
                    .build());

    public static void initialize() {}
}
~~~

记得在主类里调用 `ModItemGroups.initialize()`。

如果你不想新建物品组，只是想把某个物品**加进原版已有的物品组**（比如「建筑方块」或「工具」标签页），可以用 Fabric 的事件：

~~~java
ItemGroupEvents.modifyEntriesEvent(ItemGroups.BUILDING_BLOCKS)
        .register(entries -> entries.accept(ModBlocks.FABRIC_BLOCK));
~~~

这种方式也可以用来往**自己**的物品组里追加物品，适合物品分散在不同类里、不想集中写在 `displayItems` 的情况。

## 添加翻译

你可能已经注意到，现在物品、方块、物品组显示出来的名字都是一长串翻译键（比如 `item.tutorial.fabric_item`），这是因为我们还没提供翻译。语言文件放在：

```
src/main/resources/assets/tutorial/lang/zh_cn.json
```

~~~json
{
  "item.tutorial.fabric_item": "织物物品",
  "block.tutorial.fabric_block": "织物方块",
  "itemGroup.tutorial.general": "教程模组"
}
~~~

翻译键的规则很固定：
- 物品是 `item.<命名空间>.<物品名>`
- 方块是 `block.<命名空间>.<方块名>`
- 物品组是 `itemGroup.<命名空间>.<组名>`

建议同时提供一份英文 `en_us.json`，因为英文是游戏的默认回退语言，玩家把语言切成别的时如果缺翻译，会回退到英文而不是直接显示翻译键：

```
src/main/resources/assets/tutorial/lang/en_us.json
```

~~~json
{
  "item.tutorial.fabric_item": "Fabric Item",
  "block.tutorial.fabric_block": "Fabric Block",
  "itemGroup.tutorial.general": "Tutorial Mod"
}
~~~

## 添加物品提示

物品提示（tooltip）就是把鼠标悬停在物品上时，名字下面显示的那几行说明文字。有两种常见做法。

最简单的是**纯文本提示**：直接在语言文件里加一个 `.tooltip` 结尾的键，再在物品提示里引用。不过更通用、可控的方式是创建自己的物品类并重写 `appendHoverText`：

~~~java
public class FabricItem extends Item {
    public FabricItem(Properties properties) {
        super(properties);
    }

    @Override
    public void appendHoverText(ItemStack stack, TooltipContext context,
                                List<Component> tooltip, TooltipFlag flag) {
        // 默认白色文本
        tooltip.add(Component.translatable("item.tutorial.fabric_item.tooltip"));
        // 也可以格式化，比如变成灰色
        tooltip.add(Component.translatable("item.tutorial.fabric_item.tooltip.detail")
                .withStyle(ChatFormatting.GRAY));
    }
}
~~~

然后注册物品时把构造函数从 `Item::new` 换成 `FabricItem::new`：

~~~java
public static final Item FABRIC_ITEM =
        register("fabric_item", FabricItem::new, new Item.Properties());
~~~

最后在语言文件里补上对应的翻译键即可：

~~~json
{
  "item.tutorial.fabric_item.tooltip": "这是一段物品提示",
  "item.tutorial.fabric_item.tooltip.detail": "按住可以查看更多信息"
}
~~~

方块同理，可以在自定义方块类里重写对应的方法来添加提示。

## 添加配方

最后是配方，让你的物品能被合成出来。配方是纯数据文件，放在 `data` 目录下，注意目录名是单数 `recipe`：

```
src/main/resources/data/tutorial/recipe/fabric_item.json
```

**有序合成**（在工作台上摆成特定形状），下面这个配方表示用一个铁锭顶上一根木棍合成织物物品：

~~~json
{
  "type": "minecraft:crafting_shaped",
  "pattern": [
    "S",
    "I"
  ],
  "key": {
    "S": { "item": "minecraft:stick" },
    "I": { "item": "minecraft:iron_ingot" }
  },
  "result": {
    "id": "tutorial:fabric_item",
    "count": 1
  }
}
~~~

`pattern` 用字符摆出合成表的形状，`key` 把每个字符对应到具体物品，`result` 是产物。

如果不在意摆放形状，用**无序合成**更省事：

~~~json
{
  "type": "minecraft:crafting_shapeless",
  "ingredients": [
    { "item": "minecraft:iron_ingot" },
    { "item": "minecraft:stick" }
  ],
  "result": {
    "id": "tutorial:fabric_item",
    "count": 1
  }
}
~~~

> 配方、战利品表、模型这类 JSON 文件正是 Data Generation 最擅长的领域。当文件数量多起来，手写很容易出错，强烈建议回头去用[第一篇](../fabric开发指北1)里启用的数据生成功能，用 Java 代码批量生成它们。

到这里，你已经能添加物品、方块、把方块变成可放置的物品、建立自己的物品组，并为它们补上翻译、提示和配方了。这基本覆盖了一个内容模组最常用的「加东西」流程。
