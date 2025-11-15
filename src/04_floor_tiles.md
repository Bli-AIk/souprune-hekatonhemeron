bro，你喜欢大理石地板还是木地板，还是砖头地板，或者土地板？

喜欢啥都无所谓，找你画师给你画去。我们的问题是用什么胶打更合适――社区库，还是tiled？这是个问题。

所以我选择了 `LDtk`。咱总是不按常理出牌——例如拿`rust`写游戏（

# 04 - 铺地砖儿

虽然这样也有点代价。我们没有（至少目前还没有）地图编辑器可视化的动画瓦片等功能，但相对的是现代化且轻松上手（大概）的清清爽爽的... 地图编辑器。并且实体也可以在这里面定义，喜欢 overworld 的真是有福啦。

只不过。我应该早点考虑瓦片地图的。似乎它的资源处理方式不太一样...

## 先铺地砖...

然后我后悔了。`LDtk` 在我深度使用后没有看上去那么香……很多东西都和固定的瓦片分辨率绑死了！！！对于 deltarune / undertale 这样的有很多“自由放置”的 object 的游戏来说，LDtk没法用。

所以……撤离！还是 `tiled` 吧。我就不该折腾。淦。

所以我又好好看了遍 `tiled`（毕竟我以前没用过）。它其实倒也有槽点……我很喜欢搞 `RuleTile` 之类的东西，但是在 `tiled` 里面就必须分为 `Terrains` 和 `Automapping`。（主要还是Automapping）

相对于LDtk确实灵活了…… C++式的灵活（笑）。但至少别的槽点没有了。动画瓦片和自由放置都没问题。那就这样吧。~~难不成自己做个地图编辑器吗？~~

不管怎样，把地图搭上吧。我想现在开始我们必须在这个文档里面上点图了……

~~TODO: Tiled 权威教程之automapping太好用了你们知道吗~~

你知道吗？加个锤子。我到时候写文档说怎么用就得了呗，搁这儿磨叽啥啊。

随后，实话实说，我没有立刻去集成 Tiled ―― 刚和 automapping 对完线，真的很累诶。

所以你现在大概可以先去看看第八章（是的其实天数是todolist罢了~~哈哈我想先做啥就做啥~~）。我在画完瓦片图后实际上是先去搓那玩意儿了――亲爱的 `mortar`！

## ...再打胶？？

所以我把地图捏好后大概是这样的文件结构。levels文件夹位于assets文件夹下。

```
levels
├── Demo.tiled-session
├── ruins
│   ├── ruins_2.tmx
│   ├── ruins_3.tmx
│   ├── ruins_objects.tsx
│   ├── ruins.tsx
│   ├── rules
│   │   └── ruins_rules.tmx
│   └── tiles
│       ├── objects
│       │   ├── bigweb_0.png
│       │   ├── brand.png
│       │   ├── candydish_0.png
│       │   ├── candydish2_0.png
│       │   ├── candydish2_1.png
│       │   ├── candydish_bad_0.png
│       │   ├── centeredhole_0.png
│       │   ├── cheesetable_0.png
│       │   ├── colorswitch_0.png
│       │   ├── colorswitch_1.png
│       │   ├── colorswitch_2.png
│       │   ├── faceswitch_0.png
│       │   ├── faceswitch_1.png
│       │   ├── groundswitch1_0.png
│       │   ├── groundswitch1_1.png
│       │   ├── hole_0.png
│       │   ├── hole2_0.png
│       │   ├── ribbon_0.png
│       │   ├── smallweb_0.png
│       │   ├── spiketile_0.png
│       │   ├── spiketile_1.png
│       │   ├── switch_0.png
│       │   ├── switch_1.png
│       │   └── tornote_0.png
│       └── ruins.png
├── ruins.tiled-project
├── ruins.tiled-session
├── ruins.world
└── rules.txt
  
```

导入瓦片倒也简单……简单得有点过分了。

```
use crate::debug_info;
use bevy::prelude::Commands;
use bevy::prelude::*;
use bevy_ecs_tiled::prelude::*;

pub(crate) fn setup_tilemap(commands: &mut Commands, asset_server: &Res<AssetServer>) {
    commands.spawn((
        TiledMap(asset_server.load("levels/ruins/ruins_3.tmx")),
        TilemapAnchor::Center,
        TiledMapLayerZOffset(10.0),
    ));
}

pub(crate) fn filter_prototype_layers_and_set_z_order(
    mut commands: Commands,
    layers_query: Query<(Entity, &Name), Added<TiledLayer>>,
) {
    for (layer_entity, layer_name) in layers_query.iter() {
        let layer_name_str = layer_name.as_str();

        if layer_name_str.to_lowercase().contains("prototype") {
            debug_info!("Hide prototype layer: {}", layer_name_str);
            commands.entity(layer_entity).insert(Visibility::Hidden);
        } else {
            debug_info!("Show layers: {}", layer_name_str);

            let z_offset = match layer_name_str {
                name if name.contains("floor") => -2.5,
                name if name.contains("wall") && !name.contains("on_wall") => -2.0,
                name if name.contains("on_wall") => -1.5,
                name if name.contains("objects") => -1.0,
                _ => -3.0,
            };

            commands
                .entity(layer_entity)
                .insert(Transform::from_xyz(0.0, 0.0, z_offset));
            debug_info!(
                "Set the Z-axis position of layer {}: {}",
                layer_name_str,
                z_offset
            );
        }
    }
}

```

我调了一下图层之类的东西。反正现在看来显示的……相当的好。

然后，我们还要准备配置一下obj...

## 铺到外屋去

事实上，下面的部分可能是和第八章混着写的。切换房间本质上是个事件，对吧？那我们就得有事件系统……显而易见的。那是第八章的主题。

跨房间


---

企划时间：2025-10-23

开始时间：

完成时间：

关联的 PR：[#11](https://github.com/Bli-AIk/souprune/pull/11)
