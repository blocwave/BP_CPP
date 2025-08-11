# UK2Node API Documentation

## Overview
`UK2Node` is the abstract base class of all blueprint graph nodes in Unreal Engine. This class provides the fundamental functionality for all Blueprint nodes, including compilation, validation, pin management, and visual representation.

## Class Definition
```cpp
UCLASS(Abstract, MinimalAPI)
class UK2Node : public UEdGraphNode
```

## Inheritance Hierarchy
- `UObject`
  - `UEdGraphNode`
    - **`UK2Node`** (Abstract base class)

## Key Properties
- Abstract base class - cannot be instantiated directly
- Inherits from `UEdGraphNode` providing basic graph node functionality
- Provides Blueprint-specific node behavior and compilation support

## Core Functionality

### Compilation-Related Methods
- **`CreateNodeHandler(FKismetCompilerContext& CompilerContext)`** - Creates a node handler for compilation
- **`ExpandNode(FKismetCompilerContext& CompilerContext, UEdGraph* SourceGraph)`** - Expands the node during compilation
- **`ValidateNodeDuringCompilation(FCompilerResultsLog& MessageLog)`** - Validates the node during compilation
- **`EarlyValidation(FCompilerResultsLog& MessageLog)`** - Performs early validation before main compilation

### Pin Management
- **`AllocateDefaultPins()`** - Allocates the default pins for the node
- **`ReconstructNode()`** - Reconstructs the node and its pins
- **`CreatePinsForFunctionEntryExit(const UFunction* Function, bool bForFunctionEntry)`** - Creates pins for function entry/exit
- **`ReallocatePinsDuringReconstruction(TArray<UEdGraphPin*>& OldPins)`** - Reallocates pins during reconstruction
- **`CanSplitPin(const UEdGraphPin* Pin)`** - Determines if a pin can be split
- **`ExpandSplitPin(FKismetCompilerContext* CompilerContext, UEdGraph* SourceGraph, UEdGraphPin* Pin)`** - Expands split pins

### Visual Representation
- **`GetNodeTitle(ENodeTitleType::Type TitleType)`** - Gets the display title for the node
- **`GetNodeTitleColor()`** - Gets the color for the node title
- **`GetCompactNodeTitle()`** - Gets the compact title for small node display
- **`GetCornerIcon()`** - Gets the corner icon for the node
- **`DrawNodeAsEntry()`** - Whether to draw the node as an entry node
- **`DrawNodeAsExit()`** - Whether to draw the node as an exit node
- **`DrawNodeAsVariable()`** - Whether to draw the node as a variable node

### Blueprint Integration
- **`GetBlueprint()`** - Gets the Blueprint that owns this node
- **`GetBlueprintClassFromNode()`** - Gets the Blueprint class from the node
- **`HasValidBlueprint()`** - Checks if the node has a valid Blueprint
- **`ClearCachedBlueprintData(UBlueprint* Blueprint)`** - Clears cached Blueprint data

### Node Behavior
- **`IsNodePure()`** - Determines if the node is pure (no execution pins)
- **`IsNodeSafeToIgnore()`** - Determines if the node can be safely ignored during compilation
- **`NodeCausesStructuralBlueprintChange()`** - Whether this node causes structural changes to the Blueprint
- **`CanJumpToDefinition()`** - Whether the node supports jumping to definition
- **`JumpToDefinition()`** - Jumps to the definition of the referenced element

### Menu and Actions
- **`GetMenuActions(FBlueprintActionDatabaseRegistrar& ActionRegistrar)`** - Gets menu actions for this node type
- **`GetMenuCategory()`** - Gets the menu category for this node
- **`IsActionFilteredOut(FBlueprintActionFilter const& Filter)`** - Determines if the action should be filtered out

### Connection and Validation
- **`IsConnectionDisallowed(const UEdGraphPin* MyPin, const UEdGraphPin* OtherPin, FString& OutReason)`** - Checks if a connection is disallowed
- **`AutowireNewNode(UEdGraphPin* FromPin)`** - Autowires a new node when placed
- **`NotifyPinConnectionListChanged(UEdGraphPin* Pin)`** - Notifies when pin connections change

### Reference Handling
- **`ReferencesFunction(const FName& InFunctionName, const UStruct* InScope)`** - Checks if the node references a function
- **`ReferencesVariable(const FName& InVarName, const UStruct* InScope)`** - Checks if the node references a variable
- **`HandleFunctionRenamed(...)`** - Handles function renaming
- **`HandleVariableRenamed(...)`** - Handles variable renaming

### Utility Methods
- **`GetDocumentationLink()`** - Gets the documentation link for the node
- **`GetDocumentationExcerptName()`** - Gets the documentation excerpt name
- **`GetToolTipHeading()`** - Gets the tooltip heading
- **`GetPinHoverText(const UEdGraphPin& Pin, FString& HoverTextOut)`** - Gets hover text for pins
- **`GetPinMetaData(FName InPinName, FName InKey)`** - Gets metadata for pins

## Usage in Compilation
UK2Node serves as the foundation for Blueprint compilation. During compilation:

1. **Early Validation** - `EarlyValidation()` is called first
2. **Node Handler Creation** - `CreateNodeHandler()` creates appropriate compiler handlers
3. **Node Expansion** - `ExpandNode()` expands the node into lower-level constructs
4. **Final Validation** - `ValidateNodeDuringCompilation()` performs final checks

## Derived Classes
Common derived classes include:
- `UK2Node_CallFunction` - Function call nodes
- `UK2Node_Event` - Event nodes
- `UK2Node_Variable` - Variable access nodes
- `UK2Node_IfThenElse` - Conditional nodes
- `UK2Node_ForEachLoop` - Loop nodes
- Many others for specific Blueprint functionality

## Important Notes
- This is an abstract class and cannot be instantiated directly
- All Blueprint nodes inherit from this class
- Provides the foundation for Blueprint visual scripting system
- Critical for Blueprint compilation and editor functionality
- Handles the bridge between visual Blueprint representation and underlying code generation