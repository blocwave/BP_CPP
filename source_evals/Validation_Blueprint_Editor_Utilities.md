# Blueprint Editor Utilities and Validation System

## Overview
The FBlueprintEditorUtils class provides essential utilities for Blueprint manipulation, validation, and structural changes. This system is crucial for C++ conversion as it handles Blueprint integrity, node management, and validation workflows.

## Core Utility Functions

### Blueprint Modification and Validation

#### Node Reconstruction and Refresh
```cpp
class FBlueprintEditorUtils {
    // Complete Blueprint refresh
    static void RefreshAllNodes(UBlueprint* Blueprint);
    static void ReconstructAllNodes(UBlueprint* Blueprint);
    
    // External dependency handling
    static void RefreshExternalBlueprintDependencyNodes(
        UBlueprint* Blueprint, 
        UStruct* RefreshOnlyChild = NULL
    );
    
    // Individual graph refresh
    static void RefreshGraphNodes(const UEdGraph* Graph);
    
    // Deprecated node replacement
    static void ReplaceDeprecatedNodes(UBlueprint* Blueprint);
};
```

#### Blueprint State Management
```cpp
// Structural changes (affects compilation)
static void MarkBlueprintAsStructurallyModified(UBlueprint* Blueprint);

// Content changes (requires recompilation)
static void MarkBlueprintAsModified(
    UBlueprint* Blueprint, 
    FPropertyChangedEvent PropertyChangedEvent = FPropertyChangedEvent(nullptr)
);

// Compilation control
static bool ShouldRegenerateBlueprint(UBlueprint* Blueprint);
static bool IsCompileOnLoadDisabled(UBlueprint* Blueprint);
```

### Graph Management

#### Graph Creation and Manipulation
```cpp
// Create new graph
static class UEdGraph* CreateNewGraph(
    UObject* ParentScope, 
    const FName& GraphName, 
    TSubclassOf<class UEdGraph> GraphClass, 
    TSubclassOf<class UEdGraphSchema> SchemaClass
);

// Function graph creation
template <typename SignatureType>
static void CreateFunctionGraph(
    UBlueprint* Blueprint, 
    class UEdGraph* Graph, 
    bool bIsUserCreated, 
    SignatureType* SignatureFromObject
);

// Add various graph types
static void AddFunctionGraph(UBlueprint* Blueprint, class UEdGraph* Graph, bool bIsUserCreated, SignatureType* SignatureFromObject);
static void AddMacroGraph(UBlueprint* Blueprint, class UEdGraph* Graph, bool bIsUserCreated, UClass* SignatureFromClass);
static void AddInterfaceGraph(UBlueprint* Blueprint, class UEdGraph* Graph, UClass* InterfaceClass);
static void AddUbergraphPage(UBlueprint* Blueprint, class UEdGraph* Graph);
```

#### Graph Removal and Cleanup
```cpp
// Graph removal
static void RemoveGraphs(UBlueprint* Blueprint, const TArray<class UEdGraph*>& GraphsToRemove);
static void RemoveGraph(UBlueprint* Blueprint, class UEdGraph* GraphToRemove, EGraphRemoveFlags::Type Flags = EGraphRemoveFlags::Default);

// Graph naming
static void RenameGraph(class UEdGraph* Graph, const FString& NewName);
static void RenameGraphWithSuggestion(class UEdGraph* Graph, TSharedPtr<class INameValidatorInterface> NameValidator, const FString& DesiredName);
```

### Node Management

#### Node Utilities
```cpp
// Node removal
static void RemoveNode(UBlueprint* Blueprint, UEdGraphNode* Node, bool bDontRecompile=false);

// Graph hierarchy navigation
static UEdGraph* GetTopLevelGraph(const UEdGraph* InGraph);
static bool IsGraphReadOnly(UEdGraph* InGraph);

// Node finding
static UK2Node_Event* FindOverrideForFunction(const UBlueprint* Blueprint, const UClass* SignatureClass, FName SignatureName);
static UK2Node_Event* FindCustomEventNode(const UBlueprint* Blueprint, FName const CustomName);
```

#### Blueprint Context Resolution
```cpp
// Blueprint ownership resolution
static UBlueprint* FindBlueprintForNode(const UEdGraphNode* Node);
static UBlueprint* FindBlueprintForNodeChecked(const UEdGraphNode* Node);
static UBlueprint* FindBlueprintForGraph(const UEdGraph* Graph);
static UBlueprint* FindBlueprintForGraphChecked(const UEdGraph* Graph);
```

## Pin Change Management System

### FBasePinChangeHelper
Base class for handling pin changes across Blueprint system:
```cpp
class FBasePinChangeHelper {
public:
    // Node validation
    static bool NodeIsNotTransient(const UK2Node* Node);
    
    // Virtual methods for different node types
    virtual void EditCompositeTunnelNode(class UK2Node_Tunnel* TunnelNode) {}
    virtual void EditMacroInstance(class UK2Node_MacroInstance* MacroInstance, UBlueprint* Blueprint) {}
    virtual void EditCallSite(class UK2Node_CallFunction* CallSite, UBlueprint* Blueprint) {}
    virtual void EditDelegates(class UK2Node_BaseMCDelegate* CallSite, UBlueprint* Blueprint) {}
    virtual void EditCreateDelegates(class UK2Node_CreateDelegate* CallSite) {}
    
    // Broadcast changes
    void Broadcast(UBlueprint* InBlueprint, class UK2Node_EditablePinBase* InTargetNode, UEdGraph* Graph);
};
```

### FParamsChangedHelper
Specialized helper for tracking parameter changes:
```cpp
class FParamsChangedHelper : public FBasePinChangeHelper {
public:
    TSet<UBlueprint*> ModifiedBlueprints;
    TSet<UEdGraph*> ModifiedGraphs;
    
    // Override all virtual methods to track changes
    virtual void EditCompositeTunnelNode(class UK2Node_Tunnel* TunnelNode) override;
    virtual void EditMacroInstance(class UK2Node_MacroInstance* MacroInstance, UBlueprint* Blueprint) override;
    virtual void EditCallSite(class UK2Node_CallFunction* CallSite, UBlueprint* Blueprint) override;
    virtual void EditDelegates(class UK2Node_BaseMCDelegate* CallSite, UBlueprint* Blueprint) override;
    virtual void EditCreateDelegates(class UK2Node_CreateDelegate* CallSite) override;
};
```

## Class and Type Management

### Class Utilities
```cpp
// Skeleton class management
static UClass* GetSkeletonClass(UClass* FromClass);
static const UClass* GetSkeletonClass(const UClass* FromClass);

// Up-to-date class resolution
static UClass* GetMostUpToDateClass(UClass* FromClass);
static const UClass* GetMostUpToDateClass(const UClass* FromClass);

// Property and function resolution
static bool PropertyStillExists(FProperty* Property);
static FProperty* GetMostUpToDateProperty(FProperty* Property);
static const FProperty* GetMostUpToDateProperty(const FProperty* Property);
static UFunction* GetMostUpToDateFunction(UFunction* Function);
static const UFunction* GetMostUpToDateFunction(const UFunction* Function);
```

### Variable Management
```cpp
// Variable system updates
static void RefreshVariables(UBlueprint* Blueprint);
static void UpdateDelegatesInBlueprint(UBlueprint* Blueprint);

// Child Blueprint validation
static void ValidateBlueprintChildVariables(UBlueprint* Blueprint, const FName& VarName);
```

## Asset and Dependency Management

### Asset Handling
```cpp
// Asset preloading
static void PreloadMembers(UObject* InObject);
static void PreloadConstructionScript(UBlueprint* Blueprint);
static void PreloadConstructionScript(USimpleConstructionScript* SimpleConstructionScript);
static void PreloadBlueprintSpecificData(UBlueprint* Blueprint);

// CDO management
static void PatchNewCDOIntoLinker(UObject* CDO, FLinkerLoad* Linker, int32 ExportIndex, FUObjectSerializeContext* InLoadContext);
static void PatchCDOSubobjectsIntoExport(UObject* PreviousCDO, UObject* NewCDO);
```

### Blueprint Lifecycle
```cpp
// Blueprint creation and destruction
static void PostDuplicateBlueprint(UBlueprint* Blueprint, bool bDuplicateForPIE);
static void RemoveGeneratedClasses(UBlueprint* Blueprint);
static void RemoveStaleFunctions(UBlueprintGeneratedClass* Class, UBlueprint* Blueprint);

// External dependencies
static void LinkExternalDependencies(UBlueprint* Blueprint);
```

## Graph Removal Flags

### EGraphRemoveFlags
```cpp
namespace EGraphRemoveFlags {
    enum Type {
        None = 0x00000000,
        Recompile = 0x00000001,        // Recompile after removal
        MarkTransient = 0x00000002,    // Mark as transient
        Default = Recompile | MarkTransient
    };
}
```

Usage patterns:
- **None**: Batch operations, manual control
- **Recompile**: Immediate recompilation needed
- **MarkTransient**: Temporary removal
- **Default**: Standard removal with cleanup

## Function Analysis Utilities

### Function Validation
```cpp
// Function conversion checks
static bool IsFunctionConvertableToEvent(UBlueprint* const BlueprintObj, UFunction* const Function);

// Override analysis
static UClass* const GetOverrideFunctionClass(UBlueprint* Blueprint, const FName FuncName, UFunction** OutFunction = nullptr);
```

### Function Graph Management
```cpp
// Function creation from nodes
static void CreateMatchingFunction(UK2Node_CallFunction* InNode, TSubclassOf<class UEdGraphSchema> InSchemaClass);

// Ubergraph utilities
static FName GetUbergraphFunctionName(const UBlueprint* ForBlueprint);
```

## Blueprint Events and Notifications

### Event Delegates
```cpp
// Refresh event notifications
DECLARE_MULTICAST_DELEGATE_OneParam(FOnRefreshAllNodes, UBlueprint* /*Blueprint*/);
static FOnRefreshAllNodes OnRefreshAllNodesEvent;

DECLARE_MULTICAST_DELEGATE_OneParam(FOnReconstructAllNodes, UBlueprint* /*Blueprint*/);
static FOnReconstructAllNodes OnReconstructAllNodesEvent;
```

### Event Usage
These events allow external systems to:
1. **Track Changes**: Monitor Blueprint modifications
2. **Synchronize State**: Keep external data synchronized
3. **Validation**: Perform additional validation after changes
4. **Code Generation**: Trigger code generation workflows

## Compiler Integration Utilities

### FFunctionFromNodeHelper
Utility for extracting function information from nodes:
```cpp
struct FFunctionFromNodeHelper {
    UFunction* const Function;
    const UK2Node* const Node;
    
    static UFunction* FunctionFromNode(const UK2Node* Node);
    FFunctionFromNodeHelper(const UObject* Obj);
};
```

### Compiler Relevance
```cpp
// Compiler-relevant node tracking
struct FCompilerRelevantNodeLink {
    UK2Node* Node;
    UEdGraphPin* LinkedPin;
};

typedef TArray<FCompilerRelevantNodeLink, TInlineAllocator<4>> FCompilerRelevantNodeLinkArray;
```

## C++ Code Generation Implications

### Validation Pipeline Integration
The utilities provide validation checkpoints for C++ conversion:

1. **Pre-Conversion Validation**:
   ```cpp
   // Validate Blueprint before conversion
   FBlueprintEditorUtils::RefreshAllNodes(Blueprint);
   FBlueprintEditorUtils::ReplaceDeprecatedNodes(Blueprint);
   ```

2. **Structural Analysis**:
   ```cpp
   // Analyze Blueprint structure
   for (UEdGraph* Graph : Blueprint->FunctionGraphs) {
       UBlueprint* OwnerBlueprint = FBlueprintEditorUtils::FindBlueprintForGraph(Graph);
       // Process graph for conversion
   }
   ```

3. **Node Dependency Resolution**:
   ```cpp
   // Resolve node dependencies
   UClass* SkeletonClass = FBlueprintEditorUtils::GetSkeletonClass(GeneratedClass);
   UFunction* UpToDateFunction = FBlueprintEditorUtils::GetMostUpToDateFunction(Function);
   ```

### Change Management for Conversion
```cpp
// Track changes during conversion process
FParamsChangedHelper ChangeTracker;
// ... perform modifications ...
// ChangeTracker.ModifiedBlueprints contains all affected Blueprints
// ChangeTracker.ModifiedGraphs contains all affected graphs
```

### Asset Dependency Resolution
```cpp
// Ensure all dependencies are loaded before conversion
FBlueprintEditorUtils::PreloadMembers(Blueprint);
FBlueprintEditorUtils::PreloadConstructionScript(Blueprint);
FBlueprintEditorUtils::LinkExternalDependencies(Blueprint);
```

## Best Practices for C++ Conversion

### Pre-Conversion Preparation
1. **Validation**: Use RefreshAllNodes and validation utilities
2. **Dependency Resolution**: Ensure all external dependencies are resolved
3. **Deprecation Handling**: Replace deprecated nodes before conversion
4. **Structural Integrity**: Verify Blueprint structural integrity

### During Conversion
1. **Change Tracking**: Use change helper classes to track modifications
2. **Context Resolution**: Use Blueprint finding utilities for context
3. **Type Resolution**: Use most up-to-date type resolution functions
4. **Asset Management**: Properly handle asset references and dependencies

### Post-Conversion Validation
1. **Structural Validation**: Verify converted code maintains Blueprint semantics
2. **Reference Validation**: Ensure all references are properly converted
3. **Dependency Validation**: Verify all dependencies are satisfied
4. **Performance Validation**: Check for performance implications of conversion