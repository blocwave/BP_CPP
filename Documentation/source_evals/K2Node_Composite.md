# K2Node_Composite.cpp - Composite Node Implementation

**File**: `/Engine/Source/Editor/BlueprintGraph/Private/K2Node_Composite.cpp`

## Overview
This file implements the UK2Node_Composite class, which represents composite/collapsed nodes in Blueprint graphs. Composite nodes encapsulate subgraphs of Blueprint logic into reusable, collapsible units that appear as single nodes in the parent graph while maintaining their internal complexity in bound subgraphs.

## Role in Compilation Pipeline
- **Graph Encapsulation**: Wraps complex node networks into single composite nodes
- **Subgraph Management**: Maintains relationships between parent graphs and bound subgraphs  
- **Pin Interface**: Creates unified input/output interfaces for encapsulated logic
- **Node Lifecycle**: Handles creation, copying, and destruction of composite nodes
- **Schema Validation**: Ensures subgraph nodes are compatible with parent graph schema

## Key Functions and Algorithms

### Pin Allocation System
```cpp
void UK2Node_Composite::AllocateDefaultPins()
```
- **Purpose**: Creates external pins that mirror internal tunnel node interfaces
- **Algorithm**:
  1. **Output Source Processing**: 
     ```cpp
     if (OutputSourceNode) {
         for (UEdGraphPin* PortPin : OutputSourceNode->Pins) {
             if (PortPin->Direction == EGPD_Input) {
                 // Create complementary output pin on composite node
                 UEdGraphPin* NewPin = CreatePin(GetComplementaryDirection(PortPin->Direction), 
                                               PortPin->PinType, PortPin->PinName);
             }
         }
     }
     ```
  2. **Input Sink Processing**: Similar process for input pins from InputSinkNode
  3. **Default Value Propagation**: Copies default values from tunnel pins
  4. **Wildcard Pin Caching**: Updates wildcard pin type information

### Subgraph Lifecycle Management
```cpp
void UK2Node_Composite::DestroyNode()
```
- **Purpose**: Safely removes composite node and optionally its bound subgraph
- **Algorithm**:
  1. Captures reference to BoundGraph before Super::DestroyNode()
  2. Calls parent destruction logic
  3. Removes bound graph from Blueprint if exclusively owned
  4. Triggers recompilation to update dependent systems
- **Safety**: Prevents dangling graph references and ensures clean removal

### Post-Paste Integration
```cpp
void UK2Node_Composite::PostPasteNode()
```
- **Purpose**: Handles composite node integration after copy/paste operations
- **Complex Algorithm**:
  1. **Graph Validation**: Ensures BoundGraph != ParentGraph (prevents recursion)
  2. **Schema Compatibility Check**:
     ```cpp
     if (!ParentSchema->CanEncapuslateNode(*Node)) {
         FBlueprintEditorUtils::RemoveNode(BP, Node, true);
         continue;
     }
     ```
  3. **Event Node Deduplication**: Removes duplicate event nodes that already exist in Blueprint
  4. **Tunnel Node Resolution**: Updates InputSinkNode/OutputSourceNode pointers
  5. **Schema Synchronization**: Updates subgraph schema to match parent
  6. **Graph Relationship Setup**: Adds subgraph to ParentGraph.SubGraphs array

### Tunnel Node Identification and Linking
```cpp
// Tunnel node identification logic in PostPasteNode():
if (Node->GetClass() == UK2Node_Tunnel::StaticClass()) {
    UK2Node_Tunnel* Tunnel = CastChecked<UK2Node_Tunnel>(Node);
    
    if (Tunnel->bCanHaveInputs && !Tunnel->bCanHaveOutputs) {
        // This is an output source (takes inputs, provides to composite output)
        OutputSourceNode = Tunnel;
        Tunnel->InputSinkNode = this;
    }
    else if (Tunnel->bCanHaveOutputs && !Tunnel->bCanHaveInputs) {
        // This is an input sink (receives from composite inputs)
        InputSinkNode = Tunnel; 
        Tunnel->OutputSourceNode = this;
    }
}
```

### Wildcard Pin Management
```cpp
void UK2Node_Composite::CacheWildcardPins()
```
- **Purpose**: Updates wildcard pin types based on connected pins
- **Integration**: Works with Blueprint's wildcard type resolution system
- **Dynamic Typing**: Allows composite nodes to adapt to connected pin types

## Integration with Other Compiler Components

### With Blueprint Graph System
- **Parent-Child Relationships**: Maintains bidirectional graph relationships
- **Schema Enforcement**: Ensures subgraph nodes comply with parent schema
- **Graph Collections**: Integrates with Blueprint's SubGraphs array management

### With Node Validation System
- **Schema Validation**: Uses `ParentSchema->CanEncapsulateNode()` for compatibility
- **Event Deduplication**: Prevents multiple instances of unique event nodes
- **Circular Reference Prevention**: Ensures BoundGraph != ParentGraph

### With Blueprint Editor
- **Visual Representation**: Provides collapsed view of complex logic
- **Editing Interface**: Double-click expansion to edit internal subgraph
- **Debugging Support**: Maintains node relationships for debugging tools

## Relevance to BP→C++ Conversion

### Function Extraction Pattern
```cpp
// Composite Node → C++ Private Function
// Input: Composite node with internal logic
// Output: Private member function + function call

class UMyBlueprintClass : public UObject {
private:
    // Generated from composite subgraph
    void CompositeLogic_Internal(int32 InputA, float InputB, int32& OutputValue);
    
public:
    void MainFunction() {
        int32 Result;
        CompositeLogic_Internal(SomeValue, SomeFloat, Result);
        // Continue with Result usage...
    }
};
```

### Parameter Interface Translation
```cpp
// Composite pins → Function parameters
// InputSinkNode pins → Function parameters (input)  
// OutputSourceNode pins → Function parameters (output/reference)

// Example composite with inputs (A, B) and output (Result):
void UMyClass::CompositeFunction(int32 A, float B, int32& Result) {
    // Internal subgraph logic translated to C++ statements
    Result = PerformCalculation(A, B);
}
```

### Subgraph Compilation Strategy
1. **Function Extraction**: Each composite becomes a private member function
2. **Parameter Mapping**: Tunnel pins become function parameters
3. **Call Site Generation**: Composite node becomes function call
4. **Variable Flow**: Input/output pins handled via parameters/references

### Code Organization Benefits
- **Modularity**: Complex Blueprint logic becomes organized C++ functions
- **Reusability**: Composite patterns can be extracted as utility functions
- **Readability**: Reduces main function complexity through decomposition
- **Debugging**: Maintains logical boundaries in generated code

## Key Data Structures

### UK2Node_Composite Core Properties
```cpp
class UK2Node_Composite : public UK2Node_Tunnel {
    UEdGraph* BoundGraph;              // Subgraph containing internal logic
    UK2Node_Tunnel* InputSinkNode;     // Internal node receiving composite inputs
    UK2Node_Tunnel* OutputSourceNode;  // Internal node providing composite outputs
    bool bCanHaveInputs;               // Interface capability flags
    bool bCanHaveOutputs;
    bool bIsEditable;                  // Editor interaction permissions
};
```

### Tunnel Node Relationships
```cpp
// Bidirectional linking pattern:
class UK2Node_Tunnel {
    UK2Node_Composite* InputSinkNode;     // Points to composite that feeds this tunnel
    UK2Node_Composite* OutputSourceNode;  // Points to composite this tunnel feeds
};
```

### Graph Relationship Management
```cpp
// Parent graph maintains subgraph collection:
class UEdGraph {
    TArray<UEdGraph*> SubGraphs;  // Includes bound graphs from composite nodes
};
```

## Special Considerations

### Graph Schema Compatibility
- **Node Filtering**: Removes incompatible nodes during paste operations
- **Schema Propagation**: Updates subgraph schema to match parent
- **Validation Rules**: Ensures all subgraph nodes can compile in parent context

### Event Node Handling
```cpp
// Prevents duplicate event nodes:
if (UK2Node_Event* Event = Cast<UK2Node_Event>(Node)) {
    if (FBlueprintEditorUtils::FindOverrideForFunction(BP, Event->EventReference.GetMemberParentClass(), 
                                                       Event->EventReference.GetMemberName())) {
        bRemoveNode = true;  // Remove duplicate event
    }
}
```

### Recursive Prevention
- **Self-Reference Check**: Ensures BoundGraph != ParentGraph
- **Circular Reference Detection**: Prevents infinite nesting of composite nodes
- **Graph Hierarchy Validation**: Maintains acyclic graph relationships

### Memory Management
- **Shared Ownership**: Multiple composites can reference same subgraph
- **Cleanup Coordination**: Proper cleanup when last reference is removed
- **Blueprint Integration**: Subgraphs participate in Blueprint's memory management

This implementation enables complex Blueprint logic to be organized into reusable, maintainable units while providing clean translation paths to well-structured C++ code with appropriate function decomposition.