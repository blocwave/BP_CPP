# FKismetCompilerContext API Documentation

Source: https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Editor/KismetCompiler/FKismetCompilerContext?application_version=5.2

## Class Definition

```cpp
class FKismetCompilerContext : public FGraphCompilerContext
```

## Purpose

FKismetCompilerContext is the main compilation context for Blueprint classes. It orchestrates the entire Blueprint compilation process, from analyzing the Blueprint graph structure to generating the final UBlueprintGeneratedClass with executable code. This class is central to the Blueprint compilation pipeline and manages all aspects of converting visual Blueprint graphs into runtime-executable classes.

## Inheritance Hierarchy

- `FGraphCompilerContext`
  - **`FKismetCompilerContext`**
    - `FRigVMBlueprintCompilerContext`
    - `FWidgetBlueprintCompilerContext`
    - `FControlRigBlueprintCompilerContext`

## Module Information

- **Module**: KismetCompiler  
- **Header**: /Engine/Source/Editor/KismetCompiler/Public/KismetCompiler.h
- **Include**: #include "KismetCompiler.h"

## Key Properties

### Blueprint Reference
- **`Blueprint`** (UBlueprint*) - The Blueprint being compiled
- **`NewClass`** (UBlueprintGeneratedClass*) - The class being generated during compilation
- **`OldClass`** (UBlueprintGeneratedClass*) - Previous version of the class (for recompilation)
- **`OldCDO`** (UObject*) - Previous Class Default Object
- **`TargetClass`** (UBlueprintGeneratedClass*) - Target class for compilation

### Compilation State
- **`bIsFullCompile`** (int32: 1) - Whether this is a full compilation or incremental
- **`CompileOptions`** (FKismetCompilerOptions) - Options controlling compilation behavior

### Graph Management  
- **`ConsolidatedEventGraph`** (UEdGraph*) - The merged ubergraph containing all event graphs
- **`UbergraphContext`** (FKismetFunctionContext*) - Context for the main ubergraph execution function
- **`FunctionList`** (TIndirectArray<FKismetFunctionContext>) - List of all functions being compiled

### Schema and Handlers
- **`Schema`** (UEdGraphSchema_K2*) - Blueprint graph schema for validation and operations
- **`NodeHandlers`** (TMap<TSubclassOf<UEdGraphNode>, FNodeHandlingFunctor*>) - Map of node types to their compilation handlers

### Name Management
- **`ClassScopeNetNameMap`** (FNetNameMapping) - Network name mapping for class scope
- **`CreatedFunctionNames`** (TSet<FString>) - Names of functions created during compilation

## Core Functionality

### Main Compilation Process
- **`Compile()`** - Main entry point for Blueprint compilation
- **`CompileClassLayout(EInternalCompilerFlags InternalFlags)`** - Compiles the class structure and layout
- **`CompileFunctions(EInternalCompilerFlags InternalFlags)`** - Compiles all Blueprint functions

### Class Management
- **`CleanAndSanitizeClass(UBlueprintGeneratedClass* ClassToClean, UObject*& OldCDO)`** - Prepares class for recompilation
- **`SpawnNewClass(const FString& NewClassName)`** - Creates new Blueprint generated class
- **`SetNewClass(UBlueprintGeneratedClass* ClassToUse)`** - Sets the target class for compilation
- **`EnsureProperGeneratedClass(UClass*& TargetClass)`** - Validates target class type

### Function Compilation
- **`CreateFunctionList()`** - Builds list of functions to compile from Blueprint graphs
- **`CreateFunctionContext()`** - Creates compilation context for individual functions
- **`PrecompileFunction(FKismetFunctionContext& Context, EInternalCompilerFlags InternalFlags)`** - Pre-compilation setup
- **`CompileFunction(FKismetFunctionContext& Context)`** - Main function compilation
- **`PostcompileFunction(FKismetFunctionContext& Context)`** - Post-compilation cleanup

### Graph Processing
- **`CreateAndProcessUbergraph()`** - Merges event graphs and creates main execution function
- **`MergeUbergraphPagesIn(UEdGraph* Ubergraph)`** - Combines multiple event graph pages
- **`ProcessOneFunctionGraph(UEdGraph* SourceGraph, bool bInternalFunction)`** - Processes individual function graphs
- **`ExpansionStep(UEdGraph* Graph, bool bAllowUbergraphExpansions)`** - Expands nodes that require it
- **`ExpandTunnelsAndMacros(UEdGraph* SourceGraph)`** - Expands macro instances and collapse tunnels

### Pin and Connection Management
- **`MovePinLinksToIntermediate(UEdGraphPin& SourcePin, UEdGraphPin& IntermediatePin)`** - Safely moves pin connections
- **`CopyPinLinksToIntermediate(UEdGraphPin& SourcePin, UEdGraphPin& IntermediatePin)`** - Copies pin connections
- **`CheckConnectionResponse(const FPinConnectionResponse& Response, const UEdGraphNode* Node)`** - Validates connection operations

### Variable and Property Management
- **`CreateClassVariablesFromBlueprint()`** - Creates class properties from Blueprint variables
- **`CreateVariable(const FName Name, const FEdGraphPinType& Type)`** - Creates new class variable
- **`CreatePropertiesFromList(...)`** - Creates properties from terminal lists
- **`SetPropertyDefaultValue(const FProperty* PropertyToSet, FString& Value)`** - Sets default values for properties

### Node Spawning and Management
- **`SpawnIntermediateNode<NodeType>(UEdGraphNode* SourceNode, UEdGraph* ParentGraph)`** - Creates intermediate nodes during compilation
- **`SpawnIntermediateEventNode<NodeType>(...)`** - Creates intermediate event nodes
- **`SpawnInternalVariable(...)`** - Creates temporary variables for compilation
- **`SpawnIntermediateFunctionGraph(...)`** - Creates intermediate function graphs

### Validation
- **`ValidateGeneratedClass(UBlueprintGeneratedClass* Class)`** - Validates the compiled class
- **`ValidateFunctionGraphNames()`** - Ensures function names are valid
- **`ValidateVariableNames()`** - Validates Blueprint variable names
- **`ValidateTimelineNames()`** - Validates timeline component names
- **`ValidateLink(const UEdGraphPin* PinA, const UEdGraphPin* PinB)`** - Validates pin connections

### Utility Methods
- **`GetSchema()`** - Returns the Blueprint graph schema
- **`GetCompilerForBP(UBlueprint* BP, ...)`** - Static method to get appropriate compiler for Blueprint type
- **`RegisterCompilerForBP(UClass* BPClass, CompilerContextFactoryFunction FactoryFunction)`** - Registers custom compilers

## Compilation Pipeline

The Blueprint compilation process follows these major steps:

1. **Pre-Compilation** - Initialize compiler context and prepare Blueprint
2. **Class Layout** - Create class structure, properties, and function signatures  
3. **Function List Creation** - Identify all functions that need compilation
4. **Graph Processing** - Merge event graphs, expand macros, validate connections
5. **Function Compilation** - Convert graph logic to executable statements
6. **Post-Compilation** - Finalize class, set metadata, validate results

## Usage Examples

```cpp
// Creating a compiler context
FKismetCompilerContext CompilerContext(
    SourceBlueprint, 
    MessageLog, 
    CompilerOptions
);

// Full compilation
CompilerContext.Compile();

// Or separate phases
CompilerContext.CompileClassLayout(InternalFlags);
CompilerContext.CompileFunctions(InternalFlags);
```

## Important Notes

- This class orchestrates the entire Blueprint compilation process
- Handles both full compilation and incremental recompilation
- Manages complex graph transformations including macro expansion
- Creates and manages intermediate representations during compilation
- Validates all aspects of Blueprint structure and connections
- Thread-safe compilation is critical for editor performance
- Custom Blueprint types can provide specialized compiler contexts

## Related Classes

- **UBlueprint** - The Blueprint asset being compiled
- **UBlueprintGeneratedClass** - The output class generated by compilation
- **FKismetFunctionContext** - Individual function compilation context
- **UEdGraphSchema_K2** - Blueprint graph schema for validation
- **FNodeHandlingFunctor** - Base class for node-specific compilation handlers