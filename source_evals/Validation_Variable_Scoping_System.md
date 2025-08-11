# Variable Scoping and Temporary Variable System

## Overview
The Blueprint variable scoping system handles temporary variables, local variables, and their lifetime management. This system is critical for C++ conversion as it determines variable scope, lifetime, and access patterns in generated code.

## Temporary Variable System

### UK2Node_TemporaryVariable Core Structure
```cpp
UCLASS(MinimalAPI)
class UK2Node_TemporaryVariable : public UK2Node {
    GENERATED_UCLASS_BODY()

    // Variable type definition
    UPROPERTY()
    struct FEdGraphPinType VariableType;

    // Persistence flag for macro-generated variables
    UPROPERTY()
    bool bIsPersistent;

    // Variable pin accessor
    UEdGraphPin* GetVariablePin();
};
```

### Key Properties
1. **VariableType**: Defines the type of the temporary variable
2. **bIsPersistent**: Controls variable lifetime and naming
3. **Pure Node**: Temporary variables are pure (no execution flow)
4. **Single Pin**: Provides single output pin for variable value

### Node Behavior
```cpp
class UK2Node_TemporaryVariable {
    // Pure node (no execution pins)
    virtual bool IsNodePure() const override;
    
    // Post-reconstruction setup
    virtual void PostReconstructNode() override;
    
    // Connection change handling
    virtual void NotifyPinConnectionListChanged(UEdGraphPin* Pin) override;
    
    // Compiler integration
    virtual class FNodeHandlingFunctor* CreateNodeHandler(
        class FKismetCompilerContext& CompilerContext
    ) const override;
};
```

## Local Variable System (Deprecated)

### UDEPRECATED_K2Node_LocalVariable
```cpp
UCLASS(MinimalAPI, deprecated)
class UDEPRECATED_K2Node_LocalVariable : public UK2Node_TemporaryVariable {
    GENERATED_UCLASS_BODY()

    // Custom variable name
    UPROPERTY()
    FName CustomVariableName;

    // Variable documentation
    UPROPERTY()
    FText VariableTooltip;

    // Causes structural blueprint changes
    virtual bool NodeCausesStructuralBlueprintChange() const override { return true; }
    
    // Shows in property panel
    virtual bool ShouldShowNodeProperties() const override { return true; }
    
    // Variable type modification
    void ChangeVariableType(const FEdGraphPinType& InVariableType);
};
```

### Deprecation and Migration
The local variable system has been deprecated in favor of:
1. **Function Parameters**: Use function inputs/outputs for data passing
2. **Member Variables**: Use Blueprint member variables for persistent state
3. **Temporary Variables**: Use temporary variables for intermediate calculations
4. **Return Values**: Use function return values for single outputs

## Variable Lifetime Management

### Temporary Variable Lifetime
Temporary variables have different lifetime characteristics:

1. **Transient Variables** (`bIsPersistent = false`):
   - Created during node execution
   - Destroyed after use
   - No persistent storage
   - Suitable for intermediate calculations

2. **Persistent Variables** (`bIsPersistent = true`):
   - Created with stable names based on macro GUID
   - May persist across function calls
   - CPF_SaveGame flag applied
   - Used for macro-generated variables

### Scope Determination
Variable scope is determined by:
- **Graph Context**: Function vs. event graph vs. macro
- **Node Placement**: Location within execution flow
- **Connection Pattern**: How variables are connected and used
- **Persistence Flag**: Whether variable needs to persist

## Pin Management and Connections

### Variable Pin Creation
```cpp
UEdGraphPin* GetVariablePin();
```

Temporary variables create a single output pin:
- **Direction**: Output only
- **Type**: Matches VariableType property
- **Name**: Usually "Variable" or similar
- **Category**: Based on variable type

### Connection Handling
```cpp
virtual void NotifyPinConnectionListChanged(UEdGraphPin* Pin) override;
```

When connections change:
1. **Type Propagation**: Variable type may be inferred from connections
2. **Validation**: Ensure connections are type-compatible
3. **Reconstruction**: May trigger node reconstruction if needed
4. **Default Values**: Update default values based on connections

### Pin Reconstruction
```cpp
virtual void PostReconstructNode() override;
```

During reconstruction:
1. **Pin Creation**: Create variable pin with correct type
2. **Connection Restoration**: Restore previous connections if valid
3. **Default Value Setup**: Initialize default values
4. **Validation**: Validate new pin configuration

## Graph Context and Compatibility

### Graph Compatibility
```cpp
virtual bool IsCompatibleWithGraph(UEdGraph const* TargetGraph) const override;
virtual bool CanPasteHere(const UEdGraph* TargetGraph) const override;
```

Compatibility checks include:
1. **Graph Type**: Function graphs vs event graphs vs macros
2. **Execution Context**: Pure vs impure function contexts
3. **Variable Scope**: Local scope vs global scope requirements
4. **Blueprint Type**: Different blueprint types have different restrictions

### Context-Specific Behavior
Different graph contexts affect variable behavior:

#### Function Graphs
- Variables can be function-local
- Lifetime tied to function execution
- May be optimized as stack variables

#### Event Graphs
- Variables typically have broader scope
- May need to persist between events
- Often implemented as member variables

#### Macro Graphs
- Variables can be macro-local
- May use persistent storage with GUID-based names
- Special handling for macro expansion

## Compiler Integration

### Node Handler Creation
```cpp
virtual class FNodeHandlingFunctor* CreateNodeHandler(
    class FKismetCompilerContext& CompilerContext
) const override;
```

The node handler manages:
1. **Variable Declaration**: Generate variable declarations
2. **Scope Management**: Determine appropriate variable scope
3. **Lifetime Management**: Handle variable construction/destruction
4. **Initialization**: Initialize variables with default values
5. **Optimization**: Optimize variable usage patterns

### Compilation Process
During compilation:
1. **Scope Analysis**: Determine variable scope requirements
2. **Type Resolution**: Resolve final variable types
3. **Naming**: Generate unique variable names
4. **Storage**: Determine storage method (stack, member, etc.)
5. **Initialization**: Generate initialization code

## Blueprint Signature Integration

### FBlueprintNodeSignature
```cpp
virtual FBlueprintNodeSignature GetSignature() const override;
```

Temporary variables contribute to Blueprint signatures:
- **Type Information**: Variable type affects signature
- **Scope Information**: Variable scope affects function signature
- **Usage Pattern**: How variable is used affects optimization

### Signature Components
Variable signatures include:
1. **Variable Type**: The FEdGraphPinType of the variable
2. **Persistence Flag**: Whether variable is persistent
3. **Scope Context**: The scope context of the variable
4. **Usage Pattern**: How the variable is accessed and modified

## Variable Type System Integration

### Type Inference and Validation
Variable types are determined through:
1. **Explicit Type**: Set during node creation or configuration
2. **Connection Inference**: Inferred from connected pins
3. **Context Inference**: Inferred from usage context
4. **Default Type**: Default to appropriate type if ambiguous

### Type Conversion and Coercion
The system handles:
1. **Automatic Promotion**: Promote compatible types (int->float)
2. **Explicit Conversion**: Insert conversion nodes when needed
3. **Type Validation**: Validate type assignments are legal
4. **Error Reporting**: Report type mismatches clearly

## C++ Code Generation Implications

### Variable Declaration Strategies

#### Stack Variables (Transient)
```cpp
// For temporary variables in functions
void GeneratedFunction() {
    float TempVariable = 0.0f;  // Stack allocation
    // ... use variable ...
}  // Automatic cleanup
```

#### Member Variables (Persistent)
```cpp
// For persistent variables in classes
class GeneratedClass : public UObject {
    UPROPERTY()
    float PersistentVariable;  // Member variable
};
```

#### Parameter Variables
```cpp
// For function parameter passing
void GeneratedFunction(float InputVariable, float& OutputVariable) {
    OutputVariable = InputVariable * 2.0f;
}
```

### Scope Translation
Blueprint variable scopes map to C++ scopes:

1. **Function Scope**: Local variables within functions
2. **Class Scope**: Member variables in generated classes
3. **Global Scope**: Static variables (rarely used)
4. **Parameter Scope**: Function parameters and references

### Lifetime Management
```cpp
// RAII-style lifetime management
class VariableLifetimeManager {
    VariableLifetimeManager() { /* Constructor */ }
    ~VariableLifetimeManager() { /* Destructor */ }
};

// Usage in generated code
void GeneratedFunction() {
    VariableLifetimeManager VarManager;  // Auto-cleanup
    // ... use variables ...
}  // Automatic destruction
```

### Type Safety and Validation
Generated code must maintain type safety:
```cpp
// Template-based type safety
template<typename T>
class TypedVariable {
    T Value;
public:
    TypedVariable(const T& InitValue) : Value(InitValue) {}
    const T& Get() const { return Value; }
    void Set(const T& NewValue) { Value = NewValue; }
};

// Usage
TypedVariable<float> FloatVar(0.0f);
TypedVariable<FVector> VectorVar(FVector::ZeroVector);
```

## Best Practices for C++ Conversion

### Variable Scope Analysis
1. **Scope Detection**: Analyze Blueprint graph to determine proper scope
2. **Lifetime Analysis**: Determine variable lifetime requirements
3. **Usage Pattern**: Analyze how variables are accessed and modified
4. **Optimization Opportunities**: Identify optimization opportunities

### Code Generation Strategy
1. **Scope Mapping**: Map Blueprint scopes to appropriate C++ scopes
2. **Type Preservation**: Maintain type safety in generated code
3. **Naming Convention**: Use consistent naming conventions
4. **Memory Management**: Proper memory management for complex types

### Performance Optimization
1. **Stack Allocation**: Prefer stack allocation for temporary variables
2. **Reference Passing**: Use references for large objects
3. **Move Semantics**: Use move semantics where appropriate
4. **Const Correctness**: Maintain const correctness in generated code

### Error Handling and Validation
1. **Scope Validation**: Validate variable scopes are legal
2. **Type Validation**: Ensure all variable types are valid
3. **Initialization**: Ensure all variables are properly initialized
4. **Cleanup**: Ensure proper cleanup of resources