# Udonite documentation

Udonite compiles ordinary Unity C# to Udon for VRChat worlds. These pages cover what you need to write a behaviour, what the compiler accepts, and what it refuses.

- [Getting started](getting-started.md): install, write a behaviour, press Play, upload. ([日本語](getting-started.ja.md) / [简体中文](getting-started.zh-CN.md) / [한국어](getting-started.ko.md))
- [Language support](language-support.md): what compiles today, and what Udon can never run.
- [Networking](networking.md): synced fields, custom events, typed network events, `NetworkTransform`.
- [Migrating from UdonSharp](migrating-from-udonsharp.md): the changes a port needs.
- [Diagnostics](diagnostics.md): every `UDN` code and what to do about it.

Bugs, refusals you think should compile, and SDK breakage go in [Issues](https://github.com/Udonite/Udonite/issues). Paste the refusal line from the console; it names the exact construct.
