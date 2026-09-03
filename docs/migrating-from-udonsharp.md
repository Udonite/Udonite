# Migrating from UdonSharp

Most UdonSharp scripts are already ordinary C#. A port is usually the base class and a handful of attribute names.

## The mechanical part

| UdonSharp | Udonite |
|---|---|
| `using UdonSharp;` | `using Udonite.Package.Runtime;` |
| `: UdonSharpBehaviour` | `: UdoniteBehaviour` |
| `[UdonBehaviourSyncMode(BehaviourSyncMode.Manual)]` | same name, same enum |
| `[UdonSynced]`, `[UdonSynced(UdonSyncMode.Linear)]` | same |
| `[FieldChangeCallback(nameof(Prop))]` | same |
| `void Start()`, `void Update()`, `void Interact()` | `public override void Start()` and so on |
| `RequestSerialization()` | `this.RequestSerialization()` |
| `SendCustomNetworkEvent(target, "Name")` | `this.SendCustomNetworkEvent(target, nameof(Name))` |
| `SendCustomEventDelayedSeconds("Name", 1f)` | `this.SendCustomEventDelayedSeconds(nameof(Name), 1f)` |
| `VRCInstantiate(prefab)` | `Instantiate(prefab)` |
| `[RecursiveMethod]` | not available; recursion is refused |

Events are `virtual` on `UdoniteBehaviour`, so a Unity or VRChat event method needs `public override`. The compiler tells you when the signature does not match (`UDN0010`).

## What gets easier

The UdonSharp limits you learned to route around mostly do not apply: generics, interfaces, records, LINQ, `async`, coroutines, dictionaries, `foreach` over anything, `switch` expressions, pattern matching, string interpolation with formats, and nullable value types all compile. See [Language support](language-support.md).

## What stays the same

Udon's own limits are Udon's: no exceptions, no static mutable state, no tags, no `Awake`. Udonite refuses those with `UDN0011` and says what to write instead. Ownership, serialization and event rules are VRChat's and unchanged.

## Running both

Udonite and UdonSharp can live in the same project. Udonite only compiles classes that do not derive from `UdonSharpBehaviour`, so existing UdonSharp scripts keep working while you port one at a time.
