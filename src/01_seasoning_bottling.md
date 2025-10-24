回想起当年刚学 Unity 的时候，我抱着还没消化完的 C# 基础[^1]，看了那个至今我难以忘记的 **Ruby' Adventure[^2]** 教程。从那时起，我想我可能就明白了一些事情，一些影响我至深的事情，一些我至今也无法改变的事情――

比方说，做一个游戏的第一步是把角色状态机搓出来。

# 01 - 调料装瓶

嘛，所以说角色是调料。我没想太多需要打什么比方，反正就是说角色是调料了。路边俩npc可能是什么孜然粉胡椒粉啥的，主要角色那就油盐酱醋葱姜蒜？而主人公是什么呢...反正就搁里面随便找一个好了。

调料是做饭必不可少的一环。虽然我们是要装修厨房，但最后咋的也是要做饭的。上来就修地板砌墙也不是很有意思，对吧？再加上以前我都是先做调料，那调料大概味道也不错！熟悉，并且很有味道...往往是刺激性的。别弄大劲儿了就行。

所以我们的第一步就是搓角色状态机，让主角能站立、走起来、跑起来――这就是“状态”。很好，今儿的目标有了。

## 瓶子是现成的（？，调料是现成的

感谢 undertale 闭源――但是可以满大街找到的 toby 也懒得管的拆包，那也算是开源了。所以说，素材什么的都是现成的。之前做 UCT 存的素材还都能用呢，那可太好啦――我们不就有现成的调料了么？

流体要看容器来塑形，对于调料这种稀碎的东西来说也是这样吧...你们家里都拿什么瓶子或者小碗装调料哦？我家好像都是拿原包装，或者就是装小碗。但我们的厨房里面没有装好调料的瓶子――但现成的瓶子是有的――或者说现成的瓶子是可以现拼的。那还算是现成的吗？

上一次装调料是咋装的呢――哦不，我在 UCT 的时候，最开始的代码压根没有把调料装瓶里，用的时候都是抓一把就用了，太不讲究了。

但后来也做了些重构吧。我把很多状态写了出来――静止，走动，跑动...没有问题...然后，“坠落”？

我记得我是写了个“坠落”状态。虽然本质上来说，“坠落”就是在静止的基础上加了个动画。但很可惜，我压根没把动画和状态本身分离。想象一下之后如果要做什么剧情过场，有很多个动画，一会儿打伞一会儿被电的，那如果状态和动画一样多，那可就太哈人了。“静止”状态下仍然应该可以放任何动画，其他状态的也一样――动画和状态必须分离！但特么我当初就没这么干。谁知道我当初是咋想的呢？反正，这都是我们之后要规避的问题。

总而言之，大概做状态机不算难事，状态机就是...状态，没什么好说的。倒也有一个难点――我这还是第一次正儿八经用 bevy――但不是第一次正儿八经学rust、看ECS，也不是正儿八经第一次用游戏引擎，做游戏开发！唯一“第一次”的就是拿 rust 在 bevy 用 ECS 开发 游戏... 多看看示例就好啦！大概吧。


## 不得不先把桌子摆上。。。

以往我都是在 OOP 下写状态机。思考了一会儿，然后我发现好像状态机在 ECS 下也有点新鲜嘛！毕竟从 OOP 转向 数据驱动，那思路还是有点区别的――但对于状态机来说，数据驱动明显更舒服一些。

但是，我们还是需要先把调料本身准备好——咱现在不是在 Unity 里面了，不是把贴图拖进去就能用啦。Bevy里面的素材管理也是门学问。

不过，我不想花太多功夫在管理素材上——我是做游戏的还是搞贴图的啊喂？所以说，现在先一切从简，我们参考 [texture_atlas](https://github.com/bevyengine/bevy/blob/main/examples/2d/texture_atlas.rs) 教程来生成图集。

首先，这个实例里面定义了default_nearest。我们是像素游戏所以——没啥好说的。

嗯，然后有一个 AppState。我喜欢叫它“大状态机”。Bevy 里面应该没有 Unity 式的（令人诟病的）场景系统。所以我们用一个更好配置的“大状态机”。这是一点毛病没有的。

```rust
#[derive(Debug, Clone, Copy, Default, PartialEq, Eq, Hash, States)]
enum AppState {
    #[default]
    Setup,
    Finished,
}
```

看没，我就说做游戏就是一会儿这个一会儿那个。我们需要做状态机——我是说，给角色的“小状态机”，但是不得不先把“大状态机”处理好。之后这样的事儿多着呢，哈哈——比方说接下来我们要处理素材……服了。我本来寻思这章的标题就叫“先把调料本身准备好！”呢，合着还得先把桌子摆了？……这句不错，新标题 get。


示例里面定义个了`RpgSpriteFolder`。来保存精灵文件夹的句柄。我们做同样的事情，但是起名为`OverWorldCharacterSpriteFolder`。

```rust
#[derive(Resource, Default)]
struct OverWorldCharacterSpriteFolder(Handle<LoadedFolder>);

fn load_textures_system(mut commands: Commands, asset_server: Res<AssetServer>) {
    commands.insert_resource(OverWorldCharacterSpriteFolder(
        asset_server.load_folder("textures/overworld/characters"),
    ));
}
```
名字很长，但我的显示器放得下，并且一目了然—— OverWorld 下的角色 Sprite 文件夹。那就没问题。

这里，我把读取的路径设为了 `textures/overworld/characters`。

哦对了，现在的 `main` 方法长这样：
```rust
fn main() {
    App::new()
        .add_plugins(DefaultPlugins.set(ImagePlugin::default_nearest()))
        .init_state::<AppState>()
        .add_systems(OnEnter(AppState::Setup), load_textures_system)
        .run();
}
```

好，弄到这里就很不错了。休息一下。休息的方式就是先不敲码了，先把贴图给处理好。

直接从 UCT 搬来，把名字都改好——结构如下：

```
assets
└── textures
    └── overworld
        └── characters
            ├── flowey
            │   ├── idle
            │   │   ├── flowey-idle-0.png
            │   │   └── flowey-idle-1.png
            │   └── sink
            │       ├── flowey-sink-0.png
            │       ├── flowey-sink-1.png
            │       ├── flowey-sink-2.png
            │       ├── flowey-sink-3.png
            │       ├── flowey-sink-4.png
            │       └── flowey-sink-5.png
            ├── frisk
            │   ├── run
            │   │   ├── frisk-run-down-1.png
            │   │   ├── frisk-run-down-2.png
            │   │   ├── frisk-run-down-3.png
            │   │   ├── frisk-run-down-4.png
            │   │   ├── frisk-run-down-5.png
            │   │   ├── frisk-run-down-6.png
            │   │   ├── frisk-run-left-1.png
            │   │   ├── frisk-run-left-2.png
            │   │   ├── frisk-run-left-3.png
            │   │   ├── frisk-run-left-4.png
            │   │   ├── frisk-run-left-5.png
            │   │   ├── frisk-run-left-6.png
            │   │   ├── frisk-run-right-1.png
            │   │   ├── frisk-run-right-2.png
            │   │   ├── frisk-run-right-3.png
            │   │   ├── frisk-run-right-4.png
            │   │   ├── frisk-run-right-5.png
            │   │   ├── frisk-run-right-6.png
            │   │   ├── frisk-run-up-1.png
            │   │   ├── frisk-run-up-2.png
            │   │   ├── frisk-run-up-3.png
            │   │   ├── frisk-run-up-4.png
            │   │   ├── frisk-run-up-5.png
            │   │   ├── frisk-run-up-6.png
            │   │   └── readme.md
            │   └── walk
            │       ├── frisk-walk-down-1.png
            │       ├── frisk-walk-down-2.png
            │       ├── frisk-walk-down-3.png
            │       ├── frisk-walk-down-4.png
            │       ├── frisk-walk-left-1.png
            │       ├── frisk-walk-left-2.png
            │       ├── frisk-walk-right-1.png
            │       ├── frisk-walk-right-2.png
            │       ├── frisk-walk-up-1.png
            │       ├── frisk-walk-up-2.png
            │       ├── frisk-walk-up-3.png
            │       └── frisk-walk-up-4.png
            └── toriel
                └── walk
                    ├── toriel-walk-down-0.png
                    ├── toriel-walk-down-1.png
                    ├── toriel-walk-down-2.png
                    ├── toriel-walk-down-3.png
                    ├── toriel-walk-down-talk-0.png
                    ├── toriel-walk-down-talk-1.png
                    ├── toriel-walk-left-0.png
                    ├── toriel-walk-left-1.png
                    ├── toriel-walk-left-2.png
                    ├── toriel-walk-left-3.png
                    ├── toriel-walk-left-talk-0.png
                    ├── toriel-walk-left-talk-1.png
                    ├── toriel-walk-up-0.png
                    ├── toriel-walk-up-1.png
                    ├── toriel-walk-up-2.png
                    └── toriel-walk-up-3.png
```

虽然我们主要需要弄的是 Frisk 的状态机，但 NPC 的也可以备一份为先。

然后是 `check_textures_system`，用于“大状态机”的状态转换……直接抄了。
```rust
fn check_textures_system(
    mut next_state: ResMut<NextState<AppState>>,
    rpg_sprite_folder: Res<OverWorldCharacterSpriteFolder>,
    mut events: MessageReader<AssetEvent<LoadedFolder>>,
) {
    for event in events.read() {
        if event.is_loaded_with_dependencies(&rpg_sprite_folder.0) {
            next_state.set(AppState::Finished);
        }
    }
}
```

相应的 `main` ...
```rust
fn main() {
    App::new()
        .add_plugins(DefaultPlugins.set(ImagePlugin::default_nearest()))
        .init_state::<AppState>()
        .add_systems(OnEnter(AppState::Setup), load_textures_system)
        .add_systems(
            Update,
            check_textures_system.run_if(in_state(AppState::Setup)),
        )
        .run();
}
```

之后，资源加载部分，我们暂时只要加载一张贴图就行了。

```rust


fn setup_system(
    mut commands: Commands,
    rpg_sprite_handles: Res<OverWorldCharacterSpriteFolder>,
    asset_server: Res<AssetServer>,
    mut texture_atlases: ResMut<Assets<TextureAtlasLayout>>,
    loaded_folders: Res<Assets<LoadedFolder>>,
    mut textures: ResMut<Assets<Image>>,
) {
    commands.spawn(Camera2d);

    let loaded_folder = loaded_folders.get(&rpg_sprite_handles.0).unwrap();

    let (texture_atlas_nearest, nearest_sources, nearest_texture) = create_texture_atlas(
        loaded_folder,
        None,
        Some(ImageSampler::nearest()),
        &mut textures,
    );
    let atlas_nearest_handle = texture_atlases.add(texture_atlas_nearest);

    let frisk_handle: Handle<Image> = asset_server
        .get_handle("textures/overworld/characters/frisk/walk/frisk-walk-down-1.png")
        .unwrap();

    create_sprite_from_atlas(
        &mut commands,
        (0.0, 0.0, 0.0),
        nearest_texture,
        nearest_sources,
        atlas_nearest_handle,
        &frisk_handle,
    );
}

fn create_texture_atlas(
    folder: &LoadedFolder,
    padding: Option<UVec2>,
    sampling: Option<ImageSampler>,
    textures: &mut ResMut<Assets<Image>>,
) -> (TextureAtlasLayout, TextureAtlasSources, Handle<Image>) {
    let mut texture_atlas_builder = TextureAtlasBuilder::default();
    texture_atlas_builder.padding(padding.unwrap_or_default());

    for handle in folder.handles.iter() {
        let id = handle.id().typed_unchecked::<Image>();
        let Some(texture) = textures.get(id) else {
            warn!(
                "{} did not resolve to an `Image` asset.",
                handle.path().unwrap()
            );
            continue;
        };

        texture_atlas_builder.add_texture(Some(id), texture);
    }

    let (texture_atlas_layout, texture_atlas_sources, texture) =
        texture_atlas_builder.build().unwrap();
    let texture = textures.add(texture);

    let image = textures.get_mut(&texture).unwrap();
    image.sampler = sampling.unwrap_or_default();

    (texture_atlas_layout, texture_atlas_sources, texture)
}

fn create_sprite_from_atlas(
    commands: &mut Commands,
    translation: (f32, f32, f32),
    atlas_texture: Handle<Image>,
    atlas_sources: TextureAtlasSources,
    atlas_handle: Handle<TextureAtlasLayout>,
    vendor_handle: &Handle<Image>,
) {
    commands.spawn((
        Transform {
            translation: Vec3::new(translation.0, translation.1, translation.2),
            ..default()
        },
        Sprite::from_atlas_image(
            atlas_texture,
            atlas_sources.handle(atlas_handle, vendor_handle).unwrap(),
        ),
    ));
}

```
对应的 `main` 就是：

```rust
fn main() {
    App::new()
        .add_plugins(DefaultPlugins.set(ImagePlugin::default_nearest()))
        .init_state::<AppState>()
        .add_systems(OnEnter(AppState::Setup), load_textures_system)
        .add_systems(
            Update,
            check_textures_system.run_if(in_state(AppState::Setup)),
        )
        .add_systems(OnEnter(AppState::Finished), setup_system)
        .run();
}

```

好吧，按理来说我们这不会出任何问题，但刚刚你可能注意到我资源里面有一个 `readme.md` 在 Frisk 的 run 状态的贴图文件夹内——因为这部分贴图不是 undertale 原作中的，并且参考了UTY，所以我需要说明一下。咱肯定是要留着它的。

...啊，这玩意儿可折腾死我了。

```
ERROR bevy_asset::server: Failed to load folder. Could not find an asset loader matching: Loader Name: None; Asset Type: None; Extension: None; Path: Some("textures/overworld/characters/frisk/run/readme.md");
```
资源加载器不干了。没人管 markdown 文件的加载。虽然把那个 readme.md 删了就能解决问题，但是……太不优雅了！

所以我们就写一个 markdown 的资源加载器吧[^3]，尽管我会让它什么都不干——也许将来哪天也能用到。

```rust
use bevy::asset::io::Reader;
use bevy::asset::{AssetLoader, LoadContext};
use bevy::prelude::*;
use bevy::tasks::ConditionalSendFuture;
pub struct MarkdownPlugin;

impl Plugin for MarkdownPlugin {
    fn build(&self, app: &mut App) {
        app.init_asset::<MarkdownAsset>()
            .init_asset_loader::<MarkdownAssetLoader>();
    }
}
#[derive(Asset, TypePath, Debug)]
pub struct MarkdownAsset;
#[derive(Default)]
pub struct MarkdownAssetLoader;

impl AssetLoader for MarkdownAssetLoader {
    type Asset = MarkdownAsset;
    type Settings = ();
    type Error = std::io::Error;

    fn load<>(
        &self,
        _reader: &mut dyn Reader,
        _settings: &Self::Settings,
        load_context: &mut LoadContext<'_>,
    ) -> impl ConditionalSendFuture<Output = std::result::Result<<Self as AssetLoader>::Asset, <Self as AssetLoader>::Error>> {
        info!(
            "Successfully 'loaded' (ignored) markdown file: {:?}",
            load_context.path()
        );

        Box::pin(async move { Ok(MarkdownAsset) })
    }

    fn extensions(&self) -> &[&str] {
        &["md"]
    }
}

```

边看文档边敲，再加上点 vibe coding，反正现在可以运行了，Sprite也可以显示，但是还有一个小报错：
```
ERROR bevy_image::texture_atlas_builder: Error converting texture from 'Rgba16Float' to 'Rgba8UnormSrgb', ignoring

```

所以说，markdown文件被加载后，还是被当成图片处理了——那大概还要加一层过滤。

```rust

fn create_texture_atlas(
    folder: &LoadedFolder,
    padding: Option<UVec2>,
    sampling: Option<ImageSampler>,
    textures: &mut ResMut<Assets<Image>>,
) -> (TextureAtlasLayout, TextureAtlasSources, Handle<Image>) {
    let mut texture_atlas_builder = TextureAtlasBuilder::default();
    texture_atlas_builder.padding(padding.unwrap_or_default());
    // 我搁这儿加了过滤
    for handle in folder.handles.iter() {
        if let Some(path) = handle.path() {
            let path_str = path.to_string();
            if !path_str.ends_with(".png") && !path_str.ends_with(".jpg") && !path_str.ends_with(".jpeg") {
                continue;
            }
        }
        
        let id = handle.id().typed_unchecked::<Image>();
        let Some(texture) = textures.get(id) else {
            warn!(
                "{} did not resolve to an `Image` asset.",
                handle.path().unwrap()
            );
            continue;
        };

        texture_atlas_builder.add_texture(Some(id), texture);
    }

    let (texture_atlas_layout, texture_atlas_sources, texture) =
        texture_atlas_builder.build().unwrap();
    let texture = textures.add(texture);

    let image = textures.get_mut(&texture).unwrap();
    image.sampler = sampling.unwrap_or_default();

    (texture_atlas_layout, texture_atlas_sources, texture)
}
```

搞定了。现在一切都运行良好……但我不是要做状态机来着吗？！所以，先把这些玩意儿封装掉……别干扰我的思路为好。

以及，OOP还在追我——我现在即使是写 rust 项目，也希望尽可能的让每个文件都分开一些——单一职责也好——条理清晰也罢——你们这些 C# 遗老对类还真是有严重的依赖.png

重构完后大概就是这样：

```rust
// main.rs
mod core;
mod markdown_asset_loader;

use crate::core::resource_plugin::*;
use crate::markdown_asset_loader::MarkdownPlugin;
use bevy::asset::LoadedFolder;
use bevy::image::ImageSampler;
use bevy::prelude::*;

fn main() {
    App::new()
        .add_plugins(DefaultPlugins.set(ImagePlugin::default_nearest()))
        .add_plugins(MarkdownPlugin)
        .init_state::<AppState>()
        .add_plugins(ResourcePlugin)
        .add_systems(OnEnter(AppState::Finished), setup_system)
        .run();
}

#[derive(Debug, Clone, Copy, Default, PartialEq, Eq, Hash, States)]
enum AppState {
    #[default]
    Setup,
    Finished,
}

fn setup_system(
    mut commands: Commands,
    rpg_sprite_handles: Res<OverWorldCharacterSpriteFolder>,
    asset_server: Res<AssetServer>,
    mut texture_atlases: ResMut<Assets<TextureAtlasLayout>>,
    loaded_folders: Res<Assets<LoadedFolder>>,
    mut textures: ResMut<Assets<Image>>,
) {
    commands.spawn(Camera2d);

    let loaded_folder = loaded_folders.get(&rpg_sprite_handles.0).unwrap();

    let (texture_atlas_nearest, nearest_sources, nearest_texture) = create_texture_atlas(
        loaded_folder,
        None,
        Some(ImageSampler::nearest()),
        &mut textures,
    );
    let atlas_nearest_handle = texture_atlases.add(texture_atlas_nearest);

    let frisk_handle: Handle<Image> = asset_server
        .get_handle("textures/overworld/characters/frisk/walk/frisk-walk-down-1.png")
        .unwrap();

    create_sprite_from_atlas(
        &mut commands,
        (0.0, 0.0, 0.0),
        nearest_texture,
        nearest_sources,
        atlas_nearest_handle,
        &frisk_handle,
    );
}

fn create_texture_atlas(
    folder: &LoadedFolder,
    padding: Option<UVec2>,
    sampling: Option<ImageSampler>,
    textures: &mut ResMut<Assets<Image>>,
) -> (TextureAtlasLayout, TextureAtlasSources, Handle<Image>) {
    let mut texture_atlas_builder = TextureAtlasBuilder::default();
    texture_atlas_builder.padding(padding.unwrap_or_default());

    for handle in folder.handles.iter() {
        if let Some(path) = handle.path() {
            let path_str = path.to_string();
            if !path_str.ends_with(".png")
                && !path_str.ends_with(".jpg")
                && !path_str.ends_with(".jpeg")
            {
                continue;
            }
        }

        let id = handle.id().typed_unchecked::<Image>();
        let Some(texture) = textures.get(id) else {
            warn!(
                "{} did not resolve to an `Image` asset.",
                handle.path().unwrap()
            );
            continue;
        };

        texture_atlas_builder.add_texture(Some(id), texture);
    }

    let (texture_atlas_layout, texture_atlas_sources, texture) =
        texture_atlas_builder.build().unwrap();
    let texture = textures.add(texture);

    let image = textures.get_mut(&texture).unwrap();
    image.sampler = sampling.unwrap_or_default();

    (texture_atlas_layout, texture_atlas_sources, texture)
}

fn create_sprite_from_atlas(
    commands: &mut Commands,
    translation: (f32, f32, f32),
    atlas_texture: Handle<Image>,
    atlas_sources: TextureAtlasSources,
    atlas_handle: Handle<TextureAtlasLayout>,
    vendor_handle: &Handle<Image>,
) {
    commands.spawn((
        Transform {
            translation: Vec3::new(translation.0, translation.1, translation.2),
            ..default()
        },
        Sprite::from_atlas_image(
            atlas_texture,
            atlas_sources.handle(atlas_handle, vendor_handle).unwrap(),
        ),
    ));
}

```

```rust
use crate::AppState;
use bevy::app::{App, Plugin, Update};
use bevy::asset::LoadedFolder;
use bevy::prelude::*;

#[derive(Resource, Default)]
pub(crate) struct OverWorldCharacterSpriteFolder(pub(crate) Handle<LoadedFolder>);

pub(crate) struct ResourcePlugin;

impl Plugin for ResourcePlugin {
    fn build(&self, app: &mut App) {
        app.add_systems(OnEnter(AppState::Setup), load_textures_system)
            .add_systems(
                Update,
                check_textures_system.run_if(in_state(AppState::Setup)),
            );
    }
}

fn load_textures_system(mut commands: Commands, asset_server: Res<AssetServer>) {
    commands.insert_resource(OverWorldCharacterSpriteFolder(
        asset_server.load_folder("textures/overworld/characters"),
    ));
}

fn check_textures_system(
    mut next_state: ResMut<NextState<AppState>>,
    rpg_sprite_folder: Res<OverWorldCharacterSpriteFolder>,
    mut events: MessageReader<AssetEvent<LoadedFolder>>,
) {
    for event in events.read() {
        if event.is_loaded_with_dependencies(&rpg_sprite_folder.0) {
            next_state.set(AppState::Finished);
        }
    }
}

```

```
src
├── core
│   └── resource_plugin.rs
├── core.rs
├── main.rs
└── markdown_asset_loader.rs
```

大概如此……舒服点了。

最后，我们在真去弄调料之前，还要考虑一件事——AppState。我们要针对dr/ut来设计：Overworld, Battle, Menu。大概是这仨，之后还可以再细分，给配一个子State啥的……（例如Overworld下的自由行动/追逐战/剧情、Battle下的我方回合/敌方回合...这都是后话了）

```rust
#[derive(Debug, Clone, Copy, Default, PartialEq, Eq, Hash, States)]
enum AppState {
    #[default]
    Setup,
    Menu,
    Overworld,
    Battle,
}
```

有时候一想到我做的东西的复杂度，和原作那一堆用 `toby 码 gml 程度的能力` 生堆出来的东西的区别，我就感到忍俊不禁。

## 把调料本身准备好！

好吧，我刚刚就不该一个个给代码列出来，因为要是一重构那我光是列就得列半天—— ~~你们不会自己看git提交记录吗（~~

所以，我在刚刚的基础上加了点细节——把 Overworld 的 setup 改为了个组件；把角色相关的状态机代码准备好；把摄像机也给调好……顺带进行了一个~~小~~重构。大概是这样吧。

```
src
├── core
│   ├── core_components.rs
│   ├── overworld
│   │   ├── character
│   │   │   ├── character_system.rs
│   │   │   ├── readme.md
│   │   │   ├── states
│   │   │   │   └── state_components.rs
│   │   │   └── states.rs
│   │   ├── character.rs
│   │   └── readme.md
│   ├── overworld.rs
│   ├── resource.rs
│   └── setup.rs
├── core.rs
├── extra
│   └── markdown_asset_loader.rs
├── extra.rs
└── main.rs
```

还是不废话我都做了啥吧，有需要就去扒 git 历史记录[^4]。

不管怎样，状态已经有了：

```rust
#[derive(Component)]
pub(crate) struct Idle;
#[derive(Component)]
pub(crate) struct Walking;
#[derive(Component)]
pub(crate) struct Running;

```

但关键的是如何实现。

```rust
fn update_idle_system(query: Query<&Name, With<Idle>>) {
    //TODO
}

fn update_walking_system(query: Query<&Name, With<Walking>>) {
    //TODO
}

fn update_running_system(query: Query<&Name, With<Running>>) {
    //TODO
}

```

这套状态机不仅仅是给玩家的，NPC也包含状态机。但玩家肯定是能够以某种方式操控状态机的……输入。（靠！还得处理输入系统！）

我们的思路大概如此：

- 设一个代表 “可被玩家控制” 的组件
- 让输入的时候检测有这个组件的物体，若有，则设为Walking，若同时还按着 X 则设为 Run。

所以，新建`src/core/overworld/character/character_components.rs`，然后写了

```rust
#[derive(Component)]
struct PlayerControlled;
```

哦。我们还得有角色。所以……

```rust

#[derive(Component)]
pub(crate) struct Position(pub Vec2);

#[derive(Component)]
pub(crate) struct Rotation(pub f32);

#[derive(Component)]
pub(crate) struct Facing(pub Direction);
#[derive(Component)]
pub(crate) struct Health {
    pub(crate) current: i32,
    pub(crate) max: i32,
}
#[derive(Component)]
pub(crate) struct AnimationState {
    pub(crate) clip: String,
}
// Current state duration
#[derive(Component)]
pub(crate) struct StateTimer(pub f32);

pub(crate) enum Direction {
    Up,
    Down,
    Left,
    Right,
}
impl Direction {
    pub fn as_vec2(&self) -> Vec2 {
        match self {
            Direction::Up => Vec2::Y,
            Direction::Down => -Vec2::Y,
            Direction::Left => -Vec2::X,
            Direction::Right => Vec2::X,
        }
    }
}

```

```rust
#[derive(Bundle)]
pub struct CharacterBundle {
    position: Position,
    rotation: Rotation,
    facing: Facing,
    sprite: Sprite,
    health: Health,
    anim: AnimationState,
    state_timer: StateTimer,
    transform: Transform,
    global_transform: GlobalTransform,
}

impl CharacterBundle {
    pub fn new(spawn_pos: Vec2, facing: Direction, sprite: Sprite) -> Self {
        Self {
            position: Position(spawn_pos),
            rotation: Rotation(0.0),
            facing: Facing(facing),
            sprite,
            health: Health {
                current: 20,
                max: 20,
            },
            anim: AnimationState {
                clip: "idle_down".into(),
            },
            state_timer: StateTimer(0.0),
            transform: Transform::from_translation(spawn_pos.extend(0.0)),
            global_transform: GlobalTransform::default(),
        }
    }
}
```

好的，这些框我都搞定了。然后我睡了一觉起来（对的谁说“百日谈”就必须一个环节占正好一天的？），在准备敲码前先去 GitHub 刷刷刷，然后...

[seldom_state](https://github.com/Seldom-SE/seldom_state) 是何意味啊。有现成的有限状态机轮子...？

...那我们接下来要做的工作就简单多了。

## 把调料装进瓶子里...

我们来着手搓状态机吧！……哦还缺一个桌子，输入系统。淦！！！

好在这里也有轮子，有点 Unity Input System 感觉的轮子——[leafwing-input-manager](https://github.com/Leafwing-Studios/leafwing-input-manager).

那就来吧。`input.rs`直接码上。~~有新的旅行伙伴加入了呦~~

```rust
use bevy::prelude::*;
use leafwing_input_manager::prelude::*;

#[derive(Actionlike, PartialEq, Eq, Hash, Clone, Copy, Debug, Reflect)]
pub enum Action {
    Up,
    Down,
    Left,
    Right,
    Confirm,
    Cancel,
    Menu,
}

#[derive(Resource)]
pub(crate) struct PlayerInputSettings {
    maps: Vec<InputMap<Action>>,
}

impl PlayerInputSettings {
    pub fn get_map(&self, index: usize) -> &InputMap<Action> {
        self.maps.get(index).unwrap()
    }
    
    pub fn get_merged_map(&self) -> InputMap<Action> {
        let mut merged = InputMap::default();
        
        for map in &self.maps {
            merged.merge(map);
        }
        
        merged
    }
}

impl Default for PlayerInputSettings {
    fn default() -> Self {
        use Action::*;
        use KeyCode::*;
        let mut map_key_default = InputMap::default();

        map_key_default.insert(Up, ArrowUp);
        map_key_default.insert(Down, ArrowDown);
        map_key_default.insert(Left, ArrowLeft);
        map_key_default.insert(Right, ArrowRight);
        map_key_default.insert(Confirm, KeyZ);
        map_key_default.insert(Cancel, KeyX);
        map_key_default.insert(Menu, KeyC);

        let mut map_key_alternate_0 = InputMap::default();

        map_key_alternate_0.insert(Up, KeyW);
        map_key_alternate_0.insert(Down, KeyS);
        map_key_alternate_0.insert(Left, KeyA);
        map_key_alternate_0.insert(Right, KeyD);
        map_key_alternate_0.insert(Confirm, Enter);
        map_key_alternate_0.insert(Cancel, ShiftLeft);
        map_key_alternate_0.insert(Menu, ControlLeft);

        let mut map_key_alternate_1 = InputMap::default();

        map_key_alternate_1.insert(Cancel, ShiftRight);
        map_key_alternate_1.insert(Menu, ControlRight);

        let mut map_gamepad_default = InputMap::default();
        use GamepadButton::*;
        map_gamepad_default.insert(Up, DPadUp);
        map_gamepad_default.insert(Down, DPadDown);
        map_gamepad_default.insert(Left, DPadLeft);
        map_gamepad_default.insert(Right, DPadRight);
        map_gamepad_default.insert(Confirm, South);
        map_gamepad_default.insert(Cancel, East);
        map_gamepad_default.insert(Menu, North);

        Self {
            maps: vec![
                map_key_default,
                map_key_alternate_0,
                map_key_alternate_1,
                map_gamepad_default,
            ],
        }
    }
}

```

这套按键配置也是 UCT 的遗产啊。令人感叹。


对应的，玩家生成那里是这么写的

```rust
    commands.spawn((
        Idle,
        PlayerControlled,
        StateMachine::default()
            .trans::<Idle, _>(is_walking, Walking)
            .trans::<Walking, _>(is_walking.not(), Idle)
            .set_trans_logging(true),
        player_input.get_merged_map(),
        ActionState::<Action>::default(),
        CharacterBundle::new(Vec2::new(0.0, 0.0), Direction::Down, sprite.clone()),
    ));

    
fn is_walking(query: Query<&ActionState<Action>, With<PlayerControlled>>) -> Result<(), ()> {
    let action_state = query.single().map_err(|_| ())?;
    if action_state.pressed(&Action::Left)
        || action_state.pressed(&Action::Right)
        || action_state.pressed(&Action::Up)
        || action_state.pressed(&Action::Down)
    {
        Ok(())
    } else {
        Err(())
    }
}

```

不赖。我们终于可以开始写状态啦！哦，还需要个速度组件...

```rust
#[derive(Component)]
pub(crate) struct Speed(pub f32);
```

还有就是，角度得改成八个：

```rust

#[derive(Default)]
pub(crate) enum Direction {
    Up,
    #[default]
    Down,
    Left,
    Right,
    UpLeft,
    UpRight,
    DownLeft,
    DownRight,
}
impl Direction {
    pub fn as_vec2(&self) -> Vec2 {
        match self {
            Direction::Up => Vec2::Y,
            Direction::Down => -Vec2::Y,
            Direction::Left => -Vec2::X,
            Direction::Right => Vec2::X,
            Direction::UpLeft => Vec2::new(-1.0, 1.0).normalize(),
            Direction::UpRight => Vec2::new(1.0, 1.0).normalize(),
            Direction::DownLeft => Vec2::new(-1.0, -1.0).normalize(),
            Direction::DownRight => Vec2::new(1.0, -1.0).normalize(),
        }
    }
}
```

然后就好啦！看看我们最后的代码吧——

```rust
use crate::core::core_components::{Facing, Position, Speed};
use crate::core::input::Action;
use crate::core::overworld::character::character_components::{Idle, Running, Walking};
use bevy::prelude::*;
use leafwing_input_manager::action_state::*;

pub(crate) fn update_idle_system(
    mut query: Query<(&mut Position, &mut Facing, &Speed), With<Idle>>,
) {
    // TODO: 实现 Idle 状态逻辑
}

pub(crate) fn update_walking_system(
    time: Res<Time>,
    mut query: Query<(&mut Position, &mut Facing, &Speed, &ActionState<Action>), With<Walking>>,
) {
    for (mut pos, mut facing, speed, action_state) in query.iter_mut() {
        use crate::core::core_components::*;
        
        // 计算移动方向
        let mut direction_vec = Vec2::ZERO;
        if action_state.pressed(&Action::Up) { direction_vec.y += 1.0; }
        if action_state.pressed(&Action::Down) { direction_vec.y -= 1.0; }
        if action_state.pressed(&Action::Left) { direction_vec.x -= 1.0; }
        if action_state.pressed(&Action::Right) { direction_vec.x += 1.0; }
        
        let new_direction = match (direction_vec.x as i32, direction_vec.y as i32) {
            (0, 1) => Some(Direction::Up),
            (0, -1) => Some(Direction::Down),
            (-1, 0) => Some(Direction::Left),
            (1, 0) => Some(Direction::Right),
            (-1, 1) => Some(Direction::UpLeft),
            (1, 1) => Some(Direction::UpRight),
            (-1, -1) => Some(Direction::DownLeft),
            (1, -1) => Some(Direction::DownRight),
            _ => None,
        };
        
        if let Some(direction) = new_direction {
            facing.value = direction;
            pos.value += facing.value.as_vec2() * speed.value * time.delta_secs();
        }
    }
}

pub(crate) fn update_running_system(
    mut query: Query<(&mut Position, &mut Facing, &Speed), With<Running>>,
) {
    for (mut pos, mut facing, speed) in query.iter_mut() {
        // TODO: 实现 Running 状态逻辑
    }
}

```

站立状态下，目前我们不用做任何事情。行走……就是行走。

但是站立还是要留着，因为之后还得处理动画呢……嗯……

不管怎样，小小的庆祝一下！因为现在动起来了！

```
 ______________________
< 动起来呀，动起来呀！ >
 ----------------------
        \
         \
            _~^~^~_
        \) /  o o  \ (/
          '_   -   _'
          / '-----' \
```

~~是 Ferris 老师让我动起来的！动起来呀，动起来呀！~~

## ...然后倒入另一个瓶子

写到刚才，我本来寻思要不删掉跑步状态...但是我还是想把它留下来。

为啥呢，因为跑步状态本身很多新兴的 fangame 大概都有了。即便你想复古，那直接删掉这个状态就得了――总比塞进 walking 要好！

并且，之后我们也许可以做点有意思的。比方说玩家跑向某个 npc 和走向某个 npc 的对话不一样...之类的。发展空间大概很多。

所以，留着好啦！最重要的是我那份跑步动画不能白瞎了（

```rust
fn apply_walking_step(
    mut pos: Mut<Position>,
    mut facing: Mut<Facing>,
    speed: f32,
    action_state: &ActionState<Action>,
    delta_secs: f32,
) {
    use crate::core::core_components::*;

    let mut direction_vec = Vec2::ZERO;
    if action_state.pressed(&Action::Up) {
        direction_vec.y += 1.0;
    }
    if action_state.pressed(&Action::Down) {
        direction_vec.y -= 1.0;
    }
    if action_state.pressed(&Action::Left) {
        direction_vec.x -= 1.0;
    }
    if action_state.pressed(&Action::Right) {
        direction_vec.x += 1.0;
    }

    let new_direction = match (direction_vec.x as i32, direction_vec.y as i32) {
        (0, 1) => Some(Direction::Up),
        (0, -1) => Some(Direction::Down),
        (-1, 0) => Some(Direction::Left),
        (1, 0) => Some(Direction::Right),
        (-1, 1) => Some(Direction::UpLeft),
        (1, 1) => Some(Direction::UpRight),
        (-1, -1) => Some(Direction::DownLeft),
        (1, -1) => Some(Direction::DownRight),
        _ => None,
    };

    if let Some(direction) = new_direction {
        facing.value = direction;
        pos.value += facing.value.as_vec2() * speed * delta_secs;
    }
}

pub(crate) fn update_walking_system(
    time: Res<Time>,
    mut query: Query<(&mut Position, &mut Facing, &Speed, &ActionState<Action>), With<Walking>>,
) {
    for (pos, facing, speed, action_state) in query.iter_mut() {
        apply_walking_step(pos, facing, speed.value, action_state, time.delta_secs());
    }
}
pub(crate) fn update_running_system(
    time: Res<Time>,
    mut query: Query<(&mut Position, &mut Facing, &Speed, &ActionState<Action>), With<Running>>,
) {
    for (pos, facing, speed, action_state) in query.iter_mut() {
        apply_walking_step(pos, facing, speed.value * 2.0, action_state, time.delta_secs());
    }
}
```

然后还有配套的状态机转换：

```rust

fn setup_overworld_system(
    mut commands: Commands,
    rpg_sprite_handles: Res<OverWorldCharacterSpriteFolder>,
    asset_server: Res<AssetServer>,
    mut texture_atlases: ResMut<Assets<TextureAtlasLayout>>,
    loaded_folders: Res<Assets<LoadedFolder>>,
    mut textures: ResMut<Assets<Image>>,
    player_input: Res<PlayerInputSettings>,
) {
    let loaded_folder = loaded_folders.get(&rpg_sprite_handles.0).unwrap();

    let (texture_atlas_nearest, nearest_sources, nearest_texture) = create_texture_atlas(
        loaded_folder,
        None,
        Some(ImageSampler::nearest()),
        &mut textures,
    );
    let atlas_nearest_handle = texture_atlases.add(texture_atlas_nearest);

    let frisk_handle: Handle<Image> = asset_server
        .get_handle("textures/overworld/characters/frisk/walk/frisk-walk-down-1.png")
        .unwrap();

    let sprite = Sprite::from_atlas_image(
        nearest_texture,
        nearest_sources
            .handle(atlas_nearest_handle, &frisk_handle)
            .unwrap(),
    );

    commands.spawn((
        Idle,
        PlayerControlled,
        StateMachine::default()
            .trans::<Idle, _>(is_walking, Walking)
            .trans::<Walking, _>(is_walking.not(), Idle)
            .trans::<Running, _>(is_walking.not(), Idle)
            .trans::<Walking, _>(is_running, Running)
            .trans::<Running, _>(is_running.not(), Walking)
            .set_trans_logging(true),
        player_input.get_merged_map(),
        ActionState::<Action>::default(),
        CharacterBundle::new(Vec2::new(0.0, 0.0), Direction::Down, sprite.clone()),
    ));
}

fn is_walking(query: Query<&ActionState<Action>, With<PlayerControlled>>) -> Result<(), ()> {
    let action_state = query.single().map_err(|_| ())?;
    if action_state.pressed(&Action::Left)
        || action_state.pressed(&Action::Right)
        || action_state.pressed(&Action::Up)
        || action_state.pressed(&Action::Down)
    {
        Ok(())
    } else {
        Err(())
    }
}

fn is_running(query: Query<&ActionState<Action>, With<PlayerControlled>>) -> Result<(), ()> {
    let action_state = query.single().map_err(|_| ())?;
    if action_state.pressed(&Action::Cancel) {
        Ok(())
    } else {
        Err(())
    }
}

```

好了。换个瓶子这一步倒是简单。编程就是一开始很难搞然后中间貌似好整了，弄到后面又完犊子了的一个过程。。。哈哈。

只不过现在的瓶子可太瓶子了。

## 那我缺的这个标签谁给我补啊

现在这游戏里面没有任何动画。如果我还在 unity，那我就可以光速用 animator 不太优雅地解决战斗。但这里是 RUST！所以...

先想想怎么把帧动画给处理了吧……先看看目前的资源结构！

```
assets
└── textures
    └── overworld
        └── characters
            ├── flowey
            │   ├── idle
            │   │   ├── flowey-idle-0.png
            │   │   └── flowey-idle-1.png
            │   └── sink
            │       ├── flowey-sink-0.png
            │       ├── flowey-sink-1.png
            │       ├── flowey-sink-2.png
            │       ├── flowey-sink-3.png
            │       ├── flowey-sink-4.png
            │       └── flowey-sink-5.png
            ├── frisk
            │   ├── run
            │   │   ├── frisk-run-down-1.png
            │   │   ├── frisk-run-down-2.png
            │   │   ├── frisk-run-down-3.png
            │   │   ├── frisk-run-down-4.png
            │   │   ├── frisk-run-down-5.png
            │   │   ├── frisk-run-down-6.png
            │   │   ├── frisk-run-left-1.png
            │   │   ├── frisk-run-left-2.png
            │   │   ├── frisk-run-left-3.png
            │   │   ├── frisk-run-left-4.png
            │   │   ├── frisk-run-left-5.png
            │   │   ├── frisk-run-left-6.png
            │   │   ├── frisk-run-right-1.png
            │   │   ├── frisk-run-right-2.png
            │   │   ├── frisk-run-right-3.png
            │   │   ├── frisk-run-right-4.png
            │   │   ├── frisk-run-right-5.png
            │   │   ├── frisk-run-right-6.png
            │   │   ├── frisk-run-up-1.png
            │   │   ├── frisk-run-up-2.png
            │   │   ├── frisk-run-up-3.png
            │   │   ├── frisk-run-up-4.png
            │   │   ├── frisk-run-up-5.png
            │   │   ├── frisk-run-up-6.png
            │   │   └── readme.md
            │   └── walk
            │       ├── frisk-walk-down-1.png
            │       ├── frisk-walk-down-2.png
            │       ├── frisk-walk-down-3.png
            │       ├── frisk-walk-down-4.png
            │       ├── frisk-walk-left-1.png
            │       ├── frisk-walk-left-2.png
            │       ├── frisk-walk-right-1.png
            │       ├── frisk-walk-right-2.png
            │       ├── frisk-walk-up-1.png
            │       ├── frisk-walk-up-2.png
            │       ├── frisk-walk-up-3.png
            │       └── frisk-walk-up-4.png
            └── toriel
                └── walk
                    ├── toriel-walk-down-0.png
                    ├── toriel-walk-down-1.png
                    ├── toriel-walk-down-2.png
                    ├── toriel-walk-down-3.png
                    ├── toriel-walk-down-talk-0.png
                    ├── toriel-walk-down-talk-1.png
                    ├── toriel-walk-left-0.png
                    ├── toriel-walk-left-1.png
                    ├── toriel-walk-left-2.png
                    ├── toriel-walk-left-3.png
                    ├── toriel-walk-left-talk-0.png
                    ├── toriel-walk-left-talk-1.png
                    ├── toriel-walk-up-0.png
                    ├── toriel-walk-up-1.png
                    ├── toriel-walk-up-2.png
                    └── toriel-walk-up-3.png
```

好吧，这着实也不是很优雅。很多命名和路径都重复了——没必要。让我们把一切以路径为主吧。

```
assets
└── textures
    └── overworld
        └── characters
            ├── flowey
            │   ├── idle
            │   │   ├── 0.png
            │   │   └── 1.png
            │   └── sink
            │       ├── 0.png
            │       ├── 1.png
            │       ├── 2.png
            │       ├── 3.png
            │       ├── 4.png
            │       └── 5.png
            ├── frisk
            │   ├── run
            │   │   ├── down
            │   │   │   ├── 1.png
            │   │   │   ├── 2.png
            │   │   │   ├── 3.png
            │   │   │   ├── 4.png
            │   │   │   ├── 5.png
            │   │   │   └── 6.png
            │   │   ├── readme.md
            │   │   ├── side
            │   │   │   ├── 1.png
            │   │   │   ├── 2.png
            │   │   │   ├── 3.png
            │   │   │   ├── 4.png
            │   │   │   ├── 5.png
            │   │   │   └── 6.png
            │   │   └── up
            │   │       ├── 1.png
            │   │       ├── 2.png
            │   │       ├── 3.png
            │   │       ├── 4.png
            │   │       ├── 5.png
            │   │       └── 6.png
            │   └── walk
            │       ├── down
            │       │   ├── 1.png
            │       │   ├── 2.png
            │       │   ├── 3.png
            │       │   └── 4.png
            │       ├── side
            │       │   ├── 1.png
            │       │   └── 2.png
            │       └── up
            │           ├── 1.png
            │           ├── 2.png
            │           ├── 3.png
            │           └── 4.png
            └── toriel
                └── walk
                    ├── down
                    │   ├── 0.png
                    │   ├── 1.png
                    │   ├── 2.png
                    │   ├── 3.png
                    │   └── talk
                    │       ├── 0.png
                    │       └── 1.png
                    ├── side
                    │   ├── 0.png
                    │   ├── 1.png
                    │   ├── 2.png
                    │   ├── 3.png
                    │   └── talk
                    │       ├── 0.png
                    │       └── 1.png
                    └── up
                        ├── 0.png
                        ├── 1.png
                        ├── 2.png
                        └── 3.png
```

这就行了，路径和名字没有重复——干净，利索，并且喜欢全局搜索的小朋友们有福了。我本来寻思本文的之后还要来一些部分来写一下动画的实现……但那有点超纲了！咱不今儿不就是要搓状态机吗？我怎么做了这么多杂七杂八的，都写到一千来行了？？？

所以说……啊。就这样得了。PR 提上去，第一天的事儿就完成了。之后的事情之后再说吧！嗯大概就这样。

我们明天就能看到 Frisk 真正的动起来咯……！

```
 __________
< 动起来呀 >
 ----------
        \
         \
                                        
          ██████████████████            
        ██████████████████████          
      ██████████████████████████        
    ██████████████████████████████      
    ████████████████████████████████    
  ████████  ████████  ██████████████    
  ██████      ████    ████  ████████    
  ██████                      ████████  
████████                      ██  ████  
████████  ████        ████        ████  
██████                          ██████  
██████          ████          ████████  
████████                  ████████████  
████████████            ██████████████  
  ██  ████████████    ██████████  ██    
                ██    ██      ██        
              ██████████████            
        ██████  ████████  ██████        
      ██  ██                ██  ██      
    ██  ████████████████████████  ██    
    ██  ██                    ██  ██    
    ████████████████████████████████    
    ██  ██                    ██  ██    
    ██████                    ██████    
        ██      ██████        ██        
        ██      ██  ██        ██        
        ██████████  ████████████        
            ██████  ████████            
          ████████  ██████████          
```

---

企划时间：2025-10-22

开始时间：2025-10-22

完成时间：2025-10-23

关联的 PR：[#6](https://github.com/Bli-AIk/souprune/pull/6)



[^1]: [【C#基础 + WinForms-哔哩哔哩】](https://www.bilibili.com/video/BV1Gx411U7Hb)，虽然讲的磨叽点吧，但毕竟是引我上路的教程，还是挺好的。C#真是最好学的编程语言了。

[^2]: [【Unity2D官方入门教程 Ruby' Adventure 完整版~-哔哩哔哩】](https://www.bilibili.com/BV1V4411W787)，我个人觉得是~~针对野路子来说~~相当权威的 Unity 入门教程，要是大伙儿都看过一遍就太好了。

[^3]: 官方参考[见此](https://docs.rs/bevy/latest/bevy/asset/trait.AssetLoader.html)。

[^4]: [feat(core): Adding a standard character state machine #6](https://github.com/Bli-AIk/souprune/pull/6)
