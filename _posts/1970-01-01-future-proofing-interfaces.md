---
title: 🛡️ Future-Proofing C++ Interfaces for Blueprint
tags: tutorial gamedev programming unreal-engine c++ blueprint
highlight: true
related_tags: unreal-engine
---

TODO: introduction

{% include toc.md %}

# C++ `UInterface`

Interfaces are a powerful abstraction mechanism. They are ideal for modeling imperative communication between different systems, while avoiding the rigidity of inheritance. Most programming languages provide some way to write interfaces, and so does C++ with pure virtual functions. However, if we want full engine compatibility, we cannot use pure virtual functions. Instead we have to use Unreal Engine's built-in `UInterface` class. Following is a short recap of how `UInterface` is used.

## Quick Recap

We will use an actual use case from my game project as an example. Say we wanted to have various pieces of equipment like items, weapons, and armor in our game that can be equipped to a character. We can create a data-asset `UEquipmentAsset` that defines *how* a piece of equipment gets equipped, what it looks like and which socket it gets attached to. Then we can create an interface `IEquippable` that needs to be implemented in order to provide our equipment system with a particular `UEquipmentAsset` instance. That way we can implement the interface on *any* class, regardless of whether it is an item, weapon, or armor. The calling code does not need to know *what* exactly we are equipping, it only needs to know that it is "something equippable".

We declare a `UInterface` by using the `UINTERFACE` macro and 2 separate class declarations:

```cpp
// Declaring a UINTERFACE that can be called from Blueprint.
// Not our actual interface, but required by UE.
// This class remains empty, do not put any functions in here!
UINTERFACE(BlueprintType)
class UEquippable : public UInterface
{
    GENERATED_BODY()
};

// This is our actual interface class.
class IEquippable
{
    GENERATED_BODY()

public:
    // Declaring a UFUNCTION that can be called from Blueprint.
    UFUNCTION(BlueprintCallable)
    const UEquipmentAsset* GetEquipmentAsset() const;
};
```

To use the interface we only need to inherit from `IEquippable` and implement the function:

```cpp
UCLASS()
class AEquippableActor : public AActor, public IEquippable
{
    GENERATED_BODY()

public:
    virtual const UEquipmentAsset* GetEquipmentAsset() const override;
};
```

And now we can call it for example like so:

```cpp
void Equip(const UObject* Object)
{
    if (const IEquippable* Equippable = Cast<IEquippable>(Object))
    {
        const UEquipmentAsset* EquipmentAsset = Equippable->GetEquipmentAsset();
        // Do something with EquipmentAsset...
    }
}
```

## The Blueprint Question

# Future-Proofing `UInterface`

## Static Function Wrapper

```cs
class IEquippable
{
    GENERATED_BODY()

public:
    static const UEquipmentAsset* GetEquipmentAsset(const UObject* Target);

    UFUNCTION(BlueprintCallable)
    const UEquipmentAsset* GetEquipmentAsset() const;
};
```

```cs
const UEquipment* IEquippable::GetEquipmentAsset(const UObject* Target)
{
    const auto* Equippable = Cast<IEquippable>(Target);
    return Equippable ? Equippable->GetEquipment() : nullptr;
}
```

## Easy Blueprint Compatibility

```cs
class IEquippable
{
    GENERATED_BODY()

public:
    static const UEquipmentAsset* GetEquipmentAsset(const UObject* Target);

    UFUNCTION(BlueprintNativeEvent, BlueprintCallable)
    const UEquipmentAsset* GetEquipmentAsset() const;
};
```

```cs
const UEquipment* IEquippable::GetEquipmentAsset(const UObject* Target)
{
    return IsValid(Target) && Target->Implements<UEquippable>() ? Execute_GetEquipment(Target) : nullptr;
}
```

# Final Thoughts
