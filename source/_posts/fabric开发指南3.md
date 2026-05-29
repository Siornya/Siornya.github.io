---
title: fabric开发指南-#3添加到物品组
date: 2023-02-06 22:28:32
categories: fabric
tags: 
- minecraft
- fabric
- 开发
---

# 关于本文

更新于 2023-02-06 22:28:32 (Minecraft 1.19.3)

以前这部分内容只是作为一个小节放在前面的，不过1.19.3改动还挺大，就单独拎出来了w

以前版本的写法在下面，可以直接往下拉

# 介绍



# 1.19.3



# 1.19.2及以前版本

但是如果我们注册了很多个物品，我们总不能把所有物品都放到现有的物品组的最后吧，而且自己的mod有一个专属的页面总归是有必要的。

```Java
// build方法可以进行快速的创建
public class ExampleMod implements ModInitializer {
	public static final ItemGroup ITEM_GROUP = FabricItemGroupBuilder.build(
		new Identifier("tutorial", "general"),
		() -> new ItemStack(Blocks.COBBLESTONE));

// create方法可以进行精细的创建
	public static final ItemGroup OTHER_GROUP = FabricItemGroupBuilder.create(
		new Identifier("tutorial", "other"))
        //这一行代码可以设置物品组分页标签的图标
		.icon(() -> new ItemStack(Items.BOWL))
		.build();
}
```

以下是~~较为~~完整的示例代码

```Java
public class ExampleMod implements ModInitalizer {
    public static final Item public static final Item FABRIC_ITEM = new Item(new FabricItemSettings());
    public static final ItemGroup EXAMPLE_GROUP = FabricItemGroupBuilder.create(
		new Identifier("tutorial", "general"))
        //这一行代码可以设置物品组分页标签的图标
		.icon(() -> new ItemStack(Items.BOWL))
        .appendItems(stacks -> {
                stacks.add(new ItemStack(FABRIC_ITEM));
				stacks.add(ItemStack.EMPTY);
				stacks.add(new ItemStack(Items.IRON_SHOVEL));
            })
		.build();
}
```

这个物品组中应该会有一个你创建的物品，中间空了一格，然后是一个铁锹。这个物品组的完整翻译键是`ItemGroup.mod_ID.general`
