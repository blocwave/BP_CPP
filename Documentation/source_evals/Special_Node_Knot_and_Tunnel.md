# K2Node_Knot and K2Node_Tunnel Analysis - Visual Organization Nodes

## K2Node_Knot (Reroute Node)

### Overview
K2Node_Knot is a purely visual node that provides wire organization without affecting compilation. It acts as a passthrough connection point that automatically adapts to connected pin types.

### Key Properties
- **Pure Visual Node**: `IsCompilerRelevant() override { return false; }`
- **Two Pin Structure**: Input and Output with wildcard types
- **Type Propagation**: Automatically inherits type from connected pins
- **Recursion Protection**: `bRecursionGuard` prevents infinite loops

### Unique Expansion Patterns

#### 1. Type Propagation Algorithm
```cpp
void PropagatePinType()
{
    UEdGraphPin* MyInputPin  = GetInputPin();
    UEdGraphPin* MyOutputPin = GetOutputPin();

    // Prefer input connections for type determination
    for (UEdGraphPin* Inputs : MyInputPin->LinkedTo)
    {
        if (Inputs->PinType.PinCategory != UEdGraphSchema_K2::PC_Wildcard)
        {
            PropagatePinTypeFromDirection(true);
            return;
        }
    }

    // Fallback to output connections
    for (UEdGraphPin* Outputs : MyOutputPin->LinkedTo)
    {
        if (Outputs->PinType.PinCategory != UEdGraphSchema_K2::PC_Wildcard)
        {
            PropagatePinTypeFromDirection(false);
            return;
        }
    }

    // Revert to wildcard if no typed connections
    if (MyInputPin->LinkedTo.Num() == 0 && MyOutputPin->LinkedTo.Num() == 0)
    {
        MyInputPin->PinType.ResetToDefaults();
        MyInputPin->PinType.PinCategory = UEdGraphSchema_K2::PC_Wildcard;
        MyOutputPin->PinType.ResetToDefaults();
        MyOutputPin->PinType.PinCategory = UEdGraphSchema_K2::PC_Wildcard;
    }
}
```

#### 2. Compilation Elimination
```cpp
void ExpandNode(class FKismetCompilerContext& CompilerContext, UEdGraph* SourceGraph)
{
    const UEdGraphSchema_K2* K2Schema = GetDefault<UEdGraphSchema_K2>();
    UEdGraphPin* MyInputPin = GetInputPin();
    UEdGraphPin* MyOutputPin = GetOutputPin();

    // Directly connect input and output, removing the knot
    K2Schema->CombineTwoPinNetsAndRemoveOldPins(MyInputPin, MyOutputPin);
}
```

### C++ Conversion Requirements
**NONE** - Knot nodes are purely visual and are eliminated during compilation. They require no C++ equivalent.

## K2Node_Tunnel (Collapsed Graph Interface)

### Overview
K2Node_Tunnel provides entry/exit points for collapsed graphs, function graphs, and macro graphs. They define the interface boundary between graph levels.

### Key Properties
- **OutputSourceNode**: Reference to source tunnel for output pins
- **InputSinkNode**: Reference to destination tunnel for input pins  
- **bCanHaveInputs/bCanHaveOutputs**: Direction capabilities
- **MetaData**: Function metadata (tooltip, category, etc.)

### Graph Interface Patterns

#### 1. Tunnel Pairing System
```cpp
// Tunnel nodes work in pairs:
// Entry Tunnel (in subgraph): bCanHaveOutputs = true, receives external inputs
// Exit Tunnel (in subgraph): bCanHaveInputs = true, sends to external outputs
// Instance Node: Mirrors both tunnel types as complementary pins

class UK2Node_Tunnel : public UK2Node_EditablePinBase
{
    // Input pins go to outputs of InputSinkNode
    UK2Node_Tunnel* InputSinkNode;
    
    // Output pins come from inputs of OutputSourceNode  
    UK2Node_Tunnel* OutputSourceNode;
};
```

#### 2. Interface Metadata
```cpp
struct FKismetUserDeclaredFunctionMetadata MetaData
{
    FText ToolTip;
    FText Category; 
    FText Keywords;
    FLinearColor InstanceTitleColor;
    FText CompactNodeTitle;
    uint32 bCallInEditor : 1;
    uint32 bThreadSafe : 1;
    // ... additional metadata
};
```

### C++ Code Generation Requirements

#### 1. Function Boundary Translation
```cpp
// Blueprint function with tunnels becomes C++ function
UFUNCTION(BlueprintCallable, Category = "MyCategory")
ReturnType MyFunction(ParamType InputParam)
{
    // Entry tunnel translates to function parameters
    // Exit tunnel translates to return statement
    
    // Function body (collapsed graph content)
    ProcessType IntermediateResult = ProcessInput(InputParam);
    return CalculateResult(IntermediateResult);
}
```

#### 2. Macro Graph Interface
```cpp
// Tunnel nodes in macros become inline scope boundaries
{
    // Entry tunnel: parameter capture
    LocalType TunnelInput = ExternalInput;
    
    // Collapsed graph logic
    LocalType ProcessedResult = MacroLogic(TunnelInput);
    
    // Exit tunnel: result assignment
    ExternalOutput = ProcessedResult;
}
```

## Combined Special Properties

### 1. Visual vs Functional Distinction
- **K2Node_Knot**: Pure visual aid, eliminated during compilation
- **K2Node_Tunnel**: Functional boundary, becomes C++ function interface

### 2. Type System Integration
- **Knots**: Dynamic type adaptation through propagation
- **Tunnels**: Static type definition for graph interfaces

### 3. Graph Structure Role
- **Knots**: Wire organization within single graph
- **Tunnels**: Communication between graph hierarchies

## Implementation Priority

### K2Node_Knot
**LOW** - Visual-only nodes with no compilation impact. Can be ignored in Blueprint to C++ conversion.

### K2Node_Tunnel  
**HIGH** - Essential for function boundaries and graph hierarchy. Required for proper C++ function generation from Blueprint functions and macros.

## Dependencies
- **K2Node_Knot**: UEdGraphSchema_K2 (pin type system)
- **K2Node_Tunnel**: UK2Node_EditablePinBase, FKismetUserDeclaredFunctionMetadata, graph hierarchy system