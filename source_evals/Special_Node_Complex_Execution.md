# Complex Execution Special Nodes Analysis

## K2Node_MultiGate - Sequential Multi-Path Execution

### Overview
K2Node_MultiGate implements sequential execution across multiple output paths with state tracking. It extends K2Node_ExecutionSequence to provide ordered, stateful execution control.

### Key Properties
- **Inherits from K2Node_ExecutionSequence**: Base sequential execution functionality
- **DataNode**: Reference to UK2Node_TemporaryVariable for state storage
- **Stateful Execution**: Tracks which outputs have been executed
- **Multiple Control Modes**: Random, sequential, looping execution

### Pin Structure
```cpp
// Input Pins:
- Execute (trigger)
- Reset (resets execution state)
- IsRandom (boolean - random vs sequential)  
- Loop (boolean - loop back to start)
- Start Index (integer - starting output)

// Output Pins:
- Out 0, Out 1, Out 2... (execution outputs)
```

### Unique Expansion Patterns

#### 1. State Management System
```cpp
// Uses bit masking for tracking executed outputs
void GetMarkBitFunction(FName& FunctionName, UClass** FunctionClass);     // Mark output as used
void GetHasUnmarkedBitFunction(FName& FunctionName, UClass** FunctionClass); // Check for unused outputs
void GetUnmarkedBitFunction(FName& FunctionName, UClass** FunctionClass);    // Get next unused output
void GetClearAllBitsFunction(FName& FunctionName, UClass** FunctionClass);   // Reset all state
```

#### 2. Complex Expansion Logic
```cpp
void ExpandNode(FKismetCompilerContext& CompilerContext, UEdGraph* SourceGraph)
{
    // Creates complex node network:
    // 1. State variable for tracking executed outputs
    // 2. Conditional logic for random vs sequential
    // 3. Bit manipulation for tracking used outputs
    // 4. Looping logic for continuous execution
    // 5. Branch nodes for each output path
}
```

### C++ Code Generation Requirements

```cpp
// Generated pattern for MultiGate
class FMultiGateState
{
    int32 UsedOutputMask = 0;
    int32 LastOutputIndex = -1;
    bool bIsRandom;
    bool bShouldLoop;
    int32 StartIndex;
};

void ExecuteMultiGate(FMultiGateState& State)
{
    int32 NextOutput;
    
    if (State.bIsRandom)
    {
        // Find random unused output
        NextOutput = FindRandomUnmarkedBit(State.UsedOutputMask, NumOutputs);
    }
    else
    {
        // Find next sequential output
        NextOutput = FindNextUnmarkedBit(State.UsedOutputMask, State.LastOutputIndex);
    }
    
    if (NextOutput != -1)
    {
        MarkBitUsed(State.UsedOutputMask, NextOutput);
        State.LastOutputIndex = NextOutput;
        
        // Execute the specific output
        switch(NextOutput)
        {
            case 0: ExecuteOutput0(); break;
            case 1: ExecuteOutput1(); break;
            // ... etc
        }
    }
    else if (State.bShouldLoop)
    {
        // Reset and start over
        State.UsedOutputMask = 0;
        State.LastOutputIndex = State.StartIndex - 1;
        ExecuteMultiGate(State); // Recursive call
    }
}
```

## K2Node_FormatText - Dynamic Text Formatting

### Overview  
K2Node_FormatText implements dynamic text formatting with variable argument lists. It parses format strings and creates pins for each format argument.

### Key Properties
- **Dynamic Pin Creation**: Pins created based on format string parsing
- **PinNames Array**: Stores argument names extracted from format string
- **Pure Node**: No execution pins, returns formatted text immediately
- **Format String Parsing**: Analyzes text for {ArgumentName} patterns

### Unique Expansion Patterns

#### 1. Format String Analysis
```cpp
void PostEditChangeProperty(struct FPropertyChangedEvent& PropertyChangedEvent)
{
    // Parse format string like "Hello {PlayerName}, you have {Score} points!"
    // Create pins for each argument: PlayerName (String), Score (Integer)
    ReconstructNode(); // Rebuild pins based on format string
}
```

#### 2. Dynamic Pin Management
```cpp
void PinConnectionListChanged(UEdGraphPin* Pin)
{
    // When format string changes, dynamically add/remove argument pins
    // Preserve connections where possible during reconstruction
    
    if (Pin == GetFormatPin()) // Format string input changed
    {
        ReparseFormatString();
        ReallocatePinsDuringReconstruction(OldPins);
    }
}
```

### C++ Code Generation Requirements

```cpp
// Blueprint FormatText becomes C++ FText::Format call
FText FormatTextResult = FText::Format(
    LOCTEXT("FormatString", "Hello {PlayerName}, you have {Score} points!"),
    FFormatNamedArguments{
        {TEXT("PlayerName"), FText::FromString(PlayerNameValue)},
        {TEXT("Score"), FText::AsNumber(ScoreValue)}
    }
);

// Or using FString::Printf for simple cases
FString Result = FString::Printf(TEXT("Hello %s, you have %d points!"), 
                                *PlayerName, Score);
FText FormatTextResult = FText::FromString(Result);
```

## K2Node_MakeMap - Map Literal Construction  

### Overview
K2Node_MakeMap creates map literals with dynamic key-value pair management. It allows runtime construction of TMap<KeyType, ValueType> with arbitrary number of elements.

### Key Properties
- **Dynamic Pair Management**: Add/remove key-value pairs dynamically  
- **Template Type Resolution**: Resolves TMap<K,V> types from connected pins
- **Pure Construction**: Creates map without side effects
- **Duplicate Key Handling**: Validates unique keys at compile time

### Unique Expansion Patterns

#### 1. Pair Pin Creation
```cpp
// Creates paired pins for each map entry
void AddPinToMap()
{
    int32 NewPairIndex = GetPairCount();
    
    // Create key pin
    UEdGraphPin* KeyPin = CreatePin(EGPD_Input, KeyType, 
                                   FName(*FString::Printf(TEXT("Key_%d"), NewPairIndex)));
    
    // Create value pin  
    UEdGraphPin* ValuePin = CreatePin(EGPD_Input, ValueType,
                                     FName(*FString::Printf(TEXT("Value_%d"), NewPairIndex)));
}
```

#### 2. Map Construction Expansion
```cpp
void ExpandNode(FKismetCompilerContext& CompilerContext, UEdGraph* SourceGraph)
{
    // Expands to multiple TMap::Add() calls
    // 1. Create empty map variable
    // 2. For each key-value pair, add to map
    // 3. Return constructed map
}
```

### C++ Code Generation Requirements

```cpp
// Blueprint MakeMap becomes C++ TMap construction
TMap<KeyType, ValueType> ConstructedMap;

// Add each key-value pair
ConstructedMap.Add(Key0Value, Value0Value);
ConstructedMap.Add(Key1Value, Value1Value);
// ... for each pair

// Or using initializer list (C++11):
TMap<FString, int32> ConstructedMap = {
    {TEXT("FirstKey"), 100},
    {TEXT("SecondKey"), 200},
    {TEXT("ThirdKey"), 300}
};
```

## Variable Access Special Nodes

### K2Node_VariableGet/Set Context-Specific Behavior

These nodes have context-sensitive behavior based on:

- **Local Variables**: Direct variable access
- **Member Variables**: this->VariableName access  
- **Component Variables**: GetComponent()->VariableName
- **Global Variables**: Static class access
- **Interface Variables**: Interface function calls

### C++ Generation Patterns

```cpp
// Local Variable
LocalType VariableName = Value;

// Member Variable  
this->MemberVariable = Value;

// Component Variable
if (UComponentType* Comp = GetComponentByClass<UComponentType>())
{
    Comp->ComponentVariable = Value;
}

// Static/Global Variable
UClassType::StaticVariable = Value;
```

## Implementation Priority

### K2Node_MultiGate  
**HIGH** - Complex state management essential for flow control (~8% of Blueprint nodes)

### K2Node_FormatText
**MEDIUM** - String formatting is common but not critical (~3% of Blueprint nodes)

### K2Node_MakeMap
**MEDIUM** - Collection construction used in data-heavy Blueprints (~5% of Blueprint nodes)

### Variable Nodes Context Handling
**CRITICAL** - Variable access represents ~40% of all Blueprint operations

## Dependencies
- K2Node_ExecutionSequence (MultiGate base)
- Temporary variable system (state storage)
- Text formatting system (FormatText)
- Collection type system (MakeMap)
- Variable resolution system (all variable nodes)