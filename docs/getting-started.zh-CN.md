[English](getting-started.md) | [日本語](getting-started.ja.md) | 简体中文 | [한국어](getting-started.ko.md)

# 快速开始

## 安装

1. 打开 VRChat Creator Companion，进入 **Settings → Packages → Add Repository**。
2. 粘贴 `https://udonite.github.io/vpm/index.json`，点击 **Add**。
3. 打开你的世界项目，从包列表中添加 **Udonite**。

Udonite 需要 VRChat Worlds SDK 3.10 或更高版本，以及 Unity 2022.3。它依赖的 C# 编译器库已经随包一起提供，所以不需要再装别的东西。

## 编写一个 behaviour

继承 `UdoniteBehaviour`，然后像在 Unity 里的任何地方一样写 C#：

```csharp
using Udonite.Package.Runtime;
using UnityEngine;

public class Door : UdoniteBehaviour
{
    public Transform hinge;
    public float openAngle = 90f;
    public float speed = 2f;

    private bool open;
    private float current;

    public override void Interact()
    {
        open = !open;
    }

    public override void Update()
    {
        float goal = open ? openAngle : 0f;
        current = Mathf.MoveTowards(current, goal, speed * 90f * Time.deltaTime);
        hinge.localRotation = Quaternion.Euler(0f, current, 0f);
    }
}
```

Unity 和 VRChat 的事件（`Start`、`Update`、`Interact`、`OnPlayerJoined`、`OnPickup` 等）都是 `UdoniteBehaviour` 上的 `virtual` 方法，需要哪个就 `override` 哪个。public 字段会显示在 Inspector 里，并把值带进世界，和 `MonoBehaviour` 完全一样。

普通的 `MonoBehaviour` 也能编译，VRChat 事件同样会触发：一个名为 `OnPlayerJoined(VRCPlayerApi player)` 的方法，无论类是否继承 `UdoniteBehaviour`，都会在有玩家加入时被调用。区别在于，`MonoBehaviour` 上没有可以 `override` 的东西，所以方法名和参数列表必须由你自己精确对上；一旦拼错，它就变成一个永远不会运行的普通方法，而且没有编译器会提醒你。`UdoniteBehaviour` 把每个事件都变成 `virtual` 方法，于是 `override` 会替你抓住这类错误，同时还带来同步字段的特性和 VRChat 辅助方法。

## 编译与附加

Udonite 会在这几个时机自动运行：Unity 重新编译脚本时、场景保存时、以及你按下 Play 时。项目中的每个根 behaviour 都会在同一个 GameObject 上得到一个附加好的 Udon 程序。控制台每次编译输出一行，最后是一句类似 `Compiled 12 scripts.` 的汇总。

你也可以从 Unity 菜单栏的 **Udonite → Compile and Attach** 手动运行。如果你是把一个已经编译过的脚本从 Inspector 拖到 GameObject 上，就需要手动执行一次，因为仅仅修改场景不会触发重新编译。

在 Play 模式下，C# 组件会被禁用，实际运行的是 Udon 程序，所以你在编辑器里测试的，就是世界里运行的。ClientSim 能覆盖本地玩家、拾取和交互，但它不会运行序列化：那些回调在 ClientSim 里不会触发。网络相关的代码请用 **Build & Test** 验证，任何需要真正跨网络的东西请用两个客户端测试。

## 当某个写法被拒绝时

Udon 跑不了的东西，会被明确地按名字拒绝。控制台的那一行会指出具体的写法、给出一个 `UDN` 代码，并告诉你应该改写成什么：

```
Door.cs(14,9): error UDN0011: Udonite does not support 'UnityEngine.GameObject.CompareTag': Udon does not expose 'tag', so there is nothing to compare in a world. Compare 'gameObject.layer' or 'gameObject.name', or hold a reference to the object you mean.
```

被拒绝的 behaviour 不会生成 Udon 程序，但项目中其余的 behaviour 仍然照常编译和附加。程序内部的任何东西都不会被悄悄丢掉。完整的代码列表见[诊断代码](diagnostics.md)（英文）。

## 排除某个脚本

不想被编译的脚本（编辑器辅助工具、永远不会在世界里运行的第三方组件等）可以排除掉。排除项记录在 `ProjectSettings/UdoniteExclusions.txt` 里，一行一条，因此在版本控制中也能干净地合并。无法解析的行会以 `UDN0007` 报出并说明正确写法；如果某个会编译的 behaviour 仍在调用被排除的脚本，则会以 `UDN0008` 报出。

## 上传

照常用 VRChat SDK 的控制面板构建并上传。真正随世界发布的是 Udonite 附加上去的 Udon 程序；C# 会留在你的项目里，供下次编辑使用。

---

中文版只有 README 和本页。其余页面更新频繁，过时的翻译比英文更容易误导人，因此特意保持英文。提交问题反馈时用中文没有问题。
