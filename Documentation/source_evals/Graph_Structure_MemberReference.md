# Graph Structure - Member Reference System Analysis

## Overview
This document analyzes the FMemberReference structure, which is critical for Blueprint to C++ conversion as it handles references to variables, functions, and other members. This system enables the resolution of Blueprint member access to proper C++ equivalents.

## FMemberReference Core Structure

### Essential Member Reference Data
```cpp
struct FMemberReference
{
protected:
    // Primary reference data
    mutable TObjectPtr<UObject> MemberParent;   // Class or Package containing member
    mutable FString MemberScope;                // Scope string for local members
    mutable FName MemberName;                   // Name of the member
    mutable FGuid MemberGuid;                   // CRITICAL: Unique member identifier
    
    // Context flags
    mutable bool bSelfContext;                  // References 'self' (this class)
    mutable bool bWasDeprecated;                // Deprecated member flag

public:
    // Member identification queries
    FName GetMemberName() const { return MemberName; }
    FGuid GetMemberGuid() const { return MemberGuid; }
    UClass* GetMemberParentClass() const { return Cast<UClass>(MemberParent); }
    UPackage* GetMemberParentPackage() const;
    bool IsSelfContext() const { return bSelfContext; }
    bool IsLocalScope() const { return !MemberScope.IsEmpty(); }
};
```

### Member Reference Types and C++ Mapping

#### 1. Self-Context Members (bSelfContext = true)
```cpp
// Blueprint: Reference to member of current class
// C++ Equivalent: Direct member access
FMemberReference SelfMember;
SelfMember.SetSelfMember(TEXT("MyVariable"));

// C++ Generation:
Blueprint Reference     C++ Equivalent
MyVariable         ->   this->MyVariable
MyFunction()       ->   this->MyFunction()
```

#### 2. External Class Members (MemberParent = UClass*)
```cpp
// Blueprint: Reference to member of another class
// C++ Equivalent: Qualified member access or static access
FMemberReference ExternalMember;
ExternalMember.SetExternalMember(TEXT("StaticFunction"), UMyClass::StaticClass());

// C++ Generation:
Blueprint Reference              C++ Equivalent
OtherClass::StaticVar       ->   UOtherClass::StaticVar
OtherClass::StaticFunc()    ->   UOtherClass::StaticFunc()
```

#### 3. Global/Package Members (MemberParent = UPackage*)
```cpp
// Blueprint: Reference to global delegate or native function
// C++ Equivalent: Global scope access
FMemberReference GlobalMember;
GlobalMember.SetGlobalField(TEXT("GlobalDelegate"), GetPackage());

// C++ Generation:
Blueprint Reference          C++ Equivalent
GlobalDelegate          ->   GlobalDelegate
NativeDelegateSignature ->   FNativeDelegateSignature
```

#### 4. Local Scope Members (MemberScope set)
```cpp
// Blueprint: Reference to local variables or parameters
// C++ Equivalent: Local variable access
FMemberReference LocalMember;
LocalMember.SetLocalMember(TEXT("LocalVar"), LocalStruct, MemberGuid);

// C++ Generation:
Blueprint Reference     C++ Equivalent
LocalVar           ->   LocalVar
FunctionParam      ->   FunctionParam
```

## Member Reference Setup Methods

### Self-Context References
```cpp
// Set reference to current class member
void SetSelfMember(FName InMemberName);
void SetSelfMember(FName InMemberName, FGuid& InMemberGuid);

// Usage in C++ generation:
// Blueprint variable "Health" becomes "this->Health"
// Blueprint function "TakeDamage" becomes "this->TakeDamage"
```

### External References
```cpp
// Set reference to external class member
void SetExternalMember(FName InMemberName, TSubclassOf<UObject> InMemberParentClass);
void SetExternalMember(FName InMemberName, TSubclassOf<UObject> InMemberParentClass, FGuid& InMemberGuid);

// Usage in C++ generation:
// Static function references
// Enum value references
// Global class member access
```

### Global Field References
```cpp
// Set reference to global scope member
void SetGlobalField(FName InFieldName, UPackage* InParentPackage);

// Usage in C++ generation:
// Global delegate types
// Native function signatures
// Package-level declarations
```

### Local Scope References
```cpp
// Set reference to local member within specific scope
void SetLocalMember(FName InMemberName, UStruct* InScope, const FGuid InMemberGuid);
void SetLocalMember(FName InMemberName, FString InScopeName, const FGuid InMemberGuid);

// Usage in C++ generation:
// Function parameters
// Local variables
// Struct member access within local scope
```

## FSimpleMemberReference - Lightweight Reference

### Simplified Structure for Pins
```cpp
struct FSimpleMemberReference
{
    TObjectPtr<UObject> MemberParent;    // Class or Package containing member
    FName MemberName;                    // Member name
    FGuid MemberGuid;                    // Unique identifier
    
    // Utility methods
    UClass* GetMemberParentClass() const { return Cast<UClass>(MemberParent); }
    
    bool operator==(const FSimpleMemberReference& Other) const {
        return (MemberParent == Other.MemberParent) &&
               (MemberName == Other.MemberName) &&
               (MemberGuid == Other.MemberGuid);
    }
};
```

### Usage in Pin Type System
- Used in FEdGraphPinType::PinSubCategoryMemberReference
- Enables pins to reference specific Blueprint variables/functions
- Critical for generating proper C++ member access in pin connections

## Member Resolution Process for C++ Generation

### 1. Member Type Classification
```cpp
enum class EMemberReferenceType
{
    SelfMember,        // this->MemberName
    ExternalStatic,    // ClassName::MemberName
    GlobalField,       // GlobalMemberName
    LocalVariable,     // LocalMemberName
    Parameter,         // ParameterName
    StructMember      // StructInstance.MemberName
};
```

### 2. C++ Code Generation Strategy
```cpp
struct MemberReferenceCodeGen
{
    FString GetCppAccessCode() const
    {
        if (bSelfContext) {
            return FString::Printf(TEXT("this->%s"), *MemberName.ToString());
        }
        else if (UClass* ParentClass = GetMemberParentClass()) {
            return FString::Printf(TEXT("%s::%s"), *ParentClass->GetName(), *MemberName.ToString());
        }
        else if (IsLocalScope()) {
            return MemberName.ToString(); // Direct local access
        }
        else {
            return MemberName.ToString(); // Global access
        }
    }
    
    TArray<FString> GetRequiredIncludes() const
    {
        TArray<FString> Includes;
        if (UClass* ParentClass = GetMemberParentClass()) {
            Includes.Add(FString::Printf(TEXT("#include \"%s.h\""), *ParentClass->GetName()));
        }
        return Includes;
    }
};
```

### 3. Member Validation and Resolution
```cpp
// Validate member reference during C++ generation
bool ValidateMemberReference() const
{
    // Check if member still exists in target class
    if (UClass* ParentClass = GetMemberParentClass()) {
        // Find property by GUID first (most reliable)
        if (MemberGuid.IsValid()) {
            return UBlueprint::GetFieldNameFromClassByGuid(ParentClass, MemberGuid) != NAME_None;
        }
        // Fallback to name-based lookup
        return ParentClass->FindPropertyByName(MemberName) != nullptr ||
               ParentClass->FindFunctionByName(MemberName) != nullptr;
    }
    return false;
}
```

## Critical Member Reference Scenarios

### Variable Access
```cpp
// Blueprint Variable Access -> C++ Member Access
FMemberReference VariableRef;
VariableRef.SetSelfMember(TEXT("Health"));

// Generated C++:
float GetHealth() const { return this->Health; }
void SetHealth(float NewHealth) { this->Health = NewHealth; }
```

### Function Calls
```cpp
// Blueprint Function Call -> C++ Function Call
FMemberReference FunctionRef;
FunctionRef.SetSelfMember(TEXT("TakeDamage"));

// Generated C++:
this->TakeDamage(DamageAmount);
```

### Static Member Access
```cpp
// Blueprint Static Access -> C++ Static Access
FMemberReference StaticRef;
StaticRef.SetExternalMember(TEXT("GetStaticValue"), UGameplayStatics::StaticClass());

// Generated C++:
UGameplayStatics::GetStaticValue();
```

### Enum Value References
```cpp
// Blueprint Enum Access -> C++ Enum Access
FMemberReference EnumRef;
EnumRef.SetExternalMember(TEXT("Red"), UMyEnum::StaticClass());

// Generated C++:
EMyEnum::Red
```

## Member Reference Serialization for BP to C++ Conversion

### Serialization Data Structure
```cpp
struct MemberReferenceSerializationData
{
    FString MemberParentClassName;   // Name of parent class
    FString MemberScope;            // Scope string
    FName MemberName;               // Member name
    FGuid MemberGuid;               // Unique identifier
    bool bSelfContext;              // Self-context flag
    bool bWasDeprecated;            // Deprecation flag
    
    // Additional resolution data
    FString MemberType;             // Variable, Function, Enum, etc.
    FString CppTypeName;            // Resolved C++ type
    TArray<FString> RequiredIncludes; // Headers needed for C++ compilation
};
```

## BP to C++ Conversion Requirements

### Essential Member Reference Data
1. **MemberGuid** - Most reliable identifier for member resolution
2. **MemberName** - Fallback identifier and C++ symbol name
3. **MemberParent** - Class/package context for qualified access
4. **bSelfContext** - Determines this-> vs qualified access
5. **MemberScope** - Local scope information for parameters/locals

### Member Resolution Pipeline
```cpp
class MemberReferenceResolver
{
public:
    // Resolve member reference to C++ equivalent
    struct ResolvedMemberInfo {
        FString CppAccessCode;          // C++ code for member access
        FString CppTypeName;            // Full C++ type
        TArray<FString> RequiredIncludes; // Headers to include
        bool bIsValid;                  // Resolution successful
        bool bIsDeprecated;             // Member is deprecated
    };
    
    ResolvedMemberInfo ResolveMemberReference(const FMemberReference& MemberRef);
};
```

### Member Reference Usage Patterns

#### Variable Get/Set Operations
```cpp
// Blueprint Get Variable -> C++ getter
Blueprint: Get MyVariable
C++: this->GetMyVariable() or direct access this->MyVariable

// Blueprint Set Variable -> C++ setter  
Blueprint: Set MyVariable
C++: this->SetMyVariable(NewValue) or direct this->MyVariable = NewValue
```

#### Function Call Operations
```cpp
// Blueprint Function Call -> C++ function call
Blueprint: Call MyFunction
C++: this->MyFunction(Parameters...)

// Blueprint Static Function -> C++ static call
Blueprint: Call StaticFunction
C++: UClassName::StaticFunction(Parameters...)
```

## Implementation Notes

### Performance Considerations
- GUID-based resolution is more reliable but potentially slower
- Name-based fallback resolution for missing GUIDs
- Cache resolved member information to avoid repeated lookups

### Thread Safety
- Member resolution should occur during preprocessing phase
- Resolved information should be cached for code generation phase
- Thread-safe access to cached resolution data

### Error Handling
- Deprecated member warnings should be preserved in generated C++
- Missing member references should generate compilation errors
- Provide clear error messages with Blueprint context

## Related Components
- See `Graph_Structure_Pin.md` for pin member references
- See `Graph_Structure_K2Node.md` for node-specific member usage
- See `K2Node_Variable.md` for variable node member references
- See `K2Node_CallFunction.md` for function call member references

## Summary

The FMemberReference system is fundamental to Blueprint to C++ conversion, providing the mechanism to resolve Blueprint member access to proper C++ code. Critical aspects include:

1. **Context determination** (self vs external vs global vs local)
2. **Member identification** through GUID and name
3. **C++ code generation** with proper qualification and access patterns
4. **Type resolution** for proper C++ type declarations
5. **Include management** for necessary header dependencies

The member reference system directly translates Blueprint member access patterns into equivalent C++ syntax, making it essential for generating properly structured and compilable C++ code from Blueprint graphs.