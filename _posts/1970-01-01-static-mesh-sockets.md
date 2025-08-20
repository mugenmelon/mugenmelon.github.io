---
title: "Why You Should Be Using Static Mesh Sockets"
tags: gamedev unreal-engine gameplay animation c++ blueprint
published: true
---

Work in progress.

{% include toc.md %}

# What Are Static Mesh Sockets?

If you have spent any amount of time in Unreal Engine you are likely already familiar with *skeletal mesh sockets*.

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

### The Problem with Attachment

All attachment functions & nodes in the engine allow us to specify the parent's *socket name* to attach the child to. Let's look at a small example to demonstrate an issue here.
I have a simple pawn with a skeletal mesh. I also have a sword static mesh that I want to attach to my pawn's skeletal mesh at socket `Socket.Prop.Primary`.

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

### The Socket Way

Maybe you already get where I am going with this. It seems we are trying to solve the wrong problem. What do we *actually* need here?

> A relative-to-something transform declared at design time.

That's right! We will *declare a relative-to-our-sword transform at design-time*! We tell Unreal: "Hey, when you attach this sword to our pawn's hand make sure you use the hilt instead of the center point".

![Sword offset socket]({{ '/assets/images/posts/static-mesh-sockets/sword-offset-socket.png' | relative_url }})

And then use the *inverse* transform of this socket to apply an appropriate offset.

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

## Animating Props with Control Rig in Sequencer

If like me you use control rig to animate in sequencer we can also make use of sockets for something else entirely.
When animating with interactive props is difficult to get the desired results with forward kinematics (FK), especially if you need both hands to stay in contact with the prop at all times.

It is much easier to animate *just the prop* and let the arms *solve themselves* with inverse kinematics (IK). Which makes a lot of sense considering how we actually use props in the real world.
Nobody thinks about their shoulder, elbow nor wrist when writing with a pen. We think about *what we write* and our arm solves this without our conscious guidance.

To enable this workflow we must first adjust our skeleton to support animating props in *mesh space*.

### Preparing the Skeleton

In order to animate a prop *with minimal influence of other bones* we bring it into *mesh space*. A clearer term for this is *root bone space*.
We create a dedicated *prop bone* parented to the *root bone*.

![Skeleton prop bone]({{ '/assets/images/posts/static-mesh-sockets/skeleton-prop-bone.png' | relative_url }})

We will attach a prop to this bone *in-game*, but not in sequencer! I recommend adding a well-named socket at this point, e.g. `Socket.Prop.Primary`.
We also add an appropriate FK control for this bone in our control rig.

And now we are ready to animate with...

### Constraints & Sockets

Let's say we wanted to create an animation to throw a prop, e.g. a barrel. We create a level sequence and add our skeleton & barrel as sequencer tracks. 

![Sequencer with skeleton and barrel]({{ '/assets/images/posts/static-mesh-sockets/sequencer-with-skeleton-and-barrel.png' | relative_url }})

So what do we do with our prop control now? The first step is to make the prop control *follow* your prop using a constraint.
Select the prop control and constrain it to the barrel. A selection of static mesh sockets will pop up.
It does not really matter *in this case* but I recommend using an [Offset Socket](#offset-sockets) here. No selection (= center point of your mesh) will do just fine though.

You will see that the prop is now constrained to and animates with the barrel. We animate the barrel - and thus the prop bone - without any parent bone chains to worry about.

![Sequencer prop constraint]({{ '/assets/images/posts/static-mesh-sockets/sequencer-prop-constraint.gif' | relative_url }})

Great! But what should we do about our IK hand controls? How do we precisely attach them to the barrel?
We *could* manually position them in each keyframe but... 

- ❌ A lot of work for each frame
- ❌ Difficult to get precise hand poses
- ❌ Fragile if we change the animation
- ❌ Have to repeat all that work if we change the mesh or proportions of the barrel
- ❌ Have to repeat all that work for other throwable props, e.g. a crate, cardboard box etc.

that is a lot of work, hard to get precise poses and very fragile if we change the barrel's transform or its proportions.
What tool can we use to do this? How about...

> A relative-to-something transform declared at design time.

Déjà vu! We will *declare a relative-to-our-barrel transform at design-time*! We tell Unreal: "Hey, when this barrel moves in sequencer make sure to place the hand IK controls exactly here".
Much like with our prior attachment example we are *declaring* a transform on the mesh that is ideal for placing your hands on.
Hence the naming convention `Socket.Grip.Primary` and `Socket.Grip.Secondary` for the main & off hand respectively.

![Barrel grip sockets]({{ '/assets/images/posts/static-mesh-sockets/barrel-grip-sockets.png' | relative_url }})

Now we simply repeat the earlier steps: Select the right hand IK control, constrain it to the barrel at socket `Socket.Grip.Primary`. Conversely for the left hand IK.
The hand IK controls are now constrained to and animate with the gripping positions on the barrel.

![Sequencer hand IK constraint]({{ '/assets/images/posts/static-mesh-sockets/sequencer-hand-ik-constraint.gif' | relative_url }})

Now we can keyframe our prop & hand IK controls for our throw animation, bake the animation and simply attach a barrel to our prop bone in-game.
The hands will continue to accurately follow the gripping positions on the barrel.

![Throw animation]({{ '/assets/images/posts/static-mesh-sockets/throw-animation.gif' | relative_url }})

This brings many advantages:

- ✅ No work per frame required, set up constraints once and they remain active for any choosen frames
- ✅ Easy to set up sockets once for precise hand positions in all animations
- ✅ When changing the animation hands adjust automatically
- ✅ When changing the mesh and proportions of the barrel hands adjust automatically
- ✅ Can easily bake out dedicated animations for other throwable props

## Hitboxes

### The Grammar of Space

### Sockets Are Shapes??

### Hitbox Accuracy

### Compatibility with Targeting System

### Final Results

## More Crazy Ideas

# Other Types of Sockets

## Proxy Actors

## `FTransform` MakeEditWidget

# Final Thoughts
