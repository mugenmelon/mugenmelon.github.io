---
title: 🛡️ Future-Proofing C++ Interfaces for Blueprint
tags: tutorial gamedev programming unreal-engine c++ blueprint
highlight: true
related_tags: unreal-engine
---

TODO: introduction

{% include toc.md %}

# `UInterface`

Interfaces are a powerful abstraction mechanism. They are ideal for modeling imperative communication between different systems, while avoiding the rigidity of inheritance. Most programming languages provide some way to write interfaces, and so does C++ with pure virtual functions. But for full compatibility Unreal Engine has a built-in `UInterface` class. Following is a short recap on how to use `UInterface`.

## Quick Recap

Say we wanted to have various pieces of equipment like items, weapons, and armor in our game that can be equipped to a character. A data-asset `UEquipmentAsset` defines *how* a piece of equipment gets equipped, what it looks like and which socket it gets attached to. But how do we add this data-asset to our various in-game items without a rigid inheritance hierarchy?
By using an interface `IEquippable` that provides us with a `UEquipmentAsset` we can make *any* class equippable, regardless of whether it is an item, weapon, or armor.

We declare a `UInterface` by using the `UINTERFACE` macro and 2 separate class declarations.

```cpp
UINTERFACE(BlueprintType)
class UEquippable : public UInterface
{
    GENERATED_BODY()
};

class IEquippable
{
    GENERATED_BODY()

public:
    UFUNCTION(BlueprintCallable)
    const UEquipmentAsset* GetEquipmentAsset() const;
};
```

To use the interface we only need to inherit from `IEquippable` and implement the function in any of our classes.

```cpp
UCLASS()
class AEquippableActor : public AActor, public IEquippable
{
    GENERATED_BODY()

public:
    // Concrete implementation omitted for brevity.
    virtual const UEquipmentAsset* GetEquipmentAsset() const override;
};
```

Then we can cast any object to `IEquippable`and get the required asset by calling `GetEquipmentAsset`.

```cpp
if (const IEquippable* Equippable = Cast<IEquippable>(SomeObject))
{
    if (const UEquipmentAsset* EquipmentAsset = Equippable->GetEquipmentAsset())
    {
        // SomeObject is equippable, execute equipment logic...
    }
}
```

![Blueprint interface cast]({{ '/assets/images/posts/future-proofing-interfaces/01-blueprint-interface-cast.png' | relative_url }})

Take note that at no point do we get an error here. We are safely checking that `SomeObject` is indeed equippable before getting the asset. This is an important detail when it comes to...

## The Blueprint Question

Let us take stock: We have an interface that can be *implemented* in C++ and *called* in both C++ and blueprint. But what if we want to *implement* the interface in blueprint? To make a `UInterface` implementable in blueprint we have to mark the `UINTERFACE` as `BlueprintType` (which we already have) and then mark the `UFUNCTION` as `BlueprintNativeEvent`:

```cpp
UINTERFACE(BlueprintType)
class UEquippable : public UInterface
{
    GENERATED_BODY()
};

class IEquippable
{
    GENERATED_BODY()

public:
    UFUNCTION(BlueprintNativeEvent, BlueprintCallable)
    const UEquipmentAsset* GetEquipmentAsset() const;
};
```

Now we can add this interface to any blueprint's "Class Settings", which gives us the option to then implement the function `GetEquipmentAsset`.

![Blueprint interface class settings]({{ '/assets/images/posts/future-proofing-interfaces/02-blueprint-interface-class-settings.png' | relative_url }})

There is one important caveat here. If we try to call the function like in [Quick Recap](#quick-recap), we will get an error: `Do not directly call Event functions in Interfaces. Call Execute_GetEquipmentAsset instead`. This is the case with all `BlueprintNativeEvent` and `BlueprintImplementableEvent` functions. They must always be called by using the static `IEquippable::Execute_GetEquipmentAsset` instead.

But that is not all! We also have to make sure this static function is only ever called on objects that *actually implement the interface*. Attempting to call it on anything else results in another error. Here is how we can handle it correctly.

```cpp
// Instead of Cast() we use Implements().
// Careful! This must be UEquippable, not IEquippable!
if (SomeObject && SomeObject->Implements<UEquippable>())
{
    // Only now is it safe to call Execute_GetEquipmentAsset.
    if (const UEquipmentAsset* EquipmentAsset = IEquippable::Execute_GetEquipmentAsset(SomeObject))
    {
        // SomeObject is equippable, execute equipment logic...
    }
}
```

This circumstance makes it a bit "boilerplate-y" to add blueprint support to a `UInterface`, especially when using it in several places. And personally I always try to avoid nesting and `if` clauses where I can to keep functions easy to read.

There is a very simple convention that addresses both of these issues.

# Future-Proofing `UInterface`

The idea is straightforward: we want want to call `IEquippable::GetEquipmentAsset` but *the way* we have to call it is in flux. Why not do it in a central, coherent place? For example, in a static function on the interface itself!

## Static Function Wrapper

For the sake of demonstration, we will go back to our C++ only interface from [Quick Recap](#quick-recap). We write a new static function that wraps the actual function call to the interface.

```cpp
class IEquippable
{
    GENERATED_BODY()

public:
    // Our static function wrapper.
    static const UEquipmentAsset* GetEquipmentAsset(const UObject* Target);

    UFUNCTION(BlueprintCallable)
    const UEquipmentAsset* GetEquipmentAsset() const;
};
```

```cpp
// Implementation of our static function wrapper.
const UEquipment* IEquippable::GetEquipmentAsset(const UObject* Target)
{
    // Instead of spreading Cast() calls throughout the codebase we only cast here.
    const IEquippable* Equippable = Cast<IEquippable>(Target);
    // We return sensible defaults where possible.
    return Equippable ? Equippable->GetEquipmentAsset() : nullptr;
}
```
*This works nicely for return types that have a sentinel value like `nullptr`. If you want to know how to best handle other return types, check out my post on [Improving Program Flow In UE5 C++ & Blueprint]({% post_url 2025-08-04-improving-program-flow %}).*
{: .caption}

Usage of `IEquippable` becomes much simpler and safer.

```cpp
if (const UEquipmentAsset* EquipmentAsset = IEquippable::GetEquipmentAsset(SomeObject))
{
    // SomeObject is equippable, execute equipment logic...
}
```

## Easy Blueprint Compatibility

In addition to that we get a very clear migration path to support blueprints implementations of this interface.

```cpp
class IEquippable
{
    GENERATED_BODY()

public:
    static const UEquipmentAsset* GetEquipmentAsset(const UObject* Target);

    // We added BlueprintNativeEvent.
    UFUNCTION(BlueprintNativeEvent, BlueprintCallable)
    const UEquipmentAsset* GetEquipmentAsset() const;
};
```

```cpp
// We only have to adjust our static wrapper function.
const UEquipment* IEquippable::GetEquipmentAsset(const UObject* Target)
{
    return IsValid(Target) && Target->Implements<UEquippable>() ? Execute_GetEquipmentAsset(Target) : nullptr;
}
```

That's all, no other code changes required anywhere for full blueprint support! Neat!

# Final Thoughts

A short a no-brainer this time, but even small improvements can have compounding effects on code readability, cognitive load and especially our motivation to keep working on our projects! Little drops of water make the mighty ocean.
