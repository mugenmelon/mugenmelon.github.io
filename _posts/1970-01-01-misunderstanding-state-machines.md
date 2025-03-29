---
title: (Mis)understanding State Machines
tags: gamedev game-architecture analysis
related_tags: game-architecture
diagrams: true
---

In this article I outline a common (mis)understanding of finite state machines (FSMs) - particularly regarding game development - 
and contrast it with what I think is a more useful and elegant way to conceptualize them.

## Putting the Cart Before the Horse...

Let us first have a quick look at some common descriptions of FSMs that I have totally not cherry-picked for my own purposes.

### What You Already Know

A few quick Google searches for FSMs in game development reveal among others the following descriptions:
> Finite State Machines (FSM) are often used while programming in order to allow for more complex series of actions. [^1]  
> State machines help you untangle hairy code by enforcing a very constrained structure on it. [^2]  
> Thanks to a finite state machine we can define complex behaviors and encapsulate them into mini single interactions which we will call state. [^3]

A more general search for the original Gang of Four's "state design pattern" shows that
> The state pattern is a behavioral software design pattern that allows an object to alter its behavior when its internal state changes. [^4]

And even Unreal Engine's documentation for their FSM equivalent (StateTree) states that
> StateTree is a general-purpose hierarchical state machine that combines the Selectors from behavior trees with States and Transitions from state machines. With StateTree, you can create highly performant logic that stays flexible and organized. [^5]

When searching for concrete examples/tutorials on how to apply this pattern to game development we are often met with some variation of the classic "enemy AI chasing the player" use case:

<pre class="mermaid">
---
title: Chase player and attack
---
stateDiagram-v2
    direction LR

    [*] --> Idle: start
    Idle --> Chase: gained sight of player?
    Chase --> Attack: player in range?
    Chase --> Idle: lost sight of player?
    Attack --> Chase: player out of range?
</pre>

### So What Is Wrong With This?

I believe all of the above examples (and many others) share one thing in common: *They are putting the cart before the horse*. They focus on the *outcome of applying the pattern* over the arguably more important *application of the pattern itself*.

While it is true that FSMs provide all of the above benefits such as:
- Reduced complexity
- Cleaner code
- Encapsulation
- Flexibility & organization

They only do so when *applied correctly* (both *technically* & *functionally*). Otherwise there is a large risk of exactly the opposite: A large, complex "god" FSM as an obstacle that programmers have to avoid at all costs. This is where I believe much of the bad reputation[^6] of FSMs stems from.

What tends to follow is that a beginner programmer takes the provided examples at face value without grasping the fundamental conditions under which FSMs provide all of these benefits. And it may work just fine at first, but eventually falls apart as soon as any real complexity is involved.[^7]

What also tends to follow is that a know-it-all "professional" attempts to implement this pattern in a large enterprise platform and completely misses the mark, resulting in hundreds of thousands of € in missed revenue and a very disgruntled team of developers that has to clean up his mess after he was let go.[^8]

## ...And Now the Horse Before the Cart

For this section I will presume a very basic understanding of FSMs and their building blocks: *states* and *transitions*. However, I will make one adjustment to the terminology to be more accurate. *States* are henceforth called *state-labels*. This is to avoid confusion with the *program state*, i.e. the set of all variables and their current values also called "data". Another important reason will be revealed at a later point.

### To Be or Not To Be?

Let us begin where it hurts the most: "Do I even *need* a state machine?"

FSMs - like most programming patterns - are but one tool in your toolbelt. We must not confuse every problem for a nail just because what we have is a hammer. Sometimes the indirection of an FSM is too much overhead. Sometimes you cannot afford to use an FSM just because the set of all possible state-labels and transitions is too small and will never grow larger.

A commonly used example is the classic:

<pre class="mermaid">
---
title: Train station turnstile
---
stateDiagram-v2
    direction LR

    [*] --> Locked
    Locked --> Unlocked: fee paid?
    Unlocked --> Locked: passenger passed?
</pre>

The turnstile unlocks as soon as the fee is paid and locks again once the passenger has passed through it. Do you see a problem?

The entire FSM can be reduced to a single boolean variable `locked = true|false`. We are no better served implementing an FSM for this than to simply set and query the `locked` variable in the appropriate functions in the turnstile's program. And it is highly unlikely that the number of state-labels or transitions will ever increase.

Having outlined the lower bound of when not to use an FSM now leaves us with the question: "When *do* we use an FSM?".

### Reduce

I mentioned that the FSM can be "reduced to a single boolean variable" which is actually a critically important detail. A reduction in one direction logically means an increase in the other. So are FSMs just cursed to be a bloated pattern? Do they always increase the complexity of our projects rather than reduce it? What exactly determines whether something is a reduction vs. an increase?

The answer is *cardinality*. The amount of something. A simple boolean variable is less effort, less maintenance and more flexibility than an equivalent FSM in virtually all cases. Were we to add another variable would it then be worth it to use this pattern?

Sometimes this can be guesstimated by looking at the cartesian product of the set of all relevant states. This is also called *state space*. In our turnstile example the boolean variable has a state space of 2 because it can assume a set of 2 possible values `[true,false]`. The FSM also has a state space of 2 because it has a set of 2 state-labels `[locked,unlocked]`. So going from 2 to 2 is not a reduction in cardinality and it can even be an increase due to the overhead an FSM brings.

That still does not answer the question when an FSM *should* be used, but at least gives us a hint about the trend we need to follow...

### High Cardinality State Space

Videogames uniquely are full of *high cardinality state space*[^9], so let us make our turnstile into a digital one:

>Hi Mark, the turnstile you've built is working well! But the playtesters find it somewhat annoying to have to pass through it each time. Can you make it so that the turnstile can be destroyed when you shoot it with your gun? Then the player can just pass through it without having to pay the fee and play an animation.

>Hi Stacy, no problem. I will add a "health" attribute to the turnstile. The code for damage calculation is already implemented, so this seems like the most straightforward approach. Let's see... 100 health should work for now.

The assignment is pretty clear. Our previous turnstile now needs a variable `health` which is clamped between 0 and 100:

```c++
// Whether the gate is locked.
bool locked = true;
// Current health of the turnstile.
float health = 100;
...
// When health is 0 unlock & destroy turnstile.
if (health <= 0)
{
    locked = false;
    playDestroyAnimation();
}
```

OK, cool, this works just fine. But wait... the state space has increased to a whopping `[true,false] x [0,1,...,99,100] = 202`![^10] I can do this in an FSM with just 3 states `[locked,unlocked,destroyed]`! Surely this must mean that I have to use an FSM now, right?

## Footnotes

[^1]: Source: [Game Manual 0](https://gm0.org/en/latest/docs/software/concepts/finite-state-machines.html)  
[^2]: Source: [Game Programming Patterns](https://gameprogrammingpatterns.com/state.html) (great read!)  
[^3]: Source: [Game Developer Tips](https://gamedevelopertips.com/finite-state-machine-game-developers/)  
[^4]: Source: [Wikipedia](https://en.wikipedia.org/wiki/State_pattern)  
[^5]: Source: [Unreal Engine Documentation](https://dev.epicgames.com/documentation/en-us/unreal-engine/state-tree-in-unreal-engine)  
[^6]: Just google "are state machines bad"  
[^7]: A common obstacle is "state explosion", which is a subset of [combinatorial explosion](https://en.wikipedia.org/wiki/Combinatorial_explosion)
[^8]: Based on a true story  
[^9]: I.e. a lot of floating point numbers  
[^10]: Don't forget to count the zero!

