---
title: 🛡️ Future-Proofing C++ Interfaces for Blueprint
tags: tutorial gamedev programming unreal-engine c++ blueprint
highlight: true
related_tags: unreal-engine
---

TODO: introduction

{% include toc.md %}

# C++ `UInterface`

Interfaces are a powerful abstraction mechanism. They are ideal for modeling imperative communication between different systems, while avoiding the rigidity of inheritance. One could even say that they are one of the cornerstones of a flexible and maintainable project. Most programming languages provide some way to write interfaces, and so does C++ with pure virtual functions, but for full compatibility Unreal Engine has a built-in `UInterface` class. Following is a short recap of how `UInterface` is used.

## Quick Recap

Say we wanted to have various pieces of equipment like items, weapons, and armor in our game that can be equipped to a character. A data-asset `UEquipmentAsset` defines *how* a piece of equipment gets equipped, what it looks like and which socket it gets attached to. But how do we add this data-asset to our various in-game items without a rigid inheritance hierarchy?
By using an interface `IEquippable` that provides us with a `UEquipmentAsset` instance we can make *any* class equippable, regardless of whether it is an item, weapon, or armor.

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

Or just as simple in blueprint:

![Blueprint interface cast]({{ '/assets/images/posts/future-proofing-interfaces/01-blueprint-interface-cast.png' | relative_url }})

Take note that at no point here do we get an error. We are safely checking that SomeObject is indeed equippable before getting the asset. This is an important detail when it comes to...

## The Blueprint Question

Let us take stock: We have an interface that can be *implemented* in C++ and *called* in both C++ and blueprint. But what if we want to be able to also *implement* the interface in blueprint? To make a `UInterface` implementable in blueprint we have to mark the `UINTERFACE` as `BlueprintType` (which we already have) and then mark the `UFUNCTION` as `BlueprintNativeEvent`:

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

Now we can add the interface to any blueprint's "Class Settings" and then implement the function `GetEquipmentAsset` in blueprint:

![Blueprint interface class settings]({{ '/assets/images/posts/future-proofing-interfaces/02-blueprint-interface-class-settings.png' | relative_url }})

However there is one important caveat. If we try to call the function as we did in [Quick Recap](#quick-recap) we will get an error: `Do not directly call Event functions in Interfaces. Call Execute_GetEquipmentAsset instead`. This is the case with all `BlueprintNativeEvent` and `BlueprintImplementableEvent` functions. As the error message suggests, we have to use a static function `IEquippable::Execute_GetEquipmentAsset` instead.

But that is not all! We also have to make sure this static function is only called on objects that *actually implement the interface*. Attempting to call it on anything else will result in yet another error. Here is how we can handle it.

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

This circumstance makes it a bit cumbersome and "boilerplate-y" to add blueprint support to a `UInterface`, especially when using them in several places across our codebase. And personally I always try to avoid nesting and `if` clauses to keep my functions as simple as possible.
But there is a very simple convention that can help relieve us of both of these issues.

# Future-Proofing `UInterface`

The idea is pretty straightforward: if we want want to call `IEquippable::GetEquipmentAsset` but *the way* we have to call it can change, why not wrap it in a central, easily maintainable place? For example, a static function on the interface itself?

## Static Function Wrapper

For the sake of demonstration, let us go back to our initial interface from [Quick Recap](#quick-recap). We will add a new static function that wraps the actual function call to the interface.

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
*This works nicely for return types that have a sentinel value like `nullptr`. If you want to know how to best handle other return types here, check out my post on [Improving Program Flow In UE5 C++ & Blueprint]({% post_url 2025-08-04-improving-program-flow %}).*
{: .caption}

Usage of this interface becomes much simpler, safer and less "boilerplate-y".

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

    UFUNCTION(BlueprintNativeEvent, BlueprintCallable)
    const UEquipmentAsset* GetEquipmentAsset() const;
};
```

```cpp
const UEquipment* IEquippable::GetEquipmentAsset(const UObject* Target)
{
    // This is the only place we have to change to support blueprints.
    return IsValid(Target) && Target->Implements<UEquippable>() ? Execute_GetEquipment(Target) : nullptr;
}
```

That's all, no other code changes required anywhere! Neat!

# Final Thoughts

A little bit of a no-brainer this time, but even small improvements can have compounding effects on code readability, cognitive load and your motivation to keep working on your projects! 
