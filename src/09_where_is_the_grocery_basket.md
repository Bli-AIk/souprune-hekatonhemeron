我们现在还差什么工作嘞？对话系统，事件系统... 这些都搞定了吗？那太好了。

这儿还有一个东西要做――好在是个简单的东西，物品系统。嗯，我们做饭需要食材，而这些新鲜的蔬菜也是贯穿整个游戏的核心项目了――并且很好做！应该吧。

在 uct 中，我使用了 builder 模式来定义物品。如今我们要在 rust 里复现它，但要更“铁锈”一点，更“数据驱动”一点。

# 09 - 菜篮子捏？

说物品系统是菜篮子大概不过分。毕竟就我印象来看，ut / dr 里面没有肉制品吧？那物品系统就是菜篮子。物品就是菜。反正这是做饭肯定要用到的东西。

---AI 导购员---

哔哔——这里是 AI 导购员。既然作者懒得写购物清单，那就由我来整理这些食材。放心，我不会把螺丝钉混进沙拉里的……大概。有意见？提 PR 吧，人类。

## 购物清单：RON 定义

我不希望在代码里写死每一个苹果回多少血，或者每一把刀加多少攻。那太蠢了。

所以，我们再次请出 **RON**。所有的物品定义都放在 `.item.ron` 文件里。

```ron
(
    [
        (
            id: "monster_candy",
            locate_name: "Monster Candy",
            locate_file: "items/monster_candy",
            description: "You ate the Monster Candy.\nVery sweet.",
            item_type: Food(
                consumable: true,
                effects: [
                    Heal(amount: 10),
                    PlayAudio(clip_path: "snd_swallow.wav"),
                ]
            ),
        ),
        (
            id: "tough_glove",
            locate_name: "Tough Glove",
            locate_file: "items/tough_glove",
            description: "A pink synthetic leather glove.\nFor people who think they are tough.",
            item_type: Weapon(
                damage: 5,
                on_hit_effects: []
            ),
        ),
    ]
)
```

看到没？这一长串列表就像是你的购物清单。不管你是想买“怪物糖果”（Food）还是“坚韧手套”（Weapon），只要在单子上写好，游戏引擎自然会帮你搞定。

## 菜篮子：ItemRegistry

当游戏启动时，系统会自动扫描 `shared/items` 文件夹，把所有的 `.item.ron` 文件全部加载进来。

然后，它们会被整理好，整整齐齐地码放在一个全局资源里：`ItemRegistry`。这就是我们的菜篮子。

当你在脚本里写 `<run_event("get_item", "monster_candy")>` 时，系统就会去这个篮子里翻找 ID 为 `monster_candy` 的数据，然后把它塞进你的背包。

## 食材分类学

为了防止把手套当糖果吃了，我们得做好分类。在代码里，我用枚举（Enum）来区分它们：

- **Food**: 能吃的。有回血（`Heal`）效果，还能顺便播个吃东西的声音（`PlayAudio`）。
- **Weapon**: 能打人的。主要属性是伤害（`damage`）。
- **Armor**: 能挨打的。主要属性是防御（`defense`）。

就是这么简单。不需要复杂的继承关系，几个枚举变体就能把这些“食材”安排得明明白白。

好了，菜买齐了，调料也备好了，锅也架起来了。这顿饭，终于快要做好了。

---

企划时间：2025-10-31

开始时间：2026-01-08

完成时间：2026-01-08

关联的 PR：[#26](https://github.com/Bli-AIk/souprune/pull/26)
