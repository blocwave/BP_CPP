# K2 Schema Actions and Node Creation System

## Overview
The EdGraphSchema_K2_Actions system provides the infrastructure for creating, managing, and validating Blueprint nodes through schema actions. This system is critical for C++ conversion as it defines how nodes are instantiated and configured.

## Core Schema Action Types

### FEdGraphSchemaAction_K2NewNode
The primary action for creating new nodes:

```cpp
USTRUCT()
struct FEdGraphSchemaAction_K2NewNode : public FEdGraphSchemaAction {
    // Template node to duplicate
    UPROPERTY()
    TObjectPtr<class UK2Node> NodeTemplate;
    
    // Whether to navigate to new node
    UPROPERTY()
    bool bGotoNode;
    
    // Primary creation method
    virtual UEdGraphNode* PerformAction(
        UEdGraph* ParentGraph, 
        UEdGraphPin* FromPin, 
        const FVector2D Location, 
        bool bSelectNewNode = true
    ) override;
};
```

### Template-Based Node Creation
```cpp
template <typename NodeType>
static NodeType* SpawnNodeFromTemplate(
    class UEdGraph* ParentGraph, 
    NodeType* InTemplateNode, 
    const FVector2D Location, 
    bool bSelectNewNode = true
);

template <typename NodeType>
static NodeType* SpawnNode(
    class UEdGraph* ParentGraph, 
    const FVector2D Location, 
    EK2NewNodeFlags Options, 
    TFunctionRef<void(NodeType*)> InitializerFn
);
```

### Node Creation Flags
```cpp
enum class EK2NewNodeFlags {
    None = 0x0,
    SelectNewNode = 0x1,
    GotoNewNode = 0x2,
};
```

## Specialized Node Actions

### FEdGraphSchemaAction_K2ViewNode
For navigating to existing nodes:
```cpp
USTRUCT()
struct FEdGraphSchemaAction_K2ViewNode : public FEdGraphSchemaAction {
    UPROPERTY()
    TObjectPtr<const UK2Node> NodePtr;
    
    virtual UEdGraphNode* PerformAction(
        class UEdGraph* ParentGraph, 
        UEdGraphPin* FromPin, 
        const FVector2D Location, 
        bool bSelectNewNode = true
    ) override;
};
```

### FEdGraphSchemaAction_K2AssignDelegate
For delegate assignment operations:
```cpp
USTRUCT()
struct FEdGraphSchemaAction_K2AssignDelegate : public FEdGraphSchemaAction_K2NewNode {
    static UEdGraphNode* AssignDelegate(
        class UK2Node* NodeTemplate, 
        class UEdGraph* ParentGraph, 
        UEdGraphPin* FromPin, 
        const FVector2D Location, 
        bool bSelectNewNode
    );
};
```

### FEdGraphSchemaAction_EventFromFunction
Creates event nodes from function signatures:
```cpp
USTRUCT()
struct FEdGraphSchemaAction_EventFromFunction : public FEdGraphSchemaAction {
    UPROPERTY()
    TObjectPtr<class UFunction> SignatureFunction;
    
    virtual UEdGraphNode* PerformAction(
        class UEdGraph* ParentGraph, 
        UEdGraphPin* FromPin, 
        const FVector2D Location, 
        bool bSelectNewNode = true
    ) override;
};
```

## Component and Actor Actions

### FEdGraphSchemaAction_K2AddComponent
For adding component nodes:
```cpp
struct FEdGraphSchemaAction_K2AddComponent : public FEdGraphSchemaAction_K2NewNode {
    // Component class to instantiate
    UPROPERTY()
    TSubclassOf<class UActorComponent> ComponentClass;
    
    // Optional asset to assign
    UPROPERTY()
    TObjectPtr<class UObject> ComponentAsset;
    
    virtual UEdGraphNode* PerformAction(
        class UEdGraph* ParentGraph, 
        UEdGraphPin* FromPin, 
        const FVector2D Location, 
        bool bSelectNewNode = true
    ) override;
};
```

### FEdGraphSchemaAction_K2AddCallOnActor
For calling functions on level actors:
```cpp
struct FEdGraphSchemaAction_K2AddCallOnActor : public FEdGraphSchemaAction_K2NewNode {
    // Target actors in level
    UPROPERTY()
    TArray<TObjectPtr<class AActor>> LevelActors;
    
    virtual UEdGraphNode* PerformAction(
        class UEdGraph* ParentGraph, 
        UEdGraphPin* FromPin, 
        const FVector2D Location, 
        bool bSelectNewNode = true
    ) override;
};
```

## Event and Custom Event Actions

### FEdGraphSchemaAction_K2AddEvent
For standard event nodes:
```cpp
struct FEdGraphSchemaAction_K2AddEvent : public FEdGraphSchemaAction_K2NewNode {
    virtual UEdGraphNode* PerformAction(
        class UEdGraph* ParentGraph, 
        UEdGraphPin* FromPin, 
        const FVector2D Location, 
        bool bSelectNewNode = true
    ) override;
    
    // Check if event already exists
    bool EventHasAlreadyBeenPlaced(
        UBlueprint const* Blueprint, 
        class UK2Node_Event const** FoundEventOut = NULL
    ) const;
};
```

### FEdGraphSchemaAction_K2AddCustomEvent
For user-defined custom events:
```cpp
struct FEdGraphSchemaAction_K2AddCustomEvent : public FEdGraphSchemaAction_K2NewNode {
    virtual UEdGraphNode* PerformAction(
        class UEdGraph* ParentGraph, 
        UEdGraphPin* FromPin, 
        const FVector2D Location, 
        bool bSelectNewNode = true
    ) override;
};
```

## Variable System Actions

### FEdGraphSchemaAction_BlueprintVariableBase
Base class for variable-related actions:
```cpp
USTRUCT()
struct FEdGraphSchemaAction_BlueprintVariableBase : public FEdGraphSchemaAction {
private:
    FName VarName;
    TWeakObjectPtr<UObject> VariableSource;
    bool bIsVarBool;
    
public:
    void SetVariableInfo(const FName& InVarName, const UObject* InOwningScope, bool bInIsVarBool);
    
    FName GetVariableName() const;
    FString GetFriendlyVariableName() const;
    UClass* GetVariableClass() const;
    UObject* GetVariableScope() const;
    
    virtual FProperty* GetProperty() const;
    virtual FEdGraphPinType GetPinType() const;
    virtual void ChangeVariableType(const FEdGraphPinType& NewPinType);
    virtual void RenameVariable(const FName& NewName);
    virtual bool IsValidName(const FName& NewName, FText& OutErrorMessage) const;
    virtual void DeleteVariable();
    virtual bool IsVariableUsed();
};
```

### FEdGraphSchemaAction_K2Var
For member variables:
```cpp
USTRUCT()
struct FEdGraphSchemaAction_K2Var : public FEdGraphSchemaAction_BlueprintVariableBase {
    static FName StaticGetTypeId();
    virtual FName GetTypeId() const override;
    virtual bool IsA(const FName& InType) const override;
};
```

### FEdGraphSchemaAction_K2LocalVar
For local variables:
```cpp
USTRUCT()
struct FEdGraphSchemaAction_K2LocalVar : public FEdGraphSchemaAction_BlueprintVariableBase {
    static FName StaticGetTypeId();
    virtual FName GetTypeId() const override;
    virtual int32 GetReorderIndexInContainer() const override;
    virtual bool IsA(const FName& InType) const override;
};
```

## Graph and Function Actions

### FEdGraphSchemaAction_K2Graph
For graph-related operations:
```cpp
USTRUCT()
struct FEdGraphSchemaAction_K2Graph : public FEdGraphSchemaAction {
    // Function or graph name
    FName FuncName;
    
    // Graph type enumeration
    EEdGraphSchemaAction_K2Graph::Type GraphType;
    
    // Associated editor graph
    UEdGraph* EdGraph;
    
    virtual bool IsParentable() const override { return true; }
    virtual void MovePersistentItemToCategory(const FText& NewCategoryName) override;
    virtual int32 GetReorderIndexInContainer() const override;
    virtual bool ReorderToBeforeAction(TSharedRef<FEdGraphSchemaAction> OtherAction) override;
    
    UFunction* GetFunction() const;
    UBlueprint* GetSourceBlueprint() const;
};
```

### Graph Type Enumeration
```cpp
UENUM()
namespace EEdGraphSchemaAction_K2Graph {
    enum Type : int {
        Graph,       // Standard event graph
        Subgraph,    // Collapsed subgraph
        Function,    // Function graph
        Interface,   // Interface implementation
        Macro,       // Macro graph
        MAX
    };
}
```

## Delegate Actions

### FEdGraphSchemaAction_K2Delegate
For delegate variable operations:
```cpp
USTRUCT()
struct FEdGraphSchemaAction_K2Delegate : public FEdGraphSchemaAction_BlueprintVariableBase {
    // Associated editor graph
    UEdGraph* EdGraph;
    
    FName GetDelegateName() const;
    UClass* GetDelegateClass() const;
    FMulticastDelegateProperty* GetDelegateProperty() const;
    
    virtual bool IsA(const FName& InType) const override;
};
```

## Input and Event System Actions

### FEdGraphSchemaAction_K2Event
For event node management:
```cpp
USTRUCT()
struct FEdGraphSchemaAction_K2Event : public FEdGraphSchemaAction_K2TargetNode {
    static FName StaticGetTypeId();
    virtual FName GetTypeId() const override;
    virtual bool IsParentable() const override { return true; }
};
```

### FEdGraphSchemaAction_K2InputAction
For input action events:
```cpp
USTRUCT()
struct FEdGraphSchemaAction_K2InputAction : public FEdGraphSchemaAction_K2TargetNode {
    static FName StaticGetTypeId();
    virtual FName GetTypeId() const override;
    virtual bool IsParentable() const override { return true; }
};
```

## Utility Actions

### FEdGraphSchemaAction_K2AddComment
For comment node creation:
```cpp
USTRUCT()
struct FEdGraphSchemaAction_K2AddComment : public FEdGraphSchemaAction {
    static FName StaticGetTypeId();
    virtual FName GetTypeId() const override;
    virtual UEdGraphNode* PerformAction(
        class UEdGraph* ParentGraph, 
        UEdGraphPin* FromPin, 
        const FVector2D Location, 
        bool bSelectNewNode = true
    ) override;
};
```

### FEdGraphSchemaAction_K2PasteHere
For paste operations:
```cpp
USTRUCT()
struct FEdGraphSchemaAction_K2PasteHere : public FEdGraphSchemaAction {
    static FName StaticGetTypeId();
    virtual FName GetTypeId() const override;
    virtual UEdGraphNode* PerformAction(
        class UEdGraph* ParentGraph, 
        UEdGraphPin* FromPin, 
        const FVector2D Location, 
        bool bSelectNewNode = true
    ) override;
};
```

## Type Reference Actions

### FEdGraphSchemaAction_K2Struct
For struct type references:
```cpp
USTRUCT()
struct FEdGraphSchemaAction_K2Struct : public FEdGraphSchemaAction {
    UStruct* Struct;
    
    static FName StaticGetTypeId();
    virtual FName GetTypeId() const override;
    
    void AddReferencedObjects(FReferenceCollector& Collector) override;
    FName GetPathName() const;
};
```

### FEdGraphSchemaAction_K2Enum
For enum type references:
```cpp
USTRUCT()
struct FEdGraphSchemaAction_K2Enum : public FEdGraphSchemaAction {
    UEnum* Enum;
    
    static FName StaticGetTypeId();
    virtual FName GetTypeId() const override;
    
    void AddReferencedObjects(FReferenceCollector& Collector) override;
    FName GetPathName() const;
};
```

## C++ Code Generation Implications

### Node Creation Patterns
1. **Template-Based Creation**: Use templates for type-safe node instantiation
2. **Initialization Callbacks**: Configure nodes immediately after creation
3. **Position Management**: Track node positions for layout generation

### Variable Management
1. **Type Safety**: Variable actions ensure type consistency
2. **Scope Management**: Local vs member variable distinctions
3. **Lifetime Management**: Proper variable lifetime tracking

### Function and Event Handling
1. **Signature Matching**: Ensure function signatures match C++ expectations
2. **Event Binding**: Proper event to function binding
3. **Delegate Management**: Correct delegate instantiation and binding

### Validation Integration
1. **Pre-Creation Validation**: Validate before node creation
2. **Post-Creation Setup**: Configure nodes after creation
3. **Context Awareness**: Consider graph context during creation

### Best Practices for C++ Conversion
1. **Action Type Mapping**: Map each action type to appropriate C++ construct
2. **Parameter Extraction**: Extract all necessary parameters for code generation
3. **Context Preservation**: Maintain creation context for proper code generation
4. **Error Handling**: Provide clear error messages for failed actions