---
title: (Mis)understanding State Machines
tags: gamedev game-architecture analysis
related_tags: game-architecture
diagrams: true
published: true
---

In this article I outline a common (mis)understanding of finite state machines (FSMs) - particularly regarding game development - 
and contrast it with what I think is a more useful and elegant way to conceptualize them.

## Unfortunate Examples

A few quick Google searches for FSMs in game development reveal among others the following descriptions:

> Finite State Machines (FSM) are often used while programming in order to allow for more complex series of actions. [^1]

> State machines help you untangle hairy code by enforcing a very constrained structure on it. [^2]

> Thanks to a finite state machine we can define complex behaviors and encapsulate them into mini single interactions which we will call state. [^3]

A more general search for the original Gang of Four's "state design pattern" shows that

> The state pattern is a behavioral software design pattern that allows an object to alter its behavior when its internal state changes. [^4]

And even Unreal Engine's documentation for their FSM equivalent states that

> StateTree is a general-purpose hierarchical state machine that combines the Selectors from behavior trees with States and Transitions from state machines. With StateTree, you can create highly performant logic that stays flexible and organized. [^5]

When searching for concrete examples/tutorials for how to use this pattern in game development we are often met with some variation of the classic "enemy AI chasing the player" use case:

<pre class="mermaid">
---
title: Chase player and attack
---
stateDiagram-v2
    [*] --> Idle
    Idle --> Chase: can see player
    Chase --> Attack: player is in attack range
    Chase --> Idle: lost sight of player
    Attack --> Chase: player is out of attack range
```
</pre>

## Footnotes

[^1]: Source: [Game Manual 0](https://gm0.org/en/latest/docs/software/concepts/finite-state-machines.html)

[^2]: Source: [Game Programming Patterns](https://gameprogrammingpatterns.com/state.html) (great read!)

[^3]: Source: [Game Developer Tips](https://gamedevelopertips.com/finite-state-machine-game-developers/)

[^4]: Source: [Wikipedia](https://en.wikipedia.org/wiki/State_pattern)

[^5]: Source: [Unreal Engine Documentation](https://dev.epicgames.com/documentation/en-us/unreal-engine/state-tree-in-unreal-engine)
