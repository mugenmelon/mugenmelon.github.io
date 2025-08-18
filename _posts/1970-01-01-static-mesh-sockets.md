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

Sound familiar? The most commonly referred to use-case in Unreal Engine documentation & tutorials is of course *actor attachment*.
But that is only the tip of the iceberg. Games are hungry for many different kinds of transforms and thus sockets can do so much more!

Let's have a look at how we can enable some powerful data-driven workflows using sockets.

## Offset Sockets

## Animating in Sequencer

## Hitboxes

## More Crazy Ideas

# Other Types of Sockets

# Final Thoughts
