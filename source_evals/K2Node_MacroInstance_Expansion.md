# K2Node_MacroInstance Expansion Analysis

## Overview
This document analyzes how Blueprint macros are represented, expanded, and inlined during compilation. Blueprint macros provide a way to create reusable node graphs that are expanded inline at compile time rather than being called as functions.

## File: K2Node_MacroInstance.cpp

### Core Macro Instance Structure

#### 1. Macro Reference System
```cpp
class UK2Node_MacroInstance : public UK2Node_EditablePinBase
{
    FGraphReference MacroGraphReference;  // Reference to the macro graph
    bool bReconstructNode;               // Flag for node reconstruction needs
    
    // Deprecated - maintained for serialization compatibility
    UEdGraph* MacroGraph_DEPRECATED;
};
```

**Graph Reference Management:**
- Uses `FGraphReference` for stable macro graph references
- Handles Blueprint asset loading and dependency tracking
- Maintains compatibility with older serialization formats

#### 2. Macro Graph Access
```cpp
UEdGraph* GetMacroGraph() const
{
    return MacroGraphReference.GetGraph();
}

UBlueprint* GetMacroBlueprint() const
{
    return MacroGraphReference.GetBlueprint();
}
```

### Pin Allocation and Management

#### 1. Dynamic Pin Creation
Macro instances dynamically create pins based on the referenced macro graph:

```cpp
void UK2Node_MacroInstance::AllocateDefaultPins()
{
    UK2Node::AllocateDefaultPins();
    const UEdGraphSchema_K2* Schema = GetDefault<UEdGraphSchema_K2>();
    
    // Preload macro blueprint and graph
    PreloadObject(MacroGraphReference.GetBlueprint());
    UEdGraph* MacroGraph = MacroGraphReference.GetGraph();
    
    if (MacroGraph != nullptr) {
        PreloadObject(MacroGraph);
        
        // Find macro entry and exit nodes to determine pins
        for (UEdGraphNode* Node : MacroGraph->Nodes) {
            if (UK2Node_MacroEntry* EntryNode = Cast<UK2Node_MacroEntry>(Node)) {
                CreatePinsFromEntryNode(EntryNode);
            }
            if (UK2Node_MacroExit* ExitNode = Cast<UK2Node_MacroExit>(Node)) {
                CreatePinsFromExitNode(ExitNode);
            }
        }
        
        // Set up pin tooltips and metadata from macro definition
        SetupPinTooltips();
    }
}
```

**Pin Inheritance:**
- Input pins from macro entry nodes become instance input pins
- Output pins from macro exit nodes become instance output pins
- Execution pins maintain proper flow control
- Wildcard pins support type inference

#### 2. Pin Type Propagation
```cpp
void UK2Node_MacroInstance::NotifyPinConnectionListChanged(UEdGraphPin* ChangedPin)
{
    Super::NotifyPinConnectionListChanged(ChangedPin);
    
    // Propagate type changes to wildcard pins in the macro
    if (ChangedPin && ChangedPin->PinType.PinCategory == UEdGraphSchema_K2::PC_Wildcard) {
        PropagateWildcardTypeToMacro(ChangedPin);
    }
    
    // Reconstruct if needed to update pin types
    if (bReconstructNode) {
        ReconstructNode();
        bReconstructNode = false;
    }
}
```

### Macro Expansion Process

#### 1. Compilation Overview
During Blueprint compilation, macro instances are expanded inline rather than compiled as function calls:

```cpp
// Pseudo-code for macro expansion during compilation
void ExpandMacroInstance(UK2Node_MacroInstance* MacroInstance)
{
    UEdGraph* MacroGraph = MacroInstance->GetMacroGraph();
    
    // 1. Clone all nodes from macro graph
    TArray<UEdGraphNode*> ClonedNodes;
    CloneMacroNodes(MacroGraph, ClonedNodes);
    
    // 2. Remap macro entry/exit nodes to instance pins
    RemapMacroConnections(MacroInstance, ClonedNodes);
    
    // 3. Replace macro instance with expanded nodes
    ReplaceMacroInstanceWithNodes(MacroInstance, ClonedNodes);
    
    // 4. Update variable references and local scope
    UpdateVariableReferences(ClonedNodes, MacroInstance->GetBlueprint());
}
```

#### 2. Node Cloning Process
**Deep Copy of Macro Nodes:**
- All nodes in macro graph are duplicated
- Node-specific data is preserved (pins, properties, metadata)
- Unique names generated to avoid conflicts
- Internal connections maintained within expanded subgraph

**Variable Scope Handling:**
- Macro local variables become function-scoped temporaries
- External variable references maintained
- Parameter passing through entry/exit pins

#### 3. Connection Remapping
**Entry Node Mapping:**
```cpp
void RemapMacroEntryConnections(UK2Node_MacroEntry* OriginalEntry, UK2Node_MacroInstance* Instance)
{
    // Map instance input pins to macro entry output pins
    for (UEdGraphPin* InstancePin : Instance->GetInputPins()) {
        UEdGraphPin* MacroPin = FindCorrespondingMacroPin(OriginalEntry, InstancePin);
        if (MacroPin && InstancePin->LinkedTo.Num() > 0) {
            // Connect instance input to macro entry output connections
            RedirectPinConnections(InstancePin, MacroPin);
        }
    }
}
```

**Exit Node Mapping:**
```cpp
void RemapMacroExitConnections(UK2Node_MacroExit* OriginalExit, UK2Node_MacroInstance* Instance)
{
    // Map macro exit input pins to instance output pins  
    for (UEdGraphPin* InstancePin : Instance->GetOutputPins()) {
        UEdGraphPin* MacroPin = FindCorrespondingMacroPin(OriginalExit, InstancePin);
        if (MacroPin && InstancePin->LinkedTo.Num() > 0) {
            // Connect macro exit input to instance output connections
            RedirectPinConnections(MacroPin, InstancePin);
        }
    }
}
```

### Metadata and Documentation

#### 1. Macro Metadata System
```cpp
FKismetUserDeclaredFunctionMetadata* UK2Node_MacroInstance::GetAssociatedGraphMetadata(const UEdGraph* AssociatedMacroGraph)
{
    if (AssociatedMacroGraph) {
        if (UBlueprint* Blueprint = FBlueprintEditorUtils::FindBlueprintForGraph(AssociatedMacroGraph)) {
            return &Blueprint->FunctionGraphs.FindChecked(AssociatedMacroGraph)->Metadata;
        }
    }
    return nullptr;
}
```

**Metadata Contents:**
- Macro description and tooltip text
- Category for organization in palette
- Keywords for searching
- Compact node title for condensed display
- Custom node color and icon

#### 2. Node Visualization
```cpp
FText UK2Node_MacroInstance::GetNodeTitle(ENodeTitleType::Type TitleType) const
{
    UEdGraph* MacroGraph = MacroGraphReference.GetGraph();
    if (FKismetUserDeclaredFunctionMetadata* Metadata = GetAssociatedGraphMetadata(MacroGraph)) {
        if (TitleType == ENodeTitleType::FullTitle) {
            return FText::Format(LOCTEXT("MacroNodeTitle", "{0}\nMacro"), 
                                FText::FromString(MacroGraph->GetName()));
        }
        return FText::FromString(MacroGraph->GetName());
    }
    return LOCTEXT("InvalidMacro", "Invalid Macro");
}

FLinearColor UK2Node_MacroInstance::GetNodeTitleColor() const
{
    if (FKismetUserDeclaredFunctionMetadata* Metadata = GetAssociatedGraphMetadata(GetMacroGraph())) {
        // Use custom color if specified in metadata
        if (Metadata->HasMetaData(FBlueprintMetadata::MD_NodeColor)) {
            return Metadata->GetNodeColor();
        }
    }
    // Default macro color (distinct from function calls)
    return FLinearColor(0.8f, 0.6f, 0.2f);  // Golden yellow
}
```

#### 3. Compact Node Mode
```cpp
bool UK2Node_MacroInstance::ShouldDrawCompact() const
{
    return !GetCompactNodeTitle().IsEmpty();
}

FText UK2Node_MacroInstance::GetCompactNodeTitle() const
{
    if (FKismetUserDeclaredFunctionMetadata* Metadata = GetAssociatedGraphMetadata(GetMacroGraph())) {
        return Metadata->CompactNodeTitle;
    }
    return FText::GetEmpty();
}
```

### Asset Management and Dependencies

#### 1. External Blueprint Dependencies
```cpp
bool UK2Node_MacroInstance::HasExternalDependencies(TArray<class UStruct*>* OptionalOutput) const
{
    UBlueprint* OtherBlueprint = MacroGraphReference.GetBlueprint();
    const bool bResult = OtherBlueprint && (OtherBlueprint != GetBlueprint());
    if (bResult && OptionalOutput) {
        OptionalOutput->AddUnique(OtherBlueprint->GeneratedClass);
    }
    return bResult;
}
```

**Dependency Implications:**
- Macro blueprints must be loaded before compilation
- Changes to macro graphs trigger recompilation of dependent blueprints
- Cross-blueprint references require careful asset loading order

#### 2. Asset Preloading
```cpp
void UK2Node_MacroInstance::PreloadRequiredAssets()
{
    PreloadObject(MacroGraphReference.GetBlueprint());
    UEdGraph* MacroGraph = MacroGraphReference.GetGraph();
    if (MacroGraph) {
        PreloadObject(MacroGraph);
        
        // Preload all nodes in the macro for compilation
        for (UEdGraphNode* Node : MacroGraph->Nodes) {
            if (Node) {
                PreloadObject(Node);
                Node->PreloadRequiredAssets();
            }
        }
    }
}
```

### Editor Integration Features

#### 1. Content Browser Integration
```cpp
void UK2Node_MacroInstance::FindInContentBrowser(TWeakObjectPtr<UK2Node_MacroInstance> MacroInstance)
{
    if (MacroInstance.IsValid()) {
        if (UBlueprint* MacroBlueprint = MacroInstance->MacroGraphReference.GetBlueprint()) {
            // Navigate to macro blueprint in content browser
            TArray<UObject*> ObjectsToSyncTo;
            ObjectsToSyncTo.Add(MacroBlueprint);
            GEditor->SyncBrowserToObjects(ObjectsToSyncTo);
        }
    }
}
```

#### 2. Context Menu Actions
```cpp
void UK2Node_MacroInstance::GetNodeContextMenuActions(UToolMenu* Menu, UGraphNodeContextMenuContext* Context) const
{
    if (Context->Pin == nullptr) {
        UToolMenuSection* Section = Menu->AddSection("K2NodeMacroInstance", LOCTEXT("MacroInstanceHeader", "Macro"));
        
        // Add "Go to Macro Definition" option
        Section->AddMenuEntry(
            "GoToMacroDefinition",
            LOCTEXT("GoToMacroDefinition", "Go to Macro Definition"),
            LOCTEXT("GoToMacroDefinitionTooltip", "Opens the macro for editing"),
            FSlateIcon(),
            FUIAction(FExecuteAction::CreateStatic(&UK2Node_MacroInstance::FindInContentBrowser, 
                                                  MakeWeakObjectPtr(const_cast<UK2Node_MacroInstance*>(this))))
        );
    }
    Super::GetNodeContextMenuActions(Menu, Context);
}
```

#### 3. Paste Validation
```cpp
bool UK2Node_MacroInstance::CanPasteHere(const UEdGraph* TargetGraph) const
{
    bool bCanPaste = Super::CanPasteHere(TargetGraph);
    
    // Macro instances are not allowed in their own macro graph (prevents recursion)
    UEdGraph* MacroGraph = GetMacroGraph();
    bCanPaste &= (MacroGraph != TargetGraph);
    
    // Macros with latent functions cannot be used in function graphs
    bool const bIsTargetFuncGraph = (TargetGraph->GetSchema()->GetGraphType(TargetGraph) == GT_Function);
    bCanPaste &= (!bIsTargetFuncGraph || !FBlueprintEditorUtils::CheckIfGraphHasLatentFunctions(MacroGraph));
    
    return bCanPaste;
}
```

### Compilation Optimizations

#### 1. Inline Expansion Benefits
**Performance Advantages:**
- No function call overhead
- Direct variable access (no parameter passing)
- Improved optimization opportunities
- Better inlining by downstream compilers

**Code Size Considerations:**
- Larger compiled bytecode due to duplication
- Balanced by elimination of function setup/teardown
- Dead code elimination can remove unused branches

#### 2. Wildcard Pin Resolution
```cpp
void UK2Node_MacroInstance::PostFixupAllWildcardPins(bool bInAllWildcardPinsUnlinked)
{
    if (bInAllWildcardPinsUnlinked) {
        // Reset wildcard pins in macro when all connections removed
        UEdGraph* MacroGraph = GetMacroGraph();
        if (MacroGraph) {
            ResetWildcardPinsInMacroGraph(MacroGraph);
        }
    }
}
```

**Type Inference Flow:**
1. Connection made to macro instance wildcard pin
2. Type propagated to corresponding macro entry/exit pin
3. Type flows through macro graph connections
4. Other wildcard pins in macro resolve to compatible types
5. Instance pins update to reflect resolved types

### Error Handling and Validation

#### 1. Macro Graph Validation
```cpp
void ValidateMacroGraph(const UEdGraph* MacroGraph)
{
    if (!MacroGraph) {
        CompilerContext.MessageLog.Error(TEXT("Macro instance references invalid graph"));
        return;
    }
    
    // Check for required entry/exit nodes
    bool bHasEntry = false, bHasExit = false;
    for (const UEdGraphNode* Node : MacroGraph->Nodes) {
        if (Cast<UK2Node_MacroEntry>(Node)) bHasEntry = true;
        if (Cast<UK2Node_MacroExit>(Node)) bHasExit = true;
    }
    
    if (!bHasEntry || !bHasExit) {
        CompilerContext.MessageLog.Error(TEXT("Macro graph missing required entry or exit nodes"));
    }
}
```

#### 2. Circular Reference Detection
```cpp
bool CheckForCircularMacroReferences(UK2Node_MacroInstance* StartInstance, TSet<UEdGraph*>& VisitedGraphs)
{
    UEdGraph* MacroGraph = StartInstance->GetMacroGraph();
    if (VisitedGraphs.Contains(MacroGraph)) {
        // Circular reference detected
        return true;
    }
    
    VisitedGraphs.Add(MacroGraph);
    
    // Check all macro instances within this macro
    for (UEdGraphNode* Node : MacroGraph->Nodes) {
        if (UK2Node_MacroInstance* NestedMacro = Cast<UK2Node_MacroInstance>(Node)) {
            if (CheckForCircularMacroReferences(NestedMacro, VisitedGraphs)) {
                return true;
            }
        }
    }
    
    VisitedGraphs.Remove(MacroGraph);
    return false;
}
```

## Blueprint Integration Patterns

### 1. Macro Design Best Practices
**Reusable Logic Blocks:**
- Mathematical operations (vector math, interpolation)
- Common state management patterns
- UI interaction helpers
- Debug visualization utilities

**Parameter Design:**
- Use wildcards for generic operations
- Provide meaningful default values
- Include comprehensive tooltips
- Organize with logical pin groupings

### 2. Performance Considerations
**When to Use Macros vs Functions:**
- **Macros**: Small, frequently used logic blocks
- **Functions**: Complex logic with local state requirements
- **Macros**: Performance-critical code paths
- **Functions**: Logic that may need debugging isolation

**Compilation Impact:**
- Each macro usage increases bytecode size
- Complex macros with many branches can bloat compilation
- Consider function calls for very large macro definitions

### 3. Maintenance and Versioning
**Blueprint Dependencies:**
- Changes to macro graphs require recompilation of all users
- Version compatibility maintained through asset references
- Hot reload support for macro modifications during development

**Asset Organization:**
- Group related macros in dedicated blueprint libraries
- Use consistent naming conventions
- Provide clear documentation and examples
- Consider access restrictions for internal-only macros