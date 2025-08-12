# Wildcard Pin Type Resolution and Default Values Analysis for Blueprint to C++ Conversion

## Overview

This document analyzes the wildcard pin type resolution system and polymorphic default value handling in Unreal Engine, focusing on how Blueprint wildcard pins and their resolved types translate to C++. This is critical for the Blueprint to C++ conversion system as wildcard pins can resolve to different concrete types based on context.

## Core Wildcard Type System

### 1. Blueprint Type Promotion Infrastructure

**File:** `/Engine/Source/Editor/BlueprintGraph/Classes/BlueprintTypePromotion.h`

**Core Wildcard Resolution System:**
```cpp
class BLUEPRINTGRAPH_API FTypePromotion : private FNoncopyable
{
public:
    /** 
     * Determine what type a given set of wildcard pins would result in
     * @return Pin type that is the "highest" of all the given pins
     */
    static FEdGraphPinType GetPromotedType(const TArray<UEdGraphPin*>& WildcardPins);
    
    /** Type comparison results for promotion */
    enum class ETypeComparisonResult : uint8
    {
        TypeAHigher,    // Type A promotes over Type B
        TypeBHigher,    // Type B promotes over Type A  
        TypesEqual,     // Types are equivalent
        InvalidComparison // Cannot compare types
    };
    
    /** Check which pin type is higher in promotion hierarchy */
    static ETypeComparisonResult GetHigherType(const FEdGraphPinType& A, const FEdGraphPinType& B);
    
    /** Check if A can be promoted to type B correctly */
    static bool IsValidPromotion(const FEdGraphPinType& A, const FEdGraphPinType& B);
    
    /** Find best matching function for operation given pins to consider */
    static UFunction* FindBestMatchingFunc(FName Operation, const TArray<UEdGraphPin*>& PinsToConsider);
};
```

### 2. Wildcard Pin Type Resolution Process

**Wildcard Resolution Algorithm:**
```cpp
FEdGraphPinType ResolveWildcardPinType(const TArray<UEdGraphPin*>& ConnectedPins)
{
    TArray<UEdGraphPin*> TypedPins;
    TArray<UEdGraphPin*> WildcardPins;
    
    // Separate typed pins from wildcard pins
    for (UEdGraphPin* Pin : ConnectedPins)
    {
        if (Pin->PinType.PinCategory == UEdGraphSchema_K2::PC_Wildcard)
        {
            WildcardPins.Add(Pin);
        }
        else
        {
            TypedPins.Add(Pin);
        }
    }
    
    FEdGraphPinType ResolvedType;
    
    if (TypedPins.Num() > 0)
    {
        // Resolve to highest typed pin
        ResolvedType = TypedPins[0]->PinType;
        
        for (int32 i = 1; i < TypedPins.Num(); ++i)
        {
            FTypePromotion::ETypeComparisonResult Result = 
                FTypePromotion::GetHigherType(ResolvedType, TypedPins[i]->PinType);
            
            if (Result == FTypePromotion::ETypeComparisonResult::TypeBHigher)
            {
                ResolvedType = TypedPins[i]->PinType;
            }
        }
    }
    else if (WildcardPins.Num() > 0)
    {
        // No typed pins, resolve using type promotion system
        ResolvedType = FTypePromotion::GetPromotedType(WildcardPins);
    }
    
    // Apply resolved type to all wildcard pins
    for (UEdGraphPin* WildcardPin : WildcardPins)
    {
        WildcardPin->PinType = ResolvedType;
    }
    
    return ResolvedType;
}
```

### 3. Type Promotion Hierarchy

**Primitive Type Promotion Table:**
```cpp
TMap<FName, TArray<FName>> CreatePrimitivePromotionTable()
{
    TMap<FName, TArray<FName>> PromotionTable;
    
    // Numeric promotion hierarchy: byte -> int -> int64 -> float -> double
    PromotionTable.Add(UEdGraphSchema_K2::PC_Byte, {
        UEdGraphSchema_K2::PC_Int,
        UEdGraphSchema_K2::PC_Int64,  
        UEdGraphSchema_K2::PC_Float,
        UEdGraphSchema_K2::PC_Double
    });
    
    PromotionTable.Add(UEdGraphSchema_K2::PC_Int, {
        UEdGraphSchema_K2::PC_Int64,
        UEdGraphSchema_K2::PC_Float,
        UEdGraphSchema_K2::PC_Double
    });
    
    PromotionTable.Add(UEdGraphSchema_K2::PC_Int64, {
        UEdGraphSchema_K2::PC_Float,
        UEdGraphSchema_K2::PC_Double
    });
    
    PromotionTable.Add(UEdGraphSchema_K2::PC_Float, {
        UEdGraphSchema_K2::PC_Double
    });
    
    // Vector types promotion
    PromotionTable.Add(UEdGraphSchema_K2::PC_Struct, {});
    
    return PromotionTable;
}
```

**Type Comparison Implementation:**
```cpp
FTypePromotion::ETypeComparisonResult GetHigherType_Internal(
    const FEdGraphPinType& A, const FEdGraphPinType& B) const
{
    // Same category and subcategory
    if (A.PinCategory == B.PinCategory && A.PinSubCategory == B.PinSubCategory)
    {
        return ETypeComparisonResult::TypesEqual;
    }
    
    // Check primitive promotion table
    const TArray<FName>* PromotionsFromA = PromotionTable.Find(A.PinCategory);
    if (PromotionsFromA && PromotionsFromA->Contains(B.PinCategory))
    {
        return ETypeComparisonResult::TypeBHigher; // B is higher than A
    }
    
    const TArray<FName>* PromotionsFromB = PromotionTable.Find(B.PinCategory);
    if (PromotionsFromB && PromotionsFromB->Contains(A.PinCategory))
    {
        return ETypeComparisonResult::TypeAHigher; // A is higher than B
    }
    
    // Special case handling for structs
    if (A.PinCategory == UEdGraphSchema_K2::PC_Struct && B.PinCategory == UEdGraphSchema_K2::PC_Struct)
    {
        return CompareStructTypes(A, B);
    }
    
    // No valid promotion path
    return ETypeComparisonResult::InvalidComparison;
}

FTypePromotion::ETypeComparisonResult CompareStructTypes(
    const FEdGraphPinType& A, const FEdGraphPinType& B) const
{
    UScriptStruct* StructA = Cast<UScriptStruct>(A.PinSubCategoryObject.Get());
    UScriptStruct* StructB = Cast<UScriptStruct>(B.PinSubCategoryObject.Get());
    
    if (!StructA || !StructB)
    {
        return ETypeComparisonResult::InvalidComparison;
    }
    
    // Check for struct inheritance
    if (StructA->IsChildOf(StructB))
    {
        return ETypeComparisonResult::TypeAHigher; // A inherits from B
    }
    
    if (StructB->IsChildOf(StructA))
    {
        return ETypeComparisonResult::TypeBHigher; // B inherits from A
    }
    
    // Check for specific numeric struct promotions (Vector2D -> Vector -> Vector4)
    return CompareNumericStructTypes(StructA, StructB);
}
```

## Wildcard Default Value Resolution

### 1. Wildcard Pin Default Values

**Wildcard Pin Default Value System:**
```cpp
class UEdGraphPin
{
public:
    /** The default value for this pin */
    FString DefaultValue;
    
    /** Default object for object/class pins */
    TObjectPtr<UObject> DefaultObject;
    
    /** Default text value for text pins */
    FText DefaultTextValue;
    
    /** Pin type that may be wildcard initially */
    FEdGraphPinType PinType;
};

void ResolveWildcardPinDefault(UEdGraphPin* WildcardPin, const FEdGraphPinType& ResolvedType)
{
    FEdGraphPinType OldType = WildcardPin->PinType;
    WildcardPin->PinType = ResolvedType;
    
    // Convert default value if type changed
    if (OldType.PinCategory != ResolvedType.PinCategory)
    {
        ConvertWildcardDefaultValue(WildcardPin, OldType, ResolvedType);
    }
}

void ConvertWildcardDefaultValue(UEdGraphPin* Pin, const FEdGraphPinType& FromType, const FEdGraphPinType& ToType)
{
    if (Pin->DefaultValue.IsEmpty())
    {
        // Set default value for target type
        Pin->DefaultValue = GetDefaultValueForType(ToType);
        return;
    }
    
    // Attempt type conversion
    FString ConvertedValue;
    if (ConvertDefaultValueBetweenTypes(Pin->DefaultValue, FromType, ToType, ConvertedValue))
    {
        Pin->DefaultValue = ConvertedValue;
    }
    else
    {
        // Conversion failed, use target type default
        Pin->DefaultValue = GetDefaultValueForType(ToType);
        
        UE_LOG(LogBlueprint, Warning, TEXT("Failed to convert wildcard default value from %s to %s, using type default"),
            *UEdGraphSchema_K2::TypeToText(FromType).ToString(),
            *UEdGraphSchema_K2::TypeToText(ToType).ToString());
    }
}
```

### 2. Default Value Type Conversion

**Type-Safe Default Value Conversion:**
```cpp
bool ConvertDefaultValueBetweenTypes(const FString& SourceValue, 
                                   const FEdGraphPinType& SourceType,
                                   const FEdGraphPinType& TargetType,
                                   FString& OutConvertedValue)
{
    // Handle numeric conversions
    if (IsNumericType(SourceType) && IsNumericType(TargetType))
    {
        return ConvertNumericDefaultValue(SourceValue, SourceType, TargetType, OutConvertedValue);
    }
    
    // Handle string conversions
    if (IsStringType(SourceType) && IsStringType(TargetType))
    {
        return ConvertStringDefaultValue(SourceValue, SourceType, TargetType, OutConvertedValue);
    }
    
    // Handle struct conversions
    if (SourceType.PinCategory == UEdGraphSchema_K2::PC_Struct && 
        TargetType.PinCategory == UEdGraphSchema_K2::PC_Struct)
    {
        return ConvertStructDefaultValue(SourceValue, SourceType, TargetType, OutConvertedValue);
    }
    
    // Handle object conversions
    if (IsObjectType(SourceType) && IsObjectType(TargetType))
    {
        return ConvertObjectDefaultValue(SourceValue, SourceType, TargetType, OutConvertedValue);
    }
    
    return false; // No conversion available
}

bool ConvertNumericDefaultValue(const FString& SourceValue,
                               const FEdGraphPinType& SourceType,
                               const FEdGraphPinType& TargetType,
                               FString& OutConvertedValue)
{
    // Parse source value
    double NumericValue = 0.0;
    
    if (SourceType.PinCategory == UEdGraphSchema_K2::PC_Byte)
    {
        uint8 ByteValue = 0;
        LexFromString(ByteValue, *SourceValue);
        NumericValue = static_cast<double>(ByteValue);
    }
    else if (SourceType.PinCategory == UEdGraphSchema_K2::PC_Int)
    {
        int32 IntValue = 0;
        LexFromString(IntValue, *SourceValue);
        NumericValue = static_cast<double>(IntValue);
    }
    else if (SourceType.PinCategory == UEdGraphSchema_K2::PC_Int64)
    {
        int64 Int64Value = 0;
        LexFromString(Int64Value, *SourceValue);
        NumericValue = static_cast<double>(Int64Value);
    }
    else if (SourceType.PinCategory == UEdGraphSchema_K2::PC_Float)
    {
        float FloatValue = 0.0f;
        LexFromString(FloatValue, *SourceValue);
        NumericValue = static_cast<double>(FloatValue);
    }
    else if (SourceType.PinCategory == UEdGraphSchema_K2::PC_Double)
    {
        LexFromString(NumericValue, *SourceValue);
    }
    
    // Convert to target type
    if (TargetType.PinCategory == UEdGraphSchema_K2::PC_Byte)
    {
        uint8 ByteValue = static_cast<uint8>(FMath::Clamp(NumericValue, 0.0, 255.0));
        OutConvertedValue = LexToString(ByteValue);
    }
    else if (TargetType.PinCategory == UEdGraphSchema_K2::PC_Int)
    {
        int32 IntValue = static_cast<int32>(NumericValue);
        OutConvertedValue = LexToString(IntValue);
    }
    else if (TargetType.PinCategory == UEdGraphSchema_K2::PC_Int64)
    {
        int64 Int64Value = static_cast<int64>(NumericValue);
        OutConvertedValue = LexToString(Int64Value);
    }
    else if (TargetType.PinCategory == UEdGraphSchema_K2::PC_Float)
    {
        float FloatValue = static_cast<float>(NumericValue);
        OutConvertedValue = LexToString(FloatValue);
    }
    else if (TargetType.PinCategory == UEdGraphSchema_K2::PC_Double)
    {
        OutConvertedValue = LexToString(NumericValue);
    }
    else
    {
        return false;
    }
    
    return true;
}
```

### 3. Struct Type Default Conversion

**Vector/Transform Struct Conversions:**
```cpp
bool ConvertStructDefaultValue(const FString& SourceValue,
                              const FEdGraphPinType& SourceType,
                              const FEdGraphPinType& TargetType,
                              FString& OutConvertedValue)
{
    UScriptStruct* SourceStruct = Cast<UScriptStruct>(SourceType.PinSubCategoryObject.Get());
    UScriptStruct* TargetStruct = Cast<UScriptStruct>(TargetType.PinSubCategoryObject.Get());
    
    if (!SourceStruct || !TargetStruct)
    {
        return false;
    }
    
    // Handle Vector2D -> Vector conversion
    if (SourceStruct == TBaseStructure<FVector2D>::Get() && 
        TargetStruct == TBaseStructure<FVector>::Get())
    {
        return ConvertVector2DToVector(SourceValue, OutConvertedValue);
    }
    
    // Handle Vector -> Vector4 conversion  
    if (SourceStruct == TBaseStructure<FVector>::Get() &&
        TargetStruct == TBaseStructure<FVector4>::Get())
    {
        return ConvertVectorToVector4(SourceValue, OutConvertedValue);
    }
    
    // Handle Vector2D -> Vector4 conversion
    if (SourceStruct == TBaseStructure<FVector2D>::Get() &&
        TargetStruct == TBaseStructure<FVector4>::Get())
    {
        return ConvertVector2DToVector4(SourceValue, OutConvertedValue);
    }
    
    // Handle Rotator -> Quat conversion
    if (SourceStruct == TBaseStructure<FRotator>::Get() &&
        TargetStruct == TBaseStructure<FQuat>::Get())
    {
        return ConvertRotatorToQuat(SourceValue, OutConvertedValue);
    }
    
    return false;
}

bool ConvertVector2DToVector(const FString& SourceValue, FString& OutConvertedValue)
{
    // Parse Vector2D: X=1.0,Y=2.0
    FVector2D Vector2D;
    if (Vector2D.InitFromString(SourceValue))
    {
        FVector Vector(Vector2D.X, Vector2D.Y, 0.0f);
        OutConvertedValue = Vector.ToString();
        return true;
    }
    return false;
}

bool ConvertVectorToVector4(const FString& SourceValue, FString& OutConvertedValue)
{
    // Parse Vector: X=1.0,Y=2.0,Z=3.0
    FVector Vector;
    if (Vector.InitFromString(SourceValue))
    {
        FVector4 Vector4(Vector.X, Vector.Y, Vector.Z, 1.0f);
        OutConvertedValue = Vector4.ToString();
        return true;
    }
    return false;
}
```

## Wildcard C++ Code Generation

### 1. Template Function Generation

**Wildcard Template Resolution:**
```cpp
struct FWildcardResolution
{
    FEdGraphPinType ResolvedType;
    TArray<UEdGraphPin*> AffectedPins;
    FString TemplateSpecialization;
    
    // Generate template specialization for resolved type
    FString GenerateTemplateCode() const
    {
        if (ResolvedType.PinCategory == UEdGraphSchema_K2::PC_Int)
        {
            return TEXT("int32");
        }
        else if (ResolvedType.PinCategory == UEdGraphSchema_K2::PC_Float)
        {
            return TEXT("float");
        }
        else if (ResolvedType.PinCategory == UEdGraphSchema_K2::PC_Struct)
        {
            UScriptStruct* Struct = Cast<UScriptStruct>(ResolvedType.PinSubCategoryObject.Get());
            return Struct ? Struct->GetName() : TEXT("void");
        }
        
        return TEXT("auto"); // Fallback
    }
};

FString GenerateWildcardFunctionCall(UFunction* Function, 
                                   const TArray<FWildcardResolution>& Resolutions)
{
    FString FunctionName = Function->GetName();
    
    // Check if function requires template specialization
    if (RequiresTemplateSpecialization(Function))
    {
        FString TemplateArgs;
        for (int32 i = 0; i < Resolutions.Num(); ++i)
        {
            if (i > 0) TemplateArgs += TEXT(", ");
            TemplateArgs += Resolutions[i].GenerateTemplateCode();
        }
        
        return FString::Printf(TEXT("%s<%s>"), *FunctionName, *TemplateArgs);
    }
    
    return FunctionName;
}
```

### 2. Operator Overload Resolution

**Math Operation Wildcard Resolution:**
```cpp
struct FMathOperatorResolution
{
    FName OperatorName; // Add, Multiply, etc.
    FEdGraphPinType ResolvedType;
    UFunction* ResolvedFunction;
    
    FString GenerateCPPOperator() const
    {
        // Direct operator mapping
        if (OperatorName == TEXT("Add"))
        {
            return TEXT("+");
        }
        else if (OperatorName == TEXT("Subtract"))
        {
            return TEXT("-");
        }
        else if (OperatorName == TEXT("Multiply"))
        {
            return TEXT("*");
        }
        else if (OperatorName == TEXT("Divide"))
        {
            return TEXT("/");
        }
        
        // Function call fallback
        return ResolvedFunction ? ResolvedFunction->GetName() : TEXT("UnknownOp");
    }
    
    FString GenerateOperatorCall(const FString& LeftOperand, const FString& RightOperand) const
    {
        FString Operator = GenerateCPPOperator();
        
        if (Operator.Len() == 1) // Simple operator
        {
            return FString::Printf(TEXT("(%s %s %s)"), *LeftOperand, *Operator, *RightOperand);
        }
        else // Function call
        {
            return FString::Printf(TEXT("%s(%s, %s)"), *Operator, *LeftOperand, *RightOperand);
        }
    }
};
```

### 3. Container Wildcard Resolution

**Template Container Resolution:**
```cpp
struct FContainerWildcardResolution
{
    enum class EContainerType
    {
        Array,
        Set,
        Map
    };
    
    EContainerType ContainerType;
    FEdGraphPinType ElementType;
    FEdGraphPinType KeyType; // For maps
    
    FString GenerateContainerType() const
    {
        FString ElementTypeName = GenerateTypeName(ElementType);
        
        switch (ContainerType)
        {
        case EContainerType::Array:
            return FString::Printf(TEXT("TArray<%s>"), *ElementTypeName);
            
        case EContainerType::Set:
            return FString::Printf(TEXT("TSet<%s>"), *ElementTypeName);
            
        case EContainerType::Map:
            {
                FString KeyTypeName = GenerateTypeName(KeyType);
                return FString::Printf(TEXT("TMap<%s, %s>"), *KeyTypeName, *ElementTypeName);
            }
            
        default:
            return TEXT("TArray<int32>"); // Safe fallback
        }
    }
    
private:
    FString GenerateTypeName(const FEdGraphPinType& PinType) const
    {
        if (PinType.PinCategory == UEdGraphSchema_K2::PC_Int)
        {
            return TEXT("int32");
        }
        else if (PinType.PinCategory == UEdGraphSchema_K2::PC_Float)
        {
            return TEXT("float");
        }
        else if (PinType.PinCategory == UEdGraphSchema_K2::PC_String)
        {
            return TEXT("FString");
        }
        else if (PinType.PinCategory == UEdGraphSchema_K2::PC_Struct)
        {
            UScriptStruct* Struct = Cast<UScriptStruct>(PinType.PinSubCategoryObject.Get());
            return Struct ? Struct->GetName() : TEXT("void");
        }
        
        return TEXT("int32"); // Safe fallback
    }
};
```

## JSON Schema for Wildcard Resolution

### Wildcard Resolution Data Structure
```json
{
    "WildcardResolutions": [
        {
            "NodeId": "12345",
            "NodeType": "K2Node_CallFunction",
            "FunctionName": "Add",
            "WildcardPins": [
                {
                    "PinId": "Pin1",
                    "PinName": "A",
                    "OriginalType": "wildcard",
                    "ResolvedType": {
                        "PinCategory": "float",
                        "PinSubCategory": "",
                        "CPPType": "float"
                    },
                    "DefaultValue": "0.0",
                    "ConvertedDefaultValue": "0.0f"
                },
                {
                    "PinId": "Pin2", 
                    "PinName": "B",
                    "OriginalType": "wildcard",
                    "ResolvedType": {
                        "PinCategory": "float",
                        "PinSubCategory": "",
                        "CPPType": "float"
                    },
                    "DefaultValue": "1.0",
                    "ConvertedDefaultValue": "1.0f"
                }
            ],
            "ResolutionStrategy": "TypePromotion",
            "ResolvedFunction": "Add_FloatFloat",
            "CPPGeneration": {
                "OperatorOverload": "+",
                "FunctionCall": "UKismetMathLibrary::Add_FloatFloat",
                "DirectExpression": "(%s + %s)"
            }
        },
        {
            "NodeId": "67890",
            "NodeType": "K2Node_CallArrayFunction",
            "FunctionName": "Array_Add",
            "WildcardPins": [
                {
                    "PinId": "ArrayPin",
                    "PinName": "TargetArray",
                    "OriginalType": "wildcard array",
                    "ResolvedType": {
                        "PinCategory": "array",
                        "PinSubCategory": "int",
                        "CPPType": "TArray<int32>"
                    },
                    "DefaultValue": "()",
                    "ConvertedDefaultValue": "{}"
                },
                {
                    "PinId": "ElementPin",
                    "PinName": "NewItem",
                    "OriginalType": "wildcard",
                    "ResolvedType": {
                        "PinCategory": "int",
                        "PinSubCategory": "",
                        "CPPType": "int32"
                    },
                    "DefaultValue": "0",
                    "ConvertedDefaultValue": "0"
                }
            ],
            "ResolutionStrategy": "ContainerElementType",
            "CPPGeneration": {
                "MemberFunction": "Add",
                "DirectExpression": "%s.Add(%s)"
            }
        }
    ],
    "TypePromotionHierarchy": {
        "NumericTypes": {
            "byte": ["int", "int64", "float", "double"],
            "int": ["int64", "float", "double"],
            "int64": ["float", "double"],
            "float": ["double"]
        },
        "StructTypes": {
            "Vector2D": ["Vector", "Vector4"],
            "Vector": ["Vector4"],
            "Rotator": ["Quat"]
        }
    }
}
```

## Advanced Wildcard Resolution Patterns

### 1. Function Template Specialization

```cpp
// Blueprint wildcard function becomes template in C++
template<typename T>
T AddValues(T A, T B)
{
    return A + B;
}

// Resolved calls based on wildcard resolution
int32 IntResult = AddValues<int32>(1, 2);
float FloatResult = AddValues<float>(1.0f, 2.0f);
FVector VectorResult = AddValues<FVector>(FVector::ForwardVector, FVector::RightVector);
```

### 2. Container Template Resolution

```cpp
// Blueprint wildcard array operations
template<typename T>
void AddToArray(TArray<T>& Array, const T& Element)
{
    Array.Add(Element);
}

// Resolved calls
TArray<int32> IntArray;
AddToArray(IntArray, 42);

TArray<FString> StringArray;
AddToArray(StringArray, TEXT("Hello"));
```

### 3. Operator Overload Resolution

```cpp
// Blueprint math operations with wildcard pins resolve to appropriate operators
// Wildcard Add node with int32 pins -> direct + operator
int32 Result = Value1 + Value2;

// Wildcard Add node with FVector pins -> vector addition
FVector Result = Vector1 + Vector2;

// Wildcard Add node with custom struct -> UFunction call
FCustomStruct Result = UMyMathLibrary::Add_CustomStructCustomStruct(Struct1, Struct2);
```

## Error Handling and Fallbacks

### 1. Invalid Resolution Handling

```cpp
bool ValidateWildcardResolution(const FWildcardResolution& Resolution)
{
    // Check if resolved type is valid
    if (Resolution.ResolvedType.PinCategory == UEdGraphSchema_K2::PC_Wildcard)
    {
        UE_LOG(LogBlueprint, Error, TEXT("Wildcard resolution failed: type still wildcard"));
        return false;
    }
    
    // Check if all connected pins can accept resolved type
    for (UEdGraphPin* Pin : Resolution.AffectedPins)
    {
        if (!CanAcceptPinType(Pin, Resolution.ResolvedType))
        {
            UE_LOG(LogBlueprint, Error, TEXT("Pin %s cannot accept resolved type %s"), 
                *Pin->PinName.ToString(),
                *UEdGraphSchema_K2::TypeToText(Resolution.ResolvedType).ToString());
            return false;
        }
    }
    
    return true;
}

FEdGraphPinType GetFallbackType(const FEdGraphPinType& WildcardType)
{
    // Safe fallback types for common wildcard scenarios
    if (WildcardType.ContainerType == EPinContainerType::Array)
    {
        FEdGraphPinType ArrayType;
        ArrayType.PinCategory = UEdGraphSchema_K2::PC_Int;
        ArrayType.ContainerType = EPinContainerType::Array;
        return ArrayType; // TArray<int32>
    }
    
    if (WildcardType.ContainerType == EPinContainerType::Set)
    {
        FEdGraphPinType SetType;
        SetType.PinCategory = UEdGraphSchema_K2::PC_Int;
        SetType.ContainerType = EPinContainerType::Set;
        return SetType; // TSet<int32>
    }
    
    if (WildcardType.ContainerType == EPinContainerType::Map)
    {
        FEdGraphPinType MapType;
        MapType.PinCategory = UEdGraphSchema_K2::PC_String;
        MapType.ContainerType = EPinContainerType::Map;
        MapType.PinValueType.TerminalCategory = UEdGraphSchema_K2::PC_Int;
        return MapType; // TMap<FString, int32>
    }
    
    // Default to int32 for simple wildcards
    FEdGraphPinType IntType;
    IntType.PinCategory = UEdGraphSchema_K2::PC_Int;
    return IntType;
}
```

### 2. Type Conversion Error Handling

```cpp
void HandleDefaultValueConversionFailure(UEdGraphPin* Pin, 
                                       const FEdGraphPinType& OriginalType,
                                       const FEdGraphPinType& TargetType)
{
    // Log the conversion failure
    UE_LOG(LogBlueprint, Warning, 
        TEXT("Failed to convert default value '%s' from %s to %s for pin %s"),
        *Pin->DefaultValue,
        *UEdGraphSchema_K2::TypeToText(OriginalType).ToString(),
        *UEdGraphSchema_K2::TypeToText(TargetType).ToString(),
        *Pin->PinName.ToString());
    
    // Set appropriate default for target type
    Pin->DefaultValue = GetDefaultValueForType(TargetType);
    
    // Clear invalid object references
    if (Pin->DefaultObject && !IsValidDefaultObject(Pin->DefaultObject, TargetType))
    {
        Pin->DefaultObject = nullptr;
    }
    
    // Clear invalid text values
    if (!Pin->DefaultTextValue.IsEmpty() && TargetType.PinCategory != UEdGraphSchema_K2::PC_Text)
    {
        Pin->DefaultTextValue = FText::GetEmpty();
    }
}
```

## Conclusion

Wildcard pin type resolution and default value handling requires:

1. **Type promotion hierarchy** understanding for automatic type selection
2. **Default value conversion** between compatible types during resolution
3. **Template specialization** generation for polymorphic functions
4. **Operator overload resolution** for math and comparison operations
5. **Container template resolution** for wildcard array/set/map operations
6. **Error handling and fallbacks** for invalid or failed resolutions

This system enables accurate Blueprint to C++ conversion even for dynamic wildcard pin scenarios, ensuring type safety and correct default value semantics in the generated C++ code.