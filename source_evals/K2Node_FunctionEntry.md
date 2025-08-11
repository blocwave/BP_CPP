# K2Node_FunctionEntry.cpp - Function Entry Point Implementation

**File**: `/Engine/Source/Editor/BlueprintGraph/Private/K2Node_FunctionEntry.cpp`

## Overview
This file implements the UK2Node_FunctionEntry class and its compilation handler FKCHandler_FunctionEntry. Function entry nodes serve as the starting point for all Blueprint functions, handling parameter initialization, function signature validation, and establishing the execution context for function graphs.

## Role in Compilation Pipeline
- **Function Initialization**: Entry point for all Blueprint function execution
- **Parameter Setup**: Creates and registers input parameter terminals
- **Signature Validation**: Ensures function signatures match their declarations
- **Output Parameter Handling**: Manages output parameters for inherited functions
- **Execution Flow**: Establishes initial execution context and control flow

## Key Functions and Algorithms

### Function Entry Handler (FKCHandler_FunctionEntry)
```cpp
class FKCHandler_FunctionEntry : public FNodeHandlingFunctor
```
- **Purpose**: Compiles function entry nodes into executable statements
- **Inheritance**: Extends base node handler with function-specific logic
- **Responsibility**: Parameter registration, terminal creation, execution flow setup

### Parameter Registration System
```cpp
void RegisterFunctionInput(FKismetFunctionContext& Context, UEdGraphPin* Net, UFunction* Function)
```
- **Purpose**: Creates FBPTerminal objects for function input parameters
- **Algorithm**:
  1. Creates new FBPTerminal for the parameter
  2. Copies type and name information from the pin
  3. Checks for reference parameter flags in parent function
  4. Sets `bPassedByReference` for CPF_ReferenceParm parameters
  5. Registers terminal in function context's Parameters array
  6. Maps pin to terminal in Context.NetMap

### Output Parameter Pre-registration  
```cpp
virtual void RegisterNets(FKismetFunctionContext& Context, UEdGraphNode* Node) override
```
- **Purpose**: Pre-registers output parameters for inherited functions
- **Problem Solved**: Ensures output parameters exist even without FunctionResult nodes
- **Algorithm**:
  1. Resolves function reference to get UFunction signature
  2. Iterates through function properties using TFieldIterator
  3. Identifies output parameters (`CPF_OutParm && !CPF_ReferenceParm`)
  4. Creates FBPTerminal for each output parameter not already registered
  5. Adds terminals to Context.Results array
- **Critical for**: Function override scenarios where signature is predetermined

### Execution Statement Generation
```cpp
virtual void Compile(FKismetFunctionContext& Context, UEdGraphNode* Node) override
```
- **Purpose**: Generates initial execution statements for function entry
- **Execution Patterns**:
  
  **Ubergraph Entry** (Event Graphs):
  ```cpp
  if (EntryNode->FunctionReference.GetMemberName() == UEdGraphSchema_K2::FN_ExecuteUbergraphBase) {
      // Creates KCST_ComputedGoto statement for event dispatching
      FBlueprintCompiledStatement& ComputedGotoStatement = Context.AppendStatementForNode(Node);
      ComputedGotoStatement.Type = KCST_ComputedGoto;
      ComputedGotoStatement.LHS = EntryPointTerm;
  }
  ```
  
  **Regular Function Entry**:
  ```cpp
  else {
      // Creates simple execution flow continuation
      GenerateSimpleThenGoto(Context, *Node);
  }
  ```

### Input/Output Parameter Detection
```cpp
// Parameter classification algorithm:
const bool bIsFunctionInput = !ParamProperty->HasAnyPropertyFlags(CPF_OutParm) 
                             || ParamProperty->HasAnyPropertyFlags(CPF_ReferenceParm);
```
- **Input Parameters**: All non-output params + reference params
- **Output Parameters**: CPF_OutParm parameters that aren't CPF_ReferenceParm
- **Reference Parameters**: Both input and output (bidirectional)

## Integration with Other Compiler Components

### With Function Context Management
```cpp
FKismetFunctionContext& Context
```
- **Parameter Storage**: Manages Context.Parameters array
- **Result Storage**: Pre-populates Context.Results for output parameters  
- **Net Mapping**: Updates Context.NetMap for pin-to-terminal resolution
- **Statement Generation**: Appends to Context.StatementList

### With Function Result Nodes
- **Coordination**: FunctionResult nodes check for existing terminals before creating new ones
- **Duplication Prevention**: Avoids duplicate result terminals
- **Signature Consistency**: Ensures entry and result nodes have consistent parameter sets

### With Blueprint Function System
- **Function Resolution**: Uses FunctionReference.ResolveMember<UFunction>()
- **Signature Validation**: Compares node pins with UFunction parameters
- **Override Support**: Handles inherited function signatures correctly

## Relevance to BP→C++ Conversion

### Function Signature Generation
```cpp
// Blueprint Function Entry → C++ Function Declaration
// Input: Function entry node with pins
// Output: C++ function signature

// Example transformation:
// Blueprint: Function "CalculateValue" with (int32 InputA, float InputB) → (float Result)
// Generated C++:
float UMyBlueprintClass::CalculateValue(int32 InputA, float InputB) {
    // Function body generated from subsequent nodes
}
```

### Parameter Declaration Mapping
```cpp
// Terminal properties → C++ parameter attributes:
// - Terminal.Name → parameter name
// - Terminal.Type → C++ type (via FKismetCompilerUtilities::IsTypeCompatibleWithProperty)
// - Terminal.bPassedByReference → reference parameter (&)
// - CPF_ReferenceParm → bidirectional reference
// - CPF_OutParm → output parameter or return value
```

### Function Prolog Generation
- **Parameter Initialization**: Local variable declarations for parameters
- **Reference Handling**: Proper C++ reference semantics
- **Default Values**: Parameter default value initialization
- **Validation Code**: Parameter validation for RPC functions

### Execution Context Setup
```cpp
// Generated C++ function prolog pattern:
ReturnType UFunctionClass::FunctionName(ParamTypes...) {
    // Local variable declarations for intermediate values
    // Parameter validation (if needed)
    // Function body starts here...
    
    // For Ubergraph functions:
    switch(EntryPoint) {
        case 0: goto ExecutionNode_0;
        case 1: goto ExecutionNode_1;
        // ...
    }
}
```

### Output Parameter Handling
```cpp
// Multiple output parameters → struct return or reference parameters
// Single output → direct return value
// No outputs → void function

// Example with multiple outputs:
struct FCalculateResults {
    float PrimaryResult;
    int32 SecondaryResult;
    bool bWasSuccessful;
};

FCalculateResults UMyClass::CalculateValue(int32 Input);
```

## Key Data Structures

### FKCHandler_FunctionEntry
```cpp
class FKCHandler_FunctionEntry : public FNodeHandlingFunctor {
    // Inherits CompilerContext reference
    // Provides function-specific compilation logic
    // Manages parameter terminal creation
};
```

### Parameter Terminal Properties
```cpp
FBPTerminal* Term = new FBPTerminal();
Term->CopyFromPin(Net, Net->PinName);           // Type and name from pin
Term->bPassedByReference = /* CPF_ReferenceParm check */;  // Reference flag
Context.Parameters.Add(Term);                   // Add to parameter list
Context.NetMap.Add(Net, Term);                 // Enable pin resolution
```

## Special Considerations

### Ubergraph Handling
- **Event Dispatching**: Uses computed goto for event graph entry points
- **Entry Point Parameter**: Special handling for event ID parameter
- **Performance**: Optimized dispatch mechanism for event routing

### Function Override Support
- **Signature Matching**: Ensures overridden functions maintain parent signatures
- **Output Parameter Pre-registration**: Critical for inherited function compilation
- **Validation**: Compiler errors if signatures don't match

### Reference Parameter Semantics
- **Bidirectional Parameters**: Input and output simultaneously
- **Memory Safety**: Proper lifetime management for reference parameters  
- **Blueprint Semantics**: Reference parameters appear as both input and output pins

### Error Handling
- **Missing Parameters**: Compiler errors for incomplete function signatures
- **Type Mismatches**: Validation against parent function signatures
- **Missing Entry Points**: Error reporting for ubergraph entry point issues

This implementation is crucial for establishing the execution context of Blueprint functions and ensuring proper parameter handling during BP→C++ translation.