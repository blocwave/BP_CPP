# Math Expression System and Validation

## Overview
The UK2Node_MathExpression system provides dynamic math expression parsing and evaluation within Blueprint graphs. This composite node type converts mathematical expressions into sub-networks of math nodes, requiring special handling for C++ conversion.

## UK2Node_MathExpression Core Structure

### Class Definition
```cpp
UCLASS()
class BLUEPRINTGRAPH_API UK2Node_MathExpression : public UK2Node_Composite {
    GENERATED_UCLASS_BODY()

public:
    // The math expression to evaluate
    UPROPERTY(EditAnywhere, Category = Expression)
    FString Expression;

    // Legacy compatibility flag
    UPROPERTY()
    bool bMadeAfterRotChange;

private:
    // Cached error log for validation
    TSharedPtr<class FCompilerResultsLog> CachedMessageLog;
    
    // Performance caches for UI
    FNodeTextCache CachedNodeTitle;
    FNodeTextCache CachedDisplayExpression;
};
```

### Key Properties
1. **Expression String**: User-entered mathematical expression
2. **Composite Nature**: Contains internal sub-graph of math nodes
3. **Dynamic Parsing**: Expression parsed at edit-time and compile-time
4. **Validation Caching**: Results cached for performance

## Expression Parsing and Validation

### Core Methods
```cpp
class UK2Node_MathExpression {
    // Node behavior control
    virtual bool IsNodePure() const { return !ShouldExpandInsteadCompile(); }
    virtual bool ShouldMergeChildGraphs() const override { 
        return ShouldExpandInsteadCompile(); 
    }

    // Expression management
    virtual void OnRenameNode(const FString& NewName) override;
    virtual void PostPlacedNewNode() override;
    virtual void ReconstructNode() override;
    
    // Validation
    virtual void ValidateNodeDuringCompilation(
        class FCompilerResultsLog& MessageLog
    ) const override;

private:
    // Expansion control
    bool ShouldExpandInsteadCompile() const;
    
    // Expression processing
    void RebuildExpression(FString NewExpression);
    void ClearExpression();
    FString SanitizeDisplayExpression(FString InExpression) const;
    FText GetFullTitle(FText InExpression) const;
};
```

### Expression Parsing Process
1. **Input Validation**: Check expression syntax and operators
2. **Token Parsing**: Break expression into mathematical tokens
3. **Operator Precedence**: Apply proper mathematical operator precedence
4. **Node Generation**: Create sub-network of math operation nodes
5. **Pin Generation**: Generate input pins for variables in expression
6. **Connection Management**: Wire the generated sub-network

## Math Expression Features

### Supported Operations
The expression parser supports standard mathematical operations:

```cpp
// Basic arithmetic
+ (addition)
- (subtraction) 
* (multiplication)
/ (division)
% (modulo)

// Advanced operations
^ (power/exponentiation)
sqrt (square root)
abs (absolute value)
sin, cos, tan (trigonometric)
asin, acos, atan (inverse trigonometric)
log, ln (logarithmic)

// Comparison operations (for conditional expressions)
>, <, >=, <=, ==, !=

// Logical operations
&&, ||, ! (and, or, not)
```

### Variable Handling
Variables in expressions become input pins:
- **Automatic Pin Creation**: Variables automatically generate typed input pins
- **Type Inference**: Variable types inferred from context and usage
- **Default Values**: Variables can have default values set on pins
- **Dynamic Updating**: Pin structure updates when expression changes

### Expression Examples
```cpp
// Simple arithmetic
"A + B * C"              // Creates pins A, B, C
"(X + Y) / 2"            // Creates pins X, Y
"sqrt(A*A + B*B)"        // Creates pins A, B (Pythagorean)

// Complex expressions
"sin(Angle * 3.14159 / 180)"     // Degree to radian conversion
"A > B ? A : B"                  // Max operation (if supported)
"Position + Velocity * DeltaTime" // Physics integration
```

## Compilation and Code Generation

### Compilation Modes
The node has two compilation modes:

1. **Expansion Mode** (`ShouldExpandInsteadCompile() == true`):
   - Expression expanded to individual math nodes
   - Sub-graph merged into parent graph
   - Individual nodes compiled separately

2. **Direct Compilation Mode** (`ShouldExpandInsteadCompile() == false`):
   - Expression compiled as single unit
   - Better optimization potential
   - More complex code generation

### Node Handler Creation
```cpp
virtual class FNodeHandlingFunctor* CreateNodeHandler(
    class FKismetCompilerContext& CompilerContext
) const override;
```

The node handler manages:
- Expression parsing during compilation
- Sub-node generation and wiring
- Error reporting and validation
- Code generation for the expression

## Validation System

### Compile-Time Validation
```cpp
virtual void ValidateNodeDuringCompilation(
    class FCompilerResultsLog& MessageLog
) const override;
```

Validation checks:
1. **Syntax Validation**: Verify expression syntax is correct
2. **Variable Validation**: Ensure all variables have valid types
3. **Operator Validation**: Check operator compatibility with operand types
4. **Result Type Validation**: Verify result type is valid for context
5. **Circular Reference Check**: Prevent self-referential expressions

### Error Reporting
The system provides detailed error messages:
- **Syntax Errors**: Invalid expression syntax
- **Type Mismatches**: Incompatible operand types
- **Undefined Variables**: Variables not connected or defined
- **Invalid Operations**: Unsupported operations for given types
- **Compilation Errors**: Backend compilation failures

### Validation Caching
```cpp
private:
    TSharedPtr<class FCompilerResultsLog> CachedMessageLog;
```

Validation results are cached to avoid repeated parsing during:
- Node reconstruction
- Graph validation passes
- UI updates

## Pin Management

### Dynamic Pin Creation
Math expression nodes dynamically create pins based on parsed variables:

```cpp
// Expression: "A + B * sin(C)"
// Generated pins:
// Input: A (float/double)
// Input: B (float/double)  
// Input: C (float/double)
// Output: (return result)
```

### Pin Type Inference
The system infers pin types based on:
1. **Context Usage**: How variables are used in expression
2. **Operator Requirements**: Type requirements of operators
3. **Default Assumptions**: Default to float for ambiguous cases
4. **Connection Context**: Types of connected pins

### Pin Reconstruction
When expressions change:
1. **Clear Old Pins**: Remove pins for unused variables
2. **Add New Pins**: Create pins for new variables
3. **Preserve Connections**: Maintain connections where possible
4. **Update Types**: Update pin types if inference changes

## C++ Code Generation Implications

### Expression Evaluation Strategies

#### 1. Direct Translation
Convert expression directly to C++ mathematical expression:
```cpp
// Blueprint Expression: "A + B * C"
// Generated C++:
float Result = A + B * C;
```

#### 2. Sub-Node Expansion
Expand to individual operation nodes, then generate C++ for each:
```cpp
// Blueprint Expression: "A + B * C"
// Sub-nodes: Multiply(B, C) -> Add(A, TempResult)
float TempResult = B * C;
float Result = A + TempResult;
```

#### 3. Function Call Generation
Generate function calls for complex operations:
```cpp
// Blueprint Expression: "sqrt(A*A + B*B)"
// Generated C++:
float Result = FMath::Sqrt(A*A + B*B);
```

### Type Handling
1. **Type Promotion**: Handle automatic type promotion (int->float, float->double)
2. **Vector Operations**: Support vector math expressions when applicable
3. **Precision**: Maintain proper precision based on operand types
4. **Overflow Protection**: Add bounds checking where necessary

### Variable Binding
```cpp
// Expression variables map to C++ variables or parameters
void GeneratedFunction(float A, float B, float C, float& OutResult) {
    OutResult = A + B * C;
}
```

### Error Handling
1. **Compile-Time Validation**: Catch errors during Blueprint->C++ conversion
2. **Runtime Safety**: Add runtime checks for division by zero, invalid math
3. **NaN Handling**: Proper handling of NaN and infinity results
4. **Type Safety**: Ensure type safety in generated code

## Best Practices for C++ Conversion

### Expression Analysis
1. **Parse Expression**: Use Blueprint's parsing logic to understand structure
2. **Extract Variables**: Identify all variables and their types
3. **Analyze Operators**: Understand operator precedence and requirements
4. **Validate Types**: Ensure all operations are type-safe

### Code Generation Strategies
1. **Simple Expressions**: Direct translation for basic arithmetic
2. **Complex Expressions**: Break into sub-operations for clarity
3. **Function Calls**: Use UE math library functions for advanced operations
4. **Optimization**: Optimize generated code for performance

### Validation Integration
1. **Pre-Conversion Check**: Validate expression before generating code
2. **Type Verification**: Verify all variable types are known and valid
3. **Error Propagation**: Propagate validation errors to conversion process
4. **Fallback Handling**: Provide fallback for unsupported expressions

### Performance Considerations
1. **Constant Folding**: Evaluate constant sub-expressions at compile-time
2. **Common Sub-expressions**: Eliminate duplicate calculations
3. **Vectorization**: Use SIMD operations where possible
4. **Caching**: Cache expensive calculations when appropriate