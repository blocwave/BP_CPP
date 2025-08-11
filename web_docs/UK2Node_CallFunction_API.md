# UK2Node_CallFunction API Documentation

## Overview
`UK2Node_CallFunction` represents function call nodes in Blueprint graphs. This class creates all the pins required to call a particular UFunction and handles the compilation of function calls.

## Class Definition
```cpp
UCLASS()
class UK2Node_CallFunction : public UK2Node
```

## Inheritance Hierarchy
- `UObject`
  - `UEdGraphNode`
    - `UK2Node`
      - **`UK2Node_CallFunction`**

## Key Properties
- **Function Reference** - References the UFunction to call
- **Self Pin** - Target object pin (for non-static functions)
- **Parameter Pins** - Input pins for function parameters
- **Return Value Pin** - Output pin for function return value
- **Execution Pins** - Input/output execution flow pins

## Core Functionality

### Function Setup and Configuration
- **`SetFromFunction(const UFunction* Function)`** - Sets up the node from a UFunction
- **`GetTargetFunction()`** - Gets the target function being called
- **`GetTargetFunctionFromSkeletonClass()`** - Gets function from skeleton class
- **`GetFunctionName()`** - Gets the name of the function being called
- **`GetFunctionContextString()`** - Gets context string for the function

### Pin Creation and Management
- **`AllocateDefaultPins()`** - Creates all pins required for the function call
- **`CreatePinsForFunctionCall(const UFunction* Function)`** - Creates pins for function parameters
- **`CreateExecPinsForFunctionCall(const UFunction* Function)`** - Creates execution pins
- **`CreateSelfPin(const UFunction* Function)`** - Creates the self/target pin
- **`GetReturnValuePin()`** - Gets the return value pin

### Pin Validation and Behavior
- **`PostParameterPinCreated(UEdGraphPin* Pin)`** - Called after parameter pin creation
- **`PostReconstructNode()`** - Handles post-reconstruction logic
- **`PinDefaultValueChanged(UEdGraphPin* Pin)`** - Handles pin default value changes
- **`CanSplitPin(const UEdGraphPin* Pin)`** - Determines if a pin can be split

### Node Appearance and Information
- **`GetNodeTitle(ENodeTitleType::Type TitleType)`** - Gets the display title
- **`GetCompactNodeTitle()`** - Gets compact title for small display
- **`GetTooltipText()`** - Gets tooltip text
- **`GetIconAndTint(FLinearColor& OutColor)`** - Gets icon and color
- **`GetCornerIcon()`** - Gets corner icon

### Function Analysis
- **`IsLatentFunction()`** - Checks if the function is latent (async)
- **`CanToggleNodePurity()`** - Checks if node purity can be toggled
- **`IsNodePure()`** - Checks if the node is pure (no side effects)
- **`ShouldDrawCompact()`** - Determines compact drawing mode

### Multiple Target Support
- **`AllowMultipleSelfs(bool bInputAsArray)`** - Checks if multiple targets are allowed
- **`CanFunctionSupportMultipleTargets(UFunction const* InFunction)`** - Static check for multiple target support
- **`CallForEachElementInArrayExpansion(...)`** - Handles array expansion for multiple targets

### Compilation Support
- **`CreateNodeHandler(FKismetCompilerContext& CompilerContext)`** - Creates function call handler
- **`ExpandNode(FKismetCompilerContext& CompilerContext, UEdGraph* SourceGraph)`** - Expands during compilation
- **`ValidateNodeDuringCompilation(FCompilerResultsLog& MessageLog)`** - Validates the function call

### Wildcard and Dynamic Pins
- **`IsWildcardProperty(const UFunction* InFunction, const FProperty* InProperty)`** - Checks for wildcard properties
- **`IsStructureWildcardProperty(const UFunction* InFunction, const FName PropertyName)`** - Checks for structure wildcards
- **`GetExpandEnumPinNames(const UFunction* Function, TArray<FName>& EnumNamesToCheck)`** - Gets expandable enum pins

### Function Metadata and Documentation
- **`GetDefaultTooltipForFunction(const UFunction* Function)`** - Gets default tooltip
- **`GetDefaultCategoryForFunction(const UFunction* Function, const FText& BaseCategory)`** - Gets default category
- **`GetKeywordsForFunction(const UFunction* Function)`** - Gets search keywords
- **`GetUserFacingFunctionName(const UFunction* Function, ENodeTitleType::Type NodeTitleType)`** - Gets display name

### Pin Tooltips and Help
- **`GeneratePinTooltipFromFunction(UEdGraphPin& Pin, const UFunction* Function)`** - Generates pin tooltips
- **`GetPinHoverText(const UEdGraphPin& Pin, FString& HoverTextOut)`** - Gets pin hover text
- **`InvalidatePinTooltips()`** - Invalidates cached pin tooltips

### Editor Integration
- **`CanPasteHere(const UEdGraph* TargetGraph)`** - Checks if node can be pasted
- **`CanJumpToDefinition()`** - Checks if can jump to function definition
- **`JumpToDefinition()`** - Jumps to the function definition
- **`GetNodeContextMenuActions(UToolMenu* Menu, UGraphNodeContextMenuContext* Context)`** - Gets context menu actions

### Function Validation
- **`ValidateRequiredPins(const UFunction* Function, FCompilerResultsLog& MessageLog)`** - Validates required pins
- **`GetRequiredParamNames(const UFunction* ForFunction)`** - Gets required parameter names
- **`HasDeprecatedReference()`** - Checks for deprecated function references
- **`HasExternalDependencies(TArray<class UStruct*>* OptionalOutput)`** - Checks external dependencies

### Special Function Types
- **`CanEditorOnlyFunctionBeCalled(const UFunction* InFunction, const UObject* InObject)`** - Editor-only function validation
- **`FixupSelfMemberContext()`** - Fixes up self member context

### Static Utility Methods
- **`GetCompactNodeTitle(const UFunction* Function)`** - Static method for compact title
- **`ShouldDrawCompact(const UFunction* Function)`** - Static method for compact drawing
- **`GetPaletteIconForFunction(UFunction const* Function, FLinearColor& OutColor)`** - Gets palette icon

## Function Call Types

### Pure Functions
- No execution pins
- Called automatically when output is needed
- No side effects
- Can be called multiple times per frame

### Impure Functions
- Have execution input/output pins
- Execute in sequence
- May have side effects
- Execute once when triggered

### Latent Functions
- Async/delayed execution
- Special latent execution pin
- Common in gameplay (delays, timers, etc.)

### Static Functions
- No self pin
- Called on class rather than instance
- Utility functions

## Compilation Behavior
During compilation, function call nodes:
1. **Validate Function Reference** - Ensures function exists and is accessible
2. **Check Parameters** - Validates all required parameters are connected
3. **Generate Function Call** - Creates appropriate bytecode for function call
4. **Handle Return Values** - Sets up return value handling
5. **Process Wildcards** - Resolves wildcard parameter types

## Visual Representation
- **Color**: Blue for pure functions, white for impure
- **Shape**: Standard node with input/output pins
- **Icons**: Function-specific icons or default function icon
- **Pins**: Self, parameters, return value, execution (if impure)

## Important Notes
- Function calls are the most common node type in Blueprints
- Must match UFunction signature exactly
- Handles both C++ and Blueprint functions
- Supports function overloading resolution
- Can expand for multiple target calls
- Wildcard pins resolve types at compile time
- Editor-only functions have special handling

## Related Classes
- **`UK2Node_CallFunctionOnMember`** - Member function calls
- **`UK2Node_CallParentFunction`** - Parent class function calls
- **`UK2Node_CallDelegate`** - Delegate calls
- **`UK2Node_BaseMCDelegate`** - Multicast delegate calls