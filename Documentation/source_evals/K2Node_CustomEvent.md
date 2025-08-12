# K2Node_CustomEvent.cpp - Custom Event Node Implementation

**File**: `/Engine/Source/Editor/BlueprintGraph/Private/K2Node_CustomEvent.cpp`

## Overview
This file implements the UK2Node_CustomEvent class, which represents user-defined custom events in Blueprint graphs. Custom events are Blueprint-specific functions that can be triggered by other nodes and can have custom parameters, networking flags, and execution contexts.

## Role in Compilation Pipeline
- **Event Declaration**: Creates Blueprint-callable functions with custom signatures
- **Parameter Management**: Handles input parameter definition and validation  
- **Function Generation**: Generates UFunction metadata for custom events
- **Networking Support**: Manages RPC (Remote Procedure Call) settings
- **Name Validation**: Ensures unique event names within Blueprint scope

## Key Functions and Algorithms

### Event Function Discovery
```cpp
static UK2Node_CustomEvent const* FindCustomEventNodeFromFunction(UFunction* CustomEventFunc)
```
- **Purpose**: Locates the Blueprint node that corresponds to a generated UFunction
- **Algorithm**:
  1. Validates function is Blueprint-generated (not native C++)
  2. Extracts UBlueprintGeneratedClass from function owner
  3. Searches all CustomEvent nodes in source Blueprint
  4. Matches by CustomFunctionName against UFunction name
- **Use Case**: Event override validation and inheritance checking

### Custom Event Name Validation
```cpp
class FCustomEventNameValidator : public FKismetNameValidator
{
    virtual EValidatorResult IsValid(FString const& Name, bool bOriginal = false) override
}
```
- **Purpose**: Validates custom event names for uniqueness and legality
- **Validation Rules**:
  1. **Base Validation**: Inherits standard Kismet naming rules
  2. **Parent Override Check**: Only allows overriding other custom events
  3. **Transient Graph Handling**: Allows temporary compiler artifacts
  4. **Conflict Detection**: Prevents naming conflicts with existing events
- **Algorithm**:
  ```cpp
  // Check parent class for existing function
  UFunction* ParentFunction = FindUField<UFunction>(Blueprint->ParentClass, *Name);
  
  // Validate override target is also a custom event
  if (ParentFunction != nullptr) {
      UK2Node_CustomEvent const* OverriddenEvent = FindCustomEventNodeFromFunction(ParentFunction);
      if (OverriddenEvent == nullptr) {
          return EValidatorResult::AlreadyInUse; // Can't override native functions
      }
  }
  ```

### Node Title Generation
```cpp
FText UK2Node_CustomEvent::GetNodeTitle(ENodeTitleType::Type TitleType) const
```
- **Purpose**: Generates display title with RPC information
- **Title Format**: `"{FunctionName}{RPCString}\nCustom Event"`
- **RPC String Generation**: Uses `UK2Node_Event::GetLocalizedNetString()` for networking info
- **Performance**: Cached using `CachedNodeTitle` system to avoid expensive formatting

### Pin Allocation and Management
```cpp
void UK2Node_CustomEvent::AllocateDefaultPins()
```
- **Purpose**: Creates the node's input/output pins based on function signature
- **Pin Types**:
  - **Execution Output**: Single exec pin for event flow
  - **Parameter Inputs**: One pin per custom parameter
  - **Networking Pins**: Additional pins for RPC configuration if applicable
- **Dynamic Allocation**: Pin structure updates when parameters are modified

## Integration with Other Compiler Components

### With Blueprint Function System
```cpp
virtual UFunction* FindFunction() const override
virtual FName GetFunctionName() const override  
virtual bool HasExternalDependencies(TArray<class UStruct*>* OptionalOutput) const override
```
- **Function Lookup**: Integrates with Blueprint's function resolution system
- **Dependency Tracking**: Reports dependencies for compilation ordering
- **Signature Management**: Maintains sync between node pins and UFunction parameters

### With Event Graph Compilation
- **Event Registration**: Registers with Blueprint's event system
- **Graph Integration**: Becomes entry point for event execution subgraphs
- **Call Site Generation**: Creates callable functions that can be invoked by other nodes

### With Networking System
```cpp
// Network configuration through function flags
uint32 FunctionFlags = (FUNC_BlueprintEvent | FUNC_Public);
if (bCallInEditor) FunctionFlags |= FUNC_CallInEditor;
// RPC flags added based on networking configuration
```

## Relevance to BP→C++ Conversion

### Function Declaration Generation
- **C++ Function Signature**: `void CustomEventName(ParamType1 Param1, ParamType2 Param2)`
- **UFUNCTION Macro**: Generates appropriate UFUNCTION() with metadata
- **Parameter Mapping**: Blueprint pin types → C++ parameter types
- **Access Specifiers**: Always generates public functions

### Event Implementation Pattern
```cpp
// Generated C++ pattern:
UFUNCTION(BlueprintImplementableEvent, Category = "Events")
void OnCustomEvent(int32 InputValue, FString InputName);

// Or for events with implementation:
UFUNCTION(BlueprintCallable, Category = "Events")  
void OnCustomEvent(int32 InputValue, FString InputName);
```

### Networking Code Generation
- **RPC Functions**: Server/Client/Multicast function variants
- **Validation Functions**: `_Validate` functions for server RPCs
- **Implementation Functions**: `_Implementation` suffixed functions
- **Replication Conditions**: NetCondition and reliability settings

### Parameter Handling
```cpp
// Blueprint Parameter → C++ Parameter mapping:
// - Pin names become parameter names (sanitized)
// - Pin types use IsTypeCompatibleWithProperty() validation
// - Default values become C++ default parameters
// - Reference pins become reference parameters
```

### Event Binding Generation
- **Delegate Declarations**: Creates delegate types for event binding
- **Binding Functions**: `BindUFunction()` calls for dynamic delegates  
- **Event Dispatching**: Generated code for triggering custom events
- **Parameter Passing**: Ensures type-safe parameter forwarding

## Key Data Structures

### UK2Node_CustomEvent Properties
```cpp
class UK2Node_CustomEvent : public UK2Node_Event {
    FName CustomFunctionName;           // Event function name
    bool bCallInEditor;                 // Editor execution flag
    bool bIsDeprecated;                 // Deprecation status
    mutable FNodeTextCache CachedNodeTitle;  // Performance cache
};
```

### Function Metadata Integration
- **Function Flags**: Networking, editor callable, blueprint event flags
- **Parameter Properties**: Generated FProperty objects for each pin
- **Metadata Tags**: Blueprint-specific metadata for tools and reflection

## Special Considerations

### Event Overriding
- Custom events can override parent custom events (but not native functions)
- Override validation ensures signature compatibility
- Name validator prevents illegal override attempts

### Editor Integration
- `bCallInEditor` flag enables event execution in editor context
- Supports Blueprint debugging and testing workflows
- Integration with Blueprint editor's event visualization

### Performance Optimizations  
- Cached node titles to avoid repeated string formatting
- Efficient function lookup through Blueprint's function maps
- Minimal overhead for event registration and lookup

This implementation provides the foundation for user-defined events in Blueprints, enabling custom gameplay logic that can be efficiently translated to C++ function calls with proper networking and parameter handling.