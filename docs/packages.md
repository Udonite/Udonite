# Packages

The Udonite compiler is free and always will be. The packages built on it are for supporters.

**Nothing has shipped yet, and the list below is not settled.** These are the ideas under consideration, written down so you can see the direction before deciding whether to support, rather than after.

Which of them get built, in what order, and whether something else replaces them entirely is still being worked out. If VRChat ships something natively that covers one of these well, it gets dropped in favour of something more useful.

## Under consideration

### Players

A player registry: who is here, join and leave events, per-player data, master tracking, VR versus desktop, index-stable lists.

Every world reimplements this, usually with a fixed-size array and a bug when somebody leaves mid-frame. It is the most copy-pasted code in VRChat.

### Interactions

The set everyone needs: buttons, toggles, doors, drawers, teleporters, seats, and pickups with ownership handled. Networked, and correct for late joiners.

Nobody wants to write door number four hundred.

### Debug

An in-world console: a log view readable in VR, variable watches on behaviours, and a network panel showing owners and sync traffic.

Debugging a world you have already uploaded is genuinely miserable.

### Persistence

A typed layer over VRChat's player persistence. Named typed values instead of stringly-typed calls, a versioned schema, and a migration path for when you add a field in v2 while players still hold v1 data.

The migration story is the part that hurts to retrofit.

### Tween

Move, rotate, scale, fade and colour over time, with easing, chaining and callbacks.

Today people hand-roll `Lerp` in `Update`, or drag in animation clips for things that should be three lines.

## Supporting

Supporting today backs the direction rather than buying a specific package, because none of them exist yet. If that is not what you want, the compiler is free and always will be. Take it and owe nothing.

Udonite is built and maintained by one person. Every VRChat SDK update that breaks something is fixed by the same person who wrote the compiler, and supporting is what makes that sustainable.

Supporters get each package as it ships, and all updates while subscribed. If you stop, you keep every version you have already installed.

[Become a supporter](https://buy.polar.sh/polar_cl_fiMBXRqzbvm0c8qt6xqKKBGbaB0FOQmPMddgn06PRsC)
