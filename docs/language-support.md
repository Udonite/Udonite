# Language support

Udonite compiles a large slice of everyday C#. Every construct below is covered by tests that assemble the output and run it on the real Udon virtual machine.

## What compiles

**Types and members**
- Classes, structs, interfaces, enums, nested types, and your own classes as fields and locals.
- Generic classes and methods, with constraints.
- Interfaces, including default interface methods.
- Records and record structs: positional construction, `with`, value equality, `ToString`, deconstruction.
- Properties, auto-properties, expression-bodied members, indexers, extension methods.
- `params`, optional parameters, named arguments, `out` and `ref` parameters, tuples and deconstruction, local functions.
- Static helper classes with methods and constants. Static fields must be `const` or `readonly` (see below).

**Statements and expressions**
- `if`, `switch` statements and switch expressions, `for`, `foreach`, `while`, `do`, `break`, `continue`.
- Pattern matching: type patterns, `is not null`, relational patterns (`is > 5 and < 10`), property patterns, `when` guards.
- `?.`, `??`, `??=`, string interpolation with format specifiers and alignment, `nameof`.
- Arithmetic, bitwise and shift operators on all the primitive types, `[Flags]` enums, `HasFlag`, `char` arithmetic.
- Arrays, multidimensional arrays, ranges and indices (`arr[^1]`, `arr[1..3]`), `Array.Sort`, `Find`, `FindAll`, `Exists`, `IndexOf`, `Resize`, `Copy`, `Fill`, `ConvertAll`.
- `List<T>`, `Dictionary<K,V>`, `HashSet<T>`, `Queue<T>`, `Stack<T>` and their usual members.
- LINQ over arrays, lists and dictionaries: `Select`, `Where`, `OrderBy`, `ThenBy`, `GroupBy`, `First`, `Any`, `All`, `Sum`, `Average`, `Min`, `Max`, `Count`, `Take`, `Skip`, `TakeWhile`, `SkipWhile`, `Distinct`, `Reverse`, `Concat`, `Zip`, `Union`, `Intersect`, `Except`, `SequenceEqual`, `ToArray`, `ToList`, `ToDictionary`, `Range`, `Repeat`, and chains of them.
- Lambdas passed to LINQ and to `List<T>`/`Array` helpers are inlined at the call site.
- Delegates: `Action` and `Func` fields and locals, method groups (`Action done = Cleanup;`), multicast `+=` and `-=`, C# `event` declarations with `?.Invoke`, delegates to a public method on another behaviour, and lambdas as values (see the capture rule below).
- `StringBuilder`, `string.Format`, `Split`, `Join`, `Substring`, `Replace`, `Trim`, `PadLeft`, and the other common string members.
- `int?` and other nullable value types: `HasValue`, `Value`, `GetValueOrDefault`, `??`.

**Unity and VRChat**
- `GetComponent<T>()` and its variants, `TryGetComponent<T>(out T)`, `Instantiate`, `Destroy`, `SetActive`, transforms, physics queries, `Mathf`, `Vector3`, `Quaternion`, `Color`, `Random`, `Time`.
- Coroutines: `IEnumerator` methods with `yield return null`, `WaitForSeconds`, `WaitUntil`, nested coroutines, `StartCoroutine(Run())`, `StartCoroutine(nameof(Run))`, `StopAllCoroutines`.
- `Invoke`, `InvokeRepeating`, `CancelInvoke` for public methods.
- `async` methods returning `void`, `Task`, `UniTask` or `UniTaskVoid`, with `await Task.Delay(...)` and awaiting other async methods.
- `VRCPlayerApi`, `Networking`, pickups, stations, `UdonBehaviour` references with `SendCustomEvent`, `SetProgramVariable`, `GetProgramVariable<T>`.
- Everything in [Networking](networking.md).

## What Udon can never run

These are refused with `UDN0011`. The refusal says so, and says what to do instead.

- `try`/`catch`/`throw`: Udon has no exceptions. Check the condition first.
- Mutable `static` fields: every behaviour has its own heap, so a static would be one value per behaviour rather than one shared value. Use a field on one behaviour and reference it.
- `Awake`: Udon does not dispatch it. Use `Start`.
- `CompareTag` and `gameObject.tag`: Udon does not expose tags. Compare layers or names, or hold a reference.
- `IsInvoking`, `Application.isPlaying`, `Time.timeScale`, `AudioListener`, `new GameObject()`, `ScriptableObject` subclasses: not exposed by Udon.
- Serialized arrays of your own classes or structs in inspector fields: Udon cannot deserialize them. Use arrays of primitives, Unity types, or references to behaviours.
- A delegate pointing at a method on a plain object (not a behaviour), and a delegate in a public field Unity would serialize: neither survives the trip into a world.
- A capturing lambda kept beyond the method that created it (assigned to a field, returned, passed as an argument, or created inside a loop): the captured variables live in the behaviour's single heap, so a stored closure would see stale state on the next call. Non-capturing lambdas can be stored freely; capturing ones work as locals called within the same method.

## Not yet

Refused with `UDN0002`. These are gaps in Udonite rather than in Udon, and the ones people ask for get lifted.

- List patterns (`is [1, ..]`).
- `typeof(T)` as a free expression, and the non-generic `GetComponent(typeof(T))`.
- Recursion (`UDN0012`), capturing lambdas inside loops, `checked` arithmetic.

If a refusal blocks you, open an issue with the refusal line. The construct is usually a day's work once it is named.
