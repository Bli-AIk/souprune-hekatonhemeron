距离上一次更新有一段日子了――我不是做了不少事么？怎么才到第二篇啊？？

不管怎样，今天的任务简单（存疑）的多。实现 sprite 动画就行了――当然，它的前置要求是资源管理。

我说状态是调料，状态机是瓶，那动画...就贴在状态机上，那就是标签了――反正本来就是全都瞎比喻。无所谓。

动画离不开资源，而我们之前已经把资源的位置都安排好了。但我们还是得避免在代码中硬编码路径，不然东西很容易弄混。

解决这个方案最经典的做法就是引一个中间层，写一个配置文件――把路径做成键值对。但这样也好不优雅哦，改个路径还得同步...？

所以我就顺带写了个小工具 [chaser](https://github.com/Bli-AIk/chaser)... 尽管我还没开始写 souprune 的资源配置文件呢。

不管怎样，这些大概就是准备工作了。我们开干吧。

# 02 - 贴满标签

秉承着能不造轮子就不造轮子的原则，我打算用 [bevy_spritesheet_animation](https://github.com/merwaaan/bevy_spritesheet_animation) 来搞定动画系统。但是捏，这里还有一个小问题——我们的素材使用了 TextureAtlas 打包。

这就有点尴尬咯。这个插件自己搓了个 Spritesheet API。就它这个示例来看，要用一张铺好的图来生成动画——我才懒得搞呢。

但这也不是唯一的解决方案……对吧。至少我还看到了

```rust
let clip = Clip::from_frames([6, 7, 8, 9, 10, 11]);
```

这样的用法……嗯……所以先把配置文件做了吧。我们如果能传路径索引，那也就可以了。

所以，我们来使用 plus 版 `ini`... `toml`.

```toml
[[textures]]
# ---overworld---
# -characters-
name = "flowey_idle"
path = "textures/overworld/characters/flowey/idle"

[[textures]]
name = "flowey_sink"
path = "textures/overworld/characters/flowey/sink"

[[textures]]
name = "frisk_walk_down"
path = "textures/overworld/characters/frisk/walk/down"

[[textures]]
name = "frisk_walk_left"
path = "textures/overworld/characters/frisk/walk/side"

[[textures]]
name = "frisk_walk_right"
path = "textures/overworld/characters/frisk/walk/side"
flip_x = true

[[textures]]
name = "frisk_walk_up"
path = "textures/overworld/characters/frisk/walk/up"

[[textures]]
name = "frisk_run_down"
path = "textures/overworld/characters/frisk/run/down"

[[textures]]
name = "frisk_run_left"
path = "textures/overworld/characters/frisk/run/side"

[[textures]]
name = "frisk_run_right"
path = "textures/overworld/characters/frisk/run/side"
flip_x = true

[[textures]]
name = "frisk_run_up"
path = "textures/overworld/characters/frisk/run/up"

[[textures]]
name = "toriel_walk_down"
path = "textures/overworld/characters/toriel/walk/down"

[[textures]]
name = "toriel_walk_left"
path = "textures/overworld/characters/toriel/walk/left"

[[textures]]
name = "toriel_walk_right"
path = "textures/overworld/characters/toriel/walk/right"
flip_x = true

[[textures]]
name = "toriel_walk_up"
path = "textures/overworld/characters/toriel/walk/up"

# -objects-
# everywhere
[[textures]]
name = "chest_box"
path = "textures/overworld/objects/everywhere/chestbox.png"

[[textures]]
name = "save_point"
path = "textures/overworld/objects/everywhere/savepoint"
```
哦，对的，这里多了个 objects，里面有 chestbox 和 savepoint。chestbox是单图，savepoint是动图。单图使用文件路径，而动图就是文件夹路径。一目了然哦。

之后没准也会拓展这个配置文件，例如让你自己选定帧之类的。但目前的实现就是：单图就是单图，文件夹就是动画。

然后，我们来做动画吧——来跟 `bevy_spritesheet_animation` 对线！

## 对 线 失 败

谁懂扒完所有示例库后，发现这个库的实现完全针对切分单图而不支持不规则纹理图集的救赎感……我真不希望搞出一大堆 draw call.

所以，还是照着[示例](https://github.com/bevyengine/bevy/blob/latest/examples/2d/sprite_animation.rs)自己造轮子吧。

## 别把瓶子整掉地上了啊喂

“坠落”动画状态



---

企划时间：2025-10-23

开始时间：

完成时间：
