# Material Parameter Collection Function System

## Overview
The UK2Node_CallMaterialParameterCollectionFunction node provides specialized handling for material parameter collection functions. These nodes require special validation and code generation due to their integration with the rendering system and asset references.

## UK2Node_CallMaterialParameterCollectionFunction Structure

### Class Definition
```cpp
UCLASS(MinimalAPI)
class UK2Node_CallMaterialParameterCollectionFunction : public UK2Node_CallFunction {
    GENERATED_UCLASS_BODY()

    // EdGraphNode Interface
    virtual void PreloadRequiredAssets() override;
    virtual void PinDefaultValueChanged(UEdGraphPin* Pin) override;
    virtual void ValidateNodeDuringCompilation(class FCompilerResultsLog& MessageLog) const override;
};
```

### Key Characteristics
1. **Inherits from UK2Node_CallFunction**: Leverages standard function call infrastructure
2. **Asset Management**: Special handling for material parameter collection asset references
3. **Runtime Validation**: Additional validation for material system integration
4. **Parameter Type Safety**: Ensures parameter types match collection definitions

## Material Parameter Collection Integration

### Metadata Identification
Functions are identified as material parameter collection functions through metadata:
```cpp
// In FBlueprintMetadata constants
static const FName MD_MaterialParameterCollectionFunction;
```

Functions marked with this metadata receive special treatment:
- Asset preloading
- Parameter validation
- Runtime type checking
- Rendering system integration

### Asset Reference Management
```cpp
virtual void PreloadRequiredAssets() override;
```

The asset preloading system:
1. **Collection Asset Loading**: Ensures material parameter collection assets are loaded
2. **Parameter Definition Caching**: Caches parameter definitions for validation
3. **Dependency Tracking**: Tracks asset dependencies for proper loading order
4. **Editor Integration**: Provides asset picker functionality in editor

## Parameter Validation System

### Pin Default Value Validation
```cpp
virtual void PinDefaultValueChanged(UEdGraphPin* Pin) override;
```

When pin default values change:
1. **Parameter Name Validation**: Verify parameter names exist in collection
2. **Type Compatibility**: Check parameter types match collection definitions
3. **Value Range Validation**: Validate values are within acceptable ranges
4. **Asset Dependency Update**: Update asset references if collection changes

### Compilation Validation
```cpp
virtual void ValidateNodeDuringCompilation(class FCompilerResultsLog& MessageLog) const override;
```

Compilation-time checks include:
1. **Collection Asset Existence**: Verify referenced collections exist
2. **Parameter Existence**: Check all referenced parameters exist in collection
3. **Type Matching**: Ensure parameter types match usage
4. **Access Validation**: Verify parameters are accessible in current context
5. **Rendering Context**: Validate usage in appropriate rendering contexts

## Material Parameter Collection Functions

### Common Function Categories

#### Parameter Getters
```cpp
// Get scalar parameter from collection
GetScalarParameterValue(Collection, ParameterName) -> float

// Get vector parameter from collection  
GetVectorParameterValue(Collection, ParameterName) -> FVector4
```

#### Parameter Setters
```cpp
// Set scalar parameter in collection
SetScalarParameterValue(Collection, ParameterName, Value) -> void

// Set vector parameter in collection
SetVectorParameterValue(Collection, ParameterName, Value) -> void
```

#### Collection Management
```cpp
// Get parameter collection instance
GetParameterCollectionInstance(Collection) -> UMaterialParameterCollectionInstance*

// Check if parameter exists
HasParameter(Collection, ParameterName) -> bool
```

### Parameter Types
Material parameter collections support:
1. **Scalar Parameters**: Single float values
2. **Vector Parameters**: 4-component vectors (FVector4, FLinearColor)

### Function Signatures
Functions typically follow patterns:
```cpp
UFUNCTION(BlueprintCallable, meta = (MaterialParameterCollectionFunction = "true"))
static float GetScalarParameterValue(
    UMaterialParameterCollection* Collection,
    FName ParameterName
);

UFUNCTION(BlueprintCallable, meta = (MaterialParameterCollectionFunction = "true"))
static void SetScalarParameterValue(
    UMaterialParameterCollection* Collection,
    FName ParameterName,
    float ParameterValue
);
```

## Asset Management and References

### Collection Asset Handling
Material parameter collections are UObject assets that must be:
1. **Properly Referenced**: Hard references to prevent garbage collection
2. **Loaded at Runtime**: Available when functions execute
3. **Thread-Safe**: Safe for rendering thread access
4. **Editor Updated**: Reflect changes during development

### Parameter Name Resolution
Parameter names are resolved through:
1. **Asset Metadata**: Parameter definitions stored in collection asset
2. **Name Validation**: FName parameters validated against collection
3. **Type Information**: Parameter types extracted from collection metadata
4. **Default Values**: Default values provided by collection definitions

### Dependency Management
```cpp
virtual void PreloadRequiredAssets() override;
```

Asset dependencies include:
- **Primary Collection**: The main collection asset referenced
- **Parameter Definitions**: Metadata about available parameters
- **Type Information**: Runtime type data for parameters
- **Default Values**: Default parameter values from collection

## Validation and Error Handling

### Common Validation Errors
1. **Missing Collection**: Referenced collection asset not found
2. **Invalid Parameter**: Parameter name not found in collection
3. **Type Mismatch**: Parameter type doesn't match function expectation
4. **Access Violation**: Attempting to access parameters in invalid context
5. **Runtime Unavailability**: Collection not available at runtime

### Error Reporting
The validation system provides specific error messages:
```cpp
// Example validation errors
"Material Parameter Collection 'CollectionName' not found"
"Parameter 'ParameterName' does not exist in collection 'CollectionName'"
"Parameter 'ParameterName' is of type Scalar but Vector was expected"
"Material Parameter Collection functions require valid collection reference"
```

### Recovery Strategies
When validation fails:
1. **Asset Resolution**: Attempt to resolve missing assets
2. **Parameter Substitution**: Suggest alternative parameter names
3. **Type Coercion**: Attempt safe type conversions where possible
4. **Default Fallbacks**: Use default values when parameters unavailable

## C++ Code Generation Implications

### Function Call Generation
Material parameter collection functions generate specialized C++ calls:

```cpp
// Blueprint: GetScalarParameterValue(MyCollection, "Brightness")
// Generated C++:
float Result = 0.0f;
if (MyCollection)
{
    UMaterialParameterCollectionInstance* Instance = 
        UKismetRenderingLibrary::GetMaterialParameterCollectionInstance(
            World, MyCollection);
    if (Instance)
    {
        Result = Instance->GetScalarParameterValue(FName("Brightness"));
    }
}
```

### Asset Reference Generation
Collection asset references must be properly handled:
```cpp
// Asset references in generated code
UPROPERTY()
TObjectPtr<UMaterialParameterCollection> ParameterCollection;

// Initialization in constructor or BeginPlay
ParameterCollection = LoadObject<UMaterialParameterCollection>(
    nullptr, 
    TEXT("/Game/Materials/MyCollection.MyCollection")
);
```

### Parameter Name Handling
Parameter names are typically handled as FName constants:
```cpp
// Generate static FName constants for parameters
static const FName ParamName_Brightness(TEXT("Brightness"));
static const FName ParamName_Color(TEXT("Color"));

// Use in function calls
float Brightness = Instance->GetScalarParameterValue(ParamName_Brightness);
```

### Type Safety
Generated code must maintain type safety:
```cpp
// Type-safe parameter access
template<typename T>
T GetParameterValue(UMaterialParameterCollectionInstance* Instance, FName ParamName);

// Specialized implementations
template<>
float GetParameterValue<float>(UMaterialParameterCollectionInstance* Instance, FName ParamName)
{
    return Instance->GetScalarParameterValue(ParamName);
}

template<>
FVector4 GetParameterValue<FVector4>(UMaterialParameterCollectionInstance* Instance, FName ParamName)
{
    return Instance->GetVectorParameterValue(ParamName);
}
```

## Runtime Considerations

### Performance Implications
1. **Asset Loading**: Collections must be loaded before use
2. **Instance Creation**: Collection instances may need creation
3. **Parameter Lookup**: Parameter lookups have runtime cost
4. **Rendering Thread**: Some operations may need rendering thread synchronization

### Thread Safety
Material parameter collections interact with rendering system:
1. **Game Thread**: Parameter setting typically on game thread
2. **Render Thread**: Parameter reading may occur on render thread
3. **Synchronization**: Proper synchronization required between threads
4. **Update Timing**: Parameter updates have timing considerations

### Memory Management
1. **Collection Lifetime**: Collections must persist while referenced
2. **Instance Management**: Parameter collection instances lifecycle
3. **Parameter Caching**: Parameter values may be cached for performance
4. **Garbage Collection**: Proper GC handling for asset references

## Best Practices for C++ Conversion

### Asset Reference Management
1. **Hard References**: Use hard references for required collections
2. **Lazy Loading**: Load collections on-demand when possible
3. **Validation**: Always validate collection availability before use
4. **Error Handling**: Graceful handling of missing collections

### Parameter Access Patterns
1. **Constant Names**: Use FName constants for parameter names
2. **Type Safety**: Maintain strict type safety in parameter access
3. **Default Values**: Provide sensible defaults for missing parameters
4. **Caching**: Cache frequently accessed parameters

### Code Generation Strategy
1. **Function Mapping**: Map Blueprint functions to appropriate C++ APIs
2. **Asset Integration**: Properly integrate asset references into generated code
3. **Error Propagation**: Propagate validation errors to generated code
4. **Performance Optimization**: Optimize for runtime performance

### Validation Integration
1. **Compile-Time Checks**: Perform validation during code generation
2. **Asset Verification**: Verify all referenced assets exist
3. **Parameter Validation**: Check all parameter references are valid
4. **Type Consistency**: Ensure type consistency throughout conversion