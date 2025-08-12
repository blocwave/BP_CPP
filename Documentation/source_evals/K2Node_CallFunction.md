# K2Node_CallFunction.cpp

## File Purpose and Overview

The **K2Node_CallFunction.cpp** file implements the fundamental Blueprint node type for function calls within the Unreal Engine Blueprint system. This class represents the core mechanism by which Blueprints invoke functions - whether they are native C++ functions, Blueprint-implemented functions, or engine/library functions. It serves as the bridge between visual Blueprint graphs and the underlying function call system.

### Primary Responsibilities:
- **Function Invocation Representation**: Models function calls in the Blueprint visual graph
- **Parameter Management**: Handles input and output pin creation for function parameters
- **Dynamic Type Resolution**: Manages pins that change types based on input (polymorphic functions)
- **Metadata Processing**: Interprets function metadata to customize node behavior
- **Compilation Integration**: Provides compilation handlers for code generation

## Key Classes and Their Responsibilities

### 1. UK2Node_CallFunction
**Purpose**: The main Blueprint node class representing function calls.

**Core Functionality**:
- Inherits from UK2Node to provide base Blueprint node functionality
- Manages function reference and parameter pin creation
- Handles dynamic output type resolution
- Provides tooltip, title, and documentation integration
- Integrates with Blueprint compilation pipeline

### 2. FCustomStructureParamHelper
**Purpose**: Manages wildcard parameters that can accept custom structures.

**Key Methods**:
- `FillCustomStructureParameterNames()`: Extracts parameter names from metadata
- `HandleSinglePin()`: Updates pin type based on connections
- `UpdateCustomStructurePins()`: Synchronizes all custom structure pins

**Algorithm**:
```cpp
if (Pin->LinkedTo.Num() > 0) {
    // Copy type from connected pin
    Pin->PinType = LinkedTo->PinType;
} else {
    // Reset to wildcard state
    Pin->PinType = WildcardType;
}
```

### 3. FDynamicOutputHelper
**Purpose**: Handles functions with output types that change based on input parameters.

**Key Responsibilities**:
- **Type Picker Detection**: Identifies pins that determine output types
- **Output Conformance**: Updates output pin types to match input class selections
- **Validation**: Ensures dynamic type changes maintain valid connections

**Core Algorithm**:
```cpp
UClass* PickedClass = GetPinClass(TypePickerPin);
for (UEdGraphPin* DynamicPin : DynamicOutputPins) {
    DynamicPin->PinType.PinSubCategoryObject = PickedClass;
}
```

## Main Functions and Their Purposes

### Constructor and Initialization
**UK2Node_CallFunction::UK2Node_CallFunction()**
- Initializes base node state
- Sets up pin tooltip validation flags
- Prepares node for function binding

### Core Node Information Methods

#### GetNodeTitle()
**Purpose**: Generates display title for the node in Blueprint editor.

**Features**:
- Shows friendly function name from metadata
- Handles deprecated function warnings
- Displays context information for library functions
- Supports custom formatting for special function types

#### GetTooltipText()
**Purpose**: Provides comprehensive tooltip information.

**Components**:
- Function description from metadata
- Parameter descriptions
- Return value information
- Deprecation warnings
- Performance implications

#### GetFunctionContextString()
**Purpose**: Determines the context string displayed on the node.

**Logic**:
```cpp
if (UClass* FunctionClass = Function->GetOwnerClass()) {
    if (FunctionClass->ClassGeneratedBy) {
        // Blueprint function - show Blueprint name
        return BlueprintName;
    } else {
        // Native function - show class name
        return ClassName;
    }
}
```

### Pin Management Methods

#### AllocateDefaultPins()
**Purpose**: Creates input and output pins based on function signature.

**Process**:
1. **Parameter Analysis**: Examines UFunction parameter list
2. **Pin Creation**: Creates pins for each parameter and return value
3. **Metadata Application**: Applies special metadata (hidden pins, custom types)
4. **Dynamic Setup**: Configures dynamic output types if applicable

#### ExpandNode()
**Purpose**: Handles node expansion during compilation.

**Expansion Types**:
- **Array Functions**: Special handling for array manipulation functions
- **Math Libraries**: Optimization for mathematical operations  
- **Custom Expansions**: User-defined node expansion logic

### Dynamic Type Resolution

#### ConformOutputType()
**Purpose**: Updates output pin types based on input class selections.

**Use Cases**:
- **Spawn Actor**: Output type matches selected actor class
- **Cast**: Output type matches target cast class
- **Get Component**: Output type matches component class selection

#### UpdateCustomStructurePins()
**Purpose**: Handles wildcard pins that accept any struct type.

**Metadata Driven**:
Functions marked with `MD_CustomStructureParam` metadata get wildcard pins that adopt the type of connected structures.

## Integration with Blueprint Compilation Pipeline

### Pre-Compilation Phase
1. **Function Resolution**: Resolves function reference from name/class
2. **Pin Validation**: Ensures all pins have valid types and connections
3. **Type Checking**: Validates parameter types match function signature
4. **Metadata Processing**: Applies function-specific compilation flags

### Compilation Phase
```cpp
FNodeHandlingFunctor* CreateNodeHandler(FKismetCompilerContext& Context) const override {
    return new FKCHandler_CallFunction(Context);
}
```

**Handler Responsibilities**:
- Generates FBlueprintCompiledStatement for function call
- Handles parameter passing and return value assignment
- Manages context object resolution
- Integrates with statement compilation pipeline

### Post-Compilation
- **Bytecode Generation**: VM backend generates appropriate call instructions
- **Debugging Info**: Maintains source-to-bytecode mapping
- **Performance Profiling**: Enables function call instrumentation

## Important Data Structures Used

### UFunction Reference
**Structure**: Direct pointer to the target UFunction
**Usage**: 
- Parameter type resolution
- Metadata extraction
- Compilation validation
- Runtime invocation setup

### Pin Type Information
**FEdGraphPinType**: Comprehensive type description including:
- **Category**: Basic type classification (Object, Primitive, etc.)
- **SubCategory**: Specific type details (Class reference, etc.)
- **SubCategoryObject**: Type-specific object reference
- **Container Type**: Array, Set, Map specifications

### Metadata Mappings
**Key Metadata Tags**:
- `MD_CustomStructureParam`: Enables wildcard struct parameters
- `MD_DynamicOutputType`: Specifies type-determining parameter
- `MD_DynamicOutputParam`: Marks dynamically-typed output parameters
- `MD_Latent`: Indicates asynchronous execution requirements

## Critical Algorithms and Patterns

### 1. Lazy Pin Type Resolution
**Pattern**: Pin types are determined just-in-time during graph validation.

```cpp
void ReconstructNode() override {
    // Clear existing pins
    // Recreate pins from current function signature
    // Apply dynamic type updates
    // Restore connections where possible
}
```

### 2. Metadata-Driven Behavior
**Algorithm**: Function metadata controls node presentation and compilation.

```cpp
if (Function->HasMetaData(MD_Latent)) {
    // Add latent execution pin
    // Configure async compilation
}
if (Function->HasMetaData(MD_CustomStructureParam)) {
    // Enable wildcard type matching
    // Set up type propagation
}
```

### 3. Connection-Based Type Inference
**Pattern**: Pin types update based on connected node types.

```cpp
void NotifyPinConnectionListChanged(UEdGraphPin* Pin) override {
    if (IsCustomStructureParam(Pin)) {
        UpdatePinTypeFromConnection(Pin);
        ReconstructNode(); // Refresh dependent pins
    }
}
```

### 4. Dynamic Output Resolution
**Algorithm**: Output types change based on input class parameters.

```cpp
void ConformOutputType() {
    UClass* InputClass = ExtractClassFromPin(TypePickerPin);
    for (UEdGraphPin* OutputPin : DynamicOutputPins) {
        OutputPin->PinType.PinSubCategoryObject = InputClass;
    }
}
```

## Blueprint to C++ Conversion Relevance

This file is **essential** for Blueprint-to-C++ conversion because it:

### 1. Function Call Translation
**Blueprint Node** → **C++ Function Call**
- Pin connections → Function parameters
- Return value pins → C++ return values or out parameters
- Context pins → Object member function calls

### 2. Type System Mapping
**Dynamic Types**: Shows how Blueprint's flexible typing maps to C++ templates/casting:
```cpp
// Blueprint: Dynamic cast node
// C++ Equivalent:
if (AActor* CastResult = Cast<AActor>(SourceObject)) {
    // Use CastResult
}
```

### 3. Parameter Handling Patterns
**Custom Structure Parameters**:
```cpp
// Blueprint: Wildcard struct pin
// C++ Template Equivalent:
template<typename T>
void ProcessStruct(const T& StructData) {
    // Template-based processing
}
```

### 4. Metadata to Code Patterns
**Latent Functions**:
```cpp
// Blueprint: Latent node with execution pins
// C++ Equivalent:
void StartAsyncOperation() {
    AsyncTask([this]() {
        // Async work
        OnComplete.ExecuteIfBound();
    });
}
```

### 5. Context Resolution Patterns
**Object Member Access**:
```cpp
// Blueprint: Function call with target pin
// C++ Equivalent:
if (TargetObject) {
    TargetObject->FunctionName(Parameters);
}
```

## Conversion Implementation Guidance

### Key Mapping Patterns:
1. **Pin Connections** → **Parameter Passing**
2. **Dynamic Types** → **Template Instantiation**
3. **Execution Pins** → **Control Flow Statements**
4. **Latent Nodes** → **Async/Await Patterns**
5. **Custom Structs** → **Template Specialization**

This node type is fundamental to understanding how Blueprint visual programming maps to traditional C++ function calls, making it crucial for any Blueprint-to-C++ conversion system.