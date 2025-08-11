# FProperty Type System Analysis

## Overview
The FProperty system is the foundation of Unreal Engine's reflection and serialization system. It provides runtime type information for properties declared in UPROPERTY macros and enables Blueprint integration, networking, and serialization.

## Core Architecture

### Base Class Hierarchy

```cpp
FField (Base class for all reflection fields)
  └─ FProperty (Base property class)
      ├─ FNumericProperty (Base for all numeric types)
      │   ├─ FByteProperty (uint8)
      │   ├─ FInt8Property (int8) 
      │   ├─ FInt16Property (int16)
      │   ├─ FUInt16Property (uint16)
      │   ├─ FIntProperty (int32)
      │   ├─ FUInt32Property (uint32)
      │   ├─ FInt64Property (int64)
      │   ├─ FUInt64Property (uint64)
      │   ├─ FFloatProperty (float)
      │   └─ FDoubleProperty (double)
      ├─ FBoolProperty (bool types)
      ├─ FObjectPropertyBase (Object reference base)
      │   ├─ FObjectProperty (UObject* - strong reference)
      │   ├─ FWeakObjectProperty (TWeakObjectPtr)
      │   ├─ FLazyObjectProperty (TLazyObjectPtr)
      │   └─ FSoftObjectProperty (TSoftObjectPtr)
      ├─ FStructProperty (UStruct types)
      ├─ FArrayProperty (TArray containers)
      ├─ FMapProperty (TMap containers)
      ├─ FSetProperty (TSet containers)
      ├─ FStrProperty (FString)
      ├─ FNameProperty (FName)
      ├─ FTextProperty (FText)
      ├─ FEnumProperty (Enum types)
      ├─ FClassProperty (UClass references)
      ├─ FInterfaceProperty (Interface types)
      └─ FDelegateProperty (Delegate types)
```

## Core FProperty Structure

### Essential Members
```cpp
class FProperty : public FField
{
    // Core property metadata
    int32           ArrayDim;        // Array dimension (1 for single values)
    int32           ElementSize;     // Size of single element in bytes
    EPropertyFlags  PropertyFlags;   // Property behavior flags
    uint16          RepIndex;        // Replication index for networking
    
    // Offset within containing structure
    int32           Offset_Internal; // Memory offset within struct
    
    // Optional metadata for editor/Blueprint integration
    FString         Category;        // Editor category
    FString         DisplayName;     // Friendly display name
};
```

## Property Type Mapping

### Blueprint to C++ Type Mapping

| Blueprint Type | C++ Type | FProperty Class | Notes |
|---------------|----------|-----------------|-------|
| Boolean | bool | FBoolProperty | Bit-packed for arrays |
| Byte | uint8 | FByteProperty | 0-255 range |
| Integer | int32 | FIntProperty | Standard signed 32-bit |
| Integer64 | int64 | FInt64Property | 64-bit signed |
| Float | float | FFloatProperty | Single precision |
| String | FString | FStrProperty | Dynamic string |
| Name | FName | FNameProperty | Efficient string storage |
| Text | FText | FTextProperty | Localized text |
| Vector | FVector | FStructProperty | UStruct = Vector |
| Rotator | FRotator | FStructProperty | UStruct = Rotator |
| Transform | FTransform | FStructProperty | UStruct = Transform |
| Object Reference | UObject* | FObjectProperty | Strong reference |
| Soft Object Reference | TSoftObjectPtr | FSoftObjectProperty | Lazy loading |
| Class Reference | TSubclassOf | FClassProperty | Class pointer |
| Enum | Enum Type | FEnumProperty | Wrapped enum value |
| Array | TArray<T> | FArrayProperty | Dynamic array |
| Map | TMap<K,V> | FMapProperty | Key-value pairs |
| Set | TSet<T> | FSetProperty | Unique values |

## Template-Based Property Implementation

### TProperty Template System
```cpp
template<typename TCppType, class TPropertyBaseClass>
class TProperty : public TPropertyBaseClass
{
    // Type-safe property implementation
    typedef TCppType TCppType;
    
    // Core operations
    virtual void SetPropertyValue(void* Ptr, const TCppType& Value) const;
    virtual TCppType GetPropertyValue(const void* Ptr) const;
    virtual void CopyCompleteValue(void* Dest, const void* Src) const;
};

// Numeric property specialization
template<typename TCppType>
class TProperty_Numeric : public TProperty_WithEqualityAndSerializer<TCppType, FNumericProperty>
{
    // Numeric-specific operations like min/max clamping
    virtual void SetNumericPropertyValue(void* Ptr, const TCppType& Value) const;
};
```

## Property Creation and Registration

### Automatic Registration
Properties are automatically registered during class construction through:

1. **Static Class Construction**: FProperty instances are created in StaticClass() functions
2. **Metadata Parsing**: UHT (Unreal Header Tool) parses UPROPERTY macros
3. **Runtime Binding**: Properties are linked to their containing UStruct/UClass

### Example Property Creation
```cpp
// Generated code from UPROPERTY(EditAnywhere, BlueprintReadWrite)
// float Health;

FFloatProperty* HealthProperty = new FFloatProperty(
    GetClass(),              // Owner class
    TEXT("Health"),          // Property name  
    RF_Public | RF_Transient // Object flags
);
HealthProperty->SetPropertyFlags(CPF_Edit | CPF_BlueprintVisible | CPF_BlueprintReadWrite);
HealthProperty->SetOffset_Internal(STRUCT_OFFSET(AMyActor, Health));
HealthProperty->ElementSize = sizeof(float);
```

## Memory Layout and Access

### Property Access Patterns
```cpp
// Direct memory access via offset
float* HealthPtr = Property->ContainerPtrToValuePtr<float>(ObjectInstance);
float HealthValue = *HealthPtr;

// Type-safe access through property system
FFloatProperty* FloatProp = CastField<FFloatProperty>(Property);
float Value = FloatProp->GetPropertyValue(ObjectInstance);
FloatProp->SetPropertyValue(ObjectInstance, NewValue);

// Generic property access (used by editor/Blueprint system)
FString ValueString;
Property->ExportTextItem(ValueString, ObjectInstance, nullptr, nullptr, PPF_None);
Property->ImportText(*ValueString, ObjectInstance, PPF_None, nullptr);
```

## Property Inheritance and Polymorphism

### Virtual Function Interface
```cpp
class FProperty : public FField
{
public:
    // Serialization
    virtual void SerializeItem(FArchive& Ar, void* Value, void const* Defaults = nullptr) const;
    
    // Text import/export for editor
    virtual bool ExportTextItem(FString& ValueStr, const void* PropertyValue, const void* DefaultValue, UObject* Parent, int32 PortFlags, UObject* ExportRootScope = nullptr) const;
    virtual const TCHAR* ImportText(const TCHAR* Buffer, void* PropertyValue, int32 PortFlags, UObject* Parent, FOutputDevice* ErrorText = nullptr) const;
    
    // Property comparison
    virtual bool Identical(const void* A, const void* B, uint32 PortFlags = PPF_None) const;
    
    // Memory operations
    virtual void CopyCompleteValue(void* Dest, const void* Src) const;
    virtual void ClearValue(void* Data) const;
    virtual void DestroyValue(void* Dest) const;
    virtual void InitializeValue(void* Dest) const;
    
    // Size information
    virtual int32 GetMinAlignment() const;
    virtual int32 GetSize() const;
};
```

## Special Property Types

### Container Properties
```cpp
// Array Property - TArray<ElementType>
class FArrayProperty : public FProperty
{
    FProperty* Inner;  // Type of array elements
    EArrayPropertyFlags ArrayFlags;
    
    // Array-specific operations
    FScriptArrayHelper GetScriptArrayHelper(void* ArrayPtr) const;
};

// Map Property - TMap<KeyType, ValueType>  
class FMapProperty : public FProperty
{
    FProperty* KeyProp;    // Key type property
    FProperty* ValueProp;  // Value type property
    
    // Map-specific operations
    FScriptMapHelper GetScriptMapHelper(void* MapPtr) const;
};
```

### Object Reference Properties
```cpp
class FObjectPropertyBase : public FProperty
{
    UClass* PropertyClass;  // Required class for referenced objects
    
    // Object reference validation
    virtual bool CheckValidObject(UObject* Object) const;
    virtual UObject* GetObjectPropertyValue(const void* PropertyValuePtr) const;
    virtual void SetObjectPropertyValue(void* PropertyValuePtr, UObject* Value) const;
};
```

## Blueprint Integration

### Blueprint Type Conversion
The property system provides automatic conversion between C++ and Blueprint types:

```cpp
// Blueprint variable declaration maps to:
UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Stats")
float Health = 100.0f;

// Becomes:
FFloatProperty* Generated with flags:
- CPF_Edit (EditAnywhere)
- CPF_BlueprintVisible | CPF_BlueprintReadWrite (BlueprintReadWrite) 
- Metadata: Category="Stats"
```

### Pin Type Determination
```cpp
// Blueprint pin types derived from property types
EPinCategoryType GetBlueprintPinCategory(const FProperty* Property)
{
    if (Property->IsA<FBoolProperty>()) return PC_Boolean;
    if (Property->IsA<FIntProperty>()) return PC_Int;  
    if (Property->IsA<FFloatProperty>()) return PC_Real;
    if (Property->IsA<FStrProperty>()) return PC_String;
    if (Property->IsA<FObjectProperty>()) return PC_Object;
    // ... etc
}
```

## Reflection and Runtime Access

### Property Enumeration
```cpp
// Iterate all properties in a class
for (FProperty* Property : TFieldRange<FProperty>(MyClass))
{
    FString PropertyName = Property->GetName();
    FString PropertyType = Property->GetClass()->GetName();
    
    // Access property value on instance
    void* ValuePtr = Property->ContainerPtrToValuePtr<void>(Instance);
    
    // Export to string for display
    FString ValueString;
    Property->ExportTextItem(ValueString, ValuePtr, nullptr, nullptr, PPF_None);
}
```

## Performance Considerations

### Memory Efficiency
- Properties are allocated once per class, not per instance
- Offset-based access eliminates vtable overhead
- Template specialization provides type-safe operations
- Bit packing for boolean properties saves memory

### Runtime Performance
- Direct memory access via cached offsets
- Type casting validated at compile time where possible
- Virtual function calls only for polymorphic operations
- Minimal allocations during property access

## Summary

The FProperty system provides a comprehensive reflection mechanism that:

1. **Unifies** C++ and Blueprint type systems
2. **Enables** runtime introspection and modification
3. **Supports** automatic serialization and networking
4. **Facilitates** editor integration and visual scripting
5. **Maintains** type safety while providing generic interfaces

This system is fundamental to Unreal Engine's ability to provide seamless integration between C++ code, Blueprint visual scripting, and editor functionality.