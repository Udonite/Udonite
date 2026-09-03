# Diagnostics

Every refusal has a stable `UDN` code. Plain C# errors keep their normal `CS` codes from the C# compiler. Udonite keeps going after the first refusal so one compile shows every problem in a behaviour.

| Code | Meaning | What to do |
|---|---|---|
| `UDN0002` | Unsupported construct. Udonite does not compile this yet. | Rewrite around it, or open an issue with the line; "not yet" refusals get lifted. |
| `UDN0003` | Unsupported type. A type Udon has no representation for. | Use a supported type. An array is supported exactly when its element type is. |
| `UDN0004` | Unsupported program shape. The class as a whole cannot become one Udon program (for example a base type Udon cannot host). | Follow the guidance in the message. |
| `UDN0005` | No Udon extern. The Unity or .NET member you called is not on Udonite's verified list of externs. | If Udon exposes it, open an issue and it is added. If not, the message says so. |
| `UDN0006` | Unresolved root base type. The behaviour derives from something the compiler cannot see. | Check the base class compiles and is in a referenced assembly. |
| `UDN0007` | Malformed exclusion. An entry in the exclusions file cannot be read. | Fix or remove the line in `ProjectSettings/`. |
| `UDN0008` | Excluded code reached. A compiled behaviour calls into a script you excluded. | Un-exclude the script, or remove the call. |
| `UDN0009` | Split exclusion. An exclusion covers only part of what it must. | Exclude the whole folder or the whole script. |
| `UDN0010` | Event signature mismatch. An event override has the wrong parameters. | Match the signature on `UdoniteBehaviour`. |
| `UDN0011` | No Udon equivalent. Udon can never run this. | Write it the way the message suggests. |
| `UDN0012` | Recursion. A method calls itself, directly or through others. | Rewrite as a loop with an explicit stack. |
| `UDN0013` | Heap overflow. The program needs more variables than Udon allows. | Split the behaviour, or reduce large constant arrays. |

`UDN0002` and `UDN0011` are deliberately different codes. One means "not yet", the other means "never"; knowing which tells you whether to wait or to rewrite.

## Reading a refusal

```
Door.cs(14,9): error UDN0005: Udonite does not support 'UnityEngine.AudioListener.volume' yet. ...
```

The file and position point at the exact expression. Paste the whole line into an issue; it names the construct and the extern signature Udonite looked for, which is everything needed to add it.
