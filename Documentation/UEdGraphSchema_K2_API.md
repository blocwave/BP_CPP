# UEdGraphSchema_K2 API Documentation

Source: https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Editor/BlueprintGraph/UEdGraphSchema_K2?application_version=5.2

## Class Definition

```cpp
class UEdGraphSchema_K2 : public UEdGraphSchema
```

## Purpose

UEdGraphSchema_K2 is the schema class that defines the rules, behaviors, and visual appearance for Blueprint graphs. It serves as the central authority for all Blueprint graph operations including connection validation, node creation, pin type management, and graph editing behaviors. The "K2" designation refers to Kismet 2, the internal name for the Blueprint system.

## Inheritance Hierarchy

- `UObject`
  - `UEdGraphSchema`
    - **`UEdGraphSchema_K2`** (Blueprint graph schema)

## Key Responsibilities

### Connection Validation
- Determines which pins can be connected to each other
- Validates type compatibility between different pin types
- Handles type promotion and conversion rules
- Manages execution flow connections vs data connections

### Pin Type Management
- Defines all Blueprint pin types (boolean, integer, float, string, object references, etc.)
- Handles wildcard pin resolution and type inference
- Manages array and reference type behaviors
- Supports custom struct and enum types

### Node Creation and Menu Population
- Populates context menus with available nodes
- Filters nodes based on context and Blueprint type
- Handles node spawning and initial setup
- Manages node categories and organization

### Graph Editing Operations
- Defines drag-and-drop behaviors
- Handles pin splitting and combining
- Manages connection creation and destruction
- Supports graph refactoring operations

## Core Functionality

### Connection Management
- **`CanCreateConnection()`** - Validates potential pin connections
- **`TryCreateConnection()`** - Attempts to create a connection between pins
- **`CreateAutomaticConversionNodeAndConnections()`** - Inserts conversion nodes when needed
- **`GetConnectionResponse()`** - Provides detailed connection validation results

### Pin Type Operations
- **`ArePinsCompatible()`** - Checks if two pins can be connected
- **`DoesSupportPinWatching()`** - Determines if a pin supports debug watching
- **`IsPinDefaultValueEditable()`** - Checks if pin default values can be edited
- **`GetPinDisplayName()`** - Gets the display name for pins

### Node and Menu Management
- **`GetGraphContextActions()`** - Populates context menus with available actions
- **`GetContextMenuActions()`** - Gets context-specific menu options
- **`CreateDefaultNodesForGraph()`** - Creates default nodes for new graphs
- **`HandleGraphBeingDeleted()`** - Cleanup when graphs are deleted

### Type System Support
- **`GetDefaultValueForPin()`** - Gets appropriate default values for pin types
- **`DoesUserAllowPlacingNodeOfClass()`** - User permission checks for node placement
- **`CanPromotePinToVariable()`** - Checks if pins can become variables
- **`PromotePinToVariable()`** - Converts pins to Blueprint variables

### Graph Validation
- **`CanDeleteNode()`** - Determines if nodes can be deleted
- **`CanDeleteConnection()`** - Validates connection deletion
- **`CanCreateNewLocalVariable()`** - Checks local variable creation permissions
- **`ValidateGraphIsWellFormed()`** - Comprehensive graph structure validation

### Blueprint-Specific Features
- **`GetBlueprintVarActionMenuBuilder()`** - Creates variable-related menu actions
- **`GetFunctionMenuBuilder()`** - Creates function-related menu actions
- **`GetMacroActionMenuBuilder()`** - Creates macro-related menu actions
- **`GetEventActionMenuBuilder()`** - Creates event-related menu actions

## Pin Types and Categories

### Execution Pins
- **Exec** - White execution flow pins for sequencing
- **Delegate** - Event delegate connections

### Data Types
- **Boolean** - True/false values (red pins)
- **Integer** - Whole numbers (cyan pins)  
- **Float** - Decimal numbers (green pins)
- **String** - Text data (magenta/pink pins)
- **Name** - Unreal name identifiers (pink pins)
- **Text** - Localized text (pink pins)

### Object References
- **Object** - UObject-derived class references (blue pins)
- **Class** - Class type references (purple pins)
- **SoftObject** - Soft references to objects
- **Interface** - Interface implementations

### Containers
- **Array** - Collections of other types
- **Map** - Key-value pair collections
- **Set** - Unique value collections

### Special Types
- **Struct** - Custom data structures
- **Enum** - Enumerated values
- **Wildcard** - Type-flexible pins that resolve at compile time

## Connection Rules

### Type Compatibility
- Exact type matches are always allowed
- Numeric types can promote (int to float)
- Object references follow inheritance hierarchy
- Array types must have compatible inner types

### Execution Flow
- Execution pins connect in sequence (white exec pins)
- Data pins provide values (colored data pins)
- Mixed execution/data connections are not allowed

### Special Behaviors
- Wildcard pins adapt to connected types
- Delegate pins connect to compatible function signatures
- Interface pins can connect to implementing objects

## Usage in Blueprint System

The schema coordinates with other Blueprint systems:

1. **Editor Integration** - Works with Blueprint editor UI for visual feedback
2. **Compilation** - Provides validation during Blueprint compilation
3. **Node System** - Cooperates with UK2Node derivatives for node-specific behaviors  
4. **Type System** - Integrates with Unreal's reflection system for type information

## Customization Points

### Virtual Methods
Many behaviors can be customized by overriding virtual methods:
- Connection validation logic
- Menu population
- Pin appearance and behavior
- Type conversion rules

### Blueprint Type Variations
Different Blueprint types (Actor, Object, Interface, etc.) may use specialized schemas or override specific behaviors.

## Important Notes

- This class is central to all Blueprint graph operations
- Handles both editor-time validation and compilation-time checks
- Must maintain consistency between visual representation and compilation results
- Performance is critical as it's called frequently during editor operations
- Thread safety considerations for editor responsiveness
- Extensible for custom Blueprint node types and behaviors

## Related Classes

- **UBlueprint** - The Blueprint asset using this schema
- **UEdGraphNode** / **UK2Node** - Nodes governed by this schema
- **UEdGraphPin** - Pins managed by this schema
- **FKismetCompilerContext** - Uses schema during compilation
- **FBlueprintActionDatabase** - Action registry for menu population