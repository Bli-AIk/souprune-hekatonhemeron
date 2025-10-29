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
# Here is the configuration file of the game resources, which defines the paths of various animations and sprites.
# Each resource has a unique name and corresponding file path for easy reference in game code.
# For animation resources, horizontal flipping (flip_x) and vertical flipping (flip_y) are turned off by default.
# And looping is enabled by default.
# These can be set as needed.

# ---overworld---
# --characters--
# -flowey-
[[animations]]
name = "flowey_idle"
path = "textures/overworld/characters/flowey/idle"

[[animations]]
name = "flowey_sink"
path = "textures/overworld/characters/flowey/sink"

# -frisk-
# walk
[[animations]]
name = "frisk_idle_down"
path = "textures/overworld/characters/frisk/walk/down/0"

[[animations]]
name = "frisk_idle_left"
path = "textures/overworld/characters/frisk/walk/side/0"

[[animations]]
name = "frisk_idle_right"
path = "textures/overworld/characters/frisk/walk/side/0"
flip_x = true

[[animations]]
name = "frisk_idle_up"
path = "textures/overworld/characters/frisk/walk/up/0"
# walk
[[animations]]
name = "frisk_walk_down"
path = "textures/overworld/characters/frisk/walk/down"

[[animations]]
name = "frisk_walk_left"
path = "textures/overworld/characters/frisk/walk/side"

[[animations]]
name = "frisk_walk_right"
path = "textures/overworld/characters/frisk/walk/side"
flip_x = true

[[animations]]
name = "frisk_walk_up"
path = "textures/overworld/characters/frisk/walk/up"

# run
[[animations]]
name = "frisk_run_down"
path = "textures/overworld/characters/frisk/run/down"

[[animations]]
name = "frisk_run_left"
path = "textures/overworld/characters/frisk/run/side"

[[animations]]
name = "frisk_run_right"
path = "textures/overworld/characters/frisk/run/side"
flip_x = true

[[animations]]
name = "frisk_run_up"
path = "textures/overworld/characters/frisk/run/up"

# -toriel-
[[animations]]
name = "toriel_walk_down"
path = "textures/overworld/characters/toriel/walk/down"

[[animations]]
name = "toriel_walk_left"
path = "textures/overworld/characters/toriel/walk/left"

[[animations]]
name = "toriel_walk_right"
path = "textures/overworld/characters/toriel/walk/right"
flip_x = true

[[animations]]
name = "toriel_walk_up"
path = "textures/overworld/characters/toriel/walk/up"

# -objects-
# everywhere
[[sprites]]
name = "chest_box"
path = "textures/overworld/objects/everywhere/chestbox.png"

[[animations]]
name = "save_point"
path = "textures/overworld/objects/everywhere/savepoint"
```
哦，对的，这里多了个 objects，里面有 chestbox 和 savepoint。chestbox是单图，savepoint是动图。单图使用文件路径，而动图就是文件夹路径。一目了然哦。

之后没准也会拓展这个配置文件，例如让你自己选定帧之类的。但目前的实现就是：单图就是单图，文件夹就是动画。

然后，我们来做动画吧——来跟 `bevy_spritesheet_animation` 对线！

## 对 线 失 败

谁懂扒完所有示例库后，发现这个库的实现完全针对切分单图而不支持不规则纹理图集的救赎感……我真不希望搞出一大堆 draw call.

所以，还是照着[示例](https://github.com/bevyengine/bevy/blob/latest/examples/2d/sprite_animation.rs)自己造轮子吧。

所以我先把 `create_texture_atlas` 方法微调了一下——加入了一个 `HashMap` 来存储贴图的路径索引。

```rust
   let mut index_map = HashMap::new();
   let mut added_count = 0;

    for handle in folder.handles.iter() {
        if let Some(path) = handle.path() {
            let path_str = path.to_string();
            if !path_str.ends_with(".png")
                && !path_str.ends_with(".jpg")
                && !path_str.ends_with(".jpeg")
            {
                continue;
            }
            index_map.insert(path_str, added_count);
            added_count += 1;
        }
```

之后返回的就是

```rust
    (
        texture_atlas_layout,
        texture_atlas_sources,
        texture,
        index_map,
    )
```

而我们原本的 sprite handle 就自然而然的改为了 index...

```rust
 let loaded_folder = loaded_folders.get(&rpg_sprite_handles.0).unwrap();

    let (texture_atlas_nearest, _nearest_sources, nearest_texture, index_map) =
        create_texture_atlas(
            loaded_folder,
            None,
            Some(ImageSampler::nearest()),
            &mut textures,
        );
    println!("index_map: {:#?}", &index_map);

    let atlas_nearest_handle = texture_atlases.add(texture_atlas_nearest);

    let frisk_index = *index_map
        .get("textures/overworld/characters/frisk/walk/down/1.png")
        .expect("Frisk sprite not found in atlas");

    println!("index: {:#?}", &frisk_index);
    let sprite = Sprite::from_atlas_image(
        nearest_texture,
        TextureAtlas {
            layout: atlas_nearest_handle.clone(),
            index: frisk_index,
        },
    );
```

这样性能总比扒一遍路径强。

话虽如此，咱这时候又看到 `OverWorldCharacterSpriteFolder` —— 它只针对 `overworld` 资源文件夹。

我们先考虑一下未来复杂的情况吧。于是我额外创建了 `battle` 文件夹。

```rust
assets
├── config.toml
└── textures
    ├── battle
    │   └── ui
    │       ├── act
    │       │   ├── 0.png
    │       │   └── 1.png
    │       ├── fight
    │       │   ├── 0.png
    │       │   └── 1.png
    │       ├── item
    │       │   ├── 0.png
    │       │   └── 1.png
    │       └── mercy
    │           ├── 0.png
    │           └── 1.png
    └── overworld
        ├── characters
        │   ├── flowey
        │   │   ├── idle
        │   │   │   ├── 0.png
        │   │   │   └── 1.png
        │   │   └── sink
        │   │       ├── 0.png
        │   │       ├── 1.png
        │   │       ├── 2.png
        │   │       ├── 3.png
        │   │       ├── 4.png
        │   │       └── 5.png
        │   ├── frisk
        │   │   ├── run
        │   │   │   ├── down
        │   │   │   │   ├── 1.png
        │   │   │   │   ├── 2.png
        │   │   │   │   ├── 3.png
        │   │   │   │   ├── 4.png
        │   │   │   │   ├── 5.png
        │   │   │   │   └── 6.png
        │   │   │   ├── readme.md
        │   │   │   ├── side
        │   │   │   │   ├── 1.png
        │   │   │   │   ├── 2.png
        │   │   │   │   ├── 3.png
        │   │   │   │   ├── 4.png
        │   │   │   │   ├── 5.png
        │   │   │   │   └── 6.png
        │   │   │   └── up
        │   │   │       ├── 1.png
        │   │   │       ├── 2.png
        │   │   │       ├── 3.png
        │   │   │       ├── 4.png
        │   │   │       ├── 5.png
        │   │   │       └── 6.png
        │   │   └── walk
        │   │       ├── down
        │   │       │   ├── 1.png
        │   │       │   ├── 2.png
        │   │       │   ├── 3.png
        │   │       │   └── 4.png
        │   │       ├── side
        │   │       │   ├── 1.png
        │   │       │   └── 2.png
        │   │       └── up
        │   │           ├── 1.png
        │   │           ├── 2.png
        │   │           ├── 3.png
        │   │           └── 4.png
        │   └── toriel
        │       └── walk
        │           ├── down
        │           │   ├── 0.png
        │           │   ├── 1.png
        │           │   ├── 2.png
        │           │   ├── 3.png
        │           │   └── talk
        │           │       ├── 0.png
        │           │       └── 1.png
        │           ├── side
        │           │   ├── 0.png
        │           │   ├── 1.png
        │           │   ├── 2.png
        │           │   ├── 3.png
        │           │   └── talk
        │           │       ├── 0.png
        │           │       └── 1.png
        │           └── up
        │               ├── 0.png
        │               ├── 1.png
        │               ├── 2.png
        │               └── 3.png
        └── objects
            └── everywhere
                ├── chestbox.png
                └── savepoint
                    ├── 0.png
                    └── 1.png
```

目前放的都是 `undertale` 的贴图。之后还得把 `deltarune` 考虑上……到时候再说（



## 别把瓶子整掉地上了啊喂

“坠落”动画状态



---

企划时间：2025-10-23

开始时间：

完成时间：
