# USCS_Node API Reference

**Module:** Engine  
**Header:** /Engine/Source/Runtime/Engine/Classes/Engine/SCS_Node.h  
**Include:** `#include "Engine/SCS_Node.h"`

## Class Definition

```cpp
class USCS_Node : public UObject
```

## Overview

USCS_Node represents an individual node within a Simple Construction Script (SCS) tree. Each node corresponds to a component within a Blueprint actor's component hierarchy, containing the template for that component and managing its relationships with parent and child components.

## Inheritance Hierarchy

- UObjectBase
- UObjectBaseUtility  
- UObject
- **USCS_Node**

## Key Properties

### Core Component Data
- **ComponentClass**: The UClass type of the component this node represents
- **ComponentTemplate**: The template instance of the component that will be created
- **VariableGuid**: Unique identifier for this node

### Hierarchy Management
- **ChildNodes**: Array of child nodes in the component hierarchy
- **ParentComponentOrVariableName**: Name of the parent component or variable
- **ParentComponentOwnerClassName**: Blueprint class name that owns the parent component template
- **AttachToName**: Specific socket or bone name for attachment
- **bIsParentComponentNative**: Indicates if the parent component is native

### Editor Integration
- **EditorComponentInstance**: The scene component used for editing in the SCS editor
- **CategoryName**: Display category name in the editor

### Metadata and Optimization
- **MetaDataArray**: Array of metadata entries for this node
- **CookedComponentInstancingData**: Cached data for faster runtime instancing in cooked builds

## Primary Functions

### Hierarchy Management

- **AddChildNode(USCS_Node* InNode, bool bAddToAllNodes)**: Adds a child node to this node
- **RemoveChildNode(USCS_Node* InNode, bool bRemoveFromAllNodes)**: Removes a specific child node
- **RemoveChildNodeAt(int32 ChildIndex, bool bRemoveFromAllNodes)**: Removes a child node at a specific index
- **MoveChildNodes(USCS_Node* SourceNode, int32 InsertLocation)**: Moves child nodes from another node
- **GetChildNodes()**: Returns the array of child nodes
- **GetAllNodes()**: Returns this node and all descendant nodes as a flat array

### Parent-Child Relationships

- **SetParent(USCS_Node* InParentNode)**: Sets parent based on another SCS node
- **SetParent(const USceneComponent* InParentComponent)**: Sets parent based on component instance
- **GetParentComponentTemplate(UBlueprint* InBlueprint)**: Finds parent component template via Blueprint
- **GetParentComponentTemplate(UBlueprintGeneratedClass* BPGC)**: Finds parent component template via generated class
- **IsChildOf(USCS_Node* TestParent)**: Tests if this node is a child of another node
- **IsRootNode()**: Tests if this node is a root node

### Component Template Access

- **GetActualComponentTemplate(UBlueprintGeneratedClass* ActualBPGC)**: Gets the actual component template (accounting for overrides)
- **GetActualComponentTemplateData(UBlueprintGeneratedClass* ActualBPGC)**: Gets cooked component template data

### Variable Management

- **GetVariableName()**: Gets the variable name for this component
- **SetVariableName(const FName& NewName, bool bRenameTemplate)**: Sets the variable name
- **NameWasModified()**: Signals that the name was modified externally
- **SetOnNameChanged(const FSCSNodeNameChanged& OnChange)**: Sets callback for name changes

### Metadata

- **GetMetaData(FName Key)**: Gets a metadata value (asserts if not present)
- **SetMetaData(FName Key, FString Value)**: Sets a metadata value
- **RemoveMetaData(FName Key)**: Removes a metadata entry
- **FindMetaDataEntryIndexForKey(FName Key)**: Finds the index of a metadata entry

### Script Execution

- **ExecuteNodeOnActor(AActor* Actor, USceneComponent* ParentComponent, ...)**: Creates the component on the target actor and processes child nodes

### Utility Functions

- **GetSCS()**: Gets the owning Simple Construction Script
- **ValidateGuid()**: Ensures the GUID is valid for backward compatibility
- **SaveToTransactionBuffer()**: Saves current state for undo/redo
- **PreloadChain()**: Preloads this node and all child nodes recursively

### Static Helper Functions

- **RenameComponentTemplate(UActorComponent* ComponentTemplate, const FName& NewName)** [Protected Static]: Renames a component template and its instances

## Usage in Blueprint Construction

USCS_Node is central to Blueprint component management:

1. **Component Hierarchy**: Each node represents one component and its position in the hierarchy
2. **Template Storage**: Stores the component template that serves as the blueprint for instances
3. **Parent-Child Relationships**: Manages attachment relationships between components
4. **Editor Integration**: Provides editor-specific functionality for component editing
5. **Runtime Creation**: Executes during actor construction to create actual component instances

## Key Relationships

- **USimpleConstructionScript**: The owning construction script
- **UActorComponent**: The component template this node represents
- **USceneComponent**: Parent component for scene component nodes
- **UBlueprint/UBlueprintGeneratedClass**: The owning Blueprint classes

## Common Use Cases

- Representing individual components in Blueprint component trees
- Managing parent-child relationships between components
- Storing component configuration and metadata
- Editor operations like drag-and-drop, rename, and property editing
- Runtime component instantiation during actor construction
- Undo/redo operations in the Blueprint editor

## Deprecated Properties

The following properties are deprecated and maintained for backward compatibility:
- **bIsNative_DEPRECATED**: Previously indicated native components
- **bVariableNameAutoGenerated_DEPRECATED**: Previously tracked auto-generated names
- **NativeComponentName_DEPRECATED**: Previously stored native component names