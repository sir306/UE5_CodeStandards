# Epic C++ Coding Standard for Unreal Engine

## Overview

At Epic Games, we have a few simple coding standards and conventions. This document reflects the state of Epic Games' current coding standards. **Following the coding standards is mandatory.**

### Why Coding Standards Matter

Code conventions are important to programmers for several reasons:

- **80% of the lifetime cost** of a piece of software goes to maintenance
- Hardly any software is maintained for its whole life by the original author
- Code conventions **improve the readability** of software, allowing engineers to understand new code quickly and thoroughly
- Makes code easily understood if exposed to mod community developers
- Many conventions are **required for cross-compiler compatibility**

The coding standards below are **C++-centric**; however, the standard is expected to be followed no matter which language is used. A section may provide equivalent rules or exceptions for specific languages where applicable.

---

## Table of Contents

1. [Code Formatting](#code-formatting)
2. [Naming Conventions](#naming-conventions)
3. [Comments and Documentation](#comments-and-documentation)
4. [Type Declarations](#type-declarations)
5. [Pointer Types](#pointer-types)
6. [Use of const](#use-of-const)
7. [Virtual Functions](#virtual-functions)
8. [Boolean Parameters](#boolean-parameters)
9. [Strongly-Typed Enums](#strongly-typed-enums)
10. [Headers and Includes](#headers-and-includes)
11. [File Header and Copyright](#file-header-and-copyright)
12. [Standard Library Usage](#standard-library-usage)
13. [Modern C++ Features](#modern-c-features)
14. [Inclusive and Respectful Language](#inclusive-and-respectful-language)
15. [Portable Code](#portable-code)
16. [Asserts and Error Handling Macros](#asserts-and-error-handling-macros)
17. [Best Practices Summary](#best-practices-summary)

---

## Code Formatting

### Indentation

- **Indent code by execution block**
- **Use TABS, not spaces** for whitespace at the beginning of a line
- **Set your tab size to 4 characters**
- Spaces are sometimes necessary and allowed for keeping code aligned regardless of the number of spaces in a tab (e.g., when aligning code that follows non-tab characters)

### Braces

Epic has a **long-standing usage pattern of putting braces on a new line**. Please continue to adhere to that.

```cpp
// Correct
if (condition)
{
    DoSomething();
}

// Incorrect
if (condition) {
    DoSomething();
}
```

**Each block of execution in an if-else statement should be in braces.** This prevents editing mistakes - when braces aren't used, someone could unwittingly add another line to an if block.

```cpp
// Correct
if (bCondition)
{
    DoSomething();
}

// Incorrect - avoid this
if (bCondition)
    DoSomething();
```

### Switch Statements

Except for empty cases (multiple cases having identical code), switch case statements should **explicitly label that a case falls through** to the next case. Either include a `break`, or include a `// falls through` comment in each case.

Other code control-transfer commands (`return`, `continue`, etc.) are fine as well.

**Always have a default case**, and include a `break` just in case someone adds a new case after the default.

```cpp
switch (condition)
{
    case 1:
        ...
        // falls through
    case 2:
        ...
        break;
    case 3:
        ...
        return;
    case 4:
    case 5:
        ...
        break;
    default:
        break;
}
```

### C# Formatting

If you are writing code in C#, **please also use tabs, not spaces**. The reason is that programmers often switch between C# and C++, and most prefer to use a consistent setting for tabs. Visual Studio defaults to using spaces for C# files, so you need to remember to change this setting when working on Unreal Engine code.

---

## Naming Conventions

### General Naming Rules

Epic uses **PascalCase** extensively throughout the codebase.

### Class Naming

- Template classes are prefixed by **T**
- Classes that inherit from UObject are prefixed by **U**
- Classes that inherit from AActor are prefixed by **A**
- Classes that inherit from SWidget are prefixed by **S**
- Classes that are abstract interfaces are prefixed by **I**
- Enums are prefixed by **E**
- Boolean variables must be prefixed by **b** (e.g., `bPendingDestruction`, `bHasFadedIn`)
- Most other classes are prefixed by **F**

### File Naming

- **File names should not be prefixed where possible**
  - For example: `Scene.cpp` instead of `UScene.cpp`
  - This makes it easy to use tools like Workspace Whiz or Visual Assist's Open File in Solution, by reducing the number of letters needed to identify the file you want

### Namespaces

You can use namespaces to organize your classes, functions, and variables where appropriate.

- You can use `using` declarations to only alias specific variables within a namespace into your scope (e.g., `using Foo::FBar`)
  - However, **we don't usually do that in Unreal code**
- **Macros cannot live in a namespace** but should be prefixed with **UE_** instead (e.g., `UE_LOG`)

---

## Function Naming and Parameters

### Function Naming Conventions

**Boolean-returning functions should ask a clear yes/no question:**

```cpp
// Good - asks a question
bool IsVisible() const;
bool ShouldClearBuffer() const;
bool HasAuthority() const;
bool CanJump() const;

// Bad - not a clear question
bool Visible() const;
bool CheckBuffer() const;
```

### Parameter Naming: Out and InOut

**Reference parameters that serve as outputs should be prefixed with `Out`:**

```cpp
void GetDimensions(float& OutWidth, float& OutHeight) const;
void FindClosestActor(AActor*& OutActor, float& OutDistance) const;
```

**Parameters that are both input and output should be prefixed with `InOut`:**

```cpp
void ProcessAndModify(FData& InOutData);
void TransformInPlace(FVector& InOutLocation);
```

**Boolean output parameters specifically should be prefixed with `bOut`:**

```cpp
bool TryGetValue(int32 Key, int32& OutValue) const;
bool TryGetPlayerState(APlayerState*& OutPlayerState, bool& bOutIsValid) const;
```

**Boolean input-output parameters should be prefixed with `bInOut`:**

```cpp
void ToggleAndProcess(bool& bInOutEnabled);
```

### Parameter Order

When mixing input and output parameters:
1. Input parameters first
2. Output parameters last

```cpp
// Good - inputs first, outputs last
void CalculateStats(const TArray<int32>& InValues, float& OutAverage, float& OutMax);

// Bad - mixed order
void CalculateStats(float& OutAverage, const TArray<int32>& InValues, float& OutMax);
```

---

## Type Declarations

### Standard Types

Unreal Engine uses explicit-sized types for portability:

- `int32` / `uint32` - 32-bit signed/unsigned integer
- `int64` / `uint64` - 64-bit signed/unsigned integer
- `int16` / `uint16` - 16-bit signed/unsigned integer
- `int8` / `uint8` - 8-bit signed/unsigned integer

The `int` and `unsigned int` types **vary in size across platforms**. They are guaranteed to be at least 32 bits in width and are acceptable in code when the integer width is unimportant. **Explicitly-sized types are used in serialized or replicated formats.**

### Special Types

- **TCHAR** for a character (**NEVER assume the size of TCHAR**)
- **PTRINT** for an integer that may hold a pointer (**NEVER assume the size of PTRINT**)
- **UBOOL** for boolean values (legacy, 4 bytes) - Note: Modern code uses `bool`

### Boolean Variables

Boolean variables must be prefixed by **b**:

```cpp
// Correct
bool bPendingDestruction;
bool bHasFadedIn;

// Incorrect
bool PendingDestruction;
bool IsFadedIn;
```

---

## Comments and Documentation

### Philosophy

**Comments document intent, and code documents implementation.**

Comments are communication, and communication is vital. The following guidelines are based on Kernighan & Pike's "The Practice of Programming":

### Write Self-Documenting Code

```cpp
// Bad:
t = s + l - b;

// Good:
TotalLeaves = SmallLeaves + LargeLeaves - SmallAndLargeLeaves;
```

### Write Useful Comments

```cpp
// Bad:
// increment Leaves
++Leaves;

// Good:
// we know there is another tea leaf
++Leaves;
```

### Do Not Over Comment Bad Code — Rewrite It

```cpp
// Bad:
// total number of leaves is sum of
// small and large leaves less the
// number of leaves that are both
t = s + l - b;

// Good:
TotalLeaves = SmallLeaves + LargeLeaves - SmallAndLargeLeaves;
```

### Function Documentation Format

Each function should be documented with the following components:

**Function purpose:** Documents the problem this function solves. Comments document intent, and code documents implementation.

**Parameter comments:** Each parameter comment should include:
- Units of measure
- Range of expected values
- "Impossible" values  
- Meaning of status/error codes

**Inputs/Outputs:** Document parameters that provide values to the function that may be changed by the function. Document as both an input and an output parameter.

**Outputs:** List all output-only parameters (where the value provided by the caller is ignored). Should include meaning of return values as well as what happens to the value in case of any error/failure.

**Return comment:** Documents the expected return value, just as an output variable is documented. To avoid redundancy, an explicit `@return` comment should not be used if the sole purpose of the function is to return this value and that is already documented in the function purpose.

**Extra information:** `@warning`, `@note`, `@see`, and `@deprecated` can optionally be used to document additional relevant information. Each should be declared on their own line following the rest of the comments.

### Example Documentation

```cpp
/**
 * This class does lots of neat things. It works
 * with a few other classes to do other neat things.
 */
class Tea : public WarmBeverage
{
    /** This stores the value of tea in china. */
    float Price;
    
    /**
     * Adds boiling water to the cup.
     * @param WaterAmount - Amount of water in milliliters (0-500ml)
     * @param Temperature - Water temperature in Celsius (90-100°C)
     * @return true if water was added successfully
     */
    bool AddBoilingWater(float WaterAmount, float Temperature);
};
```

---

## Pointer Types

Unreal Engine uses several different pointer types for different purposes. Understanding when to use each is critical.

### TObjectPtr<T>

`TObjectPtr` is meant as a **drop-in replacement for raw pointers** for UObject-derived classes.

**When to use:**
- For UPROPERTY members in UObject-derived classes
- Replaces raw `UObject*` pointers in UPROPERTYs

**Benefits:**
- Supports advanced cook-time dependency tracking
- Enables barriers for garbage collection (unlocks incremental GC marking)
- Supports replication
- Strong reference that prevents garbage collection of target object

**Important notes:**
- TObjectPtr is only GC-safe when marked as UPROPERTY
- Function parameters and return values should still use raw `UObject*`
- Cannot be used for reference parameters (`UObject*&`) - use temporary raw pointer instead

```cpp
// UObject class member - use TObjectPtr
UPROPERTY()
TObjectPtr<UMyComponent> MyComponent;

// Function parameter - use raw pointer
void ProcessComponent(UMyComponent* Component);

// Return value - use raw pointer  
UMyComponent* GetComponent() const { return MyComponent; }
```

### TSharedPtr<T> and TSharedRef<T>

Thread-safe, reference-counted smart pointers for **non-UObject** classes.

**When to use:**
- For objects that are NOT derived from UObject
- When you need shared ownership semantics
- For objects that need to outlive a single owner

**Key differences:**
- `TSharedPtr` can be null
- `TSharedRef` cannot be null (guaranteed valid)

**Performance consideration:**
Avoid passing TSharedPtr/TSharedRef as function parameters where possible - pass the referenced object instead (preferably as `const&`) to avoid reference-counting overhead.

```cpp
// Non-UObject class
class FMyClass
{
    TSharedPtr<FData> SharedData;
};

// Function parameter - pass object, not smart pointer
void ProcessData(const FData& Data);  // Preferred
void ProcessData(TSharedPtr<FData> Data);  // Avoid - overhead
```

### TWeakObjectPtr<T>

Non-owning pointer to UObject that **does not prevent garbage collection**.

**When to use:**
- For non-owning references to UObjects
- For asset references
- When you need to detect if an object has been garbage collected
- In delegates/lambdas to avoid keeping objects alive

**Important:** Always check validity before dereferencing.

```cpp
TWeakObjectPtr<AActor> CachedActor;

// Always check before using
if (CachedActor.IsValid())
{
    CachedActor->DoSomething();
}

// Or use Get() with null check
if (AActor* Actor = CachedActor.Get())
{
    Actor->DoSomething();
}
```

### TStrongObjectPtr<T>

Strong pointer to UObject for **non-UObject owners** (like regular C++ classes).

**When to use:**
- When a non-UObject class needs to own a UObject
- NOT for use in UObject-derived classes (can create uncollectable cycles)
- Cannot be marked as UPROPERTY

```cpp
// In a non-UObject class
class FMyManager
{
    TStrongObjectPtr<UMyObject> OwnedObject;
};
```

### TSoftObjectPtr<T>

Soft reference that stores a path to an asset and loads **on demand**.

**When to use:**
- For optional asset references
- When you want to reduce editor startup time
- When you want to control when assets are loaded
- For large assets that shouldn't always be in memory

```cpp
UPROPERTY(EditAnywhere)
TSoftObjectPtr<UTexture2D> OptionalTexture;

// Later, load when needed
if (OptionalTexture.IsNull() == false)
{
    UTexture2D* Texture = OptionalTexture.LoadSynchronous();
}
```

### Pointer Type Selection Guide

| Use Case | Pointer Type |
|----------|-------------|
| UPROPERTY member in UObject | `TObjectPtr<T>` (or raw `T*` for legacy) |
| Function parameter (UObject) | `T*` (raw pointer) |
| Function return value (UObject) | `T*` (raw pointer) |
| Non-owning reference to UObject | `TWeakObjectPtr<T>` |
| Non-UObject shared ownership | `TSharedPtr<T>` or `TSharedRef<T>` |
| Non-UObject in UObject class | `TSharedPtr<T>` as UPROPERTY |
| UObject owned by non-UObject | `TStrongObjectPtr<T>` |
| Asset reference | `TSoftObjectPtr<T>` |
| Component in constructor | Store in `TObjectPtr<T>` marked VisibleAnywhere |

### Object Lifetime and GC Safety

When working with UObjects, pointer choice is tightly coupled to Unreal's garbage collector:

- **Hard references** to UObjects that should keep the object alive must be:
  - Members of a `UObject`-derived type, and
  - Marked with `UPROPERTY`, typically using `TObjectPtr<SomeUObject>`.

```cpp
UCLASS()
class UMyManager : public UObject
{
    GENERATED_BODY()

public:
    UPROPERTY()
    TObjectPtr<UMyAsset> Asset;         // Hard GC reference
};
```

**Raw `UObject*` members in non-UObject classes are not seen by the GC** and can dangle when the referenced object is destroyed.

If a non-UObject type needs to own a UObject, use `TStrongObjectPtr` to keep it alive safely:

```cpp
class FMyHelper
{
public:
    TStrongObjectPtr<UMyAsset> OwnedAsset;
};
```

Use `TWeakObjectPtr` for optional, non-owning references that should become nullptr automatically when the object is destroyed.

Soft references (`TSoftObjectPtr`, `TSoftClassPtr`) are for references that may not be loaded yet and can be reloaded by asset path.

**Function parameters and return types** should generally continue to use raw `UObject*` or references; they do not participate in GC.

---

## Boolean Parameters

### Avoid Multiple Boolean Parameters

Functions with multiple boolean parameters can be confusing and error-prone due to anonymous literal problems (what does `true, false, true` mean?)

```cpp
// Bad - what do these booleans mean?
FCup* Cup = MakeCupOfTea(Tea, false, true, true);
```

### Use Strongly-Typed Enums Instead

```cpp
// Good - clear and type-safe
enum class ETeaFlags
{
    None   = 0,
    Milk   = 0x01,
    Sugar  = 0x02,
    Honey  = 0x04,
    Lemon  = 0x08
};
ENUM_CLASS_FLAGS(ETeaFlags)

FCup* MakeCupOfTea(FTea* Tea, ETeaFlags Flags = ETeaFlags::None);

// Usage is self-documenting
FCup* Cup = MakeCupOfTea(Tea, ETeaFlags::Milk | ETeaFlags::Honey);
```

**Benefits of enum flags:**
- Prevents accidental transposing of arguments
- Avoids accidental conversion from pointer and integer arguments
- Removes need to repeat redundant defaults
- More efficient
- Self-documenting

### When Booleans Are Acceptable

It is acceptable to use bools as arguments when they represent the **complete state** to be passed to a function, such as a setter:

```cpp
void FWidget::SetEnabled(bool bEnabled);  // OK - clear single state
```

### Out and In/Out Parameters

When a function needs to return multiple values, prefer **out parameters** (non-const references) and a single `bool` return value to indicate success.

- Name out parameters with an `Out` or `InOut` prefix so their role is clear.
- For boolean out parameters, use the `b` prefix *and* `Out`/`InOut` (for example `bOutResult`, `bInOutDirty`).

```cpp
bool TryGetPlayerLocation(FVector& OutLocation) const;
bool TryFindResult(int32 InValue, int32& OutResult);
void UpdateBounds(const FBox& InNewBounds, FBox& InOutCombinedBounds);
```

This makes call sites easier to read and matches how many engine APIs name out parameters.

---

## Strongly-Typed Enums

Use **enum class** (strongly-typed enums) instead of old-style enums.

### Defining Enums

```cpp
// Use enum class
enum class EColorType
{
    Red,
    Green,
    Blue
};

// Old style - avoid
enum EColorType
{
    Red,
    Green,
    Blue
};
```

### Enum Prefixes

- Enum type names are prefixed with **E**
- Enum values should use **PascalCase** without prefixes

```cpp
enum class EWeaponType
{
    Knife,
    Pistol,
    Rifle
};
```

### Using Enums as Flags

For bitfield enums, use the `ENUM_CLASS_FLAGS` macro:

```cpp
enum class ETeaFlags
{
    None   = 0,
    Milk   = 0x01,
    Sugar  = 0x02,
    Honey  = 0x04,
    Lemon  = 0x08
};
ENUM_CLASS_FLAGS(ETeaFlags)

// Now you can use bitwise operators
ETeaFlags Combined = ETeaFlags::Milk | ETeaFlags::Sugar;
```

---

## Use of const

**Whenever you can use const in C++, use const.** Particularly on function parameters and class methods.

`const` is **documentation as much as it is a compiler directive**.

### Const Correctness Guidelines

All code should strive to be const-correct:

```cpp
// Passing function arguments by const pointer or reference
void ProcessData(const FString& InputData);
void AnalyzeActor(const AActor* Actor);

// Flagging methods as const if they do not modify the object
int32 GetValue() const;
FVector GetLocation() const;

// Using const iteration over containers
for (const FString& Name : NameArray)
{
    UE_LOG(LogTemp, Log, TEXT("%s"), *Name);
}
```

### Const Member Functions

Mark member functions `const` when they don't modify the object:

```cpp
class FMyClass
{
public:
    // Const - doesn't modify state
    int32 GetHealth() const { return Health; }
    bool IsAlive() const { return Health > 0; }
    
    // Non-const - modifies state
    void TakeDamage(int32 Damage) { Health -= Damage; }
    
private:
    int32 Health;
};
```

---

## Modern C++ Features

Unreal Engine is built to be massively portable to many C++ compilers, so we are careful to use features that are compatible with the compilers we might be supporting. Sometimes, features are so useful that we will wrap them in macros and use them pervasively.

### Auto Keyword

Epic's coding standard is deliberately conservative about `auto`.

- **Avoid `auto` by default.** Prefer writing the type explicitly so the code is self-documenting.
- Exceptions are mainly:
  - Storing lambdas or other types that cannot be spelled easily.
  - Very rare cases where the exact type is unimportant and the initializer makes the semantics obvious.

```cpp
// Preferred – spell out the type
TArray<FString> Names = GetNames();
TMap<FName, int32>::TConstIterator It(NamesMap);

// Acceptable – lambda type cannot be written explicitly
auto Process = [this](FMyStruct& Item)
{
    // ...
};
```

**Do not:**
- Use `auto` just to save typing for ordinary types.
- Use C++17 structured bindings (`auto [A, B] = Something;`) – they are effectively a variadic auto and are disallowed in engine code.

### nullptr

Always use `nullptr` instead of `NULL` or `0` for null pointers.

```cpp
// Good
UObject* Object = nullptr;
if (Object != nullptr)

// Bad
UObject* Object = NULL;
UObject* Object = 0;
```

### Range-Based For Loops

Use range-based for loops where appropriate for cleaner, more readable code:

```cpp
// Good - range-based for
for (AActor* Actor : ActorArray)
{
    Actor->DoSomething();
}

// Also good - const reference to avoid copies
for (const FString& Name : NameList)
{
    UE_LOG(LogTemp, Log, TEXT("%s"), *Name);
}

// Traditional for loop when index is needed
for (int32 i = 0; i < Array.Num(); ++i)
{
    ProcessElement(Array[i], i);
}
```

### Lambdas

Lambdas are acceptable but use them judiciously. Be mindful of captures, especially with UObjects.

```cpp
// Good - capture by value for simple types
TArray<int32> FilteredNumbers = Numbers.FilterByPredicate([Threshold](int32 Num)
{
    return Num > Threshold;
});

// Good - weak pointer capture for UObjects in delegates
FSimpleDelegate MyDelegate;
TWeakObjectPtr<UMyObject> WeakObject = MyObject;
MyDelegate.BindLambda([WeakObject]()
{
    if (UMyObject* StrongObject = WeakObject.Get())
    {
        StrongObject->DoSomething();
    }
});

// Be careful with captures - avoid dangling references
```

### Move Semantics

Unreal containers support move semantics. Use `MoveTemp()` to move instead of copy:

```cpp
TArray<FLargeData> SourceArray;
TArray<FLargeData> DestArray;

// Move instead of copy
DestArray = MoveTemp(SourceArray);  // SourceArray is now empty

// Move individual elements
DestArray.Add(MoveTemp(LargeObject));
```

---

## API Design and Function Patterns

### Class Organization

Classes should be organized in a consistent order:

1. Public interface first
2. Protected members
3. Private implementation

Within each section:
1. Types and constants
2. Constructors/destructor
3. Member functions
4. Member variables (UPROPERTYs first)

```cpp
class MYPROJECT_API UMyComponent : public UActorComponent
{
    GENERATED_BODY()
    
public:
    // Types
    enum class EState { Active, Inactive };
    
    // Constructor
    UMyComponent();
    
    // Public interface functions
    void Initialize();
    int32 GetValue() const;
    
protected:
    // Protected functions
    virtual void BeginPlay() override;
    
    // Protected properties
    UPROPERTY(BlueprintReadOnly)
    int32 ProtectedValue;
    
private:
    // Private functions
    void InternalUpdate();
    
    // Private properties
    UPROPERTY()
    TObjectPtr<UObject> CachedObject;
    
    // Private member variables (non-UPROPERTY)
    float InternalTimer;
};
```

### Component Creation Pattern

For components created in constructors, use `CreateDefaultSubobject`:

```cpp
// In constructor
UMyComponent::UMyComponent()
{
    // Create component - no spaces in name, can cause bugs
    MyMeshComponent = CreateDefaultSubobject<UStaticMeshComponent>(TEXT("MyMesh"));
    
    // Store in VisibleAnywhere UPROPERTY
    // Never use EditAnywhere for components created in constructor
}

// In header
UPROPERTY(VisibleAnywhere, BlueprintReadOnly)
TObjectPtr<UStaticMeshComponent> MyMeshComponent;
```

**Important:**
- Do NOT use spaces in `CreateDefaultSubobject` FName parameters (triggers bugs)
- Use `VisibleAnywhere`, NOT `EditAnywhere` for constructor-created components
- Store result in UPROPERTY marked VisibleAnywhere or VisibleDefaultsOnly
- `BlueprintReadOnly` is preferred over `BlueprintReadWrite` for component properties

### Object Creation Patterns

```cpp
// Creating UObjects
UMyObject* NewObject = NewObject<UMyObject>(Outer);

// Creating Actors
AActor* NewActor = GetWorld()->SpawnActor<AMyActor>(ActorClass, SpawnTransform);

// Components in constructors only
UActorComponent* Component = CreateDefaultSubobject<UMyComponent>(TEXT("ComponentName"));
```

### Default Member Initializers

You can initialize UPROPERTY members directly in the class declaration using **default member initializers**:

```cpp
UCLASS()
class UMyClass : public UObject
{
    GENERATED_BODY()
    
public:
    // Default member initializer
    UPROPERTY(EditAnywhere)
    int32 MaxHealth = 100;
    
    UPROPERTY(EditAnywhere)
    float Speed = 600.0f;
    
    UPROPERTY(EditAnywhere)
    bool bIsInvulnerable = false;
};
```

#### Tradeoffs of Default Member Initializers

**Advantages:**
- Cleaner header files - defaults are visible where the property is declared
- Less code in constructors
- Ensures values are initialized even if constructor is modified

**Disadvantages:**
- **Rebuild cost**: Changing a default value in a header requires recompiling all files that include it
- **Patchability**: Cannot change defaults in a patch without recompiling dependent code
- **Limitations**: Cannot use complex expressions or call functions
  - Cannot reference `this`
  - Cannot use other member variables
  - Cannot call `GetWorld()` or other methods

**Recommendation:**
- Use default member initializers for **simple, stable values** (booleans, small integers, simple floats)
- Use constructor initialization for:
  - Values that may change frequently during development
  - Complex initialization requiring function calls
  - Values that depend on other members or parameters
  - Component creation (must use `CreateDefaultSubobject`)

```cpp
// Good uses of default member initializers
UPROPERTY()
int32 MaxAmmo = 30;

UPROPERTY()
bool bAutoReload = true;

// Should be in constructor instead
UPROPERTY()
TObjectPtr<USceneComponent> RootComp;  // Needs CreateDefaultSubobject

// Constructor for complex initialization
UMyClass::UMyClass()
{
    RootComp = CreateDefaultSubobject<USceneComponent>(TEXT("Root"));
    // Complex initialization here
}
```

---

## Standard Library Usage

Historically, UE has avoided direct use of the C and C++ standard libraries for the following reasons:

- Replace slow implementations with our own that provide additional control over memory allocation
- Add new functionality before it's widely available
- Making desirable, but non-standard, behavioral changes
- Having consistent syntax across the codebase
- Avoiding constructs which are incompatible with UE's idioms

However, the standard library has matured and includes functionality that we don't want to wrap with an abstraction layer or reimplement ourselves.

### Current Guidance

**When there is a choice between a standard library feature instead of our own, prefer the option that gives superior results.** It's also important to remember that consistency is valued highly.

If a legacy UE implementation is no longer serving a purpose, we may choose to deprecate it and migrate all usage toward the standard library.

**Avoid mixing UE idioms and standard library idioms in the same API.**

### Common Idioms

The following table lists common idioms along with recommendations:

| Category | Use UE | Use STL | Notes |
|----------|--------|---------|-------|
| Containers | TArray, TMap, TSet | - | Standard containers should be avoided except in interop code |
| Strings | FString, FName, FText | - | Use UE string types |
| Smart Pointers (UObject) | TObjectPtr, TWeakObjectPtr | - | UObjects require special GC handling |
| Smart Pointers (non-UObject) | TSharedPtr, TUniquePtr | std::shared_ptr, std::unique_ptr | Either acceptable, prefer UE for consistency |
| Type Traits | - | std::enable_if, std::is_same, etc. | STL preferred, Epic type traits deprecated |
| Algorithms | Algo namespace | std::algorithms | Either acceptable |
| Numeric Limits | - | std::numeric_limits | STL preferred |

### Deprecated Epic Features

The following Epic implementations are officially deprecated in favor of STL:
- Epic type traits (TEnableIf, etc.) → Use std::enable_if, std::is_same, etc.
- Use `if constexpr` (C++17) where possible instead of SFINAE
- TLazyObjectPtr → Use TSoftObjectPtr instead

---

## Virtual Functions

When declaring a virtual function in a derived class that overrides a virtual function in the parent class, you **must use the `virtual` keyword**. 

Although it is optional according to the C++ standard (because the 'virtual' behavior is inherited), code is much clearer if all virtual function declarations use the `virtual` keyword.

```cpp
class Base
{
public:
    virtual void DoSomething();
};

class Derived : public Base
{
public:
    virtual void DoSomething() override;  // Use both 'virtual' and 'override'
};
```

---

## Headers and Includes

### Header Guards

All headers should protect against multiple includes with the **`#pragma once` directive**. Note that all compilers we use support `#pragma once`.

```cpp
#pragma once

// Header contents
```

### Minimize Physical Coupling

In general, **try to minimize physical coupling**. In particular:

- **Avoid including standard library headers from other headers**
- **If you can use forward declarations instead of including a header, do so**

```cpp
// Forward declaration preferred when possible
class UMyClass;

// Only include when you need the full definition
#include "MyClass.h"
```

---

## File Header and Copyright

For engine and plugin files that follow Epic's conventions, the first non-empty line in a source file should be the copyright notice:

```cpp
// Copyright Epic Games, Inc. All Rights Reserved.
```

This line must appear before `#pragma once` or any includes.

**For game/project code outside the engine**, use a consistent project-wide header, but do not remove or alter Epic's copyright in engine code.

Epic's continuous integration (CIS) checks for this header in files that ship with the engine and will fail builds if it is missing or modified.

When adding new engine-style modules or plugins, adopt this pattern so your code matches the rest of the ecosystem.

---

## Inclusive and Respectful Language

When you work in the Unreal Engine codebase, we encourage you to strive to use **respectful, inclusive, and professional language**.

### When This Applies

Word choice applies when you:
- Name classes, functions, data structures, types, variables, files and folders, plugins
- Write snippets of user-facing text for the UI, error messages, and notifications
- Write about code, such as in comments and changelist descriptions

### Guidelines

**Do not use:**
- Metaphors or similes that reinforce stereotypes
  - Examples include contrast "black and white" or "blacklist" and "whitelist"
- Words that refer to historical trauma or lived experience of discrimination
  - Examples include "slave", "master", and "nuke"

**Do use:**
- Refer to hypothetical people as **they, them, and their**, even in the singular

### Be Precise with Overloaded Words

Many terms we use for their technical meanings also have other meanings outside of technology. Examples include "abort", "execute", or "native". When you use words like these, **always be precise and examine the context in which they appear**.

### Terminology Replacements

The following terminology should be replaced with better alternatives:

| Avoid | Use Instead |
|-------|-------------|
| blacklist | deny list, block list, exclude list, avoid list, unapproved list, forbidden list, permission list |
| whitelist | allow list, include list, trust list, safe list, prefer list, approved list, permission list |
| master | primary, source, main, controller |
| slave | secondary, replica, subordinate, worker |

Epic Games is actively working to bring their code in line with these principles.

---

## Portable Code

Unreal Engine is built to be **massively portable to many C++ compilers**, so we are careful to use features that are compatible with the compilers we might be supporting.

Sometimes, features are so useful that we will wrap them in macros and use them pervasively.

### Platform-Specific Code

When writing platform-specific code:
- Use the platform abstraction layer where possible
- Clearly mark platform-specific sections with preprocessor directives
- Provide fallback implementations for unsupported platforms

```cpp
#if PLATFORM_WINDOWS
    // Windows-specific code
#elif PLATFORM_MAC
    // Mac-specific code
#else
    // Generic fallback
#endif
```

### Compiler Compatibility

- Unreal Engine compiles **without exceptions or RTTI**
- `dynamic_cast` is not available (it's #defined as a macro - avoid entirely)
- Use `Cast<T>()` for UObject casting instead

```cpp
// Good - UE casting
UMyClass* MyObject = Cast<UMyClass>(SomeObject);
if (MyObject)
{
    MyObject->DoSomething();
}

// Bad - dynamic_cast not available
// UMyClass* MyObject = dynamic_cast<UMyClass*>(SomeObject);
```

### Third Party Code

When integrating third-party libraries or code:

**Maintain original formatting** - Do not reformat third-party code to match Epic's style. This makes:
- Merging upstream changes easier
- Comparing with original source simpler
- Maintaining the code less error-prone

**Encapsulate appropriately** - Wrap third-party interfaces with your own abstractions where it makes sense, but don't unnecessarily duplicate code.

**Document source and version** - Clearly document:
- Where the code came from
- What version you're using
- What modifications (if any) you've made
- License information

```cpp
// ThirdParty/SomeLibrary/SomeLibrary.h
// Source: https://github.com/example/somelibrary v2.1.0
// License: MIT
// Modifications: None

// Original formatting preserved below
#include "original_style.h"

namespace some_library {
    class SomeClass {
        // Original style maintained
    };
}
```

---

## Asserts and Error Handling Macros

Unreal provides a family of assert and verification macros. They have different behaviors and intended uses:

- `check(Expr)` – Fatal assert in non-shipping builds. Use for programmer errors that must never happen.
- `checkf(Expr, TEXT("Message %d"), Value)` – Like `check` but with a formatted message.
- `checkNoEntry()`, `checkNoReentry()`, `checkNoRecursion()` – Specialized asserts for control-flow and recursion invariants.

- `ensure(Expr)` – Reports a non-fatal error but allows execution to continue. Compiles to a no-op in shipping builds.
- `ensureAlways(Expr)` – Like `ensure` but also active in shipping. Use sparingly, for issues that are serious but survivable.

- `verify(Expr)` – Like `check`, but the expression is still evaluated even when the assert is compiled out. Useful when the expression has side effects that must always run.

**Guidelines:**

- Use `check`/`checkf` for conditions that indicate a bug in your code and should never occur in valid gameplay.
- Use `ensure`/`ensureAlways` for conditions that *might* occur in the field and where you want to log and recover when possible.
- Avoid putting important side effects inside `check` expressions – they are compiled out in shipping.

### Logging Conventions

Logging is the main way to surface information without stopping execution.

- Define a log category per module or subsystem instead of using `LogTemp`:

```cpp
// In a header
DECLARE_LOG_CATEGORY_EXTERN(LogMyFeature, Log, All);

// In a single .cpp
DEFINE_LOG_CATEGORY(LogMyFeature);
```

**Use appropriate verbosity:**
- `Verbose` / `VeryVerbose` – noisy debugging output
- `Log` – general informational messages
- `Warning` – recoverable problems
- `Error` – serious failures that still allow the engine to continue
- `Fatal` – crashes with a call stack

**Avoid leaving `LogTemp` and `VeryVerbose` spam in shipping code.** Gate noisy logs behind compile-time guards or runtime conditionals as needed.

---

## Object Lifecycle and Garbage Collection

Understanding Unreal's garbage collection system is critical for safe UObject usage.

### Garbage Collection Fundamentals

**The GC only "sees" UObject pointers marked as UPROPERTY** - anything not reachable through UPROPERTYs is considered unreachable and may be deleted.

```cpp
UCLASS()
class UMyClass : public UObject
{
    GENERATED_BODY()
    
public:
    // GC-safe - marked as UPROPERTY
    UPROPERTY()
    TObjectPtr<UOtherObject> ManagedObject;
    
    // DANGER - not marked, will be garbage collected!
    UOtherObject* UnmanagedObject;  // DO NOT DO THIS
};
```

### UObject Pointer Rules by Owner Type

#### In UObject-Derived Classes (Actors, Components, etc.)

**For owning references:**
```cpp
UPROPERTY()
TObjectPtr<UMyComponent> OwnedComponent;  // Prevents GC
```

**For non-owning references:**
```cpp
UPROPERTY()
TWeakObjectPtr<AActor> CachedActor;  // Does not prevent GC, safe to check
```

**Never use raw pointers for long-lived references:**
```cpp
// DANGER - will dangle after GC!
UObject* BadPointer;  // No UPROPERTY = GC can delete it
```

#### In Non-UObject Classes (Plain C++ Classes)

**You CANNOT use UPROPERTY in non-UObject classes**. Options:

```cpp
class FMyManager  // Not a UObject
{
    // Option 1: TStrongObjectPtr - prevents GC
    TStrongObjectPtr<UMyObject> OwnedObject;
    
    // Option 2: TWeakObjectPtr - allows GC, must check validity
    TWeakObjectPtr<UMyObject> CachedObject;
    
    // NEVER: Raw pointer - will dangle!
    // UMyObject* BadPointer;
};
```

#### Static and Global Variables

**Hard UObject pointers in static/global scope are NOT GC-safe:**

```cpp
// DANGER - not safe!
static UObject* GlobalObject;  // Will be garbage collected!

// Safe options:
static TWeakObjectPtr<UObject> GlobalWeakObject;  // Check before use
static TStrongObjectPtr<UObject> GlobalStrongObject;  // Keeps alive (use carefully)
```

### Pointer Type Selection by Context

| Context | Owning Reference | Non-Owning Reference |
|---------|------------------|---------------------|
| UPROPERTY in UObject | `TObjectPtr<T>` | `TWeakObjectPtr<T>` |
| Non-UObject class | `TStrongObjectPtr<T>` | `TWeakObjectPtr<T>` |
| Function parameter | `T*` (raw pointer) | `T*` (raw pointer) |
| Function return | `T*` (raw pointer) | `T*` (raw pointer) |
| Local variable | `T*` (raw pointer) | `T*` (raw pointer) |
| Static/Global | `TStrongObjectPtr<T>` (rare) | `TWeakObjectPtr<T>` |

### UPROPERTY Timing

**UPROPERTYs are set AFTER your C++ constructor returns:**

```cpp
UCLASS()
class AMyActor : public AActor
{
    GENERATED_BODY()
    
public:
    UPROPERTY(EditAnywhere)
    int32 ConfigValue;
    
    AMyActor()
    {
        // WRONG - ConfigValue not set yet!
        // UE_LOG(LogTemp, Log, TEXT("Config: %d"), ConfigValue);
    }
    
    virtual void PostInitProperties() override
    {
        Super::PostInitProperties();
        // OK - properties are set now
        UE_LOG(LogTemp, Log, TEXT("Config: %d"), ConfigValue);
    }
};
```

**Use these callbacks to read UPROPERTYs:**
- `PostInitProperties()` - After properties set
- `PostLoad()` - After deserialization  
- `OnConstruction()` - For actors, after components created (Blueprints)
- `BeginPlay()` - When gameplay begins

### Safe Weak Pointer Usage

Always check weak pointers before dereferencing:

```cpp
TWeakObjectPtr<AActor> WeakActor;

// Method 1: IsValid()
if (WeakActor.IsValid())
{
    WeakActor->DoSomething();
}

// Method 2: Get() with null check
if (AActor* Actor = WeakActor.Get())
{
    Actor->DoSomething();
}

// Method 3: Pin to strong pointer (shared pointers)
if (TSharedPtr<FData> StrongData = WeakData.Pin())
{
    StrongData->Use();
}
```

### Avoiding Uncollectable Cycles

**Do NOT create cycles with TStrongObjectPtr:**

```cpp
// DANGER - uncollectable cycle!
UCLASS()
class UBadClass : public UObject
{
    UPROPERTY()
    TStrongObjectPtr<UBadClass> SelfReference;  // Creates uncollectable cycle!
};
```

TObjectPtr self-references are safe; TStrongObjectPtr self-references are not.

---

## Reflection and Blueprint Exposure

Unreal's reflection system has specific constraints and conventions.

### Namespace Restrictions

**UCLASS, USTRUCT, UENUM, and UINTERFACE must be in the global namespace:**

```cpp
// WRONG - inside namespace
namespace MyNamespace
{
    UCLASS()  // ERROR: UHT cannot process this
    class UMyClass : public UObject
    {
        GENERATED_BODY()
    };
}

// CORRECT - global namespace
UCLASS()
class MYPROJECT_API UMyClass : public UObject
{
    GENERATED_BODY()
};
```

Regular C++ classes (F-prefixed) can use namespaces normally.

### Blueprint-Exposed Enums

**Blueprint-exposed enums MUST be `enum class` with `uint8` underlying type:**

```cpp
// Correct for Blueprint exposure
UENUM(BlueprintType)
enum class EWeaponType : uint8
{
    Pistol,
    Rifle,
    Shotgun
};

// Won't work with Blueprints - missing uint8
UENUM(BlueprintType)
enum class EBadType  // Missing : uint8
{
    Value1,
    Value2
};
```

### UPROPERTY Specifiers

**EditAnywhere vs EditDefaultsOnly vs EditInstanceOnly:**

```cpp
UCLASS()
class AMyActor : public AActor
{
    GENERATED_BODY()
    
public:
    // Editable in both class defaults and instances
    UPROPERTY(EditAnywhere, BlueprintReadWrite)
    float EditableEverywhere;
    
    // Only editable in class defaults (Blueprint editor)
    UPROPERTY(EditDefaultsOnly, BlueprintReadWrite)
    float ClassDefault;
    
    // Only editable on placed instances
    UPROPERTY(EditInstanceOnly, BlueprintReadWrite)
    float InstanceOnly;
    
    // Components created in constructor
    UPROPERTY(VisibleAnywhere, BlueprintReadOnly)  // NOT EditAnywhere!
    TObjectPtr<USceneComponent> RootComp;
};
```

**Critical rule for constructor-created components:**
- Use `VisibleAnywhere` or `VisibleDefaultsOnly`
- **NEVER use `EditAnywhere`** - causes serialization bugs and Blueprint corruption

### Category Naming

Use consistent category naming with pipe delimiters:

```cpp
UPROPERTY(EditAnywhere, Category="MyActor|Combat")
int32 Health;

UPROPERTY(EditAnywhere, Category="MyActor|Combat")
float Damage;

UPROPERTY(EditAnywhere, Category="MyActor|Movement")
float Speed;
```

### Meta Specifiers

Common meta specifiers for better editor integration:

```cpp
UPROPERTY(EditAnywhere, meta=(ClampMin="0.0", ClampMax="1.0"))
float NormalizedValue;

UPROPERTY(EditAnywhere, meta=(UIMin="0", UIMax="100"))
int32 Percentage;

UPROPERTY(EditAnywhere, meta=(MustImplement="MyInterface"))
TSubclassOf<AActor> ActorClass;

UFUNCTION(BlueprintCallable, meta=(DeprecatedFunction, DeprecationMessage="Use NewFunction instead"))
void OldFunction();
```

---

## Best Practices Summary

### File Headers

1. ✅ **Engine/plugin code: Include Epic copyright header** as first line: `// Copyright Epic Games, Inc. All Rights Reserved.`
2. ✅ **Game/project code: Use consistent project header** (not Epic's copyright)
3. ✅ **CIS will fail builds** for engine code without proper copyright header

### Formatting

3. ✅ **Use tabs (4 spaces wide) for indentation**
4. ✅ **Put braces on new lines** (Epic's long-standing pattern)
5. ✅ **Always use braces for if/else blocks** (prevents editing mistakes)
6. ✅ **Label fall-through in switch statements** or use break
7. ✅ **Always have a default case in switch statements**

### Naming

8. ✅ **Prefix classes appropriately:**
   - UObject-derived: `U`
   - AActor-derived: `A`
   - SWidget-derived: `S`
   - Template classes: `T`
   - Enums: `E`
   - Interfaces: `I`
   - Most other classes: `F`
9. ✅ **Prefix boolean variables with `b`** (bIsAlive, bHasAuthority)
10. ✅ **Boolean-returning functions ask yes/no questions** (IsVisible, CanJump, ShouldClear)
11. ✅ **Prefix output parameters with `Out`** (OutWidth, OutHeight)
12. ✅ **Prefix boolean outputs with `bOut`** (bOutSuccess, bOutIsValid)
13. ✅ **Prefix input-output parameters with `InOut`** or `bInOut`
14. ✅ **Use PascalCase for most identifiers**
15. ✅ **Do not prefix file names** (Scene.cpp not UScene.cpp)
16. ✅ **Macros should be prefixed with `UE_`** (UE_LOG)

### Types

17. ✅ **Use explicit-sized integer types** (int32, uint32, int64, uint64)
18. ✅ **Use TCHAR for characters** (never assume size)
19. ✅ **Use nullptr instead of NULL or 0**
20. ✅ **Use enum class for strongly-typed enums**
21. ✅ **Blueprint enums must be `enum class EType : uint8`**
22. ✅ **Use ENUM_CLASS_FLAGS for bitfield enums**

### Pointers and Memory

23. ✅ **Use TObjectPtr<T> for UObject UPROPERTYs in UObject classes**
24. ✅ **Use raw pointers (T*) for function parameters/returns**
25. ✅ **Use TWeakObjectPtr for non-owning UObject references**
26. ✅ **Use TStrongObjectPtr for UObjects owned by non-UObject classes**
27. ✅ **Use TSharedPtr for non-UObject shared ownership**
28. ✅ **Use TSoftObjectPtr for asset references**
29. ✅ **Always check TWeakObjectPtr validity before dereferencing**
30. ✅ **NEVER use raw UObject* without UPROPERTY in UObject classes** (will be GC'd)
31. ✅ **NEVER use static/global raw UObject pointers** (not GC-safe)

### Functions and Methods

32. ✅ **Use const wherever possible** (parameters, methods, iteration)
33. ✅ **Mark virtual functions with `virtual` keyword** even in derived classes
34. ✅ **Document all functions** with purpose and parameter info
35. ✅ **Avoid multiple boolean parameters** - use enums instead
36. ✅ **Pass large objects by const reference**

### Memory and Objects

37. ✅ **Use CreateDefaultSubobject for components in constructors**
38. ✅ **Mark constructor components as VisibleAnywhere, NEVER EditAnywhere**
39. ✅ **Use NewObject for runtime UObject creation**
40. ✅ **Use SpawnActor for AActor creation**
41. ✅ **All UPROPERTYs are set AFTER constructor** - read in PostInitProperties/BeginPlay
42. ✅ **Use MoveTemp() for moving instead of copying large objects**
43. ✅ **Default member initializers only for simple, stable values**
44. ✅ **Constructor initialization for components, complex logic, or frequently-changing values**

### Code Organization

45. ✅ **Use `#pragma once` for header guards**
46. ✅ **Minimize physical coupling** with forward declarations
47. ✅ **Avoid including standard library headers from other headers**
48. ✅ **Organize classes: public → protected → private**
49. ✅ **Put UPROPERTYs before non-UPROPERTY members**

### Comments and Documentation

50. ✅ **Comments document intent, code documents implementation**
51. ✅ **Write self-documenting code** with clear variable names
52. ✅ **Write useful comments**, not obvious ones
53. ✅ **Don't over-comment bad code - rewrite it**
54. ✅ **Document function purpose, parameters, and return values**

### Modern C++

55. ✅ **AVOID auto except where type cannot be written (lambdas) or is extremely verbose (iterators)**
56. ✅ **NEVER use C++20 structured bindings** (effectively variadic auto)
57. ✅ **Use range-based for loops** where appropriate
58. ✅ **Use lambdas carefully** - watch captures with UObjects
59. ✅ **Prefer const references in range-based for loops**
60. ✅ **Use weak pointers in lambda captures for UObjects**

### Asserts and Error Handling

61. ✅ **Use `check()` for programmer errors that should never happen**
62. ✅ **Use `ensure()` for recoverable errors with fallback logic**
63. ✅ **Use `verify()` when expression must execute even in Shipping**
64. ✅ **Use `checkSlow()` for expensive debug-only checks**
65. ✅ **Never use logging for flow control** - use return values

### Garbage Collection and Lifecycle

66. ✅ **Only UPROPERTYs are seen by GC** - unmarked pointers will be collected
67. ✅ **Check weak pointers before dereferencing** (IsValid() or Get())
68. ✅ **Never create TStrongObjectPtr cycles** (uncollectable)
69. ✅ **Don't read UPROPERTYs in constructor** - not set until PostInitProperties
70. ✅ **Use PostInitProperties, PostLoad, BeginPlay** to read property values

### Reflection and Blueprints

71. ✅ **UCLASS/USTRUCT/UENUM must be in global namespace** (not inside namespace)
72. ✅ **Never use EditAnywhere for constructor-created components** (causes corruption)
73. ✅ **Use consistent category naming** with pipe delimiters
74. ✅ **Blueprint-exposed enums require uint8 underlying type**

### Inclusive Language

75. ✅ **Use inclusive and respectful language** everywhere
76. ✅ **Avoid metaphors that reinforce stereotypes**
77. ✅ **Use gender-neutral pronouns** (they/them/their)
78. ✅ **Replace legacy terms:**
    - blacklist → deny list, block list
    - whitelist → allow list, trust list  
    - master → primary, main, source
    - slave → secondary, replica, worker

### API Design and Third Party Code

79. ✅ **Avoid mixing UE and STL idioms in same API**
80. ✅ **Prefer superior results, but value consistency**
81. ✅ **Pass TSharedPtr/TSharedRef by reference, not by value**
82. ✅ **Maintain original formatting in third-party code**

### Logging

83. ✅ **Define proper log categories** (LogYourModule, not LogTemp)
84. ✅ **Don't leave LogTemp in production code**
85. ✅ **Use appropriate verbosity levels** (Fatal, Error, Warning, Display, Log, Verbose, VeryVerbose)
86. ✅ **Don't spam logs** - use Verbose/VeryVerbose for frequent messages

---

## Common Patterns and Anti-Patterns

### ✅ Good Patterns

```cpp
// Clear, self-documenting variable names
int32 TotalHealth = BaseHealth + BonusHealth;

// Const correctness
void ProcessData(const FString& Data) const;

// Proper pointer type usage
UPROPERTY()
TObjectPtr<UMyComponent> Component;

void UseComponent(UMyComponent* Comp);  // Function param is raw pointer

// Enum flags instead of multiple bools
enum class EOptions { None = 0, Fast = 1, Safe = 2, Verbose = 4 };
void Configure(EOptions Options);

// Range-based for with const reference
for (const FString& Name : Names)
{
    ProcessName(Name);
}

// Component creation
MyComponent = CreateDefaultSubobject<UMyComponent>(TEXT("MyComponent"));

// Weak pointer in lambda
TWeakObjectPtr<UMyObject> WeakObj = MyObject;
Delegate.BindLambda([WeakObj]() 
{
    if (UMyObject* Obj = WeakObj.Get())
    {
        Obj->DoWork();
    }
});
```

### ❌ Anti-Patterns to Avoid

```cpp
// Bad: Unclear, cryptic variable names
int32 t = b + h;

// Bad: Missing const
void ProcessData(FString Data);  // Unnecessary copy

// Bad: Multiple boolean parameters
void Setup(bool bFast, bool bSafe, bool bVerbose);  // Confusing
Setup(true, false, true);  // What does this mean?

// Bad: TObjectPtr as function parameter
void ProcessComponent(TObjectPtr<UMyComponent> Comp);  // Use raw pointer

// Bad: Not checking weak pointer
TWeakObjectPtr<AActor> WeakActor;
WeakActor->DoSomething();  // Dangerous! Might be null

// Bad: Bare auto hiding important type info
auto Data = GetData();  // What type is this?

// Bad: Not using braces for single-line blocks
if (bCondition)
    DoSomething();  // Dangerous for maintenance

// Bad: Spaces in component name
CreateDefaultSubobject<UMyComponent>(TEXT("My Component"));  // Causes bugs!

// Bad: Using EditAnywhere for constructor component
UPROPERTY(EditAnywhere)  // Wrong! Use VisibleAnywhere
TObjectPtr<UMyComponent> ConstructorComponent;

// Bad: Using standard containers for UE types
std::vector<UObject*> Objects;  // Use TArray instead

// Bad: NULL instead of nullptr
UObject* Obj = NULL;  // Use nullptr
```

---

## Additional Resources

- **Official Unreal Engine Documentation**: [dev.epicgames.com/documentation](https://dev.epicgames.com/documentation/en-us/unreal-engine/)
- **Epic C++ Coding Standard (Official)**: [Epic Developer Community - Coding Standard](https://dev.epicgames.com/documentation/en-us/unreal-engine/epic-cplusplus-coding-standard-for-unreal-engine)
- **Unreal Engine API Reference**: For implementation examples in the engine source code
- **Smart Pointers Documentation**: [Smart Pointers in Unreal Engine](https://dev.epicgames.com/documentation/en-us/unreal-engine/smart-pointers-in-unreal-engine)
- **Object Pointers Documentation**: [Object Pointers in Unreal Engine](https://dev.epicgames.com/documentation/en-us/unreal-engine/object-pointers-in-unreal-engine)

---

## Quick Reference Card

### Class Prefixes

```cpp
UMyClass      // UObject-derived
AMyActor      // AActor-derived  
SMyWidget     // SWidget-derived
FMyStruct     // Regular struct or class
TMyTemplate   // Template class
EMyEnum       // Enum
IMyInterface  // Interface
```

### Boolean Variables

```cpp
bool bIsAlive;
bool bHasAuthority;
bool bCanJump;
```

### Integer Types

```cpp
int32    // 32-bit signed
uint32   // 32-bit unsigned
int64    // 64-bit signed
uint64   // 64-bit unsigned
int16    // 16-bit signed
uint16   // 16-bit unsigned
int8     // 8-bit signed
uint8    // 8-bit unsigned
```

### Pointer Types Quick Reference

```cpp
// In UObject classes:
UPROPERTY()
TObjectPtr<UMyComponent> Component;              // UPROPERTY member

void UseIt(UMyComponent* Comp);                  // Function parameter
UMyComponent* GetIt() { return Component; }      // Return value

TWeakObjectPtr<AActor> OptionalActor;            // Non-owning reference
TSoftObjectPtr<UTexture> LazyTexture;            // Soft asset reference

// In non-UObject classes:
TSharedPtr<FMyData> SharedData;                  // Shared ownership
TUniquePtr<FMyData> UniqueData;                  // Unique ownership  
TStrongObjectPtr<UMyObject> OwnedObject;         // Own a UObject
```

### Common Macros

```cpp
UPROPERTY()                    // Property reflection
UFUNCTION()                    // Function reflection
UCLASS()                       // Class reflection
USTRUCT()                      // Struct reflection
UENUM()                        // Enum reflection
GENERATED_BODY()               // Required in classes/structs
ENUM_CLASS_FLAGS(EMyFlags)     // Enable bitwise operators on enum

UE_LOG(LogTemp, Warning, TEXT("Message"));
check(Condition);              // Assert in all builds
ensure(Condition);             // Assert but continue in shipping
checkSlow(Condition);          // Assert only in debug/development
```

### Container Types

```cpp
TArray<FString> StringArray;
TMap<FName, int32> NameToIntMap;
TSet<AActor*> ActorSet;
```

### Casting

```cpp
// UObject casting
UMyClass* Obj = Cast<UMyClass>(SomeObject);
UMyClass* Obj = CastChecked<UMyClass>(SomeObject);  // Asserts if fails

// Shared pointer casting  
TSharedPtr<FDerived> Derived = StaticCastSharedPtr<FDerived>(Base);
```

### Delegates

```cpp
// Single-cast
DECLARE_DELEGATE(FMyDelegate);
DECLARE_DELEGATE_OneParam(FMyDelegateOneParam, int32);

// Multi-cast
DECLARE_MULTICAST_DELEGATE(FMyMulticastDelegate);
DECLARE_DYNAMIC_MULTICAST_DELEGATE(FMyDynamicDelegate);
```

---

*This document is compiled from Epic Games' official Unreal Engine coding standards documentation. Always refer to the official Epic Games documentation for the most up-to-date standards.*

*Note: Epic is actively working to bring their code in line with the principles laid out in their standards, and conventions may evolve between engine versions.*

**Document Version:** Compiled for Unreal Engine 5.x  
**Last Updated:** December 2024  
**Source:** Epic Games Official Documentation
