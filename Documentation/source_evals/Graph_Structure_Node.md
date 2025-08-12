# Graph Structure - Node System Analysis

## Overview
This document analyzes the core UEdGraphNode structure and its critical data components required for Blueprint to C++ conversion. The UEdGraphNode is the fundamental building block of all Blueprint graph structures.

## UEdGraphNode Core Data Structure

### Essential Node Properties
```cpp
class UEdGraphNode : public UObject
{
    // Core node pin collection
    TArray<UEdGraphPin*> Pins;
    
    // Physical position and size (editor visual data)
    int32 NodePosX;         // X position in editor
    int32 NodePosY;         // Y position in editor  
    int32 NodeWidth;        // Node width (for resizable nodes)
    int32 NodeHeight;       // Node height (for resizable nodes)
    
    // Unique identification
    FGuid NodeGuid;         // CRITICAL: Unique identifier for node tracking
    
    // Node state and behavior flags
    ENodeEnabledState EnabledState;          // Enabled/Disabled/DevelopmentOnly
    ENodeAdvancedPins::Type AdvancedPinDisplay; // Pin visibility control
    
    // Error handling and validation
    bool bHasCompilerMessage;   // Compilation error flag
    int32 ErrorType;           // Error classification
    FString ErrorMsg;          // Error description text
    
    // User annotation
    FString NodeComment;       // User comment text
    
#if WITH_EDITORONLY_DATA
    // Editor-specific flags (preserved for conversion context)
    bool bCanResizeNode;       // Node resize capability
    bool bCanRenameNode;       // Node rename capability
    bool bCommentBubblePinned; // Comment bubble state
    bool bCommentBubbleVisible; // Comment visibility
#endif
    
    // Intermediate node marking (expansion/compilation)
    bool bIsIntermediateNode;  // Created during expansion
};
```

### Critical Node Features for BP to C++ Conversion

#### 1. Node Identification System
- **NodeGuid**: CRITICAL unique identifier that persists across node operations
- **Node class type**: Determines C++ code generation strategy
- **Position data**: Used for maintaining visual layout in generated comments

#### 2. Pin Management System
```cpp
// Pin creation and management
UEdGraphPin* CreatePin(EEdGraphPinDirection Dir, const FName PinCategory, const FName PinName);
UEdGraphPin* FindPin(const FName PinName, const EEdGraphPinDirection Direction = EGPD_MAX);
UEdGraphPin* FindPinById(const FGuid PinId);
bool RemovePin(UEdGraphPin* Pin);
```

#### 3. Node State Management
```cpp
// Enabled state affects code generation
enum class ENodeEnabledState : uint8
{
    Enabled,           // Always included in generated code
    Disabled,          // Excluded from generated code
    DevelopmentOnly    // Only included in development builds
};
```

#### 4. Advanced Pin Display Control
```cpp
enum ENodeAdvancedPins
{
    NoPins,    // No advanced pins
    Shown,     // Advanced pins are visible and included
    Hidden     // Advanced pins exist but hidden (still need C++ representation)
};
```

## Node Type Hierarchy Critical for C++ Generation

### Base Node Classes
1. **UEdGraphNode** - Base for all graph nodes
2. **UK2Node** - Blueprint-specific node base class
3. **UK2Node_CallFunction** - Function call nodes
4. **UK2Node_Variable** - Variable access nodes  
5. **UK2Node_Event** - Event handler nodes
6. **UK2Node_CustomEvent** - User-defined events

### Node Creation and Lifecycle
```cpp
// Node creation process
template <typename NodeType>
NodeType* CreateNode()
{
    NodeType* Node = Graph.CreateNode(NodeType::StaticClass());
    Node->CreateNewGuid();          // CRITICAL: Assign unique GUID
    Node->AllocateDefaultPins();    // Create required pins
    Node->PostPlacedNewNode();      // Initialize node state
    return Node;
}
```

## Data Required for C++ Conversion

### Essential Node Data
1. **NodeGuid** - Unique tracking across conversion process
2. **Node class type** - Determines C++ generation strategy
3. **Pin collection** - Complete pin data (see Graph_Structure_Pin.md)
4. **EnabledState** - Controls inclusion in generated code
5. **NodeComment** - Preserved as C++ comments
6. **Position data** - For maintaining visual correlation

### Node State Validation
```cpp
// Critical validation for C++ generation
bool IsNodeEnabled() const
{
    return (EnabledState == ENodeEnabledState::Enabled) || 
           ((EnabledState == ENodeEnabledState::DevelopmentOnly) && IsInDevelopmentMode());
}
```

### Node Connection Analysis
```cpp
// Connection traversal for data flow analysis
void ForEachNodeDirectlyConnected(TFunctionRef<void(UEdGraphNode*)> Func);
void ForEachNodeDirectlyConnectedToInputs(TFunctionRef<void(UEdGraphNode*)> Func);
void ForEachNodeDirectlyConnectedToOutputs(TFunctionRef<void(UEdGraphNode*)> Func);
```

## BP to C++ Conversion Requirements

### Node Processing Pipeline
1. **Node Discovery**: Enumerate all enabled nodes in execution order
2. **Pin Analysis**: Process all pin connections and data flow
3. **Type Resolution**: Resolve all node types and their C++ equivalents  
4. **Dependency Tracking**: Track node dependencies for proper C++ ordering
5. **Code Generation**: Generate appropriate C++ constructs per node type

### Critical Node Properties for Serialization
```cpp
struct NodeSerializationData {
    FGuid NodeGuid;                    // Unique identifier
    FString NodeClassName;             // Runtime class name
    int32 NodePosX, NodePosY;         // Position data
    ENodeEnabledState EnabledState;    // State for code inclusion
    TArray<PinSerializationData> Pins; // Complete pin data
    FString NodeComment;               // User comments
    // Additional node-specific data...
};
```

## Implementation Notes

### Memory Management
- Nodes use UObject lifecycle (GC managed)
- Pin arrays are managed by owning node
- Node destruction breaks all pin links automatically

### Thread Safety
- Graph operations are not thread-safe
- All node manipulation must occur on game thread
- Pin link operations must be atomic

### Performance Considerations
- Node GUID lookups can be expensive in large graphs
- Pin traversal should cache results when possible
- Consider using node arrays vs individual lookups

## Related Components
- See `Graph_Structure_Pin.md` for pin system details
- See `Graph_Structure_Schema.md` for validation rules
- See `Graph_Structure_K2Node.md` for Blueprint-specific nodes
- See `Graph_Structure_MemberReference.md` for variable/function references

## Summary

The UEdGraphNode system provides the foundational structure for Blueprint graphs. For successful BP to C++ conversion, the following node data is absolutely critical:

1. **NodeGuid** - Unique identification and tracking
2. **Complete pin collection** - All input/output connections and data
3. **Node class type** - Determines C++ generation strategy
4. **EnabledState** - Controls inclusion in generated code
5. **Position and comment data** - For maintaining context and documentation

The node system's hierarchical structure (UEdGraphNode → UK2Node → specific implementations) directly maps to C++ code generation strategies, making this analysis fundamental to the conversion process.