---
title: "Why You Should Be Using Static Mesh Sockets"
tags: gamedev programming unreal-engine c++ animation
published: true
---

Work in progress.

{% include toc.md %}

# What Are Static Mesh Sockets?

If you have spent any time in Unreal Engine you are likely already familiar with *skeletal mesh sockets*.

![Skeletal mesh socket example]({{ '/assets/images/posts/static-mesh-sockets/skeletal-mesh-socket-example.png' | relative_url }})

Sockets in principle are transforms (location, rotation & scale) declared at design-time in a "relative-to-something" space.
In the case of *skeletal mesh sockets* they define a transform *relative* to a *skeletal mesh*.
It is no surprise then that *static mesh sockets* define a transform *relative* to a *static mesh*.

![Static mesh socket example]({{ '/assets/images/posts/static-mesh-sockets/static-mesh-socket-example.png' | relative_url }})

To keep things simple let us set aside a few (but important) differences between *skeletal mesh* and *static mesh sockets* for now and only talk of *static mesh sockets* as per our definition:

> A relative-to-something transform declared at design time.

What I will show you here applies to *all* types of sockets which we will explore a bit in [Other Types of Sockets](#other-types-of-sockets).

# How to Define Them

Very simple once you know, but they are easily overlooked. Open up any static mesh in the editor and select the "Socket Manager" tab next to the "Details" tab.

![Socket manager]({{ '/assets/images/posts/static-mesh-sockets/socket-manager.png' | relative_url }})

The `+` button adds a moveable socket at location `(0,0,0)` to your mesh and allows you to give it a name.
I recommend using a naming convention compatible with gameplay tags (e.g. `Socket.Type.Variant`) and sticking to it for *all* of your meshes & sockets.
That way you get all the advantages of gameplay tags and can easily refer to commonly available sockets in your project without the risk of typos.

# When to Use Them

The short answer: Whenever you need

> A relative-to-something transform declared at design time.

Sound familiar? The most commonly referred to use case in Unreal Engine documentation & tutorials is of course *actor attachment*.
But that is only the tip of the iceberg. Games are hungry for many different kinds of transforms and thus sockets can do so much more!

Let's have a look at how we can enable some powerful data-driven workflows using sockets.

## Offset Sockets

We will start with something familiar: *actor attachment*.

### Basics

To be accurate we should frame this as *scene component attachment*.
In fact you can never actually attach an actor to another actor, because the engine internally only attaches the actor's *root component* to the other actor's *default attach component*.

```cs
// Engine-internal function to attach this actor to any given ParentActor.
// AttachmentRules define certain properties of what should happen with the child actor upon attachment.
// SocketName specifies the socket (i.e. relative transform) we should attach the child actor to.
bool AActor::AttachToActor(AActor* ParentActor, const FAttachmentTransformRules& AttachmentRules, FName SocketName)
{
    if (RootComponent && ParentActor)
    {
        // DefaultAttachComponent most of the time is the ParentActor's SkeletalMeshComponent, but can be overridden to provide any other USceneComponent
        USceneComponent* ParentDefaultAttachComponent = ParentActor->GetDefaultAttachComponent();
        if (ParentDefaultAttachComponent)
        {
            return RootComponent->AttachToComponent(ParentDefaultAttachComponent, AttachmentRules, SocketName);
        }
    }
    return false;
}
```

So all we are really doing is attaching `USceneComponent`s to one another. Internally the engine calls:

```cs
// ChildComponent = our RootComponent
// ParentComponent = other actor's DefaultAttachComponent
ChildComponent->AttachToComponent(ParentComponent, AttachmentRules, Socket)
```

At some point to do the actual attachment. So from now on I will refer to *parent* and *child* meaning both the component and actor.
On the blueprint side of things we get very similar functions to do attachment.

![Blueprint attach nodes]({{ '/assets/images/posts/static-mesh-sockets/blueprint-attach-nodes.png' | relative_url }})

### The Problem With Attachment

All of these functions & nodes allow us to specify the parent's *socket name* to attach the child to. Let's look at a small example to demonstrate an issue here.
I have a simple pawn with a skeletal mesh. I also have a sword static mesh that I want to attach to my pawn's skeletal mesh at socket `Socket.Hand.Main`.

![Pawn sword attachment]({{ '/assets/images/posts/static-mesh-sockets/pawn-sword-attachment.png' | relative_url }})

We can implement this in blueprint like so:

![Blueprint sword attachment simple]({{ '/assets/images/posts/static-mesh-sockets/blueprint-sword-attachment-simple.png' | relative_url }})

Which gives us this result in-game.

![Sword attached wrong]({{ '/assets/images/posts/static-mesh-sockets/sword-attached-wrong.png' | relative_url }})

Oh no! That is not how you should ever hold a sword! What happened?

Whenever we attach a child to a parent we can specify the *parent* socket name, but the child is *always* attached using the *center point of the mesh*.
When I created the sword mesh in Blender I used the *geometric center* of the mesh as its center point. Unreal will always use this point to do the attachment.

### Common Solutions

The most straightfoward solution would be to go back to Blender and set the center of the sword mesh to the hilt of the sword. And this will work just fine for many projects.
But there are some issues that we need to be aware of:

- ❌ Have to go back-and-forth between Unreal and Blender to adjust center points.
- ❌ Need to think about where to put the center point for each mesh.
- ❌ Cannot use the geometric center or center of mass of the mesh (relevant for distance checks, calculating bounds, physics etc.).
- ❌ When we rotate the sword mesh it now rotates with the hilt as its center.
- ❌ Changes to the center point affect *all* characters and use cases.

We could also try to provide an *offset* relative transform for our sword attachment.

![Blueprint sword attachment offset transform]({{ '/assets/images/posts/static-mesh-sockets/blueprint-sword-attachment-offset-transform.png' | relative_url }})

![Sword attached correct]({{ '/assets/images/posts/static-mesh-sockets/sword-attached-correct.png' | relative_url }})

Great, that worked! But now we have a set of new issues:

- ❌ Have to manually manage offset transforms for different types of weapons with varying proportions 
- ❌ Have to manually manage offset transforms for different uses cases (e.g. sheathed, unsheathed, left hand, right hand) etc.
- ❌ Designers need to understand this new system.
- ❌ Designers have to guesstimate *where* the correct transform on the mesh should be.

Oh boy, fun!

### The Solution

Maybe you already get where I am going with this. It seems we are trying to solve the wrong problem. What do we *actually* need here?

> A relative-to-something transform declared at design time.

That's right! We will *declare a relative-to-our-sword transform at design-time*!

![Sword offset socket]({{ '/assets/images/posts/static-mesh-sockets/sword-offset-socket.png' | relative_url }})

And then use the *inverse transform* of this socket to apply an appropriate offset. We are telling Unreal: "Hey, when you attach this sword to our pawn's hand make sure you use the hilt instead of the center point".

![Blueprint sword attachment offset socket manual]({{ '/assets/images/posts/static-mesh-sockets/blueprint-sword-attachment-offset-socket-manual.png' | relative_url }})

![Sword attached correct]({{ '/assets/images/posts/static-mesh-sockets/sword-attached-correct.png' | relative_url }})

Perfect! The sword is attached correctly and our prior worries about mesh center points & offset transforms are resolved:

- ✅ No back-and-forth between Unreal and Blender.
- ✅ No need to think about where the center point is.
- ✅ We get the advantages of using the geometric center or center of mass as the center point.
- ✅ When we rotate the sword mesh it rotates around the geometric center or center of mass.
- ✅ Different types of weapons with varying proportions can specify where exactly their hilt is on the mesh.
- ✅ Different use cases (e.g. sheathed, unsheathed, left hand, right hand) can be covered by declaring new offset sockets.
- ✅ Changes to an offset socket only affect one weapon and the socket's use case.
- ✅ Designers can use familiar tools and a nice UI.
- ✅ Designers do not have to guesstimate the offsets for each mesh.

If we wrap this convention in a few nice C++ functions and expose them to blueprint we won't need to think about much at all when dealing with attachments.
For blueprint projects you can wrap the previous couple of nodes in a blueprint function library.

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

Now that attachment is a solved issue we can have some fun with it.

![Attachment fun]({{ '/assets/images/posts/static-mesh-sockets/attachment-fun.png' | relative_url }})

If you wanted to be *really* strict with this you could even use `FGameplayTag` instead of `FName` as parameters to force your designers to use pre-defined gameplay tags for sockets.

## Animating in Sequencer

### Animating Props

### Constraints & Sockets to the Rescue

## Hitboxes

### The Grammar of Space

## More Crazy Ideas

# Other Types of Sockets

## Proxy Actors

## `FTransform` MakeEditWidget

# Final Thoughts
