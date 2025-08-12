# FKismetFunctionContext API Documentation

**Module**: KismetCompiler  
**Header**: `/Engine/Source/Editor/KismetCompiler/Public/KismetCompiledFunctionContext.h`  
**Include**: `#include "KismetCompiledFunctionContext.h"`

## Overview

`FKismetFunctionContext` is a critical structure in the Kismet compilation pipeline that represents the compilation context for a single Blueprint function. It manages the compilation process including statement generation, variable management, terminal creation, and execution flow control.

## Class Definition

```cpp
struct FKismetFunctionContext
```

## Key Properties

### Core Context Data
- **`UBlueprint* Blueprint`** - Blueprint source being compiled
- **`UBlueprintGeneratedClass* NewClass`** - The generated class being created
- **`UFunction* Function`** - The UFunction being compiled
- **`UEdGraph* SourceGraph`** - Source graph for the function
- **`UK2Node_FunctionEntry* EntryPoint`** - Function entry node
- **`const UEdGraphSchema_K2* Schema`** - K2 schema for validation
- **`FCompilerResultsLog& MessageLog`** - Compilation message log

### Statement Management
- **`TArray<FBlueprintCompiledStatement*> AllGeneratedStatements`** - All statements for cleanup
- **`TMap<UEdGraphNode*, TArray<FBlueprintCompiledStatement*>> StatementsPerNode`** - Statements organized by node
- **`TArray<UEdGraphNode*> LinearExecutionList`** - Linear execution schedule

### Variable and Terminal Management
- **`TIndirectArray<FBPTerminal> Locals`** - Local variables
- **`TIndirectArray<FBPTerminal> Parameters`** - Function parameters
- **`TIndirectArray<FBPTerminal> Results`** - Return values
- **`TIndirectArray<FBPTerminal> Literals`** - Literal values
- **`TIndirectArray<FBPTerminal> VariableReferences`** - Variable references
- **`TIndirectArray<FBPTerminal> EventGraphLocals`** - Event graph local variables

### Net and Mapping Management
- **`TMap<UEdGraphPin*, FBPTerminal*> NetMap`** - Map from pins to terminals
- **`TMap<UEdGraphPin*, FBPTerminal*> LiteralHackMap`** - Special literal mappings
- **`FNetNameMapping* NetNameMap`** - Variable name mapping
- **`TMap<UEdGraphPin*, UE::KismetCompiler::CastingUtils::FImplicitCastParams> ImplicitCastMap`** - Implicit cast information

### Flow Control
- **`TMap<FBlueprintCompiledStatement*, UEdGraphPin*> GotoFixupRequestMap`** - Goto fixup requests
- **`bool bUseFlowStack`** - Whether function requires flow stack

## Key Methods

### Statement Management

#### `FBlueprintCompiledStatement& AppendStatementForNode(UEdGraphNode* Node)`
Enqueues a statement to be executed when the specified node is triggered.

#### `FBlueprintCompiledStatement& PrependStatementForNode(UEdGraphNode* Node)`
Prepends a statement to the execution list for a node.

#### `void CopyAndPrependStatements(UEdGraphNode* Destination, UEdGraphNode* Source)`
Prepends statements from Source to Destination's statement list.

#### `void ResolveStatements()`
Links gotos, sorts statements, and merges adjacent ones. This is a critical compilation step.

### Terminal and Variable Management

#### `FBPTerminal* CreateLocalTerminal(ETerminalSpecification Spec)`
Creates a new local terminal with the specified properties.

#### `FBPTerminal* CreateLocalTerminalFromPinAutoChooseScope(UEdGraphPin* Net, FString NewName)`
Creates a terminal from a pin, automatically choosing the appropriate scope.

#### `FBPTerminal* RegisterLiteral(UEdGraphPin* Net)`
Registers a literal value terminal for the specified pin.

### Function Properties and Validation

#### `void MarkAsConstFunction(bool bInEnforceConstCorrectness)`
Marks the function as const and optionally enforces const correctness.

#### `void MarkAsEventGraph()`
Marks this context as representing an event graph.

#### `void MarkAsInterfaceStub()`
Marks this context as an interface stub function.

#### `void MarkAsNetFunction(uint32 InFunctionFlags)`
Marks the function as a networking function with specified flags.

#### `bool ValidatePinType(const UEdGraphPin* Pin, const FEdGraphPinType& TestType)`
Validates that a pin matches the expected type.

### Utility Methods

#### `UEdGraphPin* FindRequiredPinByName(const UEdGraphNode* Node, const FName PinName, EEdGraphPinDirection RequiredDirection)`
Finds a pin by name and validates its direction.

#### `UStruct* GetScopeFromPinType(FEdGraphPinType& Type, UClass* SelfClass)`
Returns the appropriate scope (UStruct) for a given pin type.

#### `bool DidNodeGenerateCode(UEdGraphNode* Node)`
Returns true if the specified node generated any compiled statements.

#### `void InsertWireTrace(FBlueprintCompiledStatement* GotoStatement, UEdGraphPin* AssociatedExecPin)`
Inserts wire trace debugging information for execution flow.

## Boolean Flags and State

### Function Type Flags
- **`bIsUbergraph`** - True if this represents an event graph
- **`bIsInterfaceStub`** - True if this is an interface stub
- **`bIsConstFunction`** - True if the function is const
- **`bCannotBeCalledFromOtherKismet`** - True if function is internal-only
- **`bIsSimpleStubGraphWithNoParams`** - True for simple parameter-less stubs

### Compilation Flags
- **`bCreateDebugData`** - Whether to generate debug information
- **`bEnforceConstCorrectness`** - Whether to enforce const correctness
- **`bAllocatedNetNameMap`** - Whether we allocated our own name map

## Usage in Compilation Pipeline

### 1. Initialization Phase
The context is created with references to the blueprint, class, and compilation environment.

### 2. Net Registration Phase
Pins are registered and terminals are created for variables, literals, and intermediate values.

### 3. Statement Generation Phase
Nodes are processed and converted into `FBlueprintCompiledStatement` objects that represent the actual execution logic.

### 4. Resolution Phase
`ResolveStatements()` is called to:
- Link goto statements to their targets
- Sort statements into execution order
- Merge adjacent statements for optimization
- Validate the compilation results

### 5. Backend Generation Phase
The resolved statements are passed to the backend (C++ or VM) for final code generation.

## Relationship to Other Compiler Components

- **FKismetCompilerContext**: The parent compiler context that manages multiple function contexts
- **FNodeHandlingFunctor**: Objects that process individual nodes and generate statements
- **FBlueprintCompiledStatement**: The individual execution statements generated during compilation
- **FBPTerminal**: Represents variables, literals, and intermediate values used in statements

## Best Practices

1. Always validate pin types before creating terminals
2. Use the appropriate terminal creation method based on scope requirements
3. Process statements in the correct order: generation -> resolution -> backend
4. Properly manage net name mapping to avoid variable name conflicts
5. Use debugging features (wire traces, breakpoints) for complex compilation scenarios

## Notes

- The context manages memory for all terminals through `TIndirectArray` containers
- Statement resolution is a critical phase that must be completed before backend code generation
- The context supports both event graphs (ubergraphs) and regular function graphs
- Network replication flags are managed through the `NetFlags` and related methods