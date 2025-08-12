# EPropertyFlags System Analysis

## Overview
EPropertyFlags is a 64-bit bitfield enum that controls the behavior, visibility, and capabilities of UPROPERTY-declared variables in Unreal Engine. These flags determine how properties interact with the editor, Blueprint system, networking, serialization, and runtime behavior.

## Core Flag Categories

### Editor Visibility and Editing
| Flag | Value | Description | Blueprint Equivalent |
|------|-------|-------------|---------------------|
| CPF_Edit | 0x0000000000000001 | Property is user-settable in the editor | EditAnywhere |
| CPF_EditConst | 0x0000000000020000 | Property is uneditable in the editor | VisibleAnywhere |
| CPF_DisableEditOnTemplate | 0x0000000000000800 | Disable editing of this property on an archetype/sub-blueprint | - |
| CPF_DisableEditOnInstance | 0x0000000000010000 | Disable editing on an instance of this class | EditDefaultsOnly |

### Blueprint Integration
| Flag | Value | Description | Blueprint Equivalent |
|------|-------|-------------|---------------------|
| CPF_BlueprintVisible | 0x0000000000000004 | Property can be read by blueprint code | BlueprintReadOnly/BlueprintReadWrite |
| CPF_BlueprintReadOnly | 0x0000000000000010 | Property cannot be modified by blueprint code | BlueprintReadOnly |
| CPF_BlueprintCallable | 0x0000100000000000 | MC Delegates only. Property should be exposed for calling in blueprint code | BlueprintCallable |
| CPF_BlueprintAssignable | 0x0000000010000000 | MC Delegates only. Property should be exposed for assigning in blueprint code | BlueprintAssignable |
| CPF_BlueprintAuthorityOnly | 0x0000200000000000 | MC Delegates only. This delegate accepts (only in blueprint) only events with BlueprintAuthorityOnly | BlueprintAuthorityOnly |

### Function Parameters
| Flag | Value | Description | Usage Context |
|------|-------|-------------|---------------|
| CPF_Parm | 0x0000000000000080 | Function/When call parameter | Function parameters |
| CPF_OutParm | 0x0000000000000100 | Value is copied out after function call | Out parameters |
| CPF_ReturnParm | 0x0000000000000400 | Return value | Function return values |
| CPF_ConstParm | 0x0000000000000002 | This is a constant function parameter | Const parameters |
| CPF_RequiredParm | 0x0000000000008000 | Parameter must be linked explicitly in blueprint | Required parameters |
| CPF_ReferenceParm | 0x0000000008000000 | Value is passed by reference | Reference parameters |

### Networking and Replication
| Flag | Value | Description | Blueprint Equivalent |
|------|-------|-------------|---------------------|
| CPF_Net | 0x0000000000000020 | Property is relevant to network replication | Replicated |
| CPF_RepNotify | 0x0000000100000000 | Notify actors when a property is replicated | ReplicatedUsing |
| CPF_RepSkip | 0x0000000080000000 | Not replicated. For non replicated properties in replicated structs | NotReplicated |

### Serialization and Persistence
| Flag | Value | Description | Usage Context |
|------|-------|-------------|---------------|
| CPF_Transient | 0x0000000000002000 | Property is transient: shouldn't be saved or loaded, except for Blueprint CDOs | Transient |
| CPF_DuplicateTransient | 0x0000000000200000 | Property should always be reset to the default value during any type of duplication | DuplicateTransient |
| CPF_NonPIEDuplicateTransient | 0x0000800000000000 | Property should only be copied in PIE | NonPIEDuplicateTransient |
| CPF_Config | 0x0000000000004000 | Property should be loaded/saved as permanent profile | Config |
| CPF_GlobalConfig | 0x0000000000040000 | Load config from base class, not subclass | GlobalConfig |
| CPF_SaveGame | 0x0000000001000000 | Property should be serialized for save games | SaveGame |
| CPF_SkipSerialization | 0x0080000000000000 | Property shouldn't be serialized, can still be exported to text | SkipSerialization |

### Memory and Construction
| Flag | Value | Description | Automatic Detection |
|------|-------|-------------|---------------------|
| CPF_ZeroConstructor | 0x0000000000000200 | memset is fine for construction | Detected by UHT |
| CPF_NoDestructor | 0x0001000000000000 | No destructor | Detected by UHT |
| CPF_IsPlainOldData | 0x0000000040000000 | If this is set, then the property can be memcopied instead of CopyCompleteValue | Detected by UHT |
| CPF_HasGetValueTypeHash | 0x0008000000000000 | This property can generate a meaningful hash value | Detected by UHT |

### Object References and Instancing
| Flag | Value | Description | Usage Context |
|------|-------|-------------|---------------|
| CPF_ExportObject | 0x0000000000000008 | Object can be exported with actor | Object properties |
| CPF_InstancedReference | 0x0000000000080000 | Property is a component references | Component properties |
| CPF_ContainsInstancedReference | 0x0000008000000000 | Property contains component references | Container properties |
| CPF_PersistentInstance | 0x0002000000000000 | A object referenced by the property is duplicated like a component | Instanced objects |
| CPF_AutoWeak | 0x0000004000000000 | Only used for weak pointers, means the export type is autoweak | Weak references |
| CPF_UObjectWrapper | 0x0004000000000000 | Property was parsed as a wrapper class like TSubclassOf<T> | Template wrappers |

### Editor Display and Categorization
| Flag | Value | Description | Blueprint Equivalent |
|------|-------|-------------|---------------------|
| CPF_SimpleDisplay | 0x0000020000000000 | The property is visible by default in the editor details view | Simple display |
| CPF_AdvancedDisplay | 0x0000040000000000 | The property is advanced and not visible by default in the editor details view | AdvancedDisplay |
| CPF_Protected | 0x0000080000000000 | property is protected from the perspective of script | Protected access |
| CPF_EditFixedSize | 0x0000000000000040 | Indicates that elements of an array can be modified, but its size cannot be changed | EditFixedSize |
| CPF_NoClear | 0x0000000002000000 | Hide clear (and browse) button | NoClear |

### Special Behaviors
| Flag | Value | Description | Usage Context |
|------|-------|-------------|---------------|
| CPF_NonNullable | 0x0000000000001000 | Object property can never be null | NonNullable |
| CPF_Deprecated | 0x0000000020000000 | Property is deprecated. Read it from an archive, but don't save it | Deprecated |
| CPF_Interp | 0x0000000200000000 | interpolatable property for use with cinematics | Interp |
| CPF_NonTransactional | 0x0000000400000000 | Property isn't transacted | NonTransactional |
| CPF_EditorOnly | 0x0000000800000000 | Property should only be loaded in the editor | EditorOnly |
| CPF_TextExportTransient | 0x0000400000000000 | Property shouldn't be exported to text format (e.g. copy/paste) | TextExportTransient |
| CPF_ExposeOnSpawn | 0x0001000000000000 | Property is exposed on spawn | ExposeOnSpawn |
| CPF_AssetRegistrySearchable | 0x0000010000000000 | asset instances will add properties with this flag to the asset registry automatically | AssetRegistrySearchable |

### Native Access Specifiers
| Flag | Value | Description | C++ Context |
|------|-------|-------------|-------------|
| CPF_NativeAccessSpecifierPublic | 0x0010000000000000 | Public native access specifier | public: |
| CPF_NativeAccessSpecifierProtected | 0x0020000000000000 | Protected native access specifier | protected: |
| CPF_NativeAccessSpecifierPrivate | 0x0040000000000000 | Private native access specifier | private: |

## Flag Combinations and Macros

### Common Flag Groups
```cpp
// All Native Access Specifier flags
#define CPF_NativeAccessSpecifiers (CPF_NativeAccessSpecifierPublic | CPF_NativeAccessSpecifierProtected | CPF_NativeAccessSpecifierPrivate)

// All parameter flags  
#define CPF_ParmFlags (CPF_Parm | CPF_OutParm | CPF_ReturnParm | CPF_RequiredParm | CPF_ReferenceParm | CPF_ConstParm)

// Flags that are propagated to properties inside containers
#define CPF_PropagateToArrayInner (CPF_ExportObject | CPF_PersistentInstance | CPF_InstancedReference | CPF_ContainsInstancedReference | CPF_Config | CPF_EditConst | CPF_Deprecated | CPF_EditorOnly | CPF_AutoWeak | CPF_UObjectWrapper)

// Properties that should never be set on interface properties
#define CPF_InterfaceClearMask (CPF_ExportObject|CPF_InstancedReference|CPF_ContainsInstancedReference)

// Properties that can be stripped for final release console builds
#define CPF_DevelopmentAssets (CPF_EditorOnly)

// Properties that should never be loaded or saved (computed at runtime)
#define CPF_ComputedFlags (CPF_IsPlainOldData | CPF_NoDestructor | CPF_ZeroConstructor | CPF_HasGetValueTypeHash)

// Mask of all property flags
#define CPF_AllFlags ((EPropertyFlags)0xFFFFFFFFFFFFFFFF)
```

## Blueprint Meta Flag Mapping

### UPROPERTY Specifier to Flag Mapping
```cpp
// Blueprint specifier -> Property flags mapping
EditAnywhere         -> CPF_Edit
EditDefaultsOnly     -> CPF_Edit | CPF_DisableEditOnInstance  
EditInstanceOnly     -> CPF_Edit | CPF_DisableEditOnTemplate
VisibleAnywhere      -> CPF_EditConst
VisibleDefaultsOnly  -> CPF_EditConst | CPF_DisableEditOnInstance
VisibleInstanceOnly  -> CPF_EditConst | CPF_DisableEditOnTemplate

BlueprintReadOnly    -> CPF_BlueprintVisible | CPF_BlueprintReadOnly
BlueprintReadWrite   -> CPF_BlueprintVisible

Replicated           -> CPF_Net
ReplicatedUsing      -> CPF_Net | CPF_RepNotify
NotReplicated        -> CPF_RepSkip

Transient            -> CPF_Transient
SaveGame             -> CPF_SaveGame
Config               -> CPF_Config
GlobalConfig         -> CPF_Config | CPF_GlobalConfig

Category="MyCategory"     -> (Metadata, not flags)
DisplayName="My Name"     -> (Metadata, not flags) 
ToolTip="Description"     -> (Metadata, not flags)
```

## Runtime Flag Checking

### Flag Validation Functions
```cpp
// Check if property has specific flags
bool FProperty::HasAnyPropertyFlags(EPropertyFlags FlagsToCheck) const
{
    return (PropertyFlags & FlagsToCheck) != 0;
}

bool FProperty::HasAllPropertyFlags(EPropertyFlags FlagsToCheck) const  
{
    return (PropertyFlags & FlagsToCheck) == FlagsToCheck;
}

// Common property checks
bool IsEditableInEditor() const 
{ 
    return HasAnyPropertyFlags(CPF_Edit) && !HasAnyPropertyFlags(CPF_EditConst);
}

bool IsVisibleInBlueprint() const
{
    return HasAnyPropertyFlags(CPF_BlueprintVisible);
}

bool IsWritableInBlueprint() const
{
    return HasAnyPropertyFlags(CPF_BlueprintVisible) && !HasAnyPropertyFlags(CPF_BlueprintReadOnly);
}

bool IsNetworked() const
{
    return HasAnyPropertyFlags(CPF_Net);
}
```

## Editor Integration

### Property Widget Creation
```cpp
// Editor determines widget type based on property flags
TSharedRef<SWidget> CreatePropertyWidget(const FProperty* Property)
{
    if (Property->HasAnyPropertyFlags(CPF_EditConst))
    {
        // Create read-only text widget
        return SNew(STextBlock).Text(GetPropertyValueAsText(Property));
    }
    else if (Property->HasAnyPropertyFlags(CPF_Edit))
    {
        // Create editable widget based on property type
        if (Property->IsA<FBoolProperty>())
            return SNew(SCheckBox);
        else if (Property->IsA<FFloatProperty>())
            return SNew(SNumericEntryBox<float>);
        // etc...
    }
}
```

### Detail Panel Visibility
```cpp
// Property visibility in details panel
EVisibility GetPropertyVisibility(const FProperty* Property) const
{
    // Advanced properties hidden by default
    if (Property->HasAnyPropertyFlags(CPF_AdvancedDisplay))
    {
        return bShowAdvancedProperties ? EVisibility::Visible : EVisibility::Collapsed;
    }
    
    // Editor-only properties hidden in game
    if (Property->HasAnyPropertyFlags(CPF_EditorOnly) && IsInGameWorld())
    {
        return EVisibility::Collapsed;
    }
    
    return EVisibility::Visible;
}
```

## Blueprint Compilation

### Pin Creation Logic
```cpp
// Blueprint compiler creates pins based on property flags
void CreateBlueprintPinFromProperty(const FProperty* Property, UK2Node* Node)
{
    if (!Property->HasAnyPropertyFlags(CPF_BlueprintVisible))
    {
        // Property not visible to Blueprint system
        return;
    }
    
    // Create appropriate pin type
    FEdGraphPinType PinType = GetBlueprintPinType(Property);
    
    // Set pin direction based on flags
    EPinDirection Direction = EGPD_Input;
    if (Property->HasAnyPropertyFlags(CPF_OutParm | CPF_ReturnParm))
    {
        Direction = EGPD_Output;
    }
    
    // Create pin with appropriate access level
    UEdGraphPin* Pin = Node->CreatePin(Direction, PinType, Property->GetFName());
    Pin->bHidden = Property->HasAnyPropertyFlags(CPF_BlueprintReadOnly) && Direction == EGPD_Input;
}
```

## Networking and Replication

### Replication Setup
```cpp
// Setup replication based on property flags
void SetupPropertyReplication(const FProperty* Property, FLifetimeProperty& LifetimeProperty)
{
    if (!Property->HasAnyPropertyFlags(CPF_Net))
    {
        // Property not replicated
        return;
    }
    
    // Configure replication conditions
    ELifetimeCondition Condition = COND_None;
    if (Property->HasMetaData(TEXT("ReplicatedUsing")))
    {
        LifetimeProperty.RepNotifyFunc = Property->GetMetaData(TEXT("ReplicatedUsing"));
    }
    
    // Add to replication list
    DOREPLIFETIME_CONDITION(OwnerClass, PropertyName, Condition);
}
```

## Serialization Control

### Save/Load Behavior
```cpp
// Determine if property should be serialized
bool ShouldSerializeProperty(const FProperty* Property, ESerializationContext Context) const
{
    // Transient properties not saved (except Blueprint CDOs)
    if (Property->HasAnyPropertyFlags(CPF_Transient))
    {
        return Context == SCTX_BlueprintCDO;
    }
    
    // Editor-only properties not saved in shipping builds
    if (Property->HasAnyPropertyFlags(CPF_EditorOnly))
    {
        return WITH_EDITOR;
    }
    
    // SaveGame properties only saved in save game context
    if (Property->HasAnyPropertyFlags(CPF_SaveGame))
    {
        return Context == SCTX_SaveGame;
    }
    
    // Config properties saved to config files
    if (Property->HasAnyPropertyFlags(CPF_Config))
    {
        return Context == SCTX_Config;
    }
    
    return true; // Default: serialize the property
}
```

## Memory Management

### Construction and Destruction
```cpp
// Property construction based on flags
void ConstructPropertyValue(const FProperty* Property, void* ValuePtr) const
{
    if (Property->HasAnyPropertyFlags(CPF_ZeroConstructor))
    {
        // Simple memset is sufficient
        FMemory::Memzero(ValuePtr, Property->GetSize());
    }
    else
    {
        // Call proper constructor
        Property->InitializeValue(ValuePtr);
    }
}

void DestructPropertyValue(const FProperty* Property, void* ValuePtr) const  
{
    if (!Property->HasAnyPropertyFlags(CPF_NoDestructor))
    {
        // Call proper destructor
        Property->DestroyValue(ValuePtr);
    }
    // Otherwise no destruction needed
}
```

## Performance Implications

### Runtime Checks
- Flag checks are bitwise operations (very fast)
- Flags cached at property construction time
- No dynamic allocation for flag storage
- Flags enable/disable expensive operations (replication, serialization)

### Memory Overhead
- 8 bytes per property (64-bit flags)
- Shared across all instances of a class
- No per-instance flag storage needed

## Best Practices

### Flag Selection Guidelines
1. **Use minimal required flags** - Each flag has performance implications
2. **Prefer Blueprint visibility** - Use BlueprintReadOnly instead of hiding properties
3. **Consider networking costs** - Only replicate properties that need it
4. **Group related flags** - EditAnywhere usually pairs with BlueprintReadWrite
5. **Test flag combinations** - Some combinations may conflict or be redundant

### Common Patterns
```cpp
// Standard editable property
UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Settings")
float MyFloat;
// -> CPF_Edit | CPF_BlueprintVisible

// Read-only display property  
UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = "Status")
int32 CurrentScore;
// -> CPF_EditConst | CPF_BlueprintVisible | CPF_BlueprintReadOnly

// Networked property with notification
UPROPERTY(ReplicatedUsing = OnHealthChanged, BlueprintReadOnly)
float Health;
// -> CPF_Net | CPF_RepNotify | CPF_BlueprintVisible | CPF_BlueprintReadOnly
```

## Summary

The EPropertyFlags system provides fine-grained control over property behavior across all Unreal Engine systems. Understanding these flags is crucial for:

1. **Proper Blueprint integration** - Controlling visibility and access
2. **Efficient networking** - Minimizing replication overhead  
3. **Editor workflow** - Creating intuitive property interfaces
4. **Performance optimization** - Avoiding unnecessary operations
5. **Cross-system compatibility** - Ensuring properties work correctly in all contexts

The flag system strikes a balance between flexibility and performance, enabling rich property behavior while maintaining runtime efficiency.