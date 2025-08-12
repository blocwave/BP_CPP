# UK2Node_Event API Documentation

## Overview
`UK2Node_Event` represents event nodes in Blueprint graphs. These nodes serve as entry points for Blueprint execution, triggered by various events such as gameplay events, input events, or custom events.

## Class Definition
```cpp
UCLASS(MinimalAPI)
class UK2Node_Event : public UK2Node_EditablePinBase, public IK2Node_EventNodeInterface
```

## Inheritance Hierarchy
- `UObject`
  - `UEdGraphNode`
    - `UK2Node`
      - `UK2Node_EditablePinBase`
        - **`UK2Node_Event`**

## Interfaces
- **`IK2Node_EventNodeInterface`** - Provides event-specific functionality

## Key Properties
- **Event Reference** - References the event signature/function
- **Delegate Pins** - Can have delegate pins for dynamic event binding
- **Entry Point** - Serves as an execution entry point in Blueprint graphs

## Core Functionality

### Event Configuration
- **`FindEventSignatureFunction()`** - Finds the signature function for the event
- **`FixupEventReference(bool bForce)`** - Fixes up the event reference
- **`GetFunctionName()`** - Gets the name of the function/event
- **`IsInterfaceEventNode()`** - Checks if this is an interface event node

### Event Types and Validation
- **`IsCosmeticTickEvent()`** - Checks if this is a cosmetic tick event
- **`IsUsedByAuthorityOnlyDelegate()`** - Checks if used by authority-only delegates
- **`IsFunctionEntryCompatible(const UK2Node_FunctionEntry* EntryNode)`** - Checks compatibility with function entry

### Delegate Handling
- **`GetDelegatePin()`** - Gets the delegate pin if present
- **`UpdateDelegatePin(bool bSilent)`** - Updates the delegate pin configuration

### Node Appearance
- **`GetNodeTitle(ENodeTitleType::Type TitleType)`** - Gets the display title
- **`GetNodeTitleColor()`** - Gets the color for the node (typically red for events)
- **`GetIconAndTint(FLinearColor& OutColor)`** - Gets the icon and tint color
- **`GetCornerIcon()`** - Gets the corner icon indicating event type

### Compilation Support
- **`CreateNodeHandler(FKismetCompilerContext& CompilerContext)`** - Creates event node handler
- **`ExpandNode(FKismetCompilerContext& CompilerContext, UEdGraph* SourceGraph)`** - Expands during compilation
- **`ValidateNodeDuringCompilation(FCompilerResultsLog& MessageLog)`** - Validates the event node

### Documentation and Help
- **`GetTooltipText()`** - Gets tooltip text for the event
- **`GetKeywords()`** - Gets search keywords for the event
- **`GetDocumentationLink()`** - Gets documentation link
- **`GetDocumentationExcerptName()`** - Gets documentation excerpt name

### Blueprint Integration
- **`DrawNodeAsEntry()`** - Always returns true as events are entry points
- **`CanPasteHere(const UEdGraph* TargetGraph)`** - Checks if event can be pasted in target graph
- **`IsCompatibleWithGraph(const UEdGraph* TargetGraph)`** - Checks graph compatibility
- **`PostDuplicate(bool bDuplicateForPIE)`** - Handles post-duplication logic

### Pin Management
- **`AllocateDefaultPins()`** - Creates the default pins for the event
- **`PostReconstructNode()`** - Handles post-reconstruction logic
- **`PinConnectionListChanged(UEdGraphPin* Pin)`** - Handles pin connection changes

### Search and Reference
- **`AddSearchMetaDataInfo(TArray<struct FSearchTagDataPair>& OutTaggedMetaData)`** - Adds search metadata
- **`GetFindReferenceSearchString_Impl(EGetFindReferenceSearchStringFlags InFlags)`** - Gets reference search string
- **`FindDiffs(UEdGraphNode* OtherNode, FDiffResults& Results)`** - Finds differences with another node

### Event Comparison
- **`AreEventNodesIdentical(const UK2Node_Event* InNodeA, const UK2Node_Event* InNodeB)`** - Static method to compare events

## Usage Patterns

### Common Event Types
1. **BeginPlay** - Called when the object begins play
2. **Tick** - Called every frame
3. **Input Events** - Keyboard, mouse, gamepad input
4. **Collision Events** - OnHit, OnOverlap, etc.
5. **Custom Events** - User-defined events
6. **Interface Events** - Blueprint interface implementations

### Compilation Behavior
During compilation, event nodes:
1. **Validate Event Signature** - Ensures the event reference is valid
2. **Generate Entry Point** - Creates the function entry point
3. **Handle Delegate Binding** - Sets up delegate bindings if needed
4. **Create Parameter Pins** - Generates pins for event parameters

### Network Considerations
- **Authority Checking** - Some events are authority-only
- **Replication** - Events can trigger replicated function calls
- **Cosmetic Events** - Some events are cosmetic-only (client-side)

## Visual Representation
- **Color**: Typically red to indicate event/entry point
- **Shape**: Entry node styling (no input execution pin)
- **Icon**: Lightning bolt or specific event icon
- **Pins**: Output execution pin and parameter pins

## Common Methods Override
- **`GetMenuCategory()`** - Returns "Event" or specific event category
- **`GetNodeAttributes()`** - Provides node attributes for search/filtering
- **`GetJumpTargetForDoubleClick()`** - Supports jumping to event definition

## Important Notes
- Events are always entry points (no input execution pin)
- Can have multiple output execution paths in some cases
- May bind to delegates for dynamic event handling
- Network replication behavior depends on event type
- Some events are editor-only or game-only
- Interface events must match the interface signature exactly

## Related Classes
- **`UK2Node_CustomEvent`** - Custom user-defined events
- **`UK2Node_ActorBoundEvent`** - Events bound to specific actors
- **`UK2Node_ComponentBoundEvent`** - Events bound to components
- **`UK2Node_FunctionEntry`** - Function entry points (similar concept)