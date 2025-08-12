# K2Node_IfThenElse: Blueprint Branch Node Analysis

## Overview
`UK2Node_IfThenElse` implements the fundamental conditional branching node in Blueprint graphs. It provides the basic "if-then-else" logic flow control that compiles to optimized C++ branching statements.

## Node Structure

### Pin Configuration
The node allocates four pins during `AllocateDefaultPins()`:

1. **Execute Pin** (Input): `PC_Exec` - Entry execution pin
2. **Condition Pin** (Input): `PC_Boolean` - Boolean condition to evaluate (defaults to "true")
3. **Then Pin** (Output): `PC_Exec` - Execution path when condition is true
4. **Else Pin** (Output): `PC_Exec` - Execution path when condition is false

### Pin Characteristics
- Condition pin has auto-generated default value of "true"
- Then/Else pins have friendly names ("true"/"false") for UI clarity
- All pins are essential for proper node functionality

## Compilation Process

### Compiler Handler: FKCHandler_Branch
The node uses a specialized compilation handler that implements the `FNodeHandlingFunctor` interface:

```cpp
class FKCHandler_Branch : public FNodeHandlingFunctor
{
    virtual void Compile(FKismetFunctionContext& Context, UEdGraphNode* Node) override;
};
```

### Compilation Steps

1. **Pin Validation**
   - Validates execution pin exists and has correct type (`PC_Exec`)
   - Validates condition pin exists and has correct type (`PC_Boolean`) 
   - Validates both output execution pins exist
   - Reports warnings if execution pin is not connected

2. **Terminal Resolution**
   - Resolves the condition pin to a `FBPTerminal` through the net map
   - Uses `FEdGraphUtilities::GetNetFromPin()` for net resolution
   - Validates the terminal exists in the context's NetMap

3. **Statement Generation**
   - Creates two sequential compiled statements:
   
   **Skip-If Statement (KCST_GotoIfNot):**
   ```cpp
   FBlueprintCompiledStatement& SkipIfGoto = Context.AppendStatementForNode(Node);
   SkipIfGoto.Type = KCST_GotoIfNot;
   SkipIfGoto.LHS = *CondTerm;
   Context.GotoFixupRequestMap.Add(&SkipIfGoto, ElsePin);
   ```
   
   **Then-Branch Statement (KCST_UnconditionalGoto):**
   ```cpp
   FBlueprintCompiledStatement& GotoThen = Context.AppendStatementForNode(Node);
   GotoThen.Type = KCST_UnconditionalGoto;
   GotoThen.LHS = *CondTerm;
   Context.GotoFixupRequestMap.Add(&GotoThen, ThenPin);
   ```

### C++ Code Generation Pattern

The compiled statements generate C++ code equivalent to:
```cpp
// If condition is false, jump to else branch
if (!ConditionValue)
    goto ElseLabel;

// Otherwise, execute then branch
goto ThenLabel;
```

This pattern ensures optimal branch prediction and minimizes CPU pipeline stalls.

## Data Flow Analysis

### Input Dependencies
- **Condition Input**: Must resolve to a valid boolean terminal
- **Execution Input**: Must be connected to trigger node execution

### Output Routing  
- **Execution Flow**: Splits into two mutually exclusive paths
- **Data Flow**: No data transformation - pure control flow node

### Net Map Integration
- Registers no output terminals (execution-only node)
- Reads condition terminal from context's NetMap
- Participates in goto fixup resolution system

## Execution Flow Characteristics

### Branch Prediction Optimization
The generated code structure supports:
- Predictable branching patterns
- Minimal conditional overhead
- Efficient jump table generation

### Control Flow Guarantees
- Exactly one output path is taken per execution
- No data state changes during evaluation
- Deterministic execution based on condition value

## Integration with Compiler Pipeline

### Statement Ordering
1. Node statements are appended in sequence
2. Goto fixup requests are registered for later resolution
3. Jump targets are resolved during backend compilation

### Backend Integration
- `KCST_GotoIfNot`: Compiles to conditional jump instructions
- `KCST_UnconditionalGoto`: Compiles to direct jump instructions
- Fixup system resolves jump addresses during code generation

### Error Handling
- Reports missing pins as compilation errors
- Warns about unreachable code paths
- Validates pin type compatibility

## Performance Characteristics

### Runtime Performance
- **Execution Cost**: Single boolean evaluation + conditional jump
- **Memory Overhead**: Minimal - no temporary storage required
- **Cache Impact**: Excellent - compact instruction sequence

### Compilation Performance
- **Analysis Complexity**: O(1) - simple pin validation
- **Code Generation**: O(1) - two fixed statements
- **Memory Usage**: Minimal temporary allocation

## Blueprint Editor Integration

### Visual Representation
- **Node Color**: Uses `ExecBranchNodeTitleColor` from editor settings
- **Icon**: "GraphEditor.Branch_16x" from AppStyle
- **Title**: Static "Branch" text

### User Interface
- Simple condition input with default value
- Clear true/false labeling on output pins
- Tooltip explains conditional execution behavior

### Menu Integration
- Categorized under "Flow Control" in node menu
- Spawned via `UBlueprintNodeSpawner`
- Available in all compatible graph types

## Common Usage Patterns

### Simple Conditionals
```
Event -> Branch[Condition] -> {Then: ActionA, Else: ActionB}
```

### Nested Branching
```
Branch1 -> Then: Branch2
        -> Else: Branch3
```

### Guard Patterns
```
Event -> Branch[IsValid] -> Then: MainLogic
                         -> Else: ErrorHandling
```

## Architectural Considerations

### Design Philosophy
- Minimal, focused functionality
- Optimal performance characteristics
- Clear semantic meaning in generated code

### Extension Points
- Compiler handler can be overridden
- Pin validation logic is customizable
- Error message formatting is localizable

### Limitations
- Boolean-only conditions (no multi-way branching)
- No condition caching or optimization
- Single evaluation per execution

## Related Nodes

### Functional Alternatives
- **Switch nodes**: Multi-way branching
- **Select nodes**: Data-driven conditional selection
- **Gate nodes**: Conditional execution gating

### Compilation Relatives
- **UK2Node_ExecutionSequence**: Sequential execution control
- **UK2Node_MultiGate**: Complex flow control
- **UK2Node_DoOnce**: State-based execution control

This analysis demonstrates how `UK2Node_IfThenElse` serves as the foundation for conditional logic in Blueprint graphs, providing efficient compilation to optimized C++ branching code while maintaining clear semantic meaning for Blueprint developers.