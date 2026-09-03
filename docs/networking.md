# Networking

Udonite uses VRChat's own networking underneath. The same rules apply as with any Udon behaviour: only the owner of a GameObject writes its synced state, and only a public method can be called across the network.

## Synced fields

Mark a field with `[UdonSynced]` and set the behaviour's sync mode:

```csharp
using Udonite.Package.Runtime;
using UnityEngine;
using VRC.SDKBase;
using VRC.Udon.Common;

[UdonBehaviourSyncMode(BehaviourSyncMode.Manual)]
public class Counter : UdoniteBehaviour
{
    [UdonSynced] public int count;

    public override void Interact()
    {
        if (!Networking.IsOwner(gameObject))
            Networking.SetOwner(Networking.LocalPlayer, gameObject);

        count = count + 1;
        this.RequestSerialization();
    }

    public override void OnDeserialization(DeserializationResult result)
    {
        Debug.Log("count is now " + count);
    }
}
```

- `BehaviourSyncMode.Manual` sends when you call `this.RequestSerialization()`. `Continuous` sends on its own. `None` syncs nothing.
- `[UdonSynced(UdonSyncMode.Linear)]` and `Smooth` interpolate a numeric field on the receiving side.
- `OnPreSerialization`, `OnPostSerialization` and `OnDeserialization` fire around each transfer.

### Reacting to a change

`[FieldChangeCallback]` names a property whose setter runs whenever the field's synced value arrives:

```csharp
[UdonSynced, FieldChangeCallback(nameof(Lit))]
private bool lit;

public bool Lit
{
    get => lit;
    set
    {
        lit = value;
        lamp.SetActive(value);
    }
}
```

The property's type must match the field's exactly.

## Custom events

Any public method can be sent to other clients by name:

```csharp
this.SendCustomNetworkEvent(NetworkEventTarget.All, nameof(Ring));

public void Ring()
{
    bell.Play();
}
```

`NetworkEventTarget.Owner` sends only to the object's owner. `SendCustomEventDelayedSeconds` and `SendCustomEventDelayedFrames` schedule a local call later.

## Typed network events

For a payload with structure, declare an event class with a `Serialize`/`Deserialize` pair and a handler on the receiving behaviour:

```csharp
using Udonite.Package.Runtime;
using UnityEngine;

public class ScoreEvent : NetworkEvent
{
    public int player;
    public int score;

    public void Serialize(ByteWriter writer)
    {
        writer.WriteInt32(player);
        writer.WriteInt32(score);
    }

    public void Deserialize(ByteReader reader)
    {
        player = reader.ReadInt32();
        score = reader.ReadInt32();
    }
}

public class Scoreboard : UdoniteBehaviour
{
    public void Award(int player, int score)
    {
        ScoreEvent evt = new ScoreEvent();
        evt.player = player;
        evt.score = score;
        Udonite.Network.Send(evt, this);
    }

    [NetworkEventHandler]
    public void OnScore(ScoreEvent evt)
    {
        Debug.Log("player " + evt.player + " scored " + evt.score);
    }
}
```

`Udonite.Network.Send(evt, receiver)` serializes the event and delivers it to the `[NetworkEventHandler]` method on `receiver` that takes that event type, on every client. Write it fully qualified: with `using UnityEngine;` in scope, a bare `Network` is ambiguous with Unity's obsolete `UnityEngine.Network`. It is carried by synced fields, so it reaches other clients only when called by the owner of the receiver's GameObject.

## NetworkTransform

`NetworkTransform` is a ready-made behaviour that syncs position, rotation and scale with interpolation and teleport detection. Add it to the object, compile, and move the object; with a `VRC_Pickup` on the same object, picking it up takes ownership and dropping it sends the resting pose. Send rate, per-channel thresholds and the teleport distance are inspector fields.

## Ownership

`OnOwnershipRequest` can be overridden to return `false` and refuse a transfer. `OnOwnershipTransferred` fires on every client when the owner changes. ClientSim does not fire ownership requests; test those in an uploaded world.
