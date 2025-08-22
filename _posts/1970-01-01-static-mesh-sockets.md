---
title: "Why You Should Be Using Static Mesh Sockets"
tags: gamedev unreal-engine gameplay animation c++ blueprint
published: true
---

Static mesh sockets are one of Unreal Engine’s most overlooked features. While most developers are familiar with skeletal mesh sockets for attachments, static mesh sockets open the door to smarter workflows, cleaner code, and more flexible design tools. In this article, we’ll explore how to use them for attachments, animation, hitboxes, and beyond - turning a simple concept into a powerful data-driven system that makes your game easier to build and more fun to design.

{% include toc.md %}

# What Are Static Mesh Sockets?

If you've spent any amount of time in Unreal Engine you are likely already familiar with *skeletal mesh sockets*.

![Skeletal mesh socket example]({{ '/assets/images/posts/static-mesh-sockets/skeletal-mesh-socket-example.png' | relative_url }})

Sockets in principle are transforms (location, rotation & scale) declared at design-time in a "relative-to-something" space.
In the case of skeletal mesh sockets they define a transform relative to a skeletal mesh.
It is no surprise then that *static mesh sockets* define a transform relative to a static mesh.

![Static mesh socket example]({{ '/assets/images/posts/static-mesh-sockets/static-mesh-socket-example.png' | relative_url }})

To keep things simple let us set aside a few (but important) differences between skeletal mesh and static mesh sockets for now and only talk of static mesh sockets as per our definition:

> A relative-to-something transform declared at design-time.

What I will show here applies to all types of sockets which we will explore a bit in [Other Types of Sockets](#other-types-of-sockets).

# How to Define Them

Very simple once you know how, but easily overlooked. Open up any static mesh in the editor and select the "Socket Manager" tab next to the "Details" tab.

![Socket manager]({{ '/assets/images/posts/static-mesh-sockets/socket-manager.png' | relative_url }})

The `+` button adds a moveable socket at location `(0,0,0)` to our mesh and allows us to give it a name.
I recommend using a naming convention compatible with gameplay tags (e.g. `Socket.Type.Variant`) and sticking to it for all of our meshes & sockets.
That way we get all the advantages of gameplay tags and can easily refer to commonly available sockets in our project without the risk of typos.

# When to Use Them

The most commonly referred to use case in Unreal Engine documentation & tutorials is of course *actor attachment*. I.e. if we wanted to attach something to our static mesh we would use a socket.
But that is only the tip of the iceberg. Games are hungry for many different kinds of transforms and sockets can do so much more!

Let's have a look at how we can enable some powerful data-driven workflows using sockets.

# Offset Sockets

We will start with something familiar: *skeletal mesh attachment*.

## The Problem with Attachment

All attachment functions & nodes in the engine allow us to specify the parent's socket name to attach the child to. Let's look at a small example to demonstrate an issue here.
I have a simple pawn with a skeletal mesh. I also have a sword static mesh that I want to attach to my pawn's skeletal mesh at socket `Socket.Hand.Right`.

![Pawn sword attachment]({{ '/assets/images/posts/static-mesh-sockets/pawn-sword-attachment.png' | relative_url }})

We can implement this in blueprint like so:

![Blueprint sword attachment simple]({{ '/assets/images/posts/static-mesh-sockets/blueprint-sword-attachment-simple.png' | relative_url }})

Which gives us this result in-game.

![Sword attached wrong]({{ '/assets/images/posts/static-mesh-sockets/sword-attached-wrong.png' | relative_url }})

Oh no! That is not how you should ever hold a sword! What happened?

Whenever we attach a child to a parent we can specify the *parent* socket name, but the child is always attached using the *origin of the mesh*.
When I created the sword mesh in Blender I used the *geometric center* of the mesh as its origin. Unreal will always use this point to do the attachment.

## Common Solutions

The most straightforward way would be to adjust the location/rotation of our `Socket.Hand.Right` skeletal mesh socket.
This works just fine depending on the project's complexity. If we only have a few weapon types with similar proportions this is the way to go.
But we should be aware of a few caveats here:

- ❌ Need to manage dedicated sockets per weapon type...
- ❌ ...per skeleton...
- ❌ ...per use case (i.e. sheathed, unsheathed, left hand, right hand).
- ❌ Weapons with significantly different proportions require extra sockets.

A workflow that solves these issues is to go back to Blender and set the origin to the hilt of the sword. Again, depending on the project's complexity this will work just fine.
But now there are some other aspects we need to consider:

- ❌ Have to go back-and-forth between Unreal and Blender to adjust center points.
- ❌ Need to think about where to put the origin for each mesh.
- ❌ Cannot use the geometric center or center of mass of the mesh (relevant for distance checks, calculating bounds, physics etc.).
- ❌ When we rotate the sword mesh it now rotates with the hilt as its pivot.
- ❌ Changes to the origin affect all characters and use cases.

We could also try to provide an explicit offset transform for our sword attachment.

![Blueprint sword attachment offset transform]({{ '/assets/images/posts/static-mesh-sockets/blueprint-sword-attachment-offset-transform.png' | relative_url }})

![Sword attached correct]({{ '/assets/images/posts/static-mesh-sockets/sword-attached-correct.png' | relative_url }})

Great, that worked! But now we have a set of new issues:

- ❌ Have to manually manage offset transforms for different types of weapons with varying proportions 
- ❌ Have to manually manage offset transforms for different uses cases etc.
- ❌ Designers need to understand this new system.
- ❌ Designers have to guesstimate where the correct transform on the mesh should be.

Oh boy, fun!

## The Socket Way

Maybe you already get where I am going with this. It seems we are trying to solve the wrong problem. What do we *actually* need here?
That's right! We will declare a relative-to-our-sword transform at design-time! We tell Unreal: "Hey, when you attach this sword to our pawn's hand make sure you use the hilt instead of the origin".

![Sword offset socket]({{ '/assets/images/posts/static-mesh-sockets/sword-offset-socket.png' | relative_url }})

We then use the inverse transform of this socket to apply an appropriate offset.
Why the *inverse* transform? Because if we want the hilt to overlap with the parent socket we must move the entire mesh in the *opposite* direction.

![Blueprint sword attachment offset socket manual]({{ '/assets/images/posts/static-mesh-sockets/blueprint-sword-attachment-offset-socket-manual.png' | relative_url }})

![Sword attached correct 2]({{ '/assets/images/posts/static-mesh-sockets/sword-attached-correct2.png' | relative_url }})

Perfect! The sword is attached correctly and our prior worries about mesh origins & offset transforms are resolved:

- ✅ No back-and-forth between Unreal and Blender.
- ✅ No need to think about where the origin is.
- ✅ We get the advantages of using the geometric center or center of mass as the origin.
- ✅ When we rotate the sword mesh it rotates around the geometric center or center of mass.
- ✅ Different types of weapons with varying proportions can specify where exactly their hilt is on the mesh.
- ✅ Different use cases (e.g. sheathed, unsheathed, left hand, right hand) can be covered by declaring new offset sockets.
- ✅ Changes to an offset socket only affect one weapon and the socket's use case.
- ✅ Designers can use familiar tools and a nice UI.
- ✅ Designers do not have to guesstimate the offsets for each mesh.

If we wrap this convention in a few nice C++ functions and expose them to blueprint we won't need to think about much at all when dealing with attachments.
For blueprint projects we can wrap the previous couple of nodes in a blueprint function library.

```cs
EResult UMugenSceneUtil::Attach(USceneComponent* Child, USceneComponent* Parent, const FName Socket, const FName OffsetSocket, const FAttachmentTransformRules AttachmentRules)
{
    if (IsValid(Child) && IsValid(Parent))
    {
        if (Child->AttachToComponent(Parent, AttachmentRules, Socket))
        {
            // We need Component space since we apply this transform as a relative offset
            if (const auto Transform = GetSocketTransform(Child, OffsetSocket, RTS_Component))
            {
                Child->SetRelativeTransform(Transform->Inverse());
            }
            return EResult::Success;
        }
    }
    return EResult::Failure;
}

// Only returns a valid transform if the socket truly exists
TOptional<FTransform> UMugenSceneUtil::GetSocketTransform(const USceneComponent* Component, const FName Socket, const ERelativeTransformSpace Space)
{
    return IsValid(Component) && !Socket.IsNone() && Component->DoesSocketExist(Socket) ? Component->GetSocketTransform(Socket, Space) : TOptional<FTransform>();
}
```

And voilá, our worries about attachment are a thing of the past. We can provide some actor-level utilities for these functions (leaving that up to you) and once exposed to blueprint we get:

![Blueprint sword attachment offset socket automatic]({{ '/assets/images/posts/static-mesh-sockets/blueprint-sword-attachment-offset-socket-automatic.png' | relative_url }})

Now that attachment is solved, we can have some fun with it.

![Attachment fun]({{ '/assets/images/posts/static-mesh-sockets/attachment-fun.png' | relative_url }})

If we wanted to be really strict with this we could even use `FGameplayTag` instead of `FName` as parameters to force our designers to use pre-defined gameplay tags for sockets.

Depending on our use case we should consider the performance of sequential socket lookups. Technically we are iterating a (small) array of objects multiple times. If we find a significant performance impact we could:

- Cache socket transforms after the first lookup (since they do not change)
- Use `UStaticMesh::GetSocketsByTag` to find all relevant sockets in O(n)

But the old adage applies: *Unmeasured* optimization is the root of all evil.

# Animating Props with Control Rig in Sequencer

If like me you use control rig to animate in sequencer we can also make use of sockets for something else entirely.
When animating with interactive props is difficult to get the desired results with forward kinematics (FK), especially if we need both hands to stay in contact with the prop at all times.

It is much easier to animate *just the prop* and let the arms solve themselves with inverse kinematics (IK). Which makes a lot of sense considering how we actually use props in the real world.
Nobody thinks about their shoulder, elbow or wrist when writing with a pen. We think about what we write and our arm solves this without our conscious guidance.

To enable this workflow we must first adjust our skeleton to support animating props in *mesh space*.

## Preparing the Skeleton

In order to animate a prop with minimal influence of other bones we bring it into mesh space. A clearer term for this is *root bone space*.
We create a dedicated *prop bone* parented to the *root bone*.

![Skeleton prop bone]({{ '/assets/images/posts/static-mesh-sockets/skeleton-prop-bone.png' | relative_url }})

We will attach a prop to this bone in-game, but not in sequencer! I recommend adding a well-named socket at this point, e.g. `Socket.Prop.Primary`.
We also add an appropriate FK control for this bone in our control rig.

And now we are ready to animate with...

## Constraints & Sockets

Let's say we wanted to create an animation to throw a prop, e.g. a barrel. We create a level sequence and add our skeleton & barrel as sequencer tracks. 

![Sequencer with skeleton and barrel]({{ '/assets/images/posts/static-mesh-sockets/sequencer-with-skeleton-and-barrel.png' | relative_url }})

So what do we do with our prop control now? The first step is to make the prop control follow our prop using a constraint.
Select the prop control and constrain it to the barrel. A selection of static mesh sockets will pop up.
It does not really matter in this case but I recommend using an [Offset Socket](#offset-sockets) here. No selection (= center point of our mesh) will do just fine though.

The prop is now constrained to and animates with the barrel. We animate the barrel - and thus the prop bone - without any parent bone chains to worry about.

![Sequencer prop constraint]({{ '/assets/images/posts/static-mesh-sockets/sequencer-prop-constraint.gif' | relative_url }})

Great! But what should we do about our IK hand controls? How do we precisely attach them to the barrel?
We could manually position them in each keyframe but... 

- ❌ A lot of work for each frame
- ❌ Difficult to get precise hand poses
- ❌ Fragile if we change the animation
- ❌ Have to repeat all that work if we change the mesh or proportions of the barrel
- ❌ Have to repeat all that work for other throwable props, e.g. a crate, cardboard box etc.

What tool can we use to do this? How about we declare a relative-to-our-barrel transform at design-time! We tell Unreal: "Hey, when this barrel moves in sequencer make sure to place the hand IK controls exactly here".
Much like with our prior attachment example we are declaring a transform on the mesh that is ideal for placing our hands on.
Hence the naming convention `Socket.Grip.Primary` and `Socket.Grip.Secondary` for the main & off hand respectively.

![Barrel grip sockets]({{ '/assets/images/posts/static-mesh-sockets/barrel-grip-sockets.png' | relative_url }})

Now we simply repeat the earlier steps: Select the right hand IK control, constrain it to the barrel at socket `Socket.Grip.Primary`. Conversely for the left hand IK.
The hand IK controls are now constrained to and animate with the gripping positions on the barrel.

![Sequencer hand IK constraint]({{ '/assets/images/posts/static-mesh-sockets/sequencer-hand-ik-constraint.gif' | relative_url }})

Now we can keyframe our prop & hand IK controls for our throw animation, bake the animation and simply attach a barrel to our prop bone in-game.
The hands will continue to accurately follow the gripping positions on the barrel.

![Throw animation]({{ '/assets/images/posts/static-mesh-sockets/throw-animation.gif' | relative_url }})

This brings many advantages:

- ✅ No work per frame required, set up constraints once and they remain active for any chosen frames
- ✅ Easy to set up sockets once for precise hand positions in all animations
- ✅ When changing the animation hands adjust automatically
- ✅ When changing the mesh and proportions of the barrel hands adjust automatically
- ✅ Can easily bake out dedicated animations for other throwable props

# Hitboxes

Hitboxes are at the core of most combat systems. We want some sort of geometric shape to describe *where* in space our attacks are effective.
There are a few possible ways to handle this in Unreal Engine but they boil down to two principles: *Overlap checks* and *shape/line traces*.
In the context of an action combat system traces are often better than overlap checks but the following approach may find use regardless of what traces are used for.

## The Grammar of Space

We have explored what we can do with *one* socket, but what happens when we add more sockets to the equation?

If we have 2 sockets (i.e. transforms) we can calculate a line that describes a *distance* between them.  
If we have 3 we can calculate a triangular plane that describes an *area* between them.  
If we have 4 and more we can calculate a polygon that describes a *volume* between them.

So sockets can be used to declare more than just simple points in space when composed together.
In fact we don't even need to go that far to achieve basic shapes.

We can calculate a *rectangle* using only 2 sockets: origin and extent.  
We can calculate a *sphere* using only 2 sockets: origin and radius.  
We can calculate a *capsule* using only 3 sockets: origin, radius and half-height.

If you are familiar with tracing in Unreal Engine you will notice that those are the basic shapes for sweep traces.
Can you guess where I'm going with this? Did I drive it home yet? Let's try to use sockets to define where our weapons apply damage and then trace that exact shape!

## Executing the Trace

Say we wanted to define a hitbox for a greatsword.

![Greatsword]({{ '/assets/images/posts/static-mesh-sockets/greatsword.png' | relative_url }})

We can create 3 sockets `Socket.Hitbox.Origin`, `Socket.Hitbox.Radius` and `Socket.Hitbox.Length` to define a capsule around the blade.

![Greatsword with sockets]({{ '/assets/images/posts/static-mesh-sockets/greatsword-with-sockets.png' | relative_url }})

This workflow allows us to get a very tight fit around the damaging parts of our weapons. If we use a rectangle we could make it even more precise!
I opted for a capsule here because the player should usually get some leeway with hitboxes.

Using our code from [Offset Sockets](#offset-sockets) we can provide a small C++ utility to get the required distance between our sockets:

```cs
TOptional<float> UMugenSceneUtil::GetDistanceBetweenSockets(const USceneComponent* Component, const FName FromSocket, const FName ToSocket)
{
    if (const auto From = GetSocketTransform(Component, FromSocket))
    {
        if (const auto To = GetSocketTransform(Component, ToSocket))
        {
            return FVector::Distance(From->GetLocation(), To->GetLocation());
        }
    }
    return NullOpt;
}
```

And now we can get the capsule's properties and execute a trace:

```cs
const FGameplayTag OriginSocket = FGameplayTag::RequestGameplayTag(TEXT("Socket.Hitbox.Origin"));
const FGameplayTag RadiusSocket = FGameplayTag::RequestGameplayTag(TEXT("Socket.Hitbox.Radius"));
const FGameplayTag LengthSocket = FGameplayTag::RequestGameplayTag(TEXT("Socket.Hitbox.Length"));

const FTransform CapsuleOrigin = UMugenSceneUtil::GetSocketTransform(WeaponMeshComponent, OriginSocket.GetTagName());
const float CapsuleRadius = GetDistanceBetweenSockets(WeaponMeshComponent, OriginSocket.GetTagName(), RadiusSocket.GetTagName());
const float CapsuleLength = GetDistanceBetweenSockets(WeaponMeshComponent, OriginSocket.GetTagName(), LengthSocket.GetTagName());

FHitResult Hit;
const FCollisionShape Capsule = FCollisionShape::MakeCapsule(CapsuleRadius, CapsuleLength);
// We trace from origin to origin (i.e. a length of 0) because the capsule already encompasses the blade
// And since the socket transform is returned in component space we also get the appropriate rotation
GetWorld()->SweepMultiByProfile(Hit, CapsuleOrigin.GetLocation(), CapsuleOrigin.GetLocation(), CapsuleOrigin.GetRotation(), TEXT("SomeTraceProfile"), Capsule);
```

Which is similarly easy to implement in blueprint but I would recommend executing frequent hitbox traces in C++ instead.

![Blueprint get distance between sockets]({{ '/assets/images/posts/static-mesh-sockets/blueprint-get-distance-between-sockets.png' | relative_url }})

![Blueprint capsule trace]({{ '/assets/images/posts/static-mesh-sockets/blueprint-capsule-trace.png' | relative_url }})

## Compatibility with TargetingSystem

In my project I use Unreal's TargetingSystem plugin. It provides a data-driven approach to traces by creating TargetingPreset assets.
These assets specify:

- How to trace with TargetingSelectionTasks
- What filters to apply on the traced hits with TargetingFilterTasks
- What sorting/ranking to apply on the results with TargetingSortTasks

And allow for a very powerful data-driven workflow for any tracing needs as well as great debugging tools. On top of that we can customize many aspects of these tasks.
Our hitbox shape tracing can be fully automated with a custom TargetingSelectionTask by subclassing `UTargetingSelectionTask_Trace` and overriding relevant methods.
But this is a topic for another day!

## Final Results

Now that we have a hitbox tracing system in place we can integrate it into our combat system and see the results.

![Greatsword swing hitbox]({{ '/assets/images/posts/static-mesh-sockets/greatsword-swing-hitbox.gif' | relative_url }})

There are multiple advantages we gain from this approach:

- ✅ Data-driven workflow
- ✅ Accurate hitboxes
- ✅ Visually define hitboxes directly on the weapon mesh
- ✅ Each weapon can provide its own unique hitbox

Here are a few more ideas I haven't tried yet but that are certainly possible with this workflow:

- ❓ Multiple hitboxes with different effects (e.g. hilt deals blunt damage, blade deals cutting damage)
- ❓ Damage scaling based on hit location (e.g. very tip of the blade deals less damage than the weighty center)
- ❓ More complex trace shapes composed out of the basic shapes

Speaking of ideas...

# Other Types of Sockets

We had established that sockets in Unreal Engine are really nothing but relative, design-time transforms.
All of these techniques also apply to skeletal mesh sockets, although we have to account for the fact that those can be animated, physically simulated etc.

But the list does not end there.

## Proxy Actors

The term *proxy* means a "stand-in" for something or someone else. Voting by proxy means someone else is voting for you. A proxy server handles requests in place of your own computer/server.

There is a common technique in game development to place *proxy actors* in levels instead of placing actual actors directly.
By doing so we tell the level: "Something can spawn here but we won't tell you what it is yet".
At runtime we can use this proxy actor as a - you guessed it - relative-to-the-level transform declared at design-time and spawn any actor at it.

Now I won't go into detail why we would want to do this, but here are just a few advantages:

- ✅ The actual actor is not loaded into memory until it is needed (lower memory footprint)
- ✅ We can hotswap which actor gets spawned without touching or changing the level
- ✅ DLCs and mods can easily modify our levels

There is much more to it. What I'm trying to say here: A "socket" does not need to look like or be called a "socket". It's transforms all the way down.
Well, you get it by now. Very useful stuff.

## Transform MakeEditWidget

Another relevant hidden feature in Unreal Engine is using plain `FTransform` (or `Transform` in blueprint) variables with the `MakeEditWidget` meta attribute.

```cs
UPROPERTY(EditAnywhere, meta=(MakeEditWidget))
FTransform Transform;
```

![Blueprint transform widget]({{ '/assets/images/posts/static-mesh-sockets/blueprint-transform-widget.png' | relative_url }})

Which gives us a small widget in the level e.g. on an actor. We can drag it around and place it visually which is better than guessing transform properties in the details panel.

![Transform widget result]({{ '/assets/images/posts/static-mesh-sockets/transform-widget-result.png' | relative_url }})

This utility works nicely with *invisible* actors that spawn something else, like trigger shapes or indeed our proxy actor.
Instead of using the transform of the actor directly we specify one or more named transforms to spawn other actors at.
This gives us another useful data-driven workflow and more flexibility while designing levels. Neat!

# Final Thoughts

Video games are hungry for transforms. The more we can define at design-time the less we have to worry about runtime calculations, bespoke management systems & code complexity.
We are visual creatures, computers are not. These techniques help us bridge the gap.

Thanks for reading! {% include final-words.md %}
