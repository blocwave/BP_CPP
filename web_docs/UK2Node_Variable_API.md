# UK2Node_Variable API Documentation

## Overview
`UK2Node_Variable` is the abstract base class for variable access nodes in Blueprint graphs. It handles both reading from and writing to variables, including Blueprint variables, component properties, and member variables.

## Class Definition
```cpp
UCLASS(Abstract)
class UK2Node_Variable : public UK2Node
```

## Inheritance Hierarchy
- `UObject`
  - `UEdGraphNode`
    - `UK2Node`
      - **`UK2Node_Variable`** (Abstract)

## Key Properties
- **Variable Reference** - References the property being accessed
- **Variable Scope** - The class/struct that contains the variable
- **Access Type** - Get or Set operation
- **Self Pin** - Target object pin (for instance variables)
- **Value Pin** - The variable value pin

## Core Functionality

### Variable Configuration
- **`SetFromProperty(const FProperty* Property, bool bSelfContext, UClass* OwnerClass)`** - Sets up from a property
- **`GetPropertyForVariable()`** - Gets the property this variable references
- **`GetPropertyForVariableFromSkeleton()`** - Gets property from skeleton class
- **`GetVarName()`** - Gets the variable name
- **`GetVarNameString()`** - Gets variable name as string
- **`GetVarNameText()`** - Gets variable name as text

### Pin Creation and Management
- **`CreatePinForVariable(EEdGraphPinDirection Direction, FName InPinName)`** - Creates a pin for the variable
- **`CreatePinForSelf()`** - Creates the self/target pin
- **`RecreatePinForVariable(EEdGraphPinDirection Direction, TArray<UEdGraphPin*>& OldPins, FName InPinName)`** - Recreates variable pin
- **`GetValuePin()`** - Gets the value pin for the variable

### Variable Information and Validation
- **`GetVariableSourceClass()`** - Gets the class that owns the variable
- **`CheckForErrors(const UEdGraphSchema_K2* Schema, FCompilerResultsLog& MessageLog)`** - Validates the variable node
- **`GetBlueprintVarDescription()`** - Gets Blueprint variable description

### Variable Icons and Appearance
- **`GetIconAndTint(FLinearColor& OutColor)`** - Gets icon and color for the variable
- **`GetVariableIconAndColor(const UStruct* VarScope, FName VarName, FLinearColor& IconColorOut)`** - Static method for variable icon
- **`GetVarIconFromPinType(const FEdGraphPinType& InPinType, FLinearColor& IconColorOut)`** - Gets icon from pin type
- **`DrawNodeAsVariable()`** - Always returns true as these are variable nodes

### Node Appearance
- **`GetNodeTitleColor()`** - Gets the color for variable nodes (typically green)
- **`GetCornerIcon()`** - Gets corner icon indicating variable type
- **`GetDocumentationLink()`** - Gets documentation link
- **`GetToolTipHeading()`** - Gets tooltip heading

### Variable Renaming and References
- **`HandleVariableRenamed(UBlueprint* InBlueprint, UClass* InVariableClass, UEdGraph* InGraph, const FName& InOldVarName, const FName& InNewVarName)`** - Handles variable renaming
- **`ReferencesVariable(const FName& InVarName, const UStruct* InScope)`** - Checks if references a variable
- **`DoesRenamedVariableMatch(FName OldVariableName, FName NewVariableName, UStruct* StructType)`** - Checks renamed variable match

### Component and Actor Integration
- **`GetActorComponent(const FProperty* VariableProperty)`** - Gets actor component for component variables
- **`FunctionParameterExists(const UEdGraph* InFunctionGraph, const FName InParameterName)`** - Checks function parameter existence

### Blueprint Integration
- **`AutowireNewNode(UEdGraphPin* FromPin)`** - Autowires variable node when placed
- **`CanJumpToDefinition()`** - Checks if can jump to variable definition
- **`JumpToDefinition()`** - Jumps to variable definition
- **`CanPasteHere(const UEdGraph* TargetGraph)`** - Checks paste compatibility

### Node Reconstruction
- **`ReconstructNode()`** - Reconstructs the variable node
- **`DoPinsMatchForReconstruction(const UEdGraphPin* NewPin, int32 NewPinIndex, const UEdGraphPin* OldPin, int32 OldPinIndex)`** - Matches pins during reconstruction

### Reference Management
- **`ReplaceReferences(UBlueprint* InBlueprint, UBlueprint* InReplacementBlueprint, const FMemberReference& InSource, const FMemberReference& InReplacement)`** - Replaces references
- **`RemapRestrictedLinkReference(...)`** - Remaps restricted link references

### Validation and Dependencies
- **`ValidateNodeDuringCompilation(FCompilerResultsLog& MessageLog)`** - Validates during compilation
- **`HasDeprecatedReference()`** - Checks for deprecated variable references
- **`HasExternalDependencies(TArray<class UStruct*>* OptionalOutput)`** - Checks external dependencies

### Editor Integration
- **`GetNodeContextMenuActions(UToolMenu* Menu, UGraphNodeContextMenuContext* Context)`** - Gets context menu
- **`PostPasteNode()`** - Handles post-paste logic
- **`GetNodeAttributes(TArray<TKeyValuePair<FString, FString>>& OutNodeAttributes)`** - Gets node attributes

### Search and Reference
- **`GetFindReferenceSearchString_Impl(EGetFindReferenceSearchStringFlags InFlags)`** - Gets reference search string
- **`GetDocumentationExcerptName()`** - Gets documentation excerpt name

### Deprecation Support
- **`GetDeprecationResponse(EEdGraphNodeDeprecationType DeprecationType)`** - Handles deprecation
- **`SuppressDeprecationWarning()`** - Suppresses deprecation warnings

### Metadata and Information
- **`GetPinMetaData(FName InPinName, FName InKey)`** - Gets pin metadata

## Variable Types

### Blueprint Variables
- Variables defined in Blueprint class
- Can be public, private, or protected
- Support default values and tooltips

### Component Variables
- References to actor components
- Automatically create component reference pins
- Support component-specific operations

### Member Variables
- C++ class member variables
- Exposed through UPROPERTY
- Include native and Blueprint-accessible properties

### Function Parameters
- Local variables in function graphs
- Input and output parameters
- Local variables created in functions

## Access Patterns

### Get Variables
- Read access to variable values
- Output pin provides the value
- Pure operation (no execution pins)

### Set Variables
- Write access to variable values
- Input pin accepts the new value
- Impure operation (has execution pins)

## Compilation Behavior
During compilation, variable nodes:
1. **Validate Variable Reference** - Ensures variable exists and is accessible
2. **Check Access Permissions** - Validates read/write permissions
3. **Generate Property Access** - Creates appropriate property access code
4. **Handle Self Context** - Resolves target object for instance variables
5. **Type Checking** - Ensures type compatibility

## Visual Representation
- **Color**: Green for variables
- **Shape**: Variable-specific styling
- **Icons**: Variable type icons (bool, int, float, object, etc.)
- **Pins**: Self pin (if needed), value pin, execution pins (for set)

## Derived Classes
- **`UK2Node_VariableGet`** - Variable read access
- **`UK2Node_VariableSet`** - Variable write access

## Important Notes
- Abstract class - use specific Get/Set derived classes
- Handles both Blueprint and native C++ variables
- Component variables have special handling
- Function parameters are treated as local variables
- Supports variable renaming and refactoring
- Type safety enforced at compile time
- Self context automatically resolved for instance variables

## Related Classes
- **`UK2Node_VariableGet`** - Specific implementation for getting variables
- **`UK2Node_VariableSet`** - Specific implementation for setting variables
- **`FProperty`** - Unreal's property system
- **`FMemberReference`** - References to class members