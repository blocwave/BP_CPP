# UEdGraphSchema_K2 Core Validation System

## Overview
The UEdGraphSchema_K2 class serves as the primary validation and type system controller for Blueprint graphs. It enforces connection rules, type compatibility, and node placement restrictions essential for C++ code generation.

## Schema Constants and Categories

### Pin Categories (PC_*)
```cpp
static const FName PC_Exec;          // Execution flow pins
static const FName PC_Boolean;       // Boolean values
static const FName PC_Byte;          // Byte/uint8 values  
static const FName PC_Class;         // UClass references
static const FName PC_SoftClass;     // TSoftClassPtr references
static const FName PC_Int;           // Integer values
static const FName PC_Int64;         // 64-bit integer values
static const FName PC_Float;         // Float values
static const FName PC_Double;        // Double precision values
static const FName PC_Real;          // Real number (float/double)
static const FName PC_Name;          // FName values
static const FName PC_Delegate;      // Delegate signatures
static const FName PC_MCDelegate;    // Multicast delegate signatures
static const FName PC_Object;        // UObject references
static const FName PC_Interface;     // Interface references
static const FName PC_SoftObject;    // TSoftObjectPtr references
static const FName PC_String;        // FString values
static const FName PC_Text;          // FText values
static const FName PC_Struct;        // Struct values
static const FName PC_Wildcard;      // Wildcard matching pins
static const FName PC_Enum;          // Enum values
static const FName PC_FieldPath;     // Property field paths
```

### Pin Subcategories (PSC_*)
```cpp
static const FName PSC_Self;         // Self-reference category
static const FName PSC_Index;        // Index wildcard (Int, Bool, Byte, Enum)
static const FName PSC_Bitmask;      // Bitmask field representation
```

### Special Pin Names (PN_*)
```cpp
static const FName PN_Execute;       // Main execution input
static const FName PN_Then;          // Primary execution output
static const FName PN_Completed;     // Completion execution output
static const FName PN_Self;          // Self object input
static const FName PN_Else;          // Alternative execution output
static const FName PN_ReturnValue;   // Function return value
static const FName PN_Condition;     // Boolean condition input
// ... additional specialized pin names
```

## Core Validation Methods

### Pin Connection Validation
```cpp
virtual const FPinConnectionResponse CanCreateConnection(
    const UEdGraphPin* A, 
    const UEdGraphPin* B
) const override;

virtual bool ArePinsCompatible(
    const UEdGraphPin* PinA, 
    const UEdGraphPin* PinB, 
    const UClass* CallingContext = NULL, 
    bool bIgnoreArray = false
) const override;

virtual bool ArePinTypesCompatible(
    const FEdGraphPinType& Output, 
    const FEdGraphPinType& Input, 
    const UClass* CallingContext = NULL, 
    bool bIgnoreArray = false
) const;
```

### Type Promotion and Conversion
The schema handles automatic type promotion and conversion:

```cpp
virtual bool CreateAutomaticConversionNodeAndConnections(
    UEdGraphPin* A, 
    UEdGraphPin* B
) const override;

virtual bool CreatePromotedConnection(
    UEdGraphPin* A, 
    UEdGraphPin* B
) const override;

// Find autocast functions for type conversion
struct FSearchForAutocastFunctionResults {
    FName TargetFunction;
    UClass* FunctionOwner = nullptr;
};
UE_NODISCARD virtual TOptional<FSearchForAutocastFunctionResults> 
SearchForAutocastFunction(
    const FEdGraphPinType& OutputPinType, 
    const FEdGraphPinType& InputPinType
) const;
```

### Default Value Validation
```cpp
virtual FString IsPinDefaultValid(
    const UEdGraphPin* Pin, 
    const FString& NewDefaultValue, 
    UObject* NewDefaultObject, 
    const FText& InNewDefaultText
) const override;

virtual bool DefaultValueSimpleValidation(
    const FEdGraphPinType& PinType, 
    const FName PinName, 
    const FString& NewDefaultValue, 
    UObject* NewDefaultObject, 
    const FText& InText, 
    FString* OutMsg = nullptr
) const;
```

## Pin Type Tree System

### FPinTypeTreeInfo Structure
The schema maintains a hierarchical type tree for UI and validation:

```cpp
class FPinTypeTreeInfo {
private:
    FEdGraphPinType PinType;
    uint8 PossibleObjectReferenceTypes;
    FSoftObjectPath SubCategoryObjectAssetReference;
    FText CachedDescription;
    
public:
    TArray<TSharedPtr<FPinTypeTreeInfo>> Children;
    bool bReadOnly;
    FText FriendlyName;
    FText Tooltip;
    
    const FEdGraphPinType& GetPinType(bool bForceLoadedSubCategoryObject);
    FText GetDescription() const;
    uint8 GetPossibleObjectReferenceTypes() const;
};
```

### Type Tree Generation
```cpp
void GetVariableTypeTree(
    TArray<TSharedPtr<FPinTypeTreeInfo>>& TypeTree, 
    ETypeTreeFilter TypeTreeFilter = ETypeTreeFilter::None
) const;
```

## Blueprint Metadata Constants

### Function Metadata
```cpp
// Function behavior flags
static const FName MD_Latent;                    // Latent execution
static const FName MD_CustomThunk;               // Custom thunk implementation
static const FName MD_Variadic;                  // Variable arguments
static const FName MD_Protected;                 // Kismet protected access
static const FName MD_UnsafeForConstructionScripts; // UCS unsafe

// Display and categorization
static const FName MD_FunctionCategory;          // Palette category
static const FName MD_CompactNodeTitle;          // Compact display title
static const FName MD_DisplayName;               // Override display name
static const FName MD_FunctionKeywords;          // Search keywords

// Parameter handling
static const FName MD_AutoCreateRefTerm;         // Auto-create reference terms
static const FName MD_HidePin;                   // Hide specific pins
static const FName MD_InternalUseParam;          // Internal-only parameters
static const FName MD_WorldContext;              // World context parameter
```

### Property Metadata
```cpp
// Property exposure
static const FName MD_ExposeOnSpawn;             // Spawn node exposure
static const FName MD_PropertyGetFunction;       // Custom getter
static const FName MD_PropertySetFunction;       // Custom setter
static const FName MD_Private;                   // Blueprint private access

// Special property types
static const FName MD_Bitmask;                   // Bitmask property
static const FName MD_BitmaskEnum;               // Associated enum for bitmask
static const FName MD_DataTablePin;              // DataTable row selection
```

### Type System Metadata
```cpp
// Array and container handling
static const FName MD_ArrayParam;                // Array parameter
static const FName MD_ArrayDependentParam;       // Array-dependent parameter
static const FName MD_SetParam;                  // TSet parameter
static const FName MD_MapParam;                  // TMap parameter
static const FName MD_MapKeyParam;               // TMap key parameter
static const FName MD_MapValueParam;             // TMap value parameter

// Dynamic typing
static const FName MD_DynamicOutputType;         // Dynamic return type
static const FName MD_DynamicOutputParam;        // Dynamic output parameter
static const FName MD_CustomStructureParam;      // Custom struct parameter
```

## Validation Utilities

### Property Type Conversion
```cpp
static bool GetPropertyCategoryInfo(
    const FProperty* TestProperty, 
    FName& OutCategory, 
    FName& OutSubCategory, 
    UObject*& OutSubCategoryObject, 
    bool& bOutIsWeakPointer
);

bool ConvertPropertyToPinType(
    const FProperty* Property, 
    FEdGraphPinType& TypeOut
) const;
```

### Function Validation
```cpp
// Function capability checks
static bool CanUserKismetCallFunction(const UFunction* Function);
static bool CanKismetOverrideFunction(const UFunction* Function);
static bool FunctionCanBePlacedAsEvent(const UFunction* InFunction);
static bool FunctionCanBeUsedInDelegate(const UFunction* InFunction);
static bool HasFunctionAnyOutputParameter(const UFunction* Function);
static bool HasWildcardParams(const UFunction* Function);

// Context validation
bool CanFunctionBeUsedInGraph(
    const UClass* InClass, 
    const UFunction* InFunction, 
    const UEdGraph* InDestGraph, 
    uint32 InFunctionTypes, 
    bool bInCalledForEach, 
    FText* OutReason = nullptr
) const;
```

### Variable Type Validation
```cpp
// Type allowance checks
static bool IsAllowableBlueprintVariableType(const UEnum* InEnum);
static bool IsAllowableBlueprintVariableType(const UClass* InClass, bool bAssumeBlueprintType = false);
static bool IsAllowableBlueprintVariableType(const UScriptStruct *InStruct, bool bForInternalUse = false);

// Variable access validation
static bool CanUserKismetAccessVariable(
    const FProperty* Property, 
    const UClass* InClass, 
    EDelegateFilterMode FilterMode
);
```

## C++ Code Generation Implications

### Type Safety Enforcement
The schema validation ensures:
1. **Strong Typing**: All connections must be type-compatible or have valid conversion paths
2. **Const Correctness**: Const functions and parameters are properly validated
3. **Reference Semantics**: Proper handling of value vs reference semantics
4. **Container Types**: Correct array, set, and map type handling

### Code Generation Support
1. **Default Value Generation**: Validates and formats default values for C++ generation
2. **Function Signature Matching**: Ensures Blueprint signatures match C++ expectations
3. **Metadata Preservation**: Critical metadata is maintained for code generation
4. **Type Promotion**: Automatic type promotion rules match C++ behavior

### Error Prevention
1. **Circular Dependency Detection**: Prevents invalid dependency cycles
2. **Access Level Validation**: Enforces public/protected/private access rules
3. **Const Validation**: Prevents const violations in generated code
4. **Parameter Validation**: Ensures all parameters have valid types and defaults

## Best Practices for C++ Conversion

### Pin Type Mapping
1. **Execution Pins**: Map to control flow constructs
2. **Data Pins**: Map to typed parameters and return values
3. **Self Pins**: Map to `this` pointer or class context
4. **Wildcard Pins**: Require template or variant handling

### Validation Integration
1. **Pre-Conversion Validation**: Use schema validation before code generation
2. **Type Compatibility Checks**: Verify all connections are valid
3. **Default Value Validation**: Ensure all defaults are C++ compatible
4. **Metadata Processing**: Extract all relevant metadata for code generation

### Error Handling
1. **Connection Errors**: Report incompatible pin connections
2. **Type Errors**: Flag invalid type usage
3. **Default Value Errors**: Catch invalid default values
4. **Access Errors**: Report visibility and access violations