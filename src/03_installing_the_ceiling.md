虽说我~~是某种程度上抱着检查器狗都不用的态度~~选择了bevy，但是当你真告诉我 bevy 有检查器库 bevy_inspector_egui 的时候，那咱还是忍不住要折腾一波（

这是为了调试方便！！才不是因为我依赖检查器呢！！你懂个球！！

# 03 - 装天花板

尽管我觉得没有天花板也能做饭，喝汤的也不在意厨房长啥样，更别说有没有天花板...但是来做饭的“大伙儿”大概受不了这个吧。对吧？

那就整吧。我扯的理由够多了。

## 咎由自取

我本来打算做一个带有 dock 的检查器界面，但是这样配置分辨率会超级麻烦……所以我就做了非常省事儿的工作。

顺带把这个文档也省事儿了得了。我全交给 AI 吧。

---AI编写部分---

## 为什么又是检查器

说来奇怪...我本来是一副"检查器狗都不用"的架势来选择 bevy 的，结果现在反而搞起了检查器。这大概就是所谓的真香定律吧？

不过说真的，bevy_inspector_egui 确实是个好东西。特别是在调试状态机的时候——能实时看到组件的变化，比在代码里狂加 `println!` 优雅多了。

## 添加检查器

首先在 `Cargo.toml` 里加上：

```toml
bevy-inspector-egui = "0.35.0"
```

然后新建 `src/extra/inspector.rs`：

```rust
use bevy::app::{App, Plugin};
use bevy_inspector_egui::bevy_egui::EguiPlugin;
use bevy_inspector_egui::quick::WorldInspectorPlugin;

pub struct InspectorPlugin;

impl Plugin for InspectorPlugin {
    fn build(&self, app: &mut App) {
        app.add_plugins((EguiPlugin::default(), WorldInspectorPlugin::new()));
    }
}
```

在 `main.rs` 里添加插件：

```rust
.add_plugins((
    TomlPlugin,
    InputManagerPlugin::<Action>::default(),
    StateMachinePlugin::default(),
    #[cfg(debug_assertions)]
    InspectorPlugin,
))
```

用条件编译只在 debug 模式下启用——毕竟最终发布版本里用户也用不着这个。

## 状态组件重命名

之前的状态组件命名有点问题。`Idle`、`Walking`、`Running` 这些名字太泛用了，容易和其他地方的组件冲突。

所以全部改成了带前缀的：

```rust
#[derive(Clone, Component)]
#[component(storage = "SparseSet")]
pub(crate) struct StateIdle;

#[derive(Clone, Component)]
#[component(storage = "SparseSet")]
pub(crate) struct StateWalking;

#[derive(Clone, Component)]
#[component(storage = "SparseSet")]
pub(crate) struct StateRunning;
```

相应的，所有使用这些组件的地方都要更新：

- `src/core/overworld.rs` - 状态机转换逻辑
- `src/core/overworld/character/systems.rs` - 系统查询
- `src/core/overworld/player.rs` - 动画控制

状态机的配置变成了：

```rust
StateMachine::default()
    .trans::<StateIdle, _>(is_player_walking, StateWalking)
    .trans::<StateWalking, _>(is_player_walking.not(), StateIdle)
    .trans::<StateRunning, _>(is_player_walking.not(), StateIdle)
    .trans::<StateWalking, _>(is_player_running, StateRunning)
    .trans::<StateRunning, _>(is_player_running.not(), StateWalking)
```

## 代码清理

删掉了一些调试用的 `println!`，比如动画系统里的：

```rust
// 删掉了这行
// println!("Debug - Apply animation frames: {}", clip.frame);
```

还移除了一个多余的 `character/readme.md` 文件——项目结构简化了一些。

## 依赖更新

这次加检查器带来了不少新的依赖：

- `bevy-inspector-egui` 和 `bevy-inspector-egui-derive`
- `bevy_egui` - egui 的 bevy 集成
- `arboard` - 剪贴板支持
- 一堆 egui 相关的底层依赖

Cargo.lock 文件膨胀了不少，但这是为了更好的开发体验。

## 调试体验提升

现在在 debug 模式下运行，会弹出一个检查器窗口。可以：

- 查看所有实体和它们的组件
- 实时看状态机的状态变化
- 观察动画播放情况
- 调试各种数值

虽然我嘴上说着"检查器狗都不用"，但身体还是很诚实的。有了这个工具，调试状态机和动画系统方便多了。

特别是能看到状态组件的实时切换，比之前单纯靠控制台输出清晰太多了。

## 小结

这次更新主要是：

1. 添加了 bevy_inspector_egui 检查器
2. 重命名状态组件避免命名冲突  
3. 清理了一些调试代码
4. 提升了整体的开发体验

说是"装天花板"，其实就是给开发环境加了个调试工具。虽然对最终用户没什么直接影响，但对开发者来说确实是质的提升。

真香.jpg

---THX---

反正就是这么个小功能，怎么还跟水论文似的……

大概就是这样了。日后也许还会对检查器进行更新？那就到时候再说……也许那时候你们会发现这个文章多了几段。

---

企划时间：2025-10-23

开始时间：2025-11-2

完成时间：2025-11-2

关联的 PR：[#9](https://github.com/Bli-AIk/souprune/pull/9)
