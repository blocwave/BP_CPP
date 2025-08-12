# Default Values for Primitive Types Analysis for Blueprint to C++ Conversion

## Overview

This document analyzes the default value resolution system for primitive types in Unreal Engine, focusing on how Blueprint default values translate to C++ initialization patterns. This is critical for ensuring converted C++ code compiles with proper initial values matching Blueprint behavior.

## Core Default Value Infrastructure

### 1. FProperty Default Value Methods

**File:** `/Engine/Source/Runtime/CoreUObject/Public/UObject/UnrealType.h`

**Key Default Value Interface:**
```cpp
class COREUOBJECT_API FProperty : public FField
{
public:
    // Core export/import for default values
    virtual void ExportText_Internal(FString& ValueStr, const void* PropertyValueOrContainer, 
                                   EPropertyPointerType PointerType, const void* DefaultValue, 
                                   UObject* Parent, int32 PortFlags, UObject* ExportRootScope = nullptr) const = 0;
    
    virtual const TCHAR* ImportText_Internal(const TCHAR* Buffer, void* ContainerOrPropertyPtr, 
                                           EPropertyPointerType PropertyPointerType, UObject* OwnerObject, 
                                           int32 PortFlags, FOutputDevice* ErrorText) const = 0;

    // C++ type generation
    virtual FString GetCPPMacroType(FString& ExtendedTypeText) const = 0;
    virtual FString GetCPPType(FString* ExtendedTypeText, uint32 CPPExportFlags) const = 0;
    
    // Value initialization
    virtual void InitializeValueInternal(void* Dest) const = 0;
};
```

### 2. Property Port Flags for Default Values

**File:** `/Engine/Source/Runtime/Core/Public/UObject/PropertyPortFlags.h`

**Key Flags for Default Value Handling:**
```cpp
enum EPropertyPortFlags
{
    PPF_None = 0x00000000,
    
    // Wrap property data in quotes for text serialization
    PPF_Delimited = 0x00000001,
    
    // External editor format - default values always written
    PPF_ExternalEditor = 0x00000040,
    
    // Parsing default properties - allow transient property text import
    PPF_ParsingDefaultProperties = 0x00008000,
    
    // Blueprint debugging watch values
    PPF_BlueprintDebugView = 0x02000000,
    
    // Using deprecated properties during import
    PPF_UseDeprecatedProperties = 0x08000000,
};
```

## Primitive Type Default Value Resolution

### 1. Integer Types (int8, int16, int32, int64, uint8, uint16, uint32, uint64)

**Default Value Characteristics:**
- Zero-initialized by default
- Support decimal and hexadecimal formats
- Range validation during import

**Default Resolution Pattern:**
```cpp
void ApplyIntegerDefaults(FIntProperty* Property, void* ContainerPtr, const FString& DefaultValue)
{
    int32* ValuePtr = Property->ContainerPtrToValuePtr<int32>(ContainerPtr);
    
    if (DefaultValue.IsEmpty())
    {
        *ValuePtr = 0; // Zero default
    }
    else
    {
        // Parse with support for hex format (0x prefix)
        int32 ParsedValue = 0;
        if (DefaultValue.StartsWith(TEXT("0x")))
        {
            ParsedValue = FParse::HexNumber(*DefaultValue);
        }
        else
        {
            LexFromString(ParsedValue, *DefaultValue);
        }
        
        // Apply range validation if present
        *ValuePtr = FMath::Clamp(ParsedValue, Property->GetMinValue(), Property->GetMaxValue());
    }
}
```

**C++ Generation for Integer Defaults:**
```cpp
// Blueprint: int32 with default 42
UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="Default")
int32 MyInteger = 42;

// Blueprint: uint8 with hex default 0xFF
UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="Default")
uint8 MyByte = 0xFF;

// Blueprint: int64 with large default
UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="Default")
int64 MyLargeInt = 1234567890123LL;
```

### 2. Floating Point Types (float, double)

**Default Value Characteristics:**
- Zero-initialized by default
- Support scientific notation
- Special values: infinity, NaN handling

**Default Resolution Pattern:**
```cpp
void ApplyFloatDefaults(FFloatProperty* Property, void* ContainerPtr, const FString& DefaultValue)
{
    float* ValuePtr = Property->ContainerPtrToValuePtr<float>(ContainerPtr);
    
    if (DefaultValue.IsEmpty())
    {
        *ValuePtr = 0.0f; // Zero default
    }
    else if (DefaultValue.Equals(TEXT("inf"), ESearchCase::IgnoreCase))
    {
        *ValuePtr = TNumericLimits<float>::Max();
    }
    else if (DefaultValue.Equals(TEXT("-inf"), ESearchCase::IgnoreCase))
    {
        *ValuePtr = TNumericLimits<float>::Lowest();
    }
    else if (DefaultValue.Equals(TEXT("nan"), ESearchCase::IgnoreCase))
    {
        *ValuePtr = TNumericLimits<float>::QuietNaN();
    }
    else
    {
        float ParsedValue = 0.0f;
        LexFromString(ParsedValue, *DefaultValue);
        
        // Apply range validation if present
        *ValuePtr = FMath::Clamp(ParsedValue, Property->GetMinValue(), Property->GetMaxValue());
    }
}
```

**C++ Generation for Float Defaults:**
```cpp
// Blueprint: float with default 3.14159
UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="Default")
float MyFloat = 3.14159f;

// Blueprint: double with scientific notation
UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="Default")
double MyDouble = 1.23e-4;

// Blueprint: float with special value handling
UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="Default")
float MySpecialFloat = TNumericLimits<float>::Max();
```

### 3. Boolean Type

**Default Value Characteristics:**
- False-initialized by default
- Support multiple text formats: "true"/"false", "1"/"0", "yes"/"no"

**Default Resolution Pattern:**
```cpp
void ApplyBoolDefaults(FBoolProperty* Property, void* ContainerPtr, const FString& DefaultValue)
{
    // Handle both bitfield and direct bool properties
    if (Property->IsNativeBool())
    {
        bool* BoolPtr = Property->ContainerPtrToValuePtr<bool>(ContainerPtr);
        *BoolPtr = ParseBooleanValue(DefaultValue);
    }
    else
    {
        // Bitfield property
        Property->SetPropertyValue_InContainer(ContainerPtr, ParseBooleanValue(DefaultValue));
    }
}

bool ParseBooleanValue(const FString& Value)
{
    if (Value.IsEmpty())
    {
        return false; // Default false
    }
    
    // Case-insensitive parsing of boolean values
    FString LowerValue = Value.ToLower();
    return LowerValue == TEXT("true") || 
           LowerValue == TEXT("1") || 
           LowerValue == TEXT("yes") ||
           LowerValue == TEXT("on");
}
```

**C++ Generation for Boolean Defaults:**
```cpp
// Blueprint: bool with default true
UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="Default")
bool bMyFlag = true;

// Blueprint: bitfield bool with default
UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="Default", meta=(Bitfield))
uint32 bMyBitfield : 1;

// Constructor initialization for bitfields
AMyActor::AMyActor()
{
    bMyBitfield = 1; // true
}
```

### 4. Name Type

**Default Value Characteristics:**
- NAME_None initialized by default
- String-to-FName conversion
- Case-sensitive name table lookup

**Default Resolution Pattern:**
```cpp
void ApplyNameDefaults(FNameProperty* Property, void* ContainerPtr, const FString& DefaultValue)
{
    FName* NamePtr = Property->ContainerPtrToValuePtr<FName>(ContainerPtr);
    
    if (DefaultValue.IsEmpty() || DefaultValue == TEXT("None"))
    {
        *NamePtr = NAME_None;
    }
    else
    {
        *NamePtr = FName(*DefaultValue);
    }
}
```

**C++ Generation for Name Defaults:**
```cpp
// Blueprint: FName with default "MyName"
UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="Default")
FName MyName = TEXT("MyName");

// Blueprint: FName with None default
UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="Default")
FName MyEmptyName = NAME_None;
```

### 5. String Type

**Default Value Characteristics:**
- Empty string initialized by default
- UTF-8/UTF-16 support
- Escape sequence handling

**Default Resolution Pattern:**
```cpp
void ApplyStringDefaults(FStrProperty* Property, void* ContainerPtr, const FString& DefaultValue)
{
    FString* StringPtr = Property->ContainerPtrToValuePtr<FString>(ContainerPtr);
    
    if (DefaultValue.IsEmpty())
    {
        *StringPtr = FString(); // Empty string default
    }
    else
    {
        // Process escape sequences
        FString ProcessedValue = ProcessStringEscapes(DefaultValue);
        *StringPtr = ProcessedValue;
    }
}

FString ProcessStringEscapes(const FString& InputString)
{
    FString Result = InputString;
    
    // Handle common escape sequences
    Result = Result.Replace(TEXT("\\n"), TEXT("\n"));
    Result = Result.Replace(TEXT("\\r"), TEXT("\r"));
    Result = Result.Replace(TEXT("\\t"), TEXT("\t"));
    Result = Result.Replace(TEXT("\\\""), TEXT("\""));
    Result = Result.Replace(TEXT("\\\\"), TEXT("\\"));
    
    return Result;
}
```

**C++ Generation for String Defaults:**
```cpp
// Blueprint: FString with default "Hello World"
UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="Default")
FString MyString = TEXT("Hello World");

// Blueprint: FString with escape sequences
UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="Default")
FString MyStringWithNewlines = TEXT("Line 1\nLine 2\nLine 3");

// Blueprint: Empty string
UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="Default")
FString MyEmptyString;
```

## Enum Type Default Values

### 1. Basic Enum Defaults

**File:** `/Engine/Source/Runtime/CoreUObject/Public/UObject/EnumProperty.h`

**FEnumProperty Default Handling:**
```cpp
void ApplyEnumDefaults(FEnumProperty* Property, void* ContainerPtr, const FString& DefaultValue)
{
    UEnum* Enum = Property->GetEnum();
    
    int64 EnumValue = INDEX_NONE;
    
    if (DefaultValue.IsEmpty())
    {
        // Use first enum value as default
        EnumValue = Enum->GetValueByIndex(0);
    }
    else
    {
        // Try multiple resolution strategies
        EnumValue = ResolveEnumValue(Enum, DefaultValue);
    }
    
    if (EnumValue != INDEX_NONE)
    {
        Property->GetUnderlyingProperty()->SetIntPropertyValue(ContainerPtr, EnumValue);
    }
}

int64 ResolveEnumValue(UEnum* Enum, const FString& ValueString)
{
    // Strategy 1: Direct name lookup
    int64 Value = Enum->GetValueByNameString(ValueString);
    if (Value != INDEX_NONE) return Value;
    
    // Strategy 2: Short name lookup (remove enum prefix)
    FString ShortName = ValueString;
    FString EnumPrefix = Enum->GenerateEnumPrefix();
    if (ShortName.StartsWith(EnumPrefix))
    {
        ShortName.RemoveFromStart(EnumPrefix);
        Value = Enum->GetValueByNameString(ShortName);
        if (Value != INDEX_NONE) return Value;
    }
    
    // Strategy 3: Display name lookup
    for (int32 Index = 0; Index < Enum->NumEnums(); ++Index)
    {
        if (Enum->GetDisplayNameTextByIndex(Index).ToString() == ValueString)
        {
            return Enum->GetValueByIndex(Index);
        }
    }
    
    // Strategy 4: Integer value parsing
    int64 IntValue;
    if (LexFromString(IntValue, *ValueString))
    {
        if (Enum->IsValidEnumValue(IntValue))
        {
            return IntValue;
        }
    }
    
    return INDEX_NONE; // No valid resolution found
}
```

**C++ Generation for Enum Defaults:**
```cpp
// Blueprint enum with default value
UENUM(BlueprintType)
enum class EMyEnum : uint8
{
    Value1,
    Value2,
    Value3
};

// Blueprint: EMyEnum with default Value2
UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="Default")
EMyEnum MyEnum = EMyEnum::Value2;

// Blueprint: Enum with explicit integer value
UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="Default") 
EMyEnum MyExplicitEnum = static_cast<EMyEnum>(1); // Value2
```

## GetCPPMacroType for Default Value Generation

### 1. FProperty::GetCPPMacroType Analysis

**Purpose:** Generates C++ macro type strings for UPROPERTY declarations

**Pattern Analysis:**
```cpp
// From EnumProperty.cpp
FString FEnumProperty::GetCPPMacroType(FString& ExtendedTypeText) const
{
    ExtendedTypeText = Enum->GetName();
    return TEXT("ENUM");
}

// Pattern for primitive types
FString FIntProperty::GetCPPMacroType(FString& ExtendedTypeText) const
{
    ExtendedTypeText = TEXT("");
    return TEXT("INT");
}

FString FFloatProperty::GetCPPMacroType(FString& ExtendedTypeText) const  
{
    ExtendedTypeText = TEXT("");
    return TEXT("FLOAT");
}

FString FBoolProperty::GetCPPMacroType(FString& ExtendedTypeText) const
{
    ExtendedTypeText = TEXT("");
    return TEXT("UBOOL");
}
```

**Usage in C++ Generation:**
```cpp
void GenerateUPROPERTYMacro(FProperty* Property, const FString& DefaultValue)
{
    FString ExtendedType;
    FString MacroType = Property->GetCPPMacroType(ExtendedType);
    
    FString UPropertyMacro = FString::Printf(TEXT("UPROPERTY(%s)"), 
        GeneratePropertyMeta(Property).Join(TEXT(", ")));
    
    FString CPPType = Property->GetCPPType(&ExtendedType, CPPF_None);
    
    FString Declaration = FString::Printf(TEXT("%s %s"), 
        *CPPType, *Property->GetName());
    
    if (!DefaultValue.IsEmpty())
    {
        Declaration += FString::Printf(TEXT(" = %s"), *DefaultValue);
    }
    
    Declaration += TEXT(";");
    
    // Output: UPROPERTY(...) int32 MyProperty = 42;
}
```

## ExportTextItem/ImportTextItem for Default Serialization

### 1. Text Export/Import Interface

**Core Export Interface:**
```cpp
void ExportTextItem_Direct(FString& ValueStr, const void* PropertyValue, const void* DefaultValue, 
                          UObject* Parent, int32 PortFlags, UObject* ExportRootScope = nullptr) const
{
    ExportText_Internal(ValueStr, PropertyValue, EPropertyPointerType::Direct, 
                       DefaultValue, Parent, PortFlags, ExportRootScope);
}

void ExportTextItem_InContainer(FString& ValueStr, const void* Container, const void* DefaultValue, 
                               UObject* Parent, int32 PortFlags, UObject* ExportRootScope = nullptr) const
{
    ExportText_Internal(ValueStr, Container, EPropertyPointerType::Container, 
                       DefaultValue, Parent, PortFlags, ExportRootScope);
}
```

**Core Import Interface:**
```cpp
const TCHAR* ImportText_Direct(const TCHAR* Buffer, void* PropertyPtr, UObject* OwnerObject, 
                              int32 PortFlags, FOutputDevice* ErrorText = nullptr) const
{
    if (!ValidateImportFlags(PortFlags, ErrorText) || Buffer == nullptr)
    {
        return nullptr;
    }
    
    PortFlags |= EPropertyPortFlags::PPF_UseDeprecatedProperties;
    return ImportText_Internal(Buffer, PropertyPtr, EPropertyPointerType::Direct, 
                              OwnerObject, PortFlags, ErrorText);
}

const TCHAR* ImportText_InContainer(const TCHAR* Buffer, void* Container, UObject* OwnerObject, 
                                   int32 PortFlags, FOutputDevice* ErrorText = nullptr) const
{
    if (!ValidateImportFlags(PortFlags, ErrorText) || Buffer == nullptr)
    {
        return nullptr;
    }
    
    PortFlags |= EPropertyPortFlags::PPF_UseDeprecatedProperties;
    return ImportText_Internal(Buffer, Container, EPropertyPointerType::Container, 
                              OwnerObject, PortFlags, ErrorText);
}
```

### 2. Default Value Serialization Patterns

**Export with Default Comparison:**
```cpp
void ExportPropertyWithDefaults(FProperty* Property, const void* Object, const void* DefaultObject, 
                               FString& OutText, int32 PortFlags)
{
    void* PropertyValue = Property->ContainerPtrToValuePtr<void>(Object);
    void* DefaultValue = DefaultObject ? Property->ContainerPtrToValuePtr<void>(DefaultObject) : nullptr;
    
    // Only export if different from default
    if (!DefaultValue || !Property->Identical(PropertyValue, DefaultValue, PPF_DeepComparison))
    {
        FString ValueText;
        Property->ExportTextItem_Direct(ValueText, PropertyValue, DefaultValue, 
                                       nullptr, PortFlags, nullptr);
        
        OutText += FString::Printf(TEXT("%s=%s\n"), *Property->GetName(), *ValueText);
    }
}
```

**Import with Default Fallback:**
```cpp
bool ImportPropertyWithDefaults(FProperty* Property, const TCHAR* Buffer, void* Object, 
                               const void* DefaultObject, FOutputDevice* ErrorText)
{
    void* PropertyValue = Property->ContainerPtrToValuePtr<void>(Object);
    
    // Try to import from buffer
    const TCHAR* Result = Property->ImportText_Direct(Buffer, PropertyValue, nullptr, 
                                                     PPF_ParsingDefaultProperties, ErrorText);
    
    if (!Result && DefaultObject)
    {
        // Fallback to copying from default
        void* DefaultValue = Property->ContainerPtrToValuePtr<void>(DefaultObject);
        Property->CopyCompleteValue(PropertyValue, DefaultValue);
        return true;
    }
    
    return Result != nullptr;
}
```

## JSON Schema for Default Value Storage

### Default Values Data Structure
```json
{
    "PrimitiveDefaults": [
        {
            "PropertyName": "MyInteger",
            "PropertyType": "int32",
            "CPPType": "int32",
            "MacroType": "INT",
            "DefaultValue": "42",
            "DefaultExpression": "42",
            "RequiresQuotes": false,
            "ValidationRules": {
                "MinValue": -2147483648,
                "MaxValue": 2147483647
            }
        },
        {
            "PropertyName": "MyFloat",
            "PropertyType": "float",
            "CPPType": "float",
            "MacroType": "FLOAT", 
            "DefaultValue": "3.14159",
            "DefaultExpression": "3.14159f",
            "RequiresQuotes": false,
            "SpecialValues": ["inf", "-inf", "nan"]
        },
        {
            "PropertyName": "MyBool",
            "PropertyType": "bool",
            "CPPType": "bool",
            "MacroType": "UBOOL",
            "DefaultValue": "true",
            "DefaultExpression": "true",
            "RequiresQuotes": false,
            "AcceptedFormats": ["true", "false", "1", "0", "yes", "no", "on", "off"]
        },
        {
            "PropertyName": "MyString",
            "PropertyType": "FString",
            "CPPType": "FString", 
            "MacroType": "STR",
            "DefaultValue": "Hello World",
            "DefaultExpression": "TEXT(\"Hello World\")",
            "RequiresQuotes": true,
            "EscapeSequences": ["\\n", "\\r", "\\t", "\\\"", "\\\\"]
        },
        {
            "PropertyName": "MyName",
            "PropertyType": "FName",
            "CPPType": "FName",
            "MacroType": "NAME",
            "DefaultValue": "MyName",
            "DefaultExpression": "TEXT(\"MyName\")",
            "RequiresQuotes": true,
            "SpecialValues": ["None", "NAME_None"]
        },
        {
            "PropertyName": "MyEnum",
            "PropertyType": "EMyEnum",
            "CPPType": "EMyEnum",
            "MacroType": "ENUM",
            "DefaultValue": "Value2",
            "DefaultExpression": "EMyEnum::Value2",
            "RequiresQuotes": false,
            "EnumClass": "EMyEnum",
            "ResolutionStrategies": [
                "FullName",
                "ShortName", 
                "DisplayName",
                "IntegerValue"
            ]
        }
    ]
}
```

## Blueprint to C++ Default Conversion Rules

### 1. Inline Initialization Rules

**Safe for inline initialization:**
```cpp
// Primitive types with simple defaults
int32 MyInt = 42;
float MyFloat = 3.14f;
bool bMyBool = true;
FString MyString = TEXT("Simple");
FName MyName = TEXT("SimpleName");
EMyEnum MyEnum = EMyEnum::Value1;
```

**Requires constructor initialization:**
```cpp
// Complex expressions or validation
AMyActor::AMyActor()
{
    // Range-validated values
    MyClampedInt = FMath::Clamp(42, MinValue, MaxValue);
    
    // Special float values
    MySpecialFloat = TNumericLimits<float>::Max();
    
    // Complex string processing
    MyProcessedString = ProcessStringWithEscapes(TEXT("Complex\\nString"));
    
    // Enum with complex resolution
    MyComplexEnum = ResolveEnumFromString(TEXT("DisplayName"));
}
```

### 2. Type-Specific Conversion Patterns

**Integer Type Conversions:**
```cpp
// Blueprint int32 with validation
int32 MyValidatedInt = 42; // Simple case
// OR in constructor:
MyValidatedInt = FMath::Clamp(42, GetMinValue(), GetMaxValue());
```

**Float Type Conversions:**
```cpp
// Blueprint float with special handling
float MyFloat = 3.14f; // Simple case
float MyInfiniteFloat = TNumericLimits<float>::Max(); // Special values
```

**Boolean Conversions:**
```cpp
// Blueprint bool
bool bMyBool = true; // Direct conversion

// Blueprint bitfield bool (requires constructor)
AMyActor::AMyActor()
{
    bMyBitfield = 1; // Bitfield assignment
}
```

**String Conversions:**
```cpp
// Blueprint FString
FString MyString = TEXT("Value"); // Simple strings
// Complex strings in constructor:
MyComplexString = TEXT("Line1\nLine2\nLine3");
```

**Name Conversions:**
```cpp
// Blueprint FName
FName MyName = TEXT("Value"); // Simple names
FName MyEmptyName = NAME_None; // Special None value
```

**Enum Conversions:**
```cpp
// Blueprint enum
EMyEnum MyEnum = EMyEnum::Value2; // Standard case
EMyEnum MyIntEnum = static_cast<EMyEnum>(2); // Integer-based
```

## Conclusion

Primitive type default value resolution requires:

1. **Property-specific parsing** for each primitive type
2. **Multiple resolution strategies** for enums and special values  
3. **Proper C++ expression generation** with correct syntax
4. **Validation and range checking** during value assignment
5. **Constructor vs. inline initialization** decision logic
6. **Text serialization** support via ExportText/ImportText

This foundation enables accurate Blueprint to C++ conversion while preserving all default value semantics and validation rules.