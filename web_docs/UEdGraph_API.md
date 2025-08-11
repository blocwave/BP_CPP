# UEdGraph API Documentation

Source: https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Runtime/Engine/EdGraph/UEdGraph?application_version=5.2

## Class Definition

```cpp
class UEdGraph : public UObject
```

## Inheritance Hierarchy

- UObjectBase
- UObjectBaseUtility
- UObject
- **UEdGraph**
  - UAIGraph
  - UBehaviorTreeGraph
  - UConversationGraph
  - UEnvironmentQueryGraph
  - UAnimationGraph
  - UAnimationBlendSpaceSampleGraph
  - UAnimationCustomTransitionGraph
  - UAnimationStateGraph
  - UAnimationTransitionGraph
  - UAnimationStateMachineGraph
  - UBehaviorTreeDecoratorGraph
  - UBlendSpaceGraph
  - UCustomizableObjectGraph
  - UDataflow
  - UEdGraph_ReferenceViewer
  - UGameplayAbilityGraph
  - UMaterialGraph
  - UMetasoundEditorGraphBase
  - UMetasoundEditorGraph
  - UNiagaraGraph
  - URigVMEdGraph
  - UControlRigGraph
  - USoundClassGraph
  - USoundCueGraph
  - USoundSubmixGraph

## Module Information

- **Module**: Engine
- **Header**: /Engine/Source/Runtime/Engine/Classes/EdGraph/EdGraph.h
- **Include**: `#include "EdGraph/EdGraph.h"`

## Properties

### Core Properties
- `Schema` (TSubclassOf<UEdGraphSchema>) - The schema that this graph obeys
- `Nodes` (TArray<TObjectPtr<UEdGraphNode>>) - Set of all nodes in this graph
- `SubGraphs` (TArray<TObjectPtr<UEdGraph>>) - Child graphs that are a part of this graph; the separation is purely visual

### Graph Metadata
- `GraphGuid` (FGuid) - Guid for this graph
- `InterfaceGuid` (FGuid) - Guid of interface graph this graph comes from (used for conforming)

### Graph Behavior Flags
- `bAllowDeletion` (uint32: 1) - If true, graph can be deleted from the whatever container it is in
- `bAllowRenaming` (uint32: 1) - If true, graph can be renamed; Note: Graph can also be renamed if bAllowDeletion is true currently
- `bEditable` (uint32: 1) - If true, graph can be edited by the user

## Key Methods

### Node Management

#### AddNode
```cpp
virtual void AddNode(UEdGraphNode* NodeToAdd, bool bUserAction, bool bSelectNewNode)
```
Add a node to the graph

#### RemoveNode
```cpp
bool RemoveNode(UEdGraphNode* NodeToRemove, bool bBreakAllLinks)
```
Remove a node from this graph

#### CreateNode (Protected)
```cpp
UEdGraphNode* CreateNode(TSubclassOf<UEdGraphNode> NewNodeClass, bool bFromUI, bool bSelectNewNode)
UEdGraphNode* CreateNode(TSubclassOf<UEdGraphNode> NewNodeClass, bool bSelectNewNode)
```
Creates an empty node in this graph

#### CreateUserInvokedNode (Protected)
```cpp
UEdGraphNode* CreateUserInvokedNode(TSubclassOf<UEdGraphNode> NewNodeClass, bool bSelectNewNode)
```

#### CreateIntermediateNode
```cpp
template<typename NodeClass>
NodeClass* CreateIntermediateNode()
```

### Graph Query Methods

#### GetNodesOfClass
```cpp
template<typename MinRequiredType>
void GetNodesOfClass(TArray<MinRequiredType*>& OutNodes) const
```
Gets all the nodes in the graph of a given type

#### GetNodesOfClassEx
```cpp
template<typename ArrayElementType, typename MinRequiredType>
void GetNodesOfClassEx(TArray<ArrayElementType*>& OutNodes) const
```
Finds all the nodes of a given minimum type in the graph

#### GetAllChildrenGraphs
```cpp
void GetAllChildrenGraphs(TArray<UEdGraph*>& Graphs) const
```
Get all children graphs in the specified graph

#### GetOuterGraph (Static)
```cpp
static UEdGraph* GetOuterGraph(UObject* Obj)
```
Get parent outer graph, if it exists

### Schema Access
```cpp
const UEdGraphSchema* GetSchema() const
```
Get the schema associated with this graph

### Graph Operations

#### MoveNodesToAnotherGraph
```cpp
void MoveNodesToAnotherGraph(UEdGraph* DestinationGraph, bool bIsLoading, bool bInIsCompiling)
```
Move all nodes from this graph to another graph

#### GetGoodPlaceForNewNode
```cpp
FVector2D GetGoodPlaceForNewNode()
```
Util to find a good place for a new node

#### SelectNodeSet
```cpp
void SelectNodeSet(TSet<const UEdGraphNode*> NodeSelection, bool bFromUI)
```
Queues up a select operation for a series of nodes in this graph

### Notification System

#### NotifyGraphChanged
```cpp
virtual void NotifyGraphChanged()
virtual void NotifyGraphChanged(const FEdGraphEditAction& Action) // Protected
```
Signal to listeners that the graph has changed

#### NotifyNodeChanged
```cpp
void NotifyNodeChanged(const UEdGraphNode* Node)
```
Signal to listeners that a node has changed in the graph

#### AddOnGraphChangedHandler
```cpp
FDelegateHandle AddOnGraphChangedHandler(const FOnGraphChanged::FDelegate& InHandler)
```
Add a listener for OnGraphChanged events

#### RemoveOnGraphChangedHandler
```cpp
void RemoveOnGraphChangedHandler(FDelegateHandle Handle)
```
Remove a listener for OnGraphChanged events

### Property Change Notifications

#### AddPropertyChangedNotifier
```cpp
FDelegateHandle AddPropertyChangedNotifier(const FOnPropertyChanged::FDelegate& InDelegate)
```
Add a delegate listening for property change notifications

#### RemovePropertyChangedNotifier
```cpp
void RemovePropertyChangedNotifier(FDelegateHandle InHandle)
```
Remove a delegate listening for property changed notifications

#### NotifyPreChange
```cpp
void NotifyPreChange(const FString& PropertyName)
```
Notify the graph and its associated listeners that a property is about to change

#### NotifyPostChange
```cpp
void NotifyPostChange(const FPropertyChangedEvent& PropertyChangedEvent, const FString& PropertyName)
```
Notify the graph and associated listeners that a property has changed

## Overridden from UObject

- `PostInitProperties()` - Called after the C++ constructor and after properties have been initialized
- `PostLoad()` - Do any object-specific cleanup required immediately after loading an object
- `Serialize(FStructuredArchiveRecord Record)` - Handles reading, writing, and reference collecting
- `BuildSubobjectMapping(UObject* OtherObject, TMap<UObject*, UObject*>& ObjectMapping) const` - Recursively find matching sub-objects

## Constructor

```cpp
UEdGraph(const FObjectInitializer& ObjectInitializer)
```

## Notes

- UEdGraph is the base class for all graph types in Unreal Engine
- It manages nodes, subgraphs, and provides the infrastructure for visual scripting
- The Schema property determines what types of nodes and connections are valid in the graph
- This class is fundamental to the Blueprint system and all visual scripting in UE5