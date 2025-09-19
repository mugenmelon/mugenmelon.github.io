---
title: 🛡️ Future-Proofing C++ Interfaces for Blueprint
tags: tutorial gamedev programming unreal-engine c++ blueprint
highlight: true
related_tags: unreal-engine
---

TODO: introduction

{% include toc.md %}

# C++ `UInterface`

## Quick Recap

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
    // TODO: Pure virtual? Or with default implementation? I think pure virtual doesn't actually work.
    // Or IIRC UBT creates a stub for this anyway?
    UFUNCTION(BlueprintCallable)
    const UEquipmentAsset* GetEquipmentAsset() const;
};
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
