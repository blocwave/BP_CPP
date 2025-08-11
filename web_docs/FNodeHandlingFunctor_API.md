# FNodeHandlingFunctor API Documentation

**Module**: KismetCompiler  
**Header**: `/Engine/Source/Editor/KismetCompiler/Public/KismetCompilerMisc.h`  
**Include**: `#include "KismetCompilerMisc.h"`

## Overview

`FNodeHandlingFunctor` is the base class for all node handlers in the Kismet compilation system. It provides the interface and common functionality for processing Blueprint graph nodes and converting them into compiled statements during the compilation pipeline.

## Class Definition

```cpp
class FNodeHandlingFunctor
```

## Inheritance Hierarchy

`FNodeHandlingFunctor` is the base class for numerous specialized node handlers:
- **FKCHandler_CallFunction** - Handles function call nodes
- **FKCHandler_EventEntry** - Handles event entry nodes
- **FKCHandler_MakeContainer** - Handles container creation nodes
- **FKCHandler_MathExpression** - Handles math expression nodes
- **FKCHandler_Passthru** - Handles pass-through nodes
- **FKCHandler_VariableSet** - Handles variable assignment nodes

## Core Properties

### **`FKismetCompilerContext& CompilerContext`**
Reference to the main compiler context that provides access to compilation state, message logging, and global compilation resources.

## Constructor and Destructor

### Constructor
```cpp
FNodeHandlingFunctor(FKismetCompilerContext& InCompilerContext)
```
Initializes the functor with a reference to the compiler context.

### Virtual Destructor
```cpp
virtual ~FNodeHandlingFunctor()
```
Virtual destructor to ensure proper cleanup of derived classes.

## Core Virtual Methods

### **`virtual void RegisterNets(FKismetFunctionContext& Context, UEdGraphNode* Node)`**
Registers all pins on the node as nets (connections) in the function context. This is typically called during the net registration phase before statement generation.

**Parameters:**
- `Context` - The function compilation context
- `Node` - The graph node being processed

### **`virtual void Compile(FKismetFunctionContext& Context, UEdGraphNode* Node)`**
The main compilation method that converts the node into compiled statements. This is where the node's logic is translated into executable form.

**Parameters:**
- `Context` - The function compilation context
- `Node` - The graph node being compiled

### **`virtual void Transform(FKismetFunctionContext& Context, UEdGraphNode* Node)`**
Optional transformation phase for nodes that need to modify the graph structure before compilation.

**Parameters:**
- `Context` - The function compilation context
- `Node` - The graph node being transformed

### **`virtual bool RequiresRegisterNetsBeforeScheduling() const`**
Returns true if this node type needs net registration before execution scheduling. This is primarily used for function entry and return nodes.

**Returns:** Boolean indicating if early net registration is required

## Registration and Net Management Methods

### **`virtual void RegisterNet(FKismetFunctionContext& Context, UEdGraphPin* Pin)`**
Registers a single pin as a net in the function context. Creates appropriate terminals for the pin based on its type and usage.

**Parameters:**
- `Context` - The function compilation context
- `Pin` - The pin to register

## Protected Helper Methods

### **`virtual FBPTerminal* RegisterLiteral(FKismetFunctionContext& Context, UEdGraphPin* Net)`**
Helper method to register a literal terminal for a pin that contains a constant value.

**Parameters:**
- `Context` - The function compilation context
- `Net` - The pin containing the literal value

**Returns:** Pointer to the created literal terminal

### **`void ResolveAndRegisterScopedTerm(FKismetFunctionContext& Context, UEdGraphPin* Net, TIndirectArray<FBPTerminal>& NetArray)`**
Helper function that verifies variable references exist in the appropriate scope and creates terminals for variable access.

**Parameters:**
- `Context` - The function compilation context
- `Net` - The pin referencing the variable
- `NetArray` - The terminal array to add the new terminal to

### **`bool ValidateAndRegisterNetIfLiteral(FKismetFunctionContext& Context, UEdGraphPin* Net)`**
Validates and registers a net if it contains a literal value.

**Parameters:**
- `Context` - The function compilation context
- `Net` - The pin to check and potentially register

**Returns:** True if the pin was a literal and was registered

## Statement Generation Helpers

### **`FBlueprintCompiledStatement& GenerateSimpleThenGoto(FKismetFunctionContext& Context, UEdGraphNode& Node)`**
Generates a simple goto statement for the "then" execution pin(s) of a node.

**Parameters:**
- `Context` - The function compilation context
- `Node` - The node to generate the goto for

**Returns:** Reference to the generated statement

### **`FBlueprintCompiledStatement& GenerateSimpleThenGoto(FKismetFunctionContext& Context, UEdGraphNode& Node, UEdGraphPin* ThenExecPin)`**
Generates a goto statement for a specific execution pin.

**Parameters:**
- `Context` - The function compilation context
- `Node` - The node to generate the goto for
- `ThenExecPin` - The specific execution pin to target

**Returns:** Reference to the generated statement

## Utility Methods

### **`static void SanitizeName(FString& Name)`**
Creates a sanitized version of a variable name by removing or replacing invalid characters.

**Parameters:**
- `Name` - The name to sanitize (modified in place)

## Usage in Compilation Pipeline

### 1. Registration Phase
The compiler calls `RegisterNets()` on each node handler to create terminals for all pins on the node. This establishes the data flow connections.

### 2. Transform Phase (Optional)
If needed, `Transform()` is called to modify the graph structure before compilation.

### 3. Compilation Phase
The `Compile()` method is called to generate the actual compiled statements that will execute the node's logic.

### 4. Early Registration (Special Cases)
For nodes that return true from `RequiresRegisterNetsBeforeScheduling()`, registration happens before the normal scheduling phase.

## Implementation Patterns

### Basic Node Handler Pattern
```cpp
class FKCHandler_MyNode : public FNodeHandlingFunctor
{
public:
    FKCHandler_MyNode(FKismetCompilerContext& InCompilerContext)
        : FNodeHandlingFunctor(InCompilerContext)
    {
    }

    virtual void Compile(FKismetFunctionContext& Context, UEdGraphNode* Node) override
    {
        // Cast to specific node type
        UMyK2Node* MyNode = CastChecked<UMyK2Node>(Node);
        
        // Generate statements for the node's behavior
        FBlueprintCompiledStatement& Statement = Context.AppendStatementForNode(Node);
        Statement.Type = KCST_CallFunction;
        // ... configure statement
        
        // Generate execution flow
        GenerateSimpleThenGoto(Context, *Node);
    }
};
```

### Net Registration Pattern
```cpp
virtual void RegisterNets(FKismetFunctionContext& Context, UEdGraphNode* Node) override
{
    // Let base class handle standard pins
    FNodeHandlingFunctor::RegisterNets(Context, Node);
    
    // Handle special pins
    for (UEdGraphPin* Pin : Node->Pins)
    {
        if (Pin->PinName == TEXT("SpecialPin"))
        {
            // Custom registration logic
            RegisterSpecialPin(Context, Pin);
        }
    }
}
```

## Best Practices

1. **Always call parent methods** when overriding `RegisterNets()` unless you're completely replacing the behavior
2. **Validate node types** using `CastChecked` or `Cast` at the beginning of handler methods
3. **Handle execution flow** by calling `GenerateSimpleThenGoto()` or implementing custom flow control
4. **Use helper methods** like `RegisterLiteral()` and `ValidateAndRegisterNetIfLiteral()` for common operations
5. **Sanitize names** using `SanitizeName()` when creating variable or terminal names
6. **Check pin connections** before generating statements to avoid runtime errors

## Common Node Handler Types

- **Function Call Handlers**: Process nodes that call functions or methods
- **Variable Access Handlers**: Handle getting and setting variables
- **Control Flow Handlers**: Manage branches, loops, and execution flow
- **Math Expression Handlers**: Process mathematical operations
- **Container Handlers**: Deal with arrays, sets, and maps
- **Event Handlers**: Process event entry and custom event nodes

## Error Handling

Node handlers should use the compiler context's message log for reporting errors:
```cpp
CompilerContext.MessageLog.Error(*FString::Printf(TEXT("Error in node @@: %s"), *ErrorMessage), Node);
```

## Notes

- Node handlers are instantiated once per compilation and can be reused across multiple nodes
- The functor pattern allows the compiler to be easily extended with new node types
- All memory management for terminals and statements is handled by the function context
- The compilation process is multi-threaded safe at the node handler level