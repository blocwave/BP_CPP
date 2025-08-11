# Default Values for Complex Types Analysis for Blueprint to C++ Conversion

## Overview

The Unreal Engine system for handling default values in complex types (structs, text, objects) is crucial for Blueprint to C++ conversion. This analysis examines how default values are resolved, stored, and applied across different property types, focusing on the infrastructure needed to maintain default value semantics in converted code.

## Core Default Value Infrastructure

### 1. FProperty Default Value System

**Property Default Value Storage:**
- Default values stored in class's Default Subobject (CDO)
- Complex types require special handling for construction/initialization
- Blueprint defaults can override class defaults

### 2. FTextProperty Default Values

**File:** `/Engine/Source/Runtime/CoreUObject/Public/UObject/TextProperty.h`

**FTextProperty Structure:**
```cpp
class COREUOBJECT_API FTextProperty : public FTextProperty_Super
{
public:
    typedef FTextProperty_Super::TTypeFundamentals TTypeFundamentals;
    typedef TTypeFundamentals::TCppType TCppType; // FText

    // Constructor for compiled properties
    FTextProperty(FFieldVariant InOwner, const UECodeGen_Private::FTextPropertyParams& Prop);
};
```

**FText Default Value Characteristics:**
- Supports localization from creation
- Can contain namespace/key information for localization
- May include source string for fallback
- Requires special serialization handling

**FText Default Value Resolution:**
```cpp
// FText default value resolution pattern
void ResolveFTextDefault(FTextProperty* Property, void* ContainerPtr, const FText& DefaultValue)
{
    FText* TextPtr = Property->ContainerPtrToValuePtr<FText>(ContainerPtr);
    
    if (DefaultValue.IsEmpty())
    {
        *TextPtr = FText::GetEmpty();
    }
    else if (DefaultValue.IsFromStringTable())
    {
        // Preserve string table reference
        *TextPtr = DefaultValue;
    }
    else
    {
        // Create new FText from source string
        *TextPtr = FText::FromString(DefaultValue.ToString());
    }
}
```

### 3. Struct Default Value System

**File:** `/Engine/Source/Editor/UnrealEd/Public/Kismet2/StructureEditorUtils.h`

**Structure Editor Utilities:**
```cpp
class UNREALED_API FStructureEditorUtils
{
    enum EStructureEditorChangeInfo
    {
        Unknown,
        AddedVariable,
        RemovedVariable,
        RenamedVariable,
        VariableTypeChanged,
        MovedVariable,
        DefaultValueChanged, // Key enum for default value changes
    };

    /** Change variable default value */
    static bool ChangeVariableDefaultValue(UUserDefinedStruct* Struct, FGuid VarGuid, const FString& NewDefaultValue);
    
    /** Get variable default value */
    static FString GetVariableDefaultValue(const UUserDefinedStruct* Struct, FGuid VarGuid);
};
```

**Struct Default Value Storage Pattern:**
```cpp
struct FStructVariableDescription
{
    /** Name of the variable */
    FName VarName;
    
    /** Type of the variable */
    FEdGraphPinType VarType;
    
    /** Default value as string */
    FString DefaultValue;
    
    /** Unique identifier */
    FGuid VarGuid;
    
    /** Metadata for the variable */
    TMap<FString, FString> MetaData;
};
```

## Complex Type Default Value Resolution

### 1. User-Defined Struct Defaults

**Default Value Application Process:**
```cpp
void ApplyStructDefaults(UUserDefinedStruct* Struct, void* StructPtr)
{
    for (TFieldIterator<FProperty> It(Struct); It; ++It)
    {
        FProperty* Property = *It;
        
        // Get default value from struct definition
        FString DefaultValueString = GetPropertyDefaultValue(Property);
        
        if (!DefaultValueString.IsEmpty())
        {
            // Apply default based on property type
            ApplyPropertyDefault(Property, StructPtr, DefaultValueString);
        }
        else
        {
            // Initialize with property's intrinsic default
            Property->InitializeValue_InContainer(StructPtr);
        }
    }
}
```

**Property-Specific Default Application:**
```cpp
void ApplyPropertyDefault(FProperty* Property, void* ContainerPtr, const FString& DefaultValueString)
{
    if (FTextProperty* TextProp = CastField<FTextProperty>(Property))
    {
        ApplyTextPropertyDefault(TextProp, ContainerPtr, DefaultValueString);
    }
    else if (FStructProperty* StructProp = CastField<FStructProperty>(Property))
    {
        ApplyStructPropertyDefault(StructProp, ContainerPtr, DefaultValueString);
    }
    else if (FObjectProperty* ObjectProp = CastField<FObjectProperty>(Property))
    {
        ApplyObjectPropertyDefault(ObjectProp, ContainerPtr, DefaultValueString);
    }
    else
    {
        // Use property's ImportText for simple types
        Property->ImportText(*DefaultValueString, Property->ContainerPtrToValuePtr<void>(ContainerPtr), 0, nullptr);
    }
}
```

### 2. FText Default Value Resolution

**FText Localization Support:**
```cpp
void ApplyTextPropertyDefault(FTextProperty* Property, void* ContainerPtr, const FString& DefaultValueString)
{
    FText* TextPtr = Property->ContainerPtrToValuePtr<FText>(ContainerPtr);
    
    // Parse potential localization information
    FText ParsedText;
    if (ParseTextFromString(DefaultValueString, ParsedText))
    {
        *TextPtr = ParsedText;
    }
    else
    {
        // Fallback to simple string conversion
        *TextPtr = FText::FromString(DefaultValueString);
    }
}

bool ParseTextFromString(const FString& TextString, FText& OutText)
{
    // Check for localization format: NSLOCTEXT("Namespace", "Key", "Source")
    if (TextString.StartsWith(TEXT("NSLOCTEXT(")))
    {
        return ParseNSLOCTEXT(TextString, OutText);
    }
    
    // Check for string table format: LOCTABLE("TableId", "Key")
    if (TextString.StartsWith(TEXT("LOCTABLE(")))
    {
        return ParseLOCTABLE(TextString, OutText);
    }
    
    // Simple string format
    OutText = FText::FromString(TextString);
    return true;
}
```

### 3. Object Reference Defaults

**Object Property Default Resolution:**
```cpp
void ApplyObjectPropertyDefault(FObjectProperty* Property, void* ContainerPtr, const FString& DefaultValueString)
{
    UObject** ObjectPtr = Property->ContainerPtrToValuePtr<UObject*>(ContainerPtr);
    
    if (DefaultValueString.IsEmpty() || DefaultValueString == TEXT("None"))
    {
        *ObjectPtr = nullptr;
    }
    else
    {
        // Resolve object reference
        UObject* DefaultObject = ResolveObjectReference(DefaultValueString, Property->PropertyClass);
        *ObjectPtr = DefaultObject;
    }
}

UObject* ResolveObjectReference(const FString& ObjectPath, UClass* ExpectedClass)
{
    // Handle blueprint class references
    if (ObjectPath.Contains(TEXT("_C")))
    {
        return ResolveBlueprintClassReference(ObjectPath, ExpectedClass);
    }
    
    // Handle asset references
    if (ObjectPath.StartsWith(TEXT("/")))
    {
        return LoadObject<UObject>(nullptr, *ObjectPath);
    }
    
    // Handle class references
    if (UClass* Class = FindObject<UClass>(nullptr, *ObjectPath))
    {
        if (Class->IsChildOf(ExpectedClass))
        {
            return Class->GetDefaultObject();
        }
    }
    
    return nullptr;
}
```

## Blueprint Default Value Compilation

### 1. Blueprint Property Defaults

**Blueprint Compilation Default Handling:**
```cpp
void ApplyBlueprintPropertyDefaults(UBlueprintGeneratedClass* Class, UObject* CDO)
{
    UBlueprint* Blueprint = Class->ClassGeneratedBy;
    
    // Apply defaults from Blueprint variables
    for (const FBPVariableDescription& Variable : Blueprint->NewVariables)
    {
        FProperty* Property = Class->FindPropertyByName(Variable.VarName);
        if (Property)
        {
            ApplyBlueprintVariableDefault(Property, CDO, Variable);
        }
    }
    
    // Apply component defaults
    ApplyComponentDefaults(Class, CDO);
}

void ApplyBlueprintVariableDefault(FProperty* Property, UObject* CDO, const FBPVariableDescription& Variable)
{
    // Get default value from variable description
    FString DefaultValue = Variable.DefaultValue;
    
    if (!DefaultValue.IsEmpty())
    {
        // Import the default value
        Property->ImportText(*DefaultValue, Property->ContainerPtrToValuePtr<void>(CDO), 0, CDO);
    }
    else if (Property->HasAnyPropertyFlags(CPF_HasGetValueTypeHash))
    {
        // Initialize complex types to valid defaults
        Property->InitializeValue_InContainer(CDO);
    }
}
```

### 2. Component Default Values

**Component Default Application:**
```cpp
void ApplyComponentDefaults(UBlueprintGeneratedClass* Class, UObject* CDO)
{
    if (USimpleConstructionScript* SCS = Class->SimpleConstructionScript)
    {
        for (USCS_Node* Node : SCS->GetAllNodes())
        {
            if (UActorComponent* Component = Node->GetActualComponentTemplate(Class))
            {
                ApplyComponentPropertyDefaults(Component, Node);
            }
        }
    }
}

void ApplyComponentPropertyDefaults(UActorComponent* Component, USCS_Node* Node)
{
    // Apply overridden property values
    for (const auto& PropertyPair : Node->GetCachedPropertyData())
    {
        FProperty* Property = PropertyPair.Key;
        const FBlueprintCookedComponentInstancingData& Data = PropertyPair.Value;
        
        if (Data.bHasValidCookedData)
        {
            // Apply the cooked property data
            Property->ImportText(*Data.CookedPropertyData, 
                                Property->ContainerPtrToValuePtr<void>(Component), 
                                0, Component);
        }
    }
}
```

## Type-Specific Default Value Handling

### 1. Array Default Values

**TArray Default Initialization:**
```cpp
void InitializeArrayDefaults(FArrayProperty* ArrayProperty, void* ContainerPtr, const TArray<FString>& DefaultValues)
{
    FScriptArray* Array = ArrayProperty->ContainerPtrToValuePtr<FScriptArray>(ContainerPtr);
    FProperty* InnerProperty = ArrayProperty->Inner;
    
    // Resize array to accommodate defaults
    Array->Empty(DefaultValues.Num(), InnerProperty->ElementSize);
    Array->AddUninitialized(DefaultValues.Num(), InnerProperty->ElementSize);
    
    // Initialize each element
    for (int32 Index = 0; Index < DefaultValues.Num(); ++Index)
    {
        void* ElementPtr = Array->GetData() + (Index * InnerProperty->ElementSize);
        
        if (!DefaultValues[Index].IsEmpty())
        {
            InnerProperty->ImportText(*DefaultValues[Index], ElementPtr, 0, nullptr);
        }
        else
        {
            InnerProperty->InitializeValue(ElementPtr);
        }
    }
}
```

### 2. Map Default Values

**TMap Default Initialization:**
```cpp
void InitializeMapDefaults(FMapProperty* MapProperty, void* ContainerPtr, const TMap<FString, FString>& DefaultPairs)
{
    FScriptMap* Map = MapProperty->ContainerPtrToValuePtr<FScriptMap>(ContainerPtr);
    FProperty* KeyProperty = MapProperty->KeyProp;
    FProperty* ValueProperty = MapProperty->ValueProp;
    
    // Initialize each key-value pair
    for (const auto& Pair : DefaultPairs)
    {
        void* KeyPtr = FMemory::Malloc(KeyProperty->ElementSize);
        void* ValuePtr = FMemory::Malloc(ValueProperty->ElementSize);
        
        // Initialize key and value
        KeyProperty->InitializeValue(KeyPtr);
        ValueProperty->InitializeValue(ValuePtr);
        
        // Import from strings
        if (KeyProperty->ImportText(*Pair.Key, KeyPtr, 0, nullptr) &&
            ValueProperty->ImportText(*Pair.Value, ValuePtr, 0, nullptr))
        {
            // Add to map
            MapProperty->AddPair(Map, KeyPtr, ValuePtr);
        }
        
        // Cleanup temporary memory
        KeyProperty->DestroyValue(KeyPtr);
        ValueProperty->DestroyValue(ValuePtr);
        FMemory::Free(KeyPtr);
        FMemory::Free(ValuePtr);
    }
}
```

### 3. Enum Default Values

**Enum Default Resolution:**
```cpp
void ApplyEnumDefault(FEnumProperty* EnumProperty, void* ContainerPtr, const FString& DefaultValueString)
{
    UEnum* Enum = EnumProperty->GetEnum();
    
    // Try to find enum value by name
    int64 EnumValue = Enum->GetValueByNameString(DefaultValueString);
    
    if (EnumValue == INDEX_NONE)
    {
        // Try alternate formats (short name, display name)
        for (int32 Index = 0; Index < Enum->NumEnums(); ++Index)
        {
            if (Enum->GetNameStringByIndex(Index) == DefaultValueString ||
                Enum->GetDisplayNameTextByIndex(Index).ToString() == DefaultValueString)
            {
                EnumValue = Enum->GetValueByIndex(Index);
                break;
            }
        }
    }
    
    if (EnumValue != INDEX_NONE)
    {
        EnumProperty->GetUnderlyingProperty()->SetIntPropertyValue(ContainerPtr, EnumValue);
    }
    else
    {
        // Use first valid enum value as fallback
        EnumProperty->GetUnderlyingProperty()->SetIntPropertyValue(ContainerPtr, Enum->GetValueByIndex(0));
    }
}
```

## Blueprint to C++ Default Value Conversion

### 1. Simple Type Defaults

**Blueprint Variable Default → C++:**
```cpp
// Blueprint: int32 Variable with default value 42
UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="MyCategory")
int32 MyInteger = 42;

// Blueprint: FString Variable with default value "Hello"
UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="MyCategory") 
FString MyString = TEXT("Hello");

// Blueprint: bool Variable with default value true
UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="MyCategory")
bool bMyFlag = true;
```

### 2. Complex Type Defaults

**Blueprint Struct Default → C++:**
```cpp
// Blueprint: FVector Variable with default value (1.0, 0.0, 0.0)
UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="MyCategory")
FVector MyVector = FVector(1.0f, 0.0f, 0.0f);

// Blueprint: FText Variable with localized default
UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="MyCategory")
FText MyText = NSLOCTEXT("MyNamespace", "MyKey", "Default Text");

// Constructor initialization for complex defaults
AMyActor::AMyActor()
{
    // Initialize complex structs that can't use in-line initialization
    MyComplexStruct.MemberA = TEXT("Default");
    MyComplexStruct.MemberB = 42;
    MyComplexStruct.MemberC = true;
}
```

### 3. Array and Container Defaults

**Blueprint Array Default → C++:**
```cpp
// Blueprint: TArray<int32> with default values [1, 2, 3]
UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="MyCategory")
TArray<int32> MyIntArray;

// Constructor initialization for container defaults
AMyActor::AMyActor()
{
    MyIntArray = {1, 2, 3};
    
    // Map defaults
    MyStringMap.Add(TEXT("Key1"), TEXT("Value1"));
    MyStringMap.Add(TEXT("Key2"), TEXT("Value2"));
}
```

### 4. Object Reference Defaults

**Blueprint Object Default → C++:**
```cpp
// Blueprint: UTexture2D reference with default asset
UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="MyCategory")
TObjectPtr<UTexture2D> MyTexture;

// Constructor initialization for object defaults
AMyActor::AMyActor()
{
    // Load default asset
    static ConstructorHelpers::FObjectFinder<UTexture2D> TextureFinder(TEXT("/Path/To/DefaultTexture"));
    if (TextureFinder.Succeeded())
    {
        MyTexture = TextureFinder.Object;
    }
}
```

## Data Structure Requirements for JSON Serialization

### Default Values Schema
```json
{
    "DefaultValues": [
        {
            "PropertyName": "MyInteger",
            "PropertyType": "int32",
            "DefaultValue": "42"
        },
        {
            "PropertyName": "MyText", 
            "PropertyType": "FText",
            "DefaultValue": "NSLOCTEXT(\"MyNamespace\", \"MyKey\", \"Default Text\")",
            "LocalizationInfo": {
                "Namespace": "MyNamespace",
                "Key": "MyKey",
                "SourceString": "Default Text"
            }
        },
        {
            "PropertyName": "MyVector",
            "PropertyType": "FVector",
            "DefaultValue": "1.0,0.0,0.0",
            "ConstructorInitialization": "FVector(1.0f, 0.0f, 0.0f)"
        },
        {
            "PropertyName": "MyArray",
            "PropertyType": "TArray<int32>", 
            "DefaultValue": "[1,2,3]",
            "ConstructorInitialization": "{1, 2, 3}"
        },
        {
            "PropertyName": "MyTexture",
            "PropertyType": "TObjectPtr<UTexture2D>",
            "DefaultValue": "/Path/To/DefaultTexture",
            "RequiresConstructorHelper": true,
            "ConstructorHelperPattern": "FObjectFinder"
        }
    ]
}
```

## Performance Considerations

### 1. Default Value Resolution Caching

**Cache default values to avoid repeated parsing:**
```cpp
class FDefaultValueCache
{
    TMap<FProperty*, FString> PropertyDefaultCache;
    
public:
    const FString& GetPropertyDefault(FProperty* Property)
    {
        FString* CachedDefault = PropertyDefaultCache.Find(Property);
        if (CachedDefault)
        {
            return *CachedDefault;
        }
        
        FString DefaultValue = ResolvePropertyDefault(Property);
        PropertyDefaultCache.Add(Property, DefaultValue);
        return PropertyDefaultCache[Property];
    }
};
```

### 2. Lazy Initialization

**Defer expensive default value resolution:**
```cpp
UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="MyCategory", meta=(LazyInit))
TObjectPtr<UExpensiveAsset> MyExpensiveAsset;

// Lazy initialization accessor
UExpensiveAsset* GetMyExpensiveAsset()
{
    if (!MyExpensiveAsset)
    {
        MyExpensiveAsset = LoadDefaultAsset();
    }
    return MyExpensiveAsset;
}
```

## Conclusion

Default value handling for complex types requires comprehensive understanding of:

1. **Property-specific initialization** for different types (FText, structs, objects)
2. **Constructor vs. inline initialization** patterns
3. **Asset reference resolution** using ConstructorHelpers
4. **Localization support** for FText defaults  
5. **Container initialization** for arrays and maps
6. **Performance optimization** through caching and lazy loading

Successful Blueprint to C++ conversion must preserve all default value semantics while choosing appropriate C++ initialization patterns for optimal performance and maintainability.