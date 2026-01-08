现在我们有了挂钩，但是上面空空的，啥也没有。看看那个大黑框上面的那点儿白字儿...兄弟你动啊？！

所以我们最起码需要一个打字机，你 dr / ut 的灵魂就是它了；我们还需要有逐个字符的抖动、或者变形（小幽灵说话belike），那基于 mesh 的文本动画能~~很轻松的~~实现这一点。

比较麻烦的事情是我们需要对话分支――各种对话选项，有的选项可能还需要一些条件才能显示；还有对话事件系统！这可是个难点了，比方说让角色说一半话然后切换为倒地的动画，或者放神秘爆炸特效之类的。对于 dr / ut 这种游戏，我们也许需要非常细致...例如按照索引插入事件。

很搞笑的是这些都是 UCT 已经实现的功能了，并且当年啥也不会的我为了这些东西煞费苦心...而现在又要来一遍。啊哈。并且我们还要把之前做得不好的地方给改好，比方说：

```
DemoBox1\<default><fx=flowey, evil><markPoint>啥<stop*10>呢。<waitForUpdate><default><markPoint>这是个<gradient="White to Yellow - UTC">破箱子</gradient><stop...><stop...><markEnter>哈哈，咱没做界面。
<waitForUpdate><Flowey><fx=flowey, evil><markPoint><gradient="White to Red - UTC">啊？？？</gradient><fx=flowey><stop*3>怎么说，<stop><Flowey, Menace><fx=flowey, evil><enter><-  ->你<stop>先<stop>别<stop>急<stop>。
<waitForUpdate><Flowey, Side><markPoint>现在捏，<stop>这个对话是测试<gradient="White to Yellow - UTC"><enter><-  ->打字机</gradient>来的，<fx=flowey, evil><stop><Flowey><-就像这样->。
<waitForUpdate><Flowey><-* 啪的一下就说完了一句话。-><enter><-  -><stop*3><gradient="White to Red - UTC">炫炫炫炫炫炫炫炫炫炫。</gradient>
<waitForUpdate><Flowey, Side><markPoint>啥？<stop>你没看清？<stop><enter><-  ->那我慢点？<stop*5><enter><-* -><stop><-啪的-><stop><-一下-><stop><-就说-><stop><-完了-><stop><-。-><stop*3>蛤？;
```
我的天。看到这些富文本了吗，太地狱了，我不知道我当年是怎么想的，用这么混沌的方式做文本系统，文本和事件完全耦合？这怎么干活，你还让不让文案活了？认真的吗？？？！

当然，我们其实还有其他选择，例如 Ink 或 Yarn spinner 。UCT 里集成过 Ink，而 Yarn spinner 我也在后来了解过。在对话分支这一块它们做得都跟到位，但很可惜，它们都没处理好文本和事件耦合的问题。很多时候事件就是一个tag，针对整个文本...我们明明能做得更好。并且...更`rust`。

那么，就把厨具造出来吧。别家造厨房的还能从我这拿厨具用，这多好啊。要真的有别家造厨房的就好了。。哎。

做饭的时候，最烦人的不是炒菜，而是备菜。切丝、切片、切丁……这一套下来，胳膊都酸了。

写游戏也一样，最烦人的不是写逻辑，而是搓动画。你难道想在代码里一行行地写 `Translation::new(0.0, 0.0, 0.0)` 然后下一秒 `Translation::new(10.0, 10.0, 0.0)` 吗？那得写到猴年马月去？

我们需要更趁手的工具——刀、叉、筷、勺，或者更高级点，全自动切菜机。

# 08 - 刀叉筷勺

在寻找“切菜机”的过程中，我把目光投向了一个意想不到的地方：**Alight Motion**。

这就好比你发现隔壁那家做视频剪辑的店里，有一台切菜机特别好用。虽然它是用来剪视频的，但谁规定不能拿来切洋葱呢？

---AI 工具箱---

警告：以下内容由 AI 生成。为了让这些工具看起来更酷，我可能加了点滤镜。如果你发现切菜机变成了光剑，请不要惊慌。不喜欢的话，PR 大门常打开。

## 偷懒神器：Bevy Alight Motion

于是，`bevy_alight_motion` 诞生了。

这玩意儿的核心逻辑简单粗暴：直接解析 Alight Motion 的工程文件（`.amproj`，本质上是个 XML）。

你在手机上搓好动画——不管是简单的位移，还是复杂的贝塞尔曲线缓动（Easing），甚至是图层嵌套（Pre-composition）——然后把文件往游戏资源目录里一扔。

Bevy 这边，插件会自动读取 XML，把里面的每一个图层（Layer）映射成一个 Entity，把每一条关键帧（Keyframe）转化成 ECS 组件。

```xml
<shape id="123" label="MyCoolSprite">
    <transform>
        <location>
            <kf t="0.0" v="0.0,0.0,0.0" />
            <kf t="1.0" v="100.0,100.0,0.0" e="cubicBezier 0.42 0.0 0.58 1.0" />
        </location>
    </transform>
</shape>
```

这段 XML 到了游戏里，就变成了“在 1 秒内，从 (0,0) 移动到 (100,100)，并且带有一个丝滑的缓动效果”。

我再也不用在代码里手搓 `Lerp` 了！这就是科技的力量啊，朋友们！

## 更多的小工具

除了这个大家伙，厨房里还有不少顺手的小工具：

- **chaser**: 一个简单的资源路径追踪器，确保我在代码里引用的图片路径不会因为手滑写错。
- **iyes_perf_ui**: 性能监视器，挂在角落里看 FPS，就像厨房里的温度计。

有了这些趁手的家伙什，做饭（开发）的效率那是蹭蹭往上涨。接下来，咱们得找个地方放菜了——**09 - 菜篮子捏**。

---

企划时间：2025-11-15

开始时间：2026-01-08

完成时间：2026-01-08

关联的 PR：[#23](https://github.com/Bli-AIk/souprune/pull/23)

