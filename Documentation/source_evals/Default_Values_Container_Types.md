# Default Values for Container Types Analysis for Blueprint to C++ Conversion

## Overview

This document analyzes the default value resolution system for container types (Arrays, Sets, Maps) in Unreal Engine, focusing on how Blueprint container defaults translate to C++ initialization patterns. Container default values require special handling due to their dynamic nature and element-wise initialization requirements.

## Core Container Default Value Infrastructure

### 1. TArray Default Value System

**File:** `/Engine/Source/Runtime/Core/Public/Containers/Array.h`

**Array Default Value Characteristics:**
- Empty array by default
- Element-wise initialization support
- Capacity pre-allocation optimization
- Inner property type validation

**Array Default Resolution Pattern:**
```cpp
class FArrayProperty : public FProperty
{
public:
    FProperty* Inner; // Element type property
    
    void InitializeArrayDefaults(void* ContainerPtr, const TArray<FString>& DefaultValues) const
    {
        FScriptArray* Array = ContainerPtrToValuePtr<FScriptArray>(ContainerPtr);
        
        // Clear existing contents
        Array->Empty(DefaultValues.Num(), Inner->ElementSize);
        
        if (DefaultValues.Num() > 0)
        {
            // Pre-allocate space for efficiency
            Array->AddUninitialized(DefaultValues.Num(), Inner->ElementSize);
            
            // Initialize each element
            for (int32 Index = 0; Index < DefaultValues.Num(); ++Index)
            {
                void* ElementPtr = Array->GetData() + (Index * Inner->ElementSize);
                
                if (!DefaultValues[Index].IsEmpty())
                {
                    // Import element value from string
                    Inner->ImportText_Direct(*DefaultValues[Index], ElementPtr, nullptr, 
                                           PPF_ParsingDefaultProperties, nullptr);
                }
                else
                {
                    // Initialize with element type default
                    Inner->InitializeValue(ElementPtr);
                }
            }
        }
    }
};
```

**Array ExportText/ImportText Implementation:**
```cpp
void FArrayProperty::ExportText_Internal(FString& ValueStr, const void* PropertyValueOrContainer, 
                                        EPropertyPointerType PropertyPointerType, const void* DefaultValue, 
                                        UObject* Parent, int32 PortFlags, UObject* ExportRootScope) const
{
    const FScriptArray* Array = nullptr;
    if (PropertyPointerType == EPropertyPointerType::Direct)
    {
        Array = static_cast<const FScriptArray*>(PropertyValueOrContainer);
    }
    else
    {
        Array = ContainerPtrToValuePtr<FScriptArray>(PropertyValueOrContainer);
    }
    
    if (Array->Num() == 0)
    {
        ValueStr += TEXT("()");
        return;
    }
    
    ValueStr += TEXT("(");
    
    for (int32 Index = 0; Index < Array->Num(); ++Index)
    {
        if (Index > 0)
        {
            ValueStr += TEXT(",");
        }
        
        const void* ElementPtr = Array->GetData() + (Index * Inner->ElementSize);
        
        FString ElementStr;
        Inner->ExportText_Direct(ElementStr, ElementPtr, nullptr, Parent, PortFlags, ExportRootScope);
        
        ValueStr += ElementStr;
    }
    
    ValueStr += TEXT(")");
}

const TCHAR* FArrayProperty::ImportText_Internal(const TCHAR* Buffer, void* ContainerOrPropertyPtr, 
                                               EPropertyPointerType PropertyPointerType, UObject* OwnerObject, 
                                               int32 PortFlags, FOutputDevice* ErrorText) const
{
    FScriptArray* Array = nullptr;
    if (PropertyPointerType == EPropertyPointerType::Direct)
    {
        Array = static_cast<FScriptArray*>(ContainerOrPropertyPtr);
    }
    else
    {
        Array = ContainerPtrToValuePtr<FScriptArray>(ContainerOrPropertyPtr);
    }
    
    // Clear existing array
    Array->Empty(0, Inner->ElementSize);
    
    // Skip whitespace
    Buffer = FPropertyHelpers::ReadToken(Buffer, nullptr, true);
    
    if (*Buffer == '(')
    {
        Buffer++; // Skip opening parenthesis
        
        while (*Buffer && *Buffer != ')')
        {
            // Add new uninitialized element
            int32 ElementIndex = Array->AddUninitialized(1, Inner->ElementSize);
            void* ElementPtr = Array->GetData() + (ElementIndex * Inner->ElementSize);
            
            // Initialize element to default state
            Inner->InitializeValue(ElementPtr);
            
            // Import element value
            Buffer = Inner->ImportText_Direct(Buffer, ElementPtr, OwnerObject, PortFlags, ErrorText);
            
            if (!Buffer)
            {
                return nullptr; // Import failed
            }
            
            // Skip to next element or end
            Buffer = FPropertyHelpers::ReadToken(Buffer, nullptr, true);
            if (*Buffer == ',')
            {
                Buffer++; // Skip comma
            }
        }
        
        if (*Buffer == ')')
        {
            Buffer++; // Skip closing parenthesis
        }
    }
    
    return Buffer;
}
```

### 2. TSet Default Value System

**Set Default Value Characteristics:**
- Empty set by default
- Unique element enforcement
- Hash-based element storage
- Element key validation

**Set Default Resolution Pattern:**
```cpp
class FSetProperty : public FProperty
{
public:
    FProperty* ElementProp; // Element type property
    
    void InitializeSetDefaults(void* ContainerPtr, const TArray<FString>& DefaultValues) const
    {
        FScriptSet* Set = ContainerPtrToValuePtr<FScriptSet>(ContainerPtr);
        
        // Clear existing contents
        Set->Empty(DefaultValues.Num(), ElementProp->ElementSize);
        
        for (const FString& ElementValue : DefaultValues)
        {
            if (!ElementValue.IsEmpty())
            {
                // Create temporary element for import
                void* TempElement = FMemory::Malloc(ElementProp->ElementSize);
                ElementProp->InitializeValue(TempElement);
                
                // Import element value
                if (ElementProp->ImportText_Direct(*ElementValue, TempElement, nullptr, 
                                                 PPF_ParsingDefaultProperties, nullptr))
                {
                    // Add to set (automatically handles uniqueness)
                    Set->Add(TempElement, ElementProp);
                }
                
                // Cleanup temporary element
                ElementProp->DestroyValue(TempElement);
                FMemory::Free(TempElement);
            }
        }
    }
};
```

**Set ExportText/ImportText Implementation:**
```cpp
void FSetProperty::ExportText_Internal(FString& ValueStr, const void* PropertyValueOrContainer, 
                                      EPropertyPointerType PropertyPointerType, const void* DefaultValue, 
                                      UObject* Parent, int32 PortFlags, UObject* ExportRootScope) const
{
    const FScriptSet* Set = GetSetFromContainer(PropertyValueOrContainer, PropertyPointerType);
    
    if (Set->Num() == 0)
    {
        ValueStr += TEXT("()");
        return;
    }
    
    ValueStr += TEXT("(");
    
    bool bFirst = true;
    for (FScriptSetHelper SetHelper(this, Set); SetHelper.IsValidIndex(); ++SetHelper)
    {
        if (!bFirst)
        {
            ValueStr += TEXT(",");
        }
        bFirst = false;
        
        const void* ElementPtr = SetHelper.GetElementPtr();
        
        FString ElementStr;
        ElementProp->ExportText_Direct(ElementStr, ElementPtr, nullptr, Parent, PortFlags, ExportRootScope);
        ValueStr += ElementStr;
    }
    
    ValueStr += TEXT(")");
}

const TCHAR* FSetProperty::ImportText_Internal(const TCHAR* Buffer, void* ContainerOrPropertyPtr, 
                                             EPropertyPointerType PropertyPointerType, UObject* OwnerObject, 
                                             int32 PortFlags, FOutputDevice* ErrorText) const
{
    FScriptSet* Set = GetSetFromContainer(ContainerOrPropertyPtr, PropertyPointerType);
    
    // Clear existing set
    Set->Empty(0, ElementProp);
    
    // Skip whitespace
    Buffer = FPropertyHelpers::ReadToken(Buffer, nullptr, true);
    
    if (*Buffer == '(')
    {
        Buffer++; // Skip opening parenthesis
        
        while (*Buffer && *Buffer != ')')
        {
            // Create temporary element for import
            void* TempElement = FMemory::Malloc(ElementProp->ElementSize);
            ElementProp->InitializeValue(TempElement);
            
            // Import element value
            Buffer = ElementProp->ImportText_Direct(Buffer, TempElement, OwnerObject, PortFlags, ErrorText);
            
            if (Buffer)
            {
                // Add to set
                Set->Add(TempElement, ElementProp);
            }
            
            // Cleanup
            ElementProp->DestroyValue(TempElement);
            FMemory::Free(TempElement);
            
            if (!Buffer)
            {
                return nullptr; // Import failed
            }
            
            // Skip to next element or end
            Buffer = FPropertyHelpers::ReadToken(Buffer, nullptr, true);
            if (*Buffer == ',')
            {
                Buffer++; // Skip comma
            }
        }
        
        if (*Buffer == ')')
        {
            Buffer++; // Skip closing parenthesis
        }
    }
    
    return Buffer;
}
```

### 3. TMap Default Value System

**File:** `/Engine/Source/Runtime/Core/Public/Containers/Map.h`

**Map Default Value Characteristics:**
- Empty map by default
- Key-value pair initialization
- Key uniqueness enforcement
- Both key and value type validation

**Map Default Resolution Pattern:**
```cpp
class FMapProperty : public FProperty
{
public:
    FProperty* KeyProp;   // Key type property
    FProperty* ValueProp; // Value type property
    
    void InitializeMapDefaults(void* ContainerPtr, const TMap<FString, FString>& DefaultPairs) const
    {
        FScriptMap* Map = ContainerPtrToValuePtr<FScriptMap>(ContainerPtr);
        
        // Clear existing contents
        Map->Empty(DefaultPairs.Num(), KeyProp, ValueProp);
        
        for (const auto& Pair : DefaultPairs)
        {
            if (!Pair.Key.IsEmpty())
            {
                // Create temporary key and value for import
                void* TempKey = FMemory::Malloc(KeyProp->ElementSize);
                void* TempValue = FMemory::Malloc(ValueProp->ElementSize);
                
                KeyProp->InitializeValue(TempKey);
                ValueProp->InitializeValue(TempValue);
                
                // Import key and value
                bool bKeyImported = KeyProp->ImportText_Direct(*Pair.Key, TempKey, nullptr, 
                                                             PPF_ParsingDefaultProperties, nullptr) != nullptr;
                bool bValueImported = true;
                
                if (!Pair.Value.IsEmpty())
                {
                    bValueImported = ValueProp->ImportText_Direct(*Pair.Value, TempValue, nullptr, 
                                                                PPF_ParsingDefaultProperties, nullptr) != nullptr;
                }
                
                if (bKeyImported && bValueImported)
                {
                    // Add key-value pair to map
                    Map->Add(TempKey, TempValue, KeyProp, ValueProp);
                }
                
                // Cleanup temporary memory
                KeyProp->DestroyValue(TempKey);
                ValueProp->DestroyValue(TempValue);
                FMemory::Free(TempKey);
                FMemory::Free(TempValue);
            }
        }
    }
};
```

**Map ExportText/ImportText Implementation:**
```cpp
void FMapProperty::ExportText_Internal(FString& ValueStr, const void* PropertyValueOrContainer, 
                                      EPropertyPointerType PropertyPointerType, const void* DefaultValue, 
                                      UObject* Parent, int32 PortFlags, UObject* ExportRootScope) const
{
    const FScriptMap* Map = GetMapFromContainer(PropertyValueOrContainer, PropertyPointerType);
    
    if (Map->Num() == 0)
    {
        ValueStr += TEXT("()");
        return;
    }
    
    ValueStr += TEXT("(");
    
    bool bFirst = true;
    for (FScriptMapHelper MapHelper(this, Map); MapHelper.IsValidIndex(); ++MapHelper)
    {
        if (!bFirst)
        {
            ValueStr += TEXT(",");
        }
        bFirst = false;
        
        const void* KeyPtr = MapHelper.GetKeyPtr();
        const void* ValuePtr = MapHelper.GetValuePtr();
        
        // Export as (Key=Value) pairs
        ValueStr += TEXT("(");
        
        FString KeyStr;
        KeyProp->ExportText_Direct(KeyStr, KeyPtr, nullptr, Parent, PortFlags, ExportRootScope);
        ValueStr += KeyStr;
        
        ValueStr += TEXT("=");
        
        FString ValueStr;
        ValueProp->ExportText_Direct(ValueStr, ValuePtr, nullptr, Parent, PortFlags, ExportRootScope);
        ValueStr += ValueStr;
        
        ValueStr += TEXT(")");
    }
    
    ValueStr += TEXT(")");
}

const TCHAR* FMapProperty::ImportText_Internal(const TCHAR* Buffer, void* ContainerOrPropertyPtr, 
                                             EPropertyPointerType PropertyPointerType, UObject* OwnerObject, 
                                             int32 PortFlags, FOutputDevice* ErrorText) const
{
    FScriptMap* Map = GetMapFromContainer(ContainerOrPropertyPtr, PropertyPointerType);
    
    // Clear existing map
    Map->Empty(0, KeyProp, ValueProp);
    
    // Skip whitespace
    Buffer = FPropertyHelpers::ReadToken(Buffer, nullptr, true);
    
    if (*Buffer == '(')
    {
        Buffer++; // Skip opening parenthesis
        
        while (*Buffer && *Buffer != ')')
        {
            // Expect (Key=Value) format
            if (*Buffer == '(')
            {
                Buffer++; // Skip pair opening parenthesis
                
                // Create temporary key and value
                void* TempKey = FMemory::Malloc(KeyProp->ElementSize);
                void* TempValue = FMemory::Malloc(ValueProp->ElementSize);
                
                KeyProp->InitializeValue(TempKey);
                ValueProp->InitializeValue(TempValue);
                
                // Import key
                Buffer = KeyProp->ImportText_Direct(Buffer, TempKey, OwnerObject, PortFlags, ErrorText);
                
                if (Buffer && *Buffer == '=')
                {
                    Buffer++; // Skip equals sign
                    
                    // Import value
                    Buffer = ValueProp->ImportText_Direct(Buffer, TempValue, OwnerObject, PortFlags, ErrorText);
                    
                    if (Buffer)
                    {
                        // Add to map
                        Map->Add(TempKey, TempValue, KeyProp, ValueProp);
                    }
                }
                
                // Cleanup
                KeyProp->DestroyValue(TempKey);
                ValueProp->DestroyValue(TempValue);
                FMemory::Free(TempKey);
                FMemory::Free(TempValue);
                
                if (!Buffer)
                {
                    return nullptr; // Import failed
                }
                
                // Skip pair closing parenthesis
                if (*Buffer == ')')
                {
                    Buffer++; 
                }
            }
            
            // Skip to next pair or end
            Buffer = FPropertyHelpers::ReadToken(Buffer, nullptr, true);
            if (*Buffer == ',')
            {
                Buffer++; // Skip comma
            }
        }
        
        if (*Buffer == ')')
        {
            Buffer++; // Skip closing parenthesis
        }
    }
    
    return Buffer;
}
```

## Container Type C++ Generation Patterns

### 1. TArray C++ Generation

**Simple Array Defaults:**
```cpp
// Blueprint: TArray<int32> with defaults [1, 2, 3]
UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="Default")
TArray<int32> MyIntArray;

// Constructor initialization (preferred for non-empty arrays)
AMyActor::AMyActor()
{
    MyIntArray = {1, 2, 3};
    
    // Alternative explicit initialization
    MyIntArray.Empty();
    MyIntArray.Add(1);
    MyIntArray.Add(2);
    MyIntArray.Add(3);
    
    // Pre-allocated initialization
    MyIntArray.Reserve(3);
    MyIntArray.Emplace(1);
    MyIntArray.Emplace(2);
    MyIntArray.Emplace(3);
}
```

**Complex Array Element Defaults:**
```cpp
// Blueprint: TArray<FString> with string defaults
UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="Default")
TArray<FString> MyStringArray;

// Constructor initialization for complex elements
AMyActor::AMyActor()
{
    MyStringArray = {
        TEXT("First String"),
        TEXT("Second String"),
        TEXT("Third String")
    };
}

// Blueprint: TArray<FVector> with vector defaults
UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="Default")
TArray<FVector> MyVectorArray;

AMyActor::AMyActor()
{
    MyVectorArray = {
        FVector(1.0f, 0.0f, 0.0f),
        FVector(0.0f, 1.0f, 0.0f),
        FVector(0.0f, 0.0f, 1.0f)
    };
}
```

**Object Array Defaults:**
```cpp
// Blueprint: TArray<UObject*> with object references
UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="Default")
TArray<TObjectPtr<UTexture2D>> MyTextureArray;

AMyActor::AMyActor()
{
    // Load default textures
    static ConstructorHelpers::FObjectFinder<UTexture2D> Texture1(TEXT("/Path/To/Texture1"));
    static ConstructorHelpers::FObjectFinder<UTexture2D> Texture2(TEXT("/Path/To/Texture2"));
    
    if (Texture1.Succeeded() && Texture2.Succeeded())
    {
        MyTextureArray = {
            Texture1.Object,
            Texture2.Object
        };
    }
}
```

### 2. TSet C++ Generation

**Simple Set Defaults:**
```cpp
// Blueprint: TSet<int32> with defaults {1, 2, 3}
UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="Default")
TSet<int32> MyIntSet;

// Constructor initialization
AMyActor::AMyActor()
{
    MyIntSet = {1, 2, 3};
    
    // Alternative explicit initialization
    MyIntSet.Empty();
    MyIntSet.Add(1);
    MyIntSet.Add(2);
    MyIntSet.Add(3);
}
```

**Complex Set Element Defaults:**
```cpp
// Blueprint: TSet<FString> with string defaults
UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="Default")
TSet<FString> MyStringSet;

AMyActor::AMyActor()
{
    MyStringSet = {
        TEXT("Element1"),
        TEXT("Element2"),
        TEXT("Element3")
    };
}

// Blueprint: TSet<FName> with name defaults
UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="Default")
TSet<FName> MyNameSet;

AMyActor::AMyActor()
{
    MyNameSet = {
        TEXT("Name1"),
        TEXT("Name2"),
        TEXT("Name3")
    };
}
```

### 3. TMap C++ Generation

**Simple Map Defaults:**
```cpp
// Blueprint: TMap<FString, int32> with defaults {"Key1"->1, "Key2"->2}
UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="Default")
TMap<FString, int32> MyStringIntMap;

// Constructor initialization
AMyActor::AMyActor()
{
    MyStringIntMap = {
        {TEXT("Key1"), 1},
        {TEXT("Key2"), 2},
        {TEXT("Key3"), 3}
    };
    
    // Alternative explicit initialization
    MyStringIntMap.Empty();
    MyStringIntMap.Add(TEXT("Key1"), 1);
    MyStringIntMap.Add(TEXT("Key2"), 2);
    MyStringIntMap.Add(TEXT("Key3"), 3);
}
```

**Complex Map Defaults:**
```cpp
// Blueprint: TMap<FName, FVector> with complex defaults
UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="Default")
TMap<FName, FVector> MyNameVectorMap;

AMyActor::AMyActor()
{
    MyNameVectorMap = {
        {TEXT("Forward"), FVector(1.0f, 0.0f, 0.0f)},
        {TEXT("Right"), FVector(0.0f, 1.0f, 0.0f)},
        {TEXT("Up"), FVector(0.0f, 0.0f, 1.0f)}
    };
}

// Blueprint: TMap<FString, UObject*> with object value defaults
UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="Default")
TMap<FString, TObjectPtr<UTexture2D>> MyStringTextureMap;

AMyActor::AMyActor()
{
    // Load default textures
    static ConstructorHelpers::FObjectFinder<UTexture2D> DefaultTexture(TEXT("/Path/To/Default"));
    static ConstructorHelpers::FObjectFinder<UTexture2D> AlternateTexture(TEXT("/Path/To/Alternate"));
    
    if (DefaultTexture.Succeeded() && AlternateTexture.Succeeded())
    {
        MyStringTextureMap = {
            {TEXT("Default"), DefaultTexture.Object},
            {TEXT("Alternate"), AlternateTexture.Object}
        };
    }
}
```

## Container Default Value Optimization Strategies

### 1. Pre-allocation Optimization

**Reserve Capacity for Known Sizes:**
```cpp
AMyActor::AMyActor()
{
    // Reserve capacity before adding elements
    MyLargeArray.Reserve(1000);
    for (int32 i = 0; i < 1000; ++i)
    {
        MyLargeArray.Emplace(i);
    }
    
    // Reserve capacity for maps
    MyLargeMap.Reserve(500);
    for (int32 i = 0; i < 500; ++i)
    {
        MyLargeMap.Add(FString::Printf(TEXT("Key%d"), i), i);
    }
}
```

### 2. Lazy Initialization

**Defer Container Population:**
```cpp
UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="Default", meta=(LazyInit))
TArray<UExpensiveObject*> MyExpensiveArray;

bool bArrayInitialized = false;

const TArray<UExpensiveObject*>& GetExpensiveArray()
{
    if (!bArrayInitialized)
    {
        InitializeExpensiveArray();
        bArrayInitialized = true;
    }
    return MyExpensiveArray;
}

void InitializeExpensiveArray()
{
    MyExpensiveArray.Reserve(10);
    for (int32 i = 0; i < 10; ++i)
    {
        MyExpensiveArray.Add(CreateExpensiveObject(i));
    }
}
```

### 3. Asset Reference Containers

**Container with Asset Loading:**
```cpp
// Blueprint: TArray<UTexture2D*> with asset paths
UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="Default")
TArray<TObjectPtr<UTexture2D>> MyTextureArray;

AMyActor::AMyActor()
{
    // Asset paths from Blueprint defaults
    TArray<FString> TexturePaths = {
        TEXT("/Game/Textures/Texture1"),
        TEXT("/Game/Textures/Texture2"),
        TEXT("/Game/Textures/Texture3")
    };
    
    MyTextureArray.Reserve(TexturePaths.Num());
    
    for (const FString& TexturePath : TexturePaths)
    {
        if (UTexture2D* LoadedTexture = LoadObject<UTexture2D>(nullptr, *TexturePath))
        {
            MyTextureArray.Add(LoadedTexture);
        }
        else
        {
            UE_LOG(LogTemp, Warning, TEXT("Failed to load texture: %s"), *TexturePath);
            MyTextureArray.Add(nullptr); // Maintain array size consistency
        }
    }
}
```

## JSON Schema for Container Default Values

### Container Defaults Data Structure
```json
{
    "ContainerDefaults": [
        {
            "PropertyName": "MyIntArray",
            "PropertyType": "TArray<int32>",
            "CPPType": "TArray<int32>",
            "ContainerType": "Array",
            "ElementType": "int32",
            "DefaultValues": ["1", "2", "3"],
            "InitializationPattern": "Constructor",
            "ConstructorExpression": "{1, 2, 3}",
            "RequiresReserve": false,
            "ElementCount": 3
        },
        {
            "PropertyName": "MyStringSet",
            "PropertyType": "TSet<FString>",
            "CPPType": "TSet<FString>",
            "ContainerType": "Set",
            "ElementType": "FString",
            "DefaultValues": ["Element1", "Element2", "Element3"],
            "InitializationPattern": "Constructor",
            "ConstructorExpression": "{TEXT(\"Element1\"), TEXT(\"Element2\"), TEXT(\"Element3\")}",
            "RequiresQuotes": true,
            "ElementCount": 3
        },
        {
            "PropertyName": "MyStringIntMap",
            "PropertyType": "TMap<FString, int32>",
            "CPPType": "TMap<FString, int32>",
            "ContainerType": "Map",
            "KeyType": "FString",
            "ValueType": "int32",
            "DefaultPairs": [
                {"Key": "Key1", "Value": "1"},
                {"Key": "Key2", "Value": "2"},
                {"Key": "Key3", "Value": "3"}
            ],
            "InitializationPattern": "Constructor",
            "ConstructorExpression": "{{TEXT(\"Key1\"), 1}, {TEXT(\"Key2\"), 2}, {TEXT(\"Key3\"), 3}}",
            "KeyRequiresQuotes": true,
            "ValueRequiresQuotes": false,
            "PairCount": 3
        },
        {
            "PropertyName": "MyTextureArray",
            "PropertyType": "TArray<UTexture2D*>",
            "CPPType": "TArray<TObjectPtr<UTexture2D>>",
            "ContainerType": "Array", 
            "ElementType": "TObjectPtr<UTexture2D>",
            "DefaultValues": ["/Game/Textures/Texture1", "/Game/Textures/Texture2"],
            "InitializationPattern": "ConstructorHelpers",
            "RequiresAssetLoading": true,
            "ConstructorHelperPattern": "FObjectFinder",
            "AssetPaths": [
                "/Game/Textures/Texture1",
                "/Game/Textures/Texture2"
            ]
        }
    ]
}
```

## Container Serialization Format Specifications

### Array Serialization Format
```
Format: (Element1,Element2,Element3)
Examples:
- Empty Array: ()
- Int Array: (1,2,3,4,5)
- String Array: (First String,Second String,Third String)
- Object Array: (/Game/Asset1,/Game/Asset2,None)
```

### Set Serialization Format
```
Format: (Element1,Element2,Element3)
Examples:
- Empty Set: ()
- Int Set: (1,2,3)
- String Set: (Value1,Value2,Value3)
- Name Set: (Name1,Name2,Name3)
```

### Map Serialization Format
```
Format: ((Key1=Value1),(Key2=Value2),(Key3=Value3))
Examples:
- Empty Map: ()
- String->Int Map: ((Key1=1),(Key2=2),(Key3=3))
- Name->Vector Map: ((Forward=1,0,0),(Right=0,1,0),(Up=0,0,1))
- String->Object Map: ((Default=/Game/Asset1),(Fallback=None))
```

## Advanced Container Default Patterns

### 1. Nested Container Defaults

```cpp
// Blueprint: TArray<TMap<FString, int32>> with nested defaults
UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="Default")
TArray<TMap<FString, int32>> MyNestedContainer;

AMyActor::AMyActor()
{
    MyNestedContainer.SetNum(2);
    
    // First map
    MyNestedContainer[0] = {
        {TEXT("A"), 1},
        {TEXT("B"), 2}
    };
    
    // Second map
    MyNestedContainer[1] = {
        {TEXT("C"), 3},
        {TEXT("D"), 4}
    };
}
```

### 2. Container with Custom Struct Elements

```cpp
// Custom struct for container elements
USTRUCT(BlueprintType)
struct FMyCustomStruct
{
    GENERATED_BODY()

    UPROPERTY(EditAnywhere, BlueprintReadWrite)
    FString Name = TEXT("Default");
    
    UPROPERTY(EditAnywhere, BlueprintReadWrite)
    int32 Value = 0;
    
    UPROPERTY(EditAnywhere, BlueprintReadWrite)
    bool bFlag = false;
};

// Blueprint: TArray<FMyCustomStruct> with struct defaults
UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="Default")
TArray<FMyCustomStruct> MyStructArray;

AMyActor::AMyActor()
{
    MyStructArray.SetNum(2);
    
    // Initialize struct elements
    MyStructArray[0].Name = TEXT("First");
    MyStructArray[0].Value = 10;
    MyStructArray[0].bFlag = true;
    
    MyStructArray[1].Name = TEXT("Second");
    MyStructArray[1].Value = 20;
    MyStructArray[1].bFlag = false;
}
```

## Performance and Memory Considerations

### 1. Container Size Impact

**Memory allocation patterns:**
```cpp
// Small containers - inline initialization is fine
TArray<int32> SmallArray = {1, 2, 3}; // ~12 bytes + overhead

// Large containers - consider lazy initialization
TArray<int32> LargeArray; // Initialize in BeginPlay() instead
```

### 2. Asset Reference Performance

**Avoid constructor asset loading:**
```cpp
// AVOID: Loading assets in constructor (synchronous, slow)
AMyActor::AMyActor()
{
    // BAD: Synchronous loading in constructor
    for (const FString& AssetPath : AssetPaths)
    {
        MyAssetArray.Add(LoadObject<UTexture2D>(nullptr, *AssetPath));
    }
}

// PREFERRED: Lazy loading or BeginPlay initialization
void AMyActor::BeginPlay()
{
    Super::BeginPlay();
    
    // Asynchronous asset loading
    LoadAssetsAsync();
}
```

## Conclusion

Container default value resolution requires comprehensive understanding of:

1. **Container-specific serialization** formats for Arrays, Sets, and Maps
2. **Element-wise initialization** patterns and memory management
3. **Constructor vs. inline initialization** decision logic
4. **Asset loading strategies** for object reference containers
5. **Performance optimization** through pre-allocation and lazy loading
6. **Nested container handling** for complex data structures

This infrastructure enables accurate Blueprint to C++ conversion for all container types while maintaining performance and memory efficiency.