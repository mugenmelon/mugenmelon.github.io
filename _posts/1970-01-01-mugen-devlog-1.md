---
title: "Mugen Devlog #1"
tags: gamedev unreal-engine game-design game-architecture devlog
series: Mugen Devlog
series_part: 1
related_tags: game-design game-architecture
diagrams: true
published: true
---

Hi, thank you for visiting and welcome to my first devlog!

I have spent the last 4 years learning Unreal Engine from a software engineer's perspective while building a very early prototype of what will hopefully be a fun action RPG. It took a long time and while I have made many mistakes along the way I feel that the project is at a point where I can start talking about it and sharing progress. This post is a complementary deep-dive into the first devlog on my YouTube channel:

*YouTube embed here*

We will go in-depth on both game design and game architecture, dissecting my way of designing the prototype as a one-man-army game designer & programmer. I apologize in advance for the length of this first devlog, since it contains the results of 4 years of learning. Upcoming devlogs will be a lot smaller and more focused.

For questions or feedback contact me through any of the linked channels. Open to making in-depth tutorials on any topic, so reach out!

---

# Game Design

I am neither a professional game designer, nor well versed in current gaming trends & cultures. I have my set of favorite games and let them influence my game design decisions. Some of my all-time favorites and biggest inspirations include:

* Dragon's Dogma: Dark Arisen
* Monster Hunter Freedom Unite
* Dark Souls: Prepare to Die Edition
* Sekiro
* Final Fantasy VII & IX
* Metal Gear Rising: Revengeance
* Shadow of the Colossus
* Legend of Heroes: Trails in the Sky

At the same time I try to avoid "copying" any game too much. If the reason for a game design decision is "*Because game X does it*" then I avoid making that decision unless I find a good intrinsic reason for it.

---

## Game Design Rules

The way I make the game is very opinionated but follows a few basic rules which align with my views on software engineering as well:

1. **Only work on one thing at a time**  
Getting distracted by *what ifs* leads to disaster.  
When a feature is too large: divide and conquer.

2. **Only work on the currently most important feature**  
Build the core of your game *first*, worry about secondary features later.  
What is most important? Play the game and decide with your gut.

3. **No rigid design document**  
Do not plan ahead, you'll get it wrong.  
Polaris approach: Aim only at your core principles.  
When in doubt: "*You aren't gonna need it*".

4. **Follow the fun**  
"*The game is fun. The game is a battle.  
If it's not fun, why bother? If it's not a battle, where's the fun?*"

---

## Core Design Concepts

Despite following my gut on this, I do have a few "north stars" that I aim at to inform my decisions. The core design concepts I try to follow are:

* **Meaningful exploration, treacherous journeys and cozy sojourns between**
* **The world & its inhabitants should feel alive**
* **Adapting AI partners & enemies**
* **Battle: Knowledge, preparation, strategy, then action**
* **Equipment & items should be scarce and matter**
* **Dynamic gameplay-driven music & audio**
* **Timeless artstyle over realism**

Being fully aware of the potentially massive scope behind them I will leave up for interpretation what these concepts mean *exactly*. My approach is: aim high but be unafraid to miss the mark. That being said let's dive a bit deeper into the current state of the game and its features.

---

## Movement

Currently a very simple implementation of the basic locomotion states:

<pre class="mermaid">
flowchart LR
    start@{ shape: sm-circ }
    idle["`**Idle**`"]:::redNode
    walk["`**Walk**`"]:::yellowNode
    run["`**Run**`"]:::greenNode

    start --> idle
    idle e1@-->|velocity > 0| walk
    walk e2@-->|velocity == 0| idle
    walk e3@-->|has run tag| run
    run e4@-->|no run tag| walk

    e1@{ animate: true }
    e2@{ animate: true }
    e3@{ animate: true }
    e4@{ animate: true }

    {% include mermaid-styles.html %}
</pre>

### Take a Walk

I believe that taking things slow sometimes is an important part of games, especially ones that are heavy in story & world building. That was my reasoning for having a dedicated slow walk. Yes, I am talking about the infamous "RP walk". Aura farming *is* a core game mechanic.

To me there is something inherently wrong with zooming past content as quickly as the game will allow you to. It feels like a sign that the content is just not as interesting as it probably should be. Open world games suffer heavily from this.

### Gotta Go Fast

At first I did not have running at all, but playing became very cumbersome. There is a delicate balance to strike here between slow & methodical vs. fast & aggressive.

By now I have extended the utility of "running" to also affects gameplay: Putting down an object you are carrying *while* running will instead make you throw the object. This gives a bit of depth without overcomplicating the control scheme.

---

## Props

Props are interactable objects in the world. This goes for simple props like barrels, but also [weapons](#weapons) and even the [inventory](#inventory). Some props have attributes such as "health" or "damage".

### Barrelatively Useful

At the moment there is only one *pure* prop in the game: a barrel. It came first because it was the simplest "everyday" object I could come up with for a game world aside from a functionally equivalent crate. This decision bore fruit: The game's entire interaction, state & animation system was prototyped entirely on the basis of picking up, putting down and throwing a barrel.  
And now they can be broken apart once their "health" attribute reaches 0 when attacked.

---

## Weapons

A special type of prop with an obvious use case: combat. I really enjoyed how *Dragon's Dogma* handled weapons, but would like to take it a few steps further.

### Remember the Basics of CQC!

You can equip up to 2 one-hand weapons, or 1 two-handed weapon. When no weapons are equipped, your fists serve as your weapon instead. There are no hard limits on one-handed weapons: You can equip any weapon in either your primary or secondary hand.

Equipping weapons happens entirely in-game with no UI. When you pick up a weapon without having one equipped it will automatically be equipped to a free weapon slot. To exchange your weapon you must drop it or stash it in your inventory to free up the weapon slot again.

Weapons can be sheathed and drawn. In order to attack with your weapon you must draw it beforehand. This is intentional so that the act of drawing your weapon carries strategic weight in battle. Heavy weapons take longer to draw than light ones.

The current flow of how weapons work looks roughly like this, switching between different <span class="blue">states</span> through <span class="red">actions</span>:

<pre class="mermaid">
flowchart
    start@{ shape: sm-circ }
    no-weapon["`**No weapon equipped**`"]:::blueNode
    pickup-weapon["`**Pick up weapon**`"]:::redNode
    has-weapon["`**Weapon equipped**`"]:::blueNode
    draw-weapon["`**Draw weapon**`"]:::redNode
    holding-weapon["`**Holding weapon**`"]:::blueNode
    sheathe-weapon["`**Sheathe weapon**`"]:::redNode
    drop-weapon["`**Drop weapon**`"]:::redNode
    attack["`**Attack**`"]:::redNode

    start --> no-weapon
    no-weapon --> pickup-weapon
    pickup-weapon --> holding-weapon
    holding-weapon --> sheathe-weapon & drop-weapon & attack
    sheathe-weapon --> has-weapon
    has-weapon --> draw-weapon
    draw-weapon --> holding-weapon
    drop-weapon --> no-weapon

    {% include mermaid-styles.html %}
</pre>

In addition to the weapons you have equipped - when they are sheathed - you can pick up and use an additional "temporary" weapon from the battlefield. Currently there are four weapon types available:

* *Fists*: Your default weapon. Fast but low damage.
* *Swords*: Balance between speed & damage. Deadly when dual-wielding.
* *Greatswords*: Really slow but deals serious damage.
* *Lanterns*: Provide light, but can be used as a weapon.

---

## Inventory

Another special type of prop is the inventory. It can store a certain amount of items. That's right: There is no *classic* inventory that the player has. If you want to store items then you need to use an inventory prop such as a backpack. Same rule applies to all NPCs in the game.

Inventories have a limited numbers of item slots by design. The goal is to discourage hoarding and encourage strategic thinking like:

* Which items should I bring on the next journey?
* Should I pick up this item or I leave it and come back later?
* Can I safely leave my backpack here, or does it contain something valuable?

### Backpack It Up!

Backpacks are the first and currently only inventory prop. They can be picked up and dropped like any other prop. Exactly as with weapons you can also "draw" and "sheathe" your backpack in order to view or hide the items within. After drawing your backpack it occupies both of your hands, so you cannot use any weapons while in your inventory!

### Sweet Itemptation

Right now the only items in the game are weapons. After adding it to your inventory an item can be dropped using the inventory UI widget. At present, this will simply respawn the original item wherever you stand.

---

## Game Time

The game also has a simple day night cycle. This is realized using a flexible game-time system under the hood. Each minute in real-time corresponds to an hour in game-time, which means both day and night are each 12 minutes long. Days in a week, weeks in a month and months in a year are also tracked accordingly.

The system is flexible enough to allow any kind of time-tracking imaginable:

* Want to use the term "lunar cycle" instead? Time units can be named freely
* Want 3 days in a week? Define unit relations freely
* Want time to pass slower or faster? Specify a higher/lower multiplier, even while playing
* Want another time on a different planet? Just define another set of time units

---

## AI

My philosophy on AI is:

> Anything the player can do, AI should be able to do.  
> Anything AI can do, the player should be able to do.

Therefore every game design feature I have listed so far is not exclusive to either human or AI players. Anyone can move & navigate the world, interact with props and use weapons & inventories.

The current AI agents autonomously roaming the game world are a simple implementation of the AI system, but use every single feature of the game's AI "tech stack". For fear of getting too technical now, I will only roughly outline how an AI in the game works by answering 3 simple questions.

### What Are All the Things I Can Do?

AI agents receive "impulses" (called "goals") from all kinds of sources. Each goal represents "a thing I can do". One such source is their sense of sight.

For example, when an AI agent sees a barrel it receives a goal to "pick up the barrel". Each agent tracks a list of goals. Some of them are goals that are *always* available as fallback behavior, such as roaming around the world at random.

These goals are individually configurable per AI agent: A different AI agent may never want to pick up a barrel.

### Which of Those Things Should I Do?

Each goal in the list of possible goals calculates a score for itself. The higher the score, the more important the goal. The AI selects the goal with the highest score and tries to execute it.

### How Do I Do the Thing?

Once selected, a goal starts executing its logic: navigating the character and interacting with the world. How granular a goal is and what technology is used to implement it is up to the designer. I personally prefer using small, modular goals and behavior trees for this.

---

# A Brief Respite

If you have read this far: thank you for your interest and attention.

A friendly warning: the road ahead is treacherous and brimming with monsters. We will dive into game architecture and outline a few technical concepts to understand how things are working under the hood and why they work the way they do.

---

# Game Architecture

As a software engineer this is a topic that I am passionate about. I will often go the extra mile to ensure that the pieces of a feature fit neatly into the larger puzzle, even if it does not represent tangible progress on the game itself. Granted, sometimes I take it too far. Creating a devlog with regular updates is my way of restraining that side of me.

> Why should anyone care about game architecture? If something works, it works, right?

I can 100% understand if that is your stance and you are probably right. If your goal is to get a game out of the door: Do not worry too much about architecture. Build the thing and be done with it. The experience will be much more valuable than any architecting you could have done.

But I am a fool who doesn't know better. I work alone with no financial dependencies, so I can get away with more foolery than others. My personal goals for this project are different. To me the most important benefits of maintaining a clean game architecture are:

* **A more fun development experience**  
Nobody likes having to maintain spaghetti code, not even when you're the "cook".  
But when the puzzle pieces fall into place *just right*, game development becomes a lot more fun.

* **Lower complexity when done right**  
Let's face it: We're not getting any younger or smarter.  
The less you have to think about code, the more creative you can be.  
But it takes experience to know what makes something less vs. more complex.

* **Feature scalability**  
I want to add more features and find creative solutions to game design problems.  
Getting blocked by tech is frustrating. Creative freedom is the goal.

* **Emergent systems**  
Modular code and clean boundaries often work in unexpected ways.  
Sometimes they solve problems you didn't even think you had.  
At other times the one complex feature you had planned is suddenly very easy to build.

Your mileage may vary of course. Game architecture is a topic that should always be taken with a large grain of salt. It is crucial that you make your own mistakes and experiences and decide for yourself what is right for your game.

---

## Code Modules

I am making heavy use of modules in my project to promote decoupling and cohesion. From what I have gathered so far, modules seem to generally fall into one of three categories:

* **Library modules**  
Provide shared utility classes & functions for use in other modules or blueprints.

* **Engine extension modules**  
Extend existing engine classes & methods but may also hold relevant library code.

* **Game feature modules**  
Entirely new feature that does not exist in the engine.

What follows is a quick overview of current code modules and their respective functionalities.

### Library modules

* **MugenCore**  
Core library module.  
Generic utility code that does not fall into any particular functional domain.

### Engine extension modules

* **MugenAbility**  
Extension of the GameplayAbilies plugin (GAS).  
Provides an extended `UMugenAbility`, `UMugenAbilityComponent` and improves event communication to and from other engine systems such as AnimNotify, BehaviorTree and EnhancedInput.

* **MugenAI**  
Extension of generic AI features.  
Exposes some internal details with an extended `UMugenAIPerceptionSystem` to allow sense introspection and simplifies event communication to AI BrainComponents.

* **MugenAnimation**  
Extension of generic animation features.  
Enables event communication from animation assets using an extended `UMugenSkeletalMeshComponent` and provides utility functions to synchronize animations and manage their lifecycle.

* **MugenBehaviorTree**  
Extension of BehaviorTrees.  
Provides extended functionality for BehaviorTree nodes with `UBTDecorator_MugenBlackboardBase`, `UBTService_MugenBlackboardBase`, `UBTMugenTaskNode` and implements additional generically useful nodes.

* **MugenFramework**  
Extension of Unreal Engine's framework.  
Provides lightweight extensions of core framework classes like `AMugenPawn`, `AMugenPlayerController`, `AMugenPlayerState` etc. Avoids any usage of `ACharacter`.

* **MugenInput**  
Extension of EnhancedInput plugin.  
Implements a few useful InputModifiers and exposes utility functions to blueprints.

* **MugenMovement**  
Extension of Movement components.  
Provides extended `UMugenMovementComponent` and `UMugenPawnMovementComponent` and avoids any usage of `UCharacterMovementComponent`.

* **MugenStateTree**  
Extension of StateTree plugin.  
Provides generically useful StateTree nodes (conditions, functions, tasks) and enhances the capabilities of StateTrees as general-purpose hierarchical state machines with `UMugenStateTreeComponent` and `FSTTask_State`.

* **MugenTargeting**  
Extension of TargetingSystem plugin.  
Adds more configurable options to sweep trace targeting with `UTargetingSelectionTask_MugenTrace` and exposes targeting results to blueprints and GameplayAbilities.

* **MugenUI**  
Extension of UI widgets.  
Provides an extended `UMugenUserWidget` with a simple "focus" implementation and other extended UI atomics such as `UMugenButton` and `UMugenListView`.

### Game feature modules

* **MugenAIGoal**  
Implementation of the aforementioned "AI goal" utility algorithm.  
Provides a `UAIGoal` and subclasses to execute BehaviorTree goals as well as `UAIGoalSelectionComponent` to enable autonomous selection.

* **MugenEquipment**  
Implements a generic equipment system.  
Provides `UEquipmentComponent` and uses GameplayTags to configure equipment slots, their behavior and to expose relevant functionality to other parts of the engine.

* **MugenHitbox**  
Implements generic hitboxes for e.g. applying damage.  
Provides `UHitbox` data asset and related functions to configure, target and apply hitboxes. Mainly for use in GameplayAbilities.

* **MugenInventory**  
Implements a generic inventory system.  
Provides `UInventoryComponent` for configurable per-actor item storage and `UItem` data asset to create item definitions for any object.

* **MugenStateFragment**  
Implements a configurable map of gameplay logic for use in state machines.  
Provides `UStateFragmentMap` data asset to enable control of core gameplay logic to state machines, e.g. granting abilities, applying effects, playing montages etc. Also solves the "state explosion" problem when used with UE's Chooser plugin.

---

## Game Data Flow

The way data flows through the game illustrates its architecture in several layers. Let us first look at how a human player's data (their input) flows from the input device all the way to cause some action in-game, and then see how it works for AI agents.

### Player Controller Layer

<pre class="mermaid">
flowchart
    input-device["`
        **Input Device**
        Gamepad, keyboard, mouse
    `"]:::yellowNode

    subgraph player-controller-layer["`**Player Controller Layer**`"]
        enhanced-input["`
            **Enhanced Input**
            Filters and modifies two types of input data: *raw* input and *action* input.
        `"]:::yellowNode
        player-state["`
            **Player State**
            Holds a player's current state, e.g. which game component is in focus.
        `"]:::blueNode
        player-controller["`
            **Player Controller**
            In-game representation of a human player.
        `"]:::blueNode

        subgraph possession-comps["`**Possession Components**`"]
            camera-spring-arm-comp["`**Camera Spring Arm Component**`"]
            inventory-widget-comp["`**Inventory Widget Component**`"]
        end
    end

    subgraph pawn-layer["`**Pawn Layer**`"]
        pawn["`
            **Pawn**
            The *Pawn* currently possessed by the *Player Controller*.
        `"]:::blueNode
    end

    input-device e1@==> enhanced-input
    enhanced-input e2@==> player-controller
    player-controller --o player-state
    player-controller e3@===> camera-spring-arm-comp
    player-controller e4@===> inventory-widget-comp
    player-controller e5@====> pawn

    e1@{ animate: true }
    e2@{ animate: true }
    e3@{ animate: true }
    e4@{ animate: true }
    e5@{ animate: true }

    {% include mermaid-styles.html %}
</pre>

The input going from the **Input Device** is filtered and modified by the **Enhanced Input** plugin before being handed over to the **Player Controller**. *Raw* input is directly forwarded to **Possession Components** or other downstream components. *Action* input sends a gameplay event to the possessed **Pawn**, triggering gameplay abilities.

**Possession Components** are components that are added to a **Pawn** when possessed by the **Player Controller** and removed again when unpossessed. This keeps **Pawn** classes agnostic of "player components" and enables the player to transfer player functionality when switching control to *other* **Pawns**, such as the camera or UI widgets.

As per Unreal Engine convention the **Player State** serves its namesake by storing a player's current state, keeping this responsibility away from the **Player Controller** which orchestrates input.

### Pawn Layer (Model)

<pre class="mermaid">
flowchart
    subgraph player-controller-layer["`**Player Controller Layer**`"]
        player-controller["`
            **Player Controller**
            In-game representation of a human player.
        `"]:::blueNode
    end

    subgraph pawn-layer["`**Pawn Layer**`"]
        pawn["`
            **Pawn**
            The *Pawn* currently possessed by the *Player Controller*.
        `"]:::blueNode
        subgraph pawn-components["`**Pawn Components**`"]
            ability-comp["`
                **Ability System Component**
                Triggers abilities from incoming events and owns gameplay attributes & tags.
            `"]:::redNode
            ability["`
                **Ability**
                An ability triggered by gameplay event. Orchestrates *Pawn* components for gameplay logic e.g. PickUp, Sheathe.
            `"]:::redNode
            movement-comp["`
                **Movement Component**
                Collects movement input from *Player Controller* and moves the *Pawn* in the world.
            `"]:::blueNode
            equipment-comp["`
                **Equipment Component**
                Holds equipments slots and currently equipped objects. Broadcasts equipment events.
            `"]:::blueNode
        end

        subgraph inventory-actor-group["`**Inventory Actor Group**`"]
            inventory-actor["`
                **Inventory Actor**
                An equippable actor holding an *Inventory Component*.
            `"]:::blueNode
            inventory-comp["`
                **Inventory Component**
                Holds item slots and currently stored items.
            `"]:::blueNode
        end
    end

    player-controller e1@==> pawn
    pawn e2@==> ability-comp
    pawn --o movement-comp
    pawn --o equipment-comp

    equipment-comp ---o inventory-actor
    inventory-actor --o inventory-comp

    ability-comp e3@==> ability

    e1@{ animate: true }
    e2@{ animate: true }
    e3@{ animate: true }

    {% include mermaid-styles.html %}
</pre>

### Pawn Layer (ViewModel)

### Pawn Layer (View)

---
