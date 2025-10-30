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

## 我玩大了

我怎么解释目前的情况呢…… 我这两天写的有点不知天地为何物了，所以一直没寻思写这个记录文档。所以我先拿个 AI 补一下：

---AI内容---

呀，这下真的**玩大了**！让我整理一下这次分支里到底都干了啥...

## 🎭 动画系统大改造

首先说最核心的——我彻底重写了动画系统！之前那个简陋的资源管理现在变成了一个功能完整的sprite动画框架：

### 🏗️ 架构重构
- **模块化重组**：把原来混在一起的代码拆分成了 `animation/`、`sprite/`、`overworld/` 等独立模块
- **状态机驱动**：用状态机来管理角色的 `Idle`、`Walking`、`Running` 状态
- **插件架构**：每个功能都做成了Bevy插件，代码结构清爽多了

### 🎬 动画引擎从零开始
放弃了第三方库，手搓了一套动画系统：
- `SpriteAnimationClip` - 动画片段管理
- `SpriteAnimationTimer` - 基于时间的帧切换
- `SpriteAnimationCurrentFrame` - 当前帧追踪
- 支持循环/非循环动画，方向切换立即生效

### 🗂️ 配置文件系统
从YAML迁移到TOML，建立了完整的资源配置体系：
```toml
[[animations]]
name = "frisk_walk_down"
path = "textures/overworld/characters/frisk/walk/down"

[[animations]]
name = "frisk_walk_right"
path = "textures/overworld/characters/frisk/walk/side"
flip_x = true
```

### 🏃‍♀️ 玩家控制系统
- **宏生成系统**：用宏批量生成不同状态的动画控制函数
- **方向映射**：8方向输入映射到4方向动画（对角线归并）
- **状态转换**：走路→跑步→空闲的流畅切换

### 🎯 性能优化
- **纹理图集缓存**：避免重复注册相同路径的资源
- **路径索引映射**：用HashMap缓存文件路径到索引的映射
- **即时更新**：动画切换时立即应用新sprite，消除延迟感

## 🐛 关键Bug修复

最重要的是解决了动画切换延迟问题：
- **计时器重置**：切换动画时正确重置 `SpriteAnimationTimer`
- **立即渲染**：不等下一帧，直接应用新动画的第一帧
- **状态同步**：确保输入、状态机、动画系统在同一帧内同步

## 📁 资源结构重整

建立了标准化的资源目录结构：
```
assets/textures/
├── battle/ui/        # 战斗UI按钮
│   ├── act/
│   ├── fight/
│   ├── item/
│   └── mercy/
└── overworld/        # 大地图资源
    ├── characters/   # 角色动画
    └── objects/      # 场景物件
```

## 🛠️ 开发工具链

- 添加了TOML资产加载器
- 实现了单文件/文件夹动画的自动识别
- 支持sprite翻转、循环等配置
- 完善的错误处理和调试信息

说实话，这次重构的幅度有点**过分**了...74个文件改动，新增1311行代码，删除243行。基本上把整个核心系统都推倒重来了🤯

但效果确实不错！现在按键响应超级灵敏，动画切换丝滑，而且为后续的战斗系统、UI系统都打好了基础。

总结就是：**从能跑到好用的质的飞跃！** 🚀

## 🕐 开发时间线 - Git提交逐条分析

让我按时间顺序梳理一下这次开发的具体过程，每个提交都有它的故事：

### 📚 文档与依赖阶段
**`dc6e285`** - `docs: add Undertale-Changer-Template to dependencies table`  
开始之前先把依赖表更新了，把 Undertale-Changer-Template 加进去。总得让人知道我们用了什么轮子嘛。

### 🎬 动画系统雏形
**`33baa48`** - `feat(core): add animation system and asset configuration`  
第一次尝试建立动画系统！这时候还比较简陋，但是基础框架开始有了。

**`4a27456`** - `feat(assets): migrate config from YAML to TOML format`  
YAML太丑了！果断换TOML，可读性瞬间提升。这时候开始意识到配置文件的重要性。

**`d2d1c3f`** - `refactor(core): rename resource module to sprite and extract ResolutionScale`  
模块重命名！`resource` → `sprite`，语义更清晰。同时把分辨率缩放逻辑独立出来。

### 🏃‍♀️ 角色系统建立
**`83e7cfa`** - `feat(core): implement player character animation system`  
玩家角色动画系统正式上线！这是第一个能看到动画效果的版本。

**`cfb3ca0`** - `feat(config): migrate texture definitions to animation system`  
把纹理定义迁移到动画系统中，开始建立完整的资源管理体系。

**`6e2fe0a`** - `feat(ui): add battle UI button textures`  
添加战斗UI按钮！虽然还没有战斗系统，但是资源先准备上了。前瞻性规划👍

### 🏗️ 架构重构期
**`9cbc8a0`** - `feat(ui): restructure sprite system and add TOML configuration`  
大重构开始！sprite系统重新设计，TOML配置系统完善。

**`bd4caf3`** - `refactor(core): rename core modules to components and systems`  
ECS架构优化，模块命名更加符合Bevy惯例。`components` 和 `systems` 分离。

**`97ab103`** - `feat(assets): add TOML asset loader and serde dependency`  
自定义TOML资产加载器！现在可以直接加载配置文件作为Bevy资产了。

### 🔧 代码重构与优化
**`fa28d43`** - `refactor(core): extract sprite creation logic and reorganize imports`  
代码整理，sprite创建逻辑独立，import重新组织。代码洁癖发作🤔

**`04b5856`** - `refactor(core): extract sprite loading logic and move character state functions`  
继续拆分！sprite加载逻辑独立，角色状态函数移动到合适位置。

**`0ad3af3`** - `feat(core): refactor sprite system and character components`  
sprite系统和角色组件大重构。这时候ECS架构基本成型。

**`e100065`** - `refactor(core): rename CharacterBundle to PlayerBundle and enhance components`  
语义优化！`CharacterBundle` → `PlayerBundle`，更明确这是玩家专用的。

### 🎭 动画系统成熟期
**`b3eedbc`** - `refactor(core): restructure animation components and systems`  
动画组件和系统重新设计。这是动画系统走向成熟的关键一步。

**`821eb40`** - `feat(animation): implement sprite animation system with plugin architecture`  
插件架构！动画系统变成了标准的Bevy插件，可复用性大大提升。

**`a7a896f`** - `refactor(sprite): sort animation frames by filename and update frame logging`  
细节优化：动画帧按文件名排序，调试日志完善。强迫症患者的胜利✨

**`9b61705`** - `refactor(core): update animation debug message and sprite name`  
继续完善调试信息，开发体验提升。

**`b4a5718`** - `refactor(sprite): support single file animations and update character sprites`  
支持单文件动画！现在既可以用文件夹做动画，也可以用单个文件。灵活性++

### 🎯 动画系统完善
**`1ee09df`** - `feat(anim): update Frisk character animations and sprite configuration`  
Frisk角色动画更新，配置完善。可以看到走路、跑步动画了！

**`f10990a`** - `feat(animation): refactor sprite animation system with timer-based updates`  
基于时间的动画系统！不再依赖帧率，动画播放更加稳定。

**`d449c12`** - `refactor(animation): improve code formatting and structure`  
代码格式化和结构优化。看起来舒服多了。

### 🎮 玩家控制优化
**`8a27d16`** - `refactor(overworld): extract player logic into separate module`  
玩家逻辑独立成模块！代码组织更清晰，后续维护更容易。

**`9502a6c`** - `fix(player): optimize animation system to prevent redundant clip changes`  
性能优化！避免冗余的动画切换，CPU开心了。

**`01310bd`** - `refactor(animation): add dead code allowances and macro-based animation system`  
宏系统登场！用宏批量生成动画控制函数，代码量大减。

### 🚀 性能与体验优化
**`2048a80`** - `feat(core): add texture atlas caching and duplicate registration prevention`  
纹理图集缓存！避免重复注册，内存使用更高效。

**`1ede321`** - `feat(animation): add reset method and immediate sprite update`  
**就是我们刚才修复的那个！**动画切换延迟问题终于解决，按键响应变得超级灵敏！🎉

---

整个开发过程可以看出是一个**持续迭代优化**的过程：从最初的简单实现，到架构重构，再到性能优化，最后到用户体验的精雕细琢。每一步都在让系统变得更好用、更稳定、更优雅。

---AI内容结束，辛苦啦AI大人---

## 别把瓶子整掉地上了啊喂

“坠落”动画状态



---

企划时间：2025-10-23

开始时间：2025-10-27

完成时间：
