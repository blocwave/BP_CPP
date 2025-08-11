# K2Node_DynamicCast Analysis - Special Node Type

## Overview
K2Node_DynamicCast implements runtime type casting operations with comprehensive safety checks and multiple execution modes. This node represents one of the most complex special node types due to its dual nature (pure/impure) and sophisticated type validation.

## Key Properties

### Core Configuration
- **TargetType**: `TSubclassOf<UObject>` - The target class to cast to
- **bIsPureCast**: Boolean controlling execution flow behavior

### Pin Structure
```cpp
// Input Pins
- Execute (conditional - only for impure casts)
- Object to Cast (wildcard, adapts to connected type)

// Output Pins  
- Cast Succeeded (conditional - only for impure casts)
- Cast Failed (conditional - only for impure casts)
- Casted Value (typed to TargetType)
- bSuccess (boolean, hidden for impure casts)
```

## Unique Expansion Patterns

### 1. Pure vs Impure Mode Switching
```cpp
void SetPurity(bool bNewPurity)
{
    if (bNewPurity != bIsPureCast)
    {
        bIsPureCast = bNewPurity;
        if (Pins.Num() > 0)
        {
            ReconstructNode(); // Complete pin restructure
        }
    }
}
```

**Key Insight**: The node can dynamically switch between pure and impure modes, requiring complete reconstruction and pin rewiring.

### 2. Adaptive Input Pin Typing
```cpp
void NotifyPinConnectionListChanged(UEdGraphPin* Pin)
{
    if (Pin == GetCastSourcePin())
    {
        if (Pin->LinkedTo.Num() == 0)
        {
            InputPinType.PinCategory = UEdGraphSchema_K2::PC_Wildcard;
        }
        else
        {
            const FEdGraphPinType& ConnectedPinType = Pin->LinkedTo[0]->PinType;
            if (ConnectedPinType.PinCategory == UEdGraphSchema_K2::PC_Interface)
            {
                Pin->PinFriendlyName = LOCTEXT("InterfaceInputName", "Interface");
                InputPinType.PinCategory = UEdGraphSchema_K2::PC_Interface;
            }
            else if (ConnectedPinType.PinCategory == UEdGraphSchema_K2::PC_Object)
            {
                InputPinType.PinCategory = UEdGraphSchema_K2::PC_Object;
            }
        }
    }
}
```

## C++ Code Generation Requirements

### 1. Cast Safety Implementation
The handler generates multiple cast operation types:

```cpp
// From DynamicCastHandler.cpp
EKismetCompiledStatementType CastOpType = KCST_DynamicCast;
if (bIsInputInterface)
{
    if (bIsOutputInterface)
        CastOpType = KCST_CrossInterfaceCast;
    else
        CastOpType = KCST_CastInterfaceToObj;
}
else if (bIsOutputInterface)
    CastOpType = KCST_CastObjToInterface;
```

### 2. Generated C++ Pattern
```cpp
// For impure cast:
TargetType* CastedValue = Cast<TargetType>(SourceObject);
bool bSuccess = IsValid(CastedValue);
if (bSuccess)
{
    // Execute success path
}
else
{
    // Execute failure path
}

// For pure cast:
TargetType* CastedValue = Cast<TargetType>(SourceObject);
bool bSuccess = IsValid(CastedValue);
// No execution branching, result used directly
```

### 3. Interface Casting
```cpp
// Interface to Object
ITargetInterface* InterfacePtr = Cast<ITargetInterface>(SourceObject);

// Object to Interface  
TargetClass* ObjectPtr = Cast<TargetClass>(SourceInterface);

// Cross-Interface
ITargetInterface* TargetInterface = Cast<ITargetInterface>(SourceInterface);
```

## Special Properties Not In Base K2Node

### 1. Dynamic Purity System
- **SetPurity()**: Changes execution model at runtime
- **TogglePurity()**: User-accessible mode switching
- **ReconnectPureExecPins()**: Handles connection rewiring during conversion

### 2. Cast-Specific Validation
```cpp
void ValidateNodeDuringCompilation(FCompilerResultsLog& MessageLog) const
{
    // Validates cast relationships:
    // - Equal type warnings (unnecessary cast)
    // - Inheritance hierarchy validation  
    // - Impossible cast detection
    // - Interface compatibility checks
}
```

### 3. Blueprint Class Name Resolution
```cpp
FText GetNodeTitle(ENodeTitleType::Type TitleType) const
{
    FString TargetName;
    UBlueprint* CastToBP = UBlueprint::GetBlueprintFromClass(TargetType);
    if (CastToBP != NULL)
    {
        TargetName = CastToBP->GetName(); // Use BP name, not generated class name
    }
    else
    {
        TargetName = TargetType->GetName();
    }
    // Cache formatted title for performance
}
```

## Edge Cases and Validation Requirements

### 1. Container Cast Prevention
```cpp
bool IsConnectionDisallowed(const UEdGraphPin* MyPin, const UEdGraphPin* OtherPin, FString& OutReason) const
{
    if (OtherPinType.IsContainer())
    {
        bIsDisallowed = true;
        OutReason = LOCTEXT("CannotContainerCast", "You cannot cast containers of objects.").ToString();
    }
}
```

### 2. Obsolete Class Detection
```cpp
void AllocateDefaultPins()
{
    const bool bReferenceObsoleteClass = TargetType && TargetType->HasAnyClassFlags(CLASS_NewerVersionExists);
    if (bReferenceObsoleteClass)
    {
        Message_Error(FString::Printf(TEXT("Node '%s' references obsolete class '%s'"), 
                      *GetPathName(), *TargetType->GetPathName()));
    }
}
```

### 3. Graph Compatibility Checking
```cpp
const UEdGraphSchema_K2* K2Schema = Cast<UEdGraphSchema_K2>(GetSchema());
if (!K2Schema->DoesGraphSupportImpureFunctions(GetGraph()))
{
    bIsPureCast = true; // Force pure mode in function graphs
}
```

## Blueprint to C++ Conversion Considerations

### 1. Template vs Dynamic Cast Selection
- Use `Cast<T>()` for known types at compile time
- Use `dynamic_cast` for runtime type information
- Interface casts require special handling

### 2. Failed Cast Handling Strategies
```cpp
// Option 1: Null check (pure cast equivalent)
if (TargetType* CastedPtr = Cast<TargetType>(Source))
{
    // Use CastedPtr
}

// Option 2: Branched execution (impure cast equivalent)  
TargetType* CastedPtr = Cast<TargetType>(Source);
if (IsValid(CastedPtr))
{
    // Success path
}
else
{
    // Failure path
}
```

### 3. Performance Optimization
- Cache cast results when possible
- Consider `StaticCast<T>()` for verified inheritance
- Use `CastChecked<T>()` for debug builds with assertions

## Implementation Priority
**HIGH** - Dynamic cast is fundamental to Blueprint type safety and represents ~15% of all Blueprint operations in typical projects. The dual-mode nature and complex validation make this essential for complete Blueprint to C++ conversion.

## Dependencies
- FKCHandler_DynamicCast (compilation handler)
- UEdGraphSchema_K2 (type validation)
- Blueprint class resolution system
- Interface inheritance system