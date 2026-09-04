[English](README.md) | [日本語](README.ja.md) | 简体中文 | [한국어](README.ko.md)

# Udonite

**用普通的 Unity C# 编写 VRChat 世界脚本。**

写一个 `MonoBehaviour`，Udonite 会把它编译成 Udon。没有需要另外学的方言，也没有需要背的禁用关键字列表：你在 Unity 里本来就在写的 C#，就是在世界里运行的 C#。

Udonite 是免费的，网络功能也包含在内。

世界还可以调用 [Udonite 的托管服务](https://admin.udonite.com)：从世界向 Discord 或 Pushover 发送消息，并保存计数器、成绩表或共享开关 —— 即使所有人都离开也不会丢失。有 GitHub 账号就能免费试用。

**[文档](https://docs.udonite.com/compiler?lang=zh)** · **[安装](https://udonite.com/?lang=zh)**

## 使用

写一个 `MonoBehaviour`，然后按 Play。Udonite 会编译项目中的每一个 behaviour，把 Udon 程序附加到同一个 GameObject 上，并为每次编译输出一行日志。Play 模式下运行的就是 Udon 程序本身，所以你在编辑器里看到的，就是最终上传的东西。

继承 `UdoniteBehaviour` 是可选的，但会省事一些：VRChat 事件会变成可以 `override` 的 `virtual` 方法，所以事件名拼错会变成编译错误，而不是一个永远不会被调用的方法；同步字段的特性和 VRChat 辅助方法也会一并附带。

## 你可以写什么

- **是完整的语言，不是子集。** 带约束的泛型、带默认实现的接口、record、结构体、嵌套类、扩展方法、元组与解构、局部函数、`params` 和可选参数。
- **现代 C# 语法。** switch 表达式、模式匹配（`is > 5 and < 10`、属性模式、`is not null`）、`?.`、`??=`、带格式说明符的字符串插值、范围与索引。
- **集合与 LINQ。** `List<T>`、`Dictionary<K,V>`、`HashSet<T>`、`Queue<T>`、`Stack<T>`、多维数组，以及 LINQ 链式调用（`Where`、`Select`、`OrderBy`、`GroupBy`、`Zip` 等），其中的 lambda 会就地编译展开。
- **委托与事件。** `Action` 和 `Func` 字段、方法组、多播委托的 `+=` 和 `-=`、带 `?.Invoke` 的 C# `event` 声明、作为值传递的 lambda。
- **和平常一样处理时间。** 使用 `yield return new WaitForSeconds(...)` 的协程、使用 `await Task.Delay(...)` 的 `async` 方法、`Invoke` 和 `InvokeRepeating`。
- **带负载的网络同步。** 带变更回调的同步字段、按名称调用的自定义事件，以及类型化的网络事件：定义一个带字段的类，用 `Network.Send(evt, receiver)` 发送，在以该事件类型作为参数的方法中接收。还内置了同步位置、旋转和缩放的 `NetworkTransform` 组件。
- **可空值、`TryGetComponent`、`StringBuilder`、`Array.FindAll`**，以及其他那些你会下意识用到的小东西。

```csharp
using System.Collections.Generic;
using System.Linq;
using Udonite.Package.Runtime;
using UnityEngine;

public class ScoreEvent : NetworkEvent
{
    public int player;
    public int points;

    public void Serialize(ByteWriter writer) => writer.WriteInt32(player).WriteInt32(points);
    public void Deserialize(ByteReader reader) { player = reader.ReadInt32(); points = reader.ReadInt32(); }
}

public class Scoreboard : UdoniteBehaviour
{
    public event System.Action<int> Changed;
    private readonly Dictionary<int, int> totals = new Dictionary<int, int>();

    public void Award(int player, int points) => Udonite.Network.Send(new ScoreEvent { player = player, points = points }, this);

    [NetworkEventHandler]
    public void OnScore(ScoreEvent evt)
    {
        totals.TryGetValue(evt.player, out int current);
        totals[evt.player] = current + evt.points;
        Changed?.Invoke(evt.player);
    }

    public string Leader() => totals.OrderByDescending(pair => pair.Value).Select(pair => pair.Key.ToString()).FirstOrDefault() ?? "nobody";
}
```

## Udonite 不会做什么

Udon 有一些真实存在的限制：没有异常、没有可变的静态字段、没有 `Awake`、没有 tag。Udonite 会明确地拒绝这些写法。每一次拒绝都是控制台里的一行，带一个 `UDN` 代码，说明拒绝了什么、以及应该改写成什么；不会有任何东西被悄悄丢掉。被拒绝的 behaviour 不会生成 Udon 程序，项目里其余的部分照常编译。详见[语言支持](https://docs.udonite.com/compiler/language-support?lang=zh)（英文）。

## 支持 Udonite

Udonite 编译器是免费的，而且永远都会是免费的。它由一个人开发和维护，包括每次 VRChat SDK 更新弄坏东西之后的修复。

你的支持资助这项工作，而现在也会带来世界里真正用得上的东西。Udonite 提供世界通过 URL 调用的托管服务：从世界向 Discord 或 Pushover 发送消息，并保存计数器、成绩表或共享开关 —— 即使所有人都离开也不会丢失。两者都可以免费试用，成为支持者会把各项上限提高到你基本不会碰到的程度。

编译器不在其中。它对所有人都免费，SDK 更新导致的问题也会在同一天修复给所有人。

[成为支持者](https://buy.polar.sh/polar_cl_fiMBXRqzbvm0c8qt6xqKKBGbaB0FOQmPMddgn06PRsC) · [服务介绍](https://udonite.com/?lang=zh)

## 问题反馈

Bug、你认为应该能编译却被拒绝的写法，以及 SDK 更新导致的问题，请提交到 [Issues](https://github.com/Udonite/Udonite/issues)。用中文提交完全没问题。请附上控制台里的那一行拒绝信息，它会指出确切是哪个写法。

Udonite 是第三方工具，与 VRChat Inc. 无关。
