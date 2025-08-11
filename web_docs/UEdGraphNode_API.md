# UEdGraphNode API Documentation

Source: https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Runtime/Engine/EdGraph/UEdGraphNode?application_version=5.2

## Class Definition

```cpp
class UEdGraphNode : public UObject
```

## Purpose

UEdGraphNode is the base class for all nodes in an editor graph. It provides the foundational functionality for graph-based visual scripting systems, including pin management, visual appearance, and basic node behavior. All Blueprint nodes, including UK2Node and its derivatives, inherit from this class.

## Inheritance Hierarchy

- `UObject`
  - **`UEdGraphNode`** (Base class for all graph nodes)
    - `UK2Node` (Blueprint-specific nodes)
      - `UK2Node_CallFunction`
      - `UK2Node_Event`
      - `UK2Node_Variable`
      - And many others...

## Key Properties

### Graph Relationship
- **Graph Integration** - Node belongs to a specific UEdGraph and integrates with the overall graph structure
- **Pin Management** - Manages input and output pins for connections to other nodes
- **Visual Representation** - Handles display properties, positioning, and appearance in the editor

### Node State
- **Enabled State** - Nodes can be enabled or disabled
- **Advanced Display** - Some pins can be hidden in advanced view
- **Breakpoint Support** - Debugging breakpoints can be set on executable nodes

## Core Functionality

### Pin Management
- **AllocateDefaultPins()** - Creates the default set of pins for the node
- **ReconstructNode()** - Rebuilds the node's pins and internal structure
- **GetInputPin() / GetOutputPin()** - Access specific pins by index or name
- **FindPin()** - Locate pins by name and direction
- **RemovePin()** - Remove a specific pin from the node

### Visual Representation
- **GetNodeTitle()** - Returns the display title for the node
- **GetNodeTitleColor()** - Gets the color used for the node's title bar
- **GetTooltipText()** - Provides tooltip information when hovering over the node
- **GetCornerIcon()** - Returns an icon displayed in the node's corner
- **ShouldDrawNodeAsControlPointOnly()** - Determines minimal display mode

### Node Behavior
- **IsNodeEnabled()** - Checks if the node is currently enabled
- **SetEnabledState()** - Enables or disables the node
- **CanUserDeleteNode()** - Determines if the user can delete this node
- **CanDuplicateNode()** - Checks if the node can be duplicated
- **GetCanRenameNode()** - Whether the node supports renaming

### Connection Management
- **CanCreateConnection()** - Validates potential connections between pins
- **GetConnectionResponse()** - Returns detailed connection validation results
- **BreakAllNodeLinks()** - Removes all connections to/from this node
- **BreakLinkTo()** - Breaks connection to a specific node

### Schema Integration
- **GetSchema()** - Gets the graph schema that governs this node's behavior
- **IsCompatibleWithGraph()** - Checks if the node is compatible with a given graph

### Compilation Support
- **IsNodePure()** - Indicates whether the node has side effects (pure nodes have no execution pins)
- **ShouldShowNodeProperties()** - Whether to display node properties in details panels

### Debugging Support
- **CanPlaceBreakpoints()** - Whether breakpoints can be placed on this node
- **GetInstructionCount()** - Returns number of instructions for debugging purposes

## Usage in Blueprint System

UEdGraphNode serves as the foundation for all Blueprint visual scripting nodes:

1. **Node Creation** - New nodes are created and added to graphs
2. **Pin Allocation** - Nodes create appropriate input/output pins based on their functionality  
3. **Connection Validation** - Schema rules determine valid connections between nodes
4. **Visual Display** - Editor renders nodes based on their visual properties
5. **Compilation** - UK2Node derivatives handle Blueprint-specific compilation logic

## Common Derived Classes

- **UK2Node** - Base class for all Blueprint nodes
- **UEdGraphNode_Comment** - Comment nodes for documentation
- **UAnimGraphNode_Base** - Base for Animation Blueprint nodes
- **UMaterialGraphNode** - Nodes in Material Editor graphs
- **USoundCueGraphNode** - Audio cue graph nodes

## Important Notes

- This is a base class providing fundamental graph node functionality
- Specific behavior is implemented in derived classes like UK2Node
- All visual graph editors in Unreal Engine use this as their foundation
- Pin management and connection validation are core responsibilities
- Visual appearance can be customized through virtual methods
- Thread-safe operations are important for editor responsiveness

## Related Classes

- **UEdGraph** - The graph that contains these nodes
- **UEdGraphPin** - Individual connection points on nodes
- **UEdGraphSchema** - Rules governing node behavior and connections
- **FGraphCompilerContext** - Compilation context for processing nodes