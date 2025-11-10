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

# 06 - 刀叉筷勺

我们要做两个大东西。

一个是灵感源于 Ink 和 Yarn spinner 的 DSL ―― `mortar`（包括编译器，CLI，LSP），另一个是献给bevy社区的 `bevy_dialog_tools`（包括异步/协程打字机，Mesh文本动画，以及对接 mortar 的对话事件系统）。

是的，这些都是 UCT 已经实现（或者说，姑且算是实现了吧）的功能。我们要把他们都在 souprune 中... 不，是 rust 社区中，重新实现，更好的实现，贡献给大伙儿，但愿如此。

> UCT 的遗产将会在 rust 社区熠熠生辉！
>
> ~~（或继续吃灰）~~

## 臼子...？！

首先奔赴战场的是...臼。捣蒜、捣药、舂米用的那玩意儿...臼？

不是刀叉筷勺吗，怎么蹦出来个...臼？

哦，很简单，因为我最开始只打算做 `bevy_dialog_tools`的，没寻思造 `mortar` 这个轮子，不是其他厨具来者不善，而是它才是来者捏。

所以，让我们现在把 Ink / Yarn spinner 锈掉吧。

> 你被 Rust 邪教感染。被同化之后，你感到了一种嗜血冲动，面目可憎地想要传播 Ferris 的福音，并将将来和现在的所有软件用Rust重写。
> 
> 在踏上旅途之时，你在螃蟹的标志下觉醒、重生。准备好将软件世界带向一个更好的明天。

所以，我做了 Mortar。它是一个为**游戏文本与事件系统**打造的轻量级 DSL，让故事编写与程序逻辑保持分离——写文的专心写文，绑事件专心绑事件；文字干净纯粹，事件清晰明了...巴拉巴拉。

可以用它轻松描述一段对话、一场分支剧情，甚至复杂的交互逻辑...哦可能更复杂的我得之后再加。

反正啊，下面是一个最简单的示例：

```mortar
node Start {
    text: "嗷！！！"
    events: [
        0, play_sound("神笔爆炸音效.wav")
    ]

    choice: [
        "大早上的你叫啥啊？?" -> Amious,
        "何意味。。。" -> return
    ]
}

node Amious {
    text: "一把十个亿"
}

fn play_sound(value: String)

```

总而言之和 Ink / Yarn Spinner的定位一样...它读起来就像在讲故事，而不是在写代码。大概就是这样咯。

我想尽可能简单的介绍这个东西，因为如果你想看点详细的可以去看它的 [GitHub](https://github.com/Bli-AIk/mortar) 页面或者文档。

在之后我们会去对接它！但现在，我们先给这玩意儿放一边，先来处理一些更有必要的东西...


## 刀叉

Mesh 文本动画

## 筷子

对接mortar

## 勺子

打字机



---

企划时间：2025-10-23

开始时间：

完成时间：
