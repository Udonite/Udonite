English | [日本語](getting-started.ja.md) | [简体中文](getting-started.zh-CN.md) | [한국어](getting-started.ko.md)

# Getting started

## Install

1. Open the VRChat Creator Companion and go to **Settings → Packages → Add Repository**.
2. Paste `https://udonite.github.io/vpm/index.json` and click **Add**.
3. Open your world project and add **Udonite** from the package list.

Udonite needs the VRChat Worlds SDK 3.10 or newer and Unity 2022.3. It ships its own copy of the C# compiler libraries it depends on, so nothing else has to be installed.

## Write a behaviour

Derive from `UdoniteBehaviour` and write the C# you would write anywhere else in Unity:

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

Unity and VRChat events (`Start`, `Update`, `Interact`, `OnPlayerJoined`, `OnPickup`, and the rest) are `virtual` methods on `UdoniteBehaviour`; override the ones you need. Public fields show in the inspector and carry their values into the world, exactly as they do on a `MonoBehaviour`.

A plain `MonoBehaviour` compiles too, and VRChat events still fire on it: a method named `OnPlayerJoined(VRCPlayerApi player)` runs when a player joins whether or not the class derives from `UdoniteBehaviour`. The difference is that on a `MonoBehaviour` there is nothing to `override`, so the name and parameter list have to match the event exactly by hand; a typo makes an ordinary method that never runs, with no compiler to catch it. `UdoniteBehaviour` turns every event into a `virtual` method, so `override` catches the mistake, and it adds the synced-field attributes and the VRChat helper methods.

## Compile and attach

Udonite compiles on its own whenever Unity recompiles scripts, whenever a scene is saved, and when you press Play. Each root behaviour in the project gets an Udon program attached next to it on the same GameObject. The console shows one line per compile, ending in a summary such as `Compiled 12 scripts.`

You can also run it by hand with **Udonite → Compile and Attach** in the Unity menu bar. Do that after dragging an already-compiled script onto a GameObject in the inspector, since a scene edit alone does not trigger a recompile.

In Play mode the C# component is disabled and the Udon program runs, so what you test in the editor is what runs in the world. ClientSim covers the local player, pickups and interactions, but it does not run serialization: those callbacks stay silent there. Use **Build & Test** for networked code, and two clients for anything that has to cross the network.

## When something is refused

Anything Udon cannot run is refused by name. The console line names the construct, gives a `UDN` code, and says what to write instead:

```
Door.cs(14,9): error UDN0011: Udonite does not support 'UnityEngine.GameObject.CompareTag': Udon does not expose 'tag', so there is nothing to compare in a world. Compare 'gameObject.layer' or 'gameObject.name', or hold a reference to the object you mean.
```

A refused behaviour gets no program; every other behaviour in the project still compiles and attaches. Nothing inside a program is ever silently dropped. See [Diagnostics](diagnostics.md) for every code.

## Excluding a script

A script you do not want compiled (an editor helper, a third-party component that will never run in a world) can be excluded. Exclusions live in `ProjectSettings/UdoniteExclusions.txt`, one entry per line, so they merge cleanly in version control. A line Udonite cannot read is reported as `UDN0007` with the expected form, and a compiled behaviour that still calls into an excluded script is reported as `UDN0008`.

## Upload

Build and upload with the VRChat SDK control panel as usual. The Udon programs Udonite attached are what ships; the C# stays in your project for the next edit.
