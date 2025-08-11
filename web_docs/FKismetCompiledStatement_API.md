# FKismetCompiledStatement API Documentation

Source: https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Editor/KismetCompiler/FKismetCompiledStatement?application_version=5.2

## Class Definition

```cpp
class FKismetCompiledStatement
```

## Purpose

FKismetCompiledStatement represents individual executable statements in the compiled Blueprint bytecode. During Blueprint compilation, the visual graph nodes are transformed into a series of these compiled statements that represent the actual runtime execution logic. Each statement corresponds to a specific operation such as function calls, variable assignments, control flow operations, or mathematical computations.

## Overview

The FKismetCompiledStatement is a fundamental building block in the Blueprint compilation process. It serves as an intermediate representation between the high-level visual Blueprint graphs and the final runtime bytecode that gets executed. These statements form a linear sequence of operations that preserve the execution semantics of the original Blueprint graph while being optimized for runtime performance.

## Key Properties

### Statement Type and Operation
- **Statement Type** - Identifies the kind of operation (function call, assignment, jump, etc.)
- **Operation Code** - Specific opcode for the statement type
- **Execution Context** - Context in which this statement executes

### Operands and Parameters  
- **Input Operands** - Terms that provide input values to the statement
- **Output Operands** - Terms that receive output values from the statement
- **Parameter Lists** - Additional parameters specific to the statement type

### Control Flow
- **Target Labels** - Jump targets for control flow statements
- **Conditional Flags** - Conditions for branching statements
- **Loop Metadata** - Information for loop construction and optimization

### Debugging Information
- **Source Node** - Reference back to the original Blueprint graph node
- **Debug Information** - Metadata for debugging and editor integration
- **Line Mapping** - Correlation with source Blueprint for error reporting

## Statement Types

### Function Call Statements
Represent calls to Blueprint functions, C++ functions, or built-in operations:
- **UFunctionCall** - Standard function invocation
- **DelegateCall** - Delegate/event invocation  
- **InterfaceCall** - Interface method calls
- **NativeFunctionCall** - Direct C++ function calls

### Assignment Statements  
Handle variable assignments and value transfers:
- **Assignment** - Direct value assignment to variables
- **Copy** - Memory copy operations for complex types
- **PropertyAssignment** - Assignment to object properties
- **ArrayElementAssignment** - Array index assignments

### Control Flow Statements
Manage execution flow and branching logic:
- **Jump** - Unconditional jumps to labels
- **ConditionalJump** - Conditional branching based on boolean expressions
- **Return** - Function return statements
- **GotoLabel** - Label definitions for jump targets

### Memory Management
Handle object lifecycle and memory operations:
- **ObjectConstruction** - Object creation and initialization
- **ObjectDestruction** - Cleanup and destruction operations
- **GarbageCollection** - Reference management for garbage collection

### Mathematical and Logical Operations
Basic computational statements:
- **MathematicalOperation** - Arithmetic calculations (add, subtract, multiply, etc.)
- **LogicalOperation** - Boolean logic operations (and, or, not)
- **ComparisonOperation** - Value comparison operations (equal, greater, less)
- **BitwiseOperation** - Bitwise manipulations

## Compilation Process Integration

### Node to Statement Conversion
During Blueprint compilation, graph nodes are transformed into statements:

1. **Node Analysis** - Examine node type and configuration
2. **Statement Generation** - Create appropriate compiled statements  
3. **Operand Resolution** - Map pins to statement operands
4. **Optimization** - Apply compiler optimizations to statement sequences

### Statement Optimization
Multiple optimization passes may modify statements:
- **Dead Code Elimination** - Remove unused statements
- **Constant Folding** - Pre-compute constant expressions
- **Control Flow Optimization** - Simplify branching logic
- **Inline Expansion** - Inline simple function calls

### Bytecode Generation
Statements are converted to final executable bytecode:
- **Instruction Encoding** - Convert to virtual machine instructions
- **Address Resolution** - Resolve variable and function addresses
- **Jump Target Fixup** - Calculate actual jump addresses
- **Debug Info Embedding** - Include debugging metadata

## Usage in Compiler Pipeline

FKismetCompiledStatement plays a crucial role throughout compilation:

### Statement Creation
- Created by node handlers during graph processing
- Each UK2Node may generate multiple statements
- Statements maintain links back to source nodes for debugging

### Intermediate Processing  
- Statements can be analyzed and transformed by compiler passes
- Control flow analysis operates on statement sequences
- Data flow analysis tracks variable usage across statements

### Final Code Generation
- Statements are serialized into Blueprint bytecode
- Runtime virtual machine interprets these statements during execution
- Debug information enables breakpoints and variable watching

## Memory and Performance

### Statement Efficiency
- Designed for minimal memory footprint during compilation
- Optimized for fast traversal during bytecode generation
- Temporary objects with automatic cleanup after compilation

### Runtime Impact
- Statements themselves don't exist at runtime
- They are compiled into efficient bytecode instructions
- Debug builds may retain additional metadata for editor features

## Debugging Support

### Editor Integration
- Statements maintain references to source Blueprint nodes
- Enable step-through debugging in Blueprint editor
- Support breakpoint placement and variable inspection

### Error Reporting
- Compilation errors reference both statements and source nodes
- Provides clear mapping from runtime errors to Blueprint source
- Enables accurate error highlighting in visual editor

## Important Notes

- FKismetCompiledStatement is an intermediate compilation artifact
- Not directly visible to Blueprint authors but critical for compilation
- Each statement represents one atomic operation in the execution flow
- Statements are optimized and transformed during compilation
- Final runtime uses compiled bytecode, not the statements themselves
- Essential for Blueprint debugging and error reporting capabilities

## Related Classes

- **FKismetCompilerContext** - Creates and manages compiled statements
- **FNodeHandlingFunctor** - Generates statements from Blueprint nodes
- **FBPTerminal** - Represents operands used by statements
- **FKismetFunctionContext** - Context containing statement lists
- **UK2Node** - Source Blueprint nodes that generate statements
- **UBlueprintGeneratedClass** - Contains the final compiled bytecode