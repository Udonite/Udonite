English | [日本語](README.ja.md) | [简体中文](README.zh-CN.md) | [한국어](README.ko.md)

# Udonite

**VRChat world scripting that feels like normal Unity C#.**

Write a `MonoBehaviour` and Udonite compiles it to Udon. No dialect to learn and no list of forbidden keywords to memorise: the C# you already write in Unity is the C# that runs in your world.

Udonite is free, networking included.

Worlds can call [Udonite's hosted services](https://admin.udonite.com) too: send a message from a world into
Discord or Pushover, and keep a counter, a score table or a shared switch that survives everyone leaving.
Free to try with a GitHub account.

**[Documentation](https://docs.udonite.com/compiler)** · **[Install](https://udonite.com)**

## Use

Write a `MonoBehaviour` and save. Udonite compiles whenever Unity recompiles scripts or a scene is saved, attaches the Udon program next to each behaviour, and says nothing unless it refuses something. In Play mode the Udon program is what runs, so what you see in the editor is what ships.

Deriving from `UdoniteBehaviour` instead is optional, and a little easier: VRChat events become `virtual` methods you `override`, so a misspelt event name is a compile error rather than a method that never runs, and the synced-field attributes and VRChat helper methods come with it.

## What you can write

- **The language, not a subset.** Generics with constraints, interfaces with default methods, records, structs, nested classes, extension methods, tuples and deconstruction, local functions, `params` and optional arguments.
- **Modern C# syntax.** Switch expressions, pattern matching (`is > 5 and < 10`, property patterns, `is not null`), `?.`, `??=`, string interpolation with format specifiers, ranges and indices.
- **Collections and LINQ.** `List<T>`, `Dictionary<K,V>`, `HashSet<T>`, `Queue<T>`, `Stack<T>`, multidimensional arrays, and LINQ chains (`Where`, `Select`, `OrderBy`, `GroupBy`, `Zip`, and the rest) with the lambdas compiled in place.
- **Delegates and events.** `Action` and `Func` fields, method groups, `+=` and `-=` on multicast delegates, C# `event` declarations with `?.Invoke`, lambdas as values.
- **Time, the normal way.** Coroutines with `yield return new WaitForSeconds(...)`, `async` methods with `await Task.Delay(...)`, `Invoke` and `InvokeRepeating`.
- **Networking with payloads.** Synced fields with change callbacks, custom events by name, and typed network events: declare a class with fields, send it with `Network.Send(evt, receiver)`, receive it in a method that takes the event as a parameter. A `NetworkTransform` component syncs position, rotation and scale out of the box.
- **Nullable values, `TryGetComponent`, `StringBuilder`, `Array.FindAll`,** and the other small things you reach for without thinking.

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

## What Udonite will not do

Udon has real limits: nothing to `catch`, no mutable statics, no `Awake`, no tags. Udonite refuses those by name. Every refusal is a console line with a `UDN` code that says what was refused and what to write instead; nothing is dropped silently. A refused behaviour gets no program, the rest of your project still compiles. Details in [Language support](https://docs.udonite.com/compiler/language-support).

## Supporting Udonite

The Udonite compiler is free and always will be. It is built and maintained by one person, through
every VRChat SDK update that breaks something.

Supporting funds that work, and it now gets you something a world can use. Udonite runs hosted
services your worlds call by URL: send a message from a world into Discord or Pushover, and keep a
counter, a score table or a shared switch that survives everyone leaving. Both are free to try, and
supporting raises every limit to somewhere you will not meet them.

The compiler is never part of that. It is free to everyone either way, and fixes for SDK breakage
ship to everyone the same day.

[Become a supporter](https://buy.polar.sh/polar_cl_fiMBXRqzbvm0c8qt6xqKKBGbaB0FOQmPMddgn06PRsC) · [What the services do](https://udonite.com)

## Issues

Bugs, refused constructs you think should compile, and SDK breakage go in [Issues](https://github.com/Udonite/Udonite/issues). Include the refusal line from the console; it names the exact construct.

Udonite is a third-party tool and is not affiliated with VRChat Inc.
