# USimpleConstructionScript API Reference

**Module:** Engine  
**Header:** /Engine/Source/Runtime/Engine/Classes/Engine/SimpleConstructionScript.h  
**Include:** `#include "Engine/SimpleConstructionScript.h"`

## Class Definition

```cpp
class USimpleConstructionScript : public UObject
```

## Overview

USimpleConstructionScript is a core class in Unreal Engine that manages the construction and organization of components within Blueprint actors. It serves as the foundation for Blueprint component hierarchies, handling the creation, execution, and management of component construction scripts.

## Inheritance Hierarchy

- UObjectBase
- UObjectBaseUtility  
- UObject
- **USimpleConstructionScript**

## Key Properties

- **ComponentTemplateNameSuffix**: Suffix used for component template object names

## Primary Functions

### Node Management

- **AddNode(USCS_Node* Node)**: Adds a node to the root set
- **RemoveNode(USCS_Node* Node, bool bValidateSceneRootNodes)**: Removes a node from the script
- **RemoveNodeAndPromoteChildren(USCS_Node* Node)**: Removes a node and promotes its first child to replace it if it's the root
- **FindSCSNode(const FName InName)**: Finds an SCS node by name
- **FindSCSNodeByGuid(const FGuid Guid)**: Finds an SCS node by GUID
- **FindParentNode(USCS_Node* InNode)**: Finds the parent node of the specified node

### Node Creation

- **CreateNode(UClass* NewComponentClass, FName NewComponentVariableName)**: Creates a new SCS node using the given class
- **CreateNodeAndRenameComponent(UActorComponent* ExistingTemplate)**: Creates a new SCS node using an existing component template
- **GenerateNewComponentName(const UClass* ComponentClass, FName DesiredName)**: Generates a unique name for new components

### Script Execution

- **ExecuteScriptOnActor(AActor* Actor, ...)**: Executes the construction script on the specified actor, creating components according to the script's node hierarchy

### Hierarchy Access

- **GetRootNodes()**: Returns read-only access to the root node set
- **GetAllNodes()**: Returns all nodes in the tree as a flat list
- **GetDefaultSceneRootNode()**: Provides access to the default scene root node
- **GetSceneRootComponentTemplate(bool bShouldUseDefaultRoot, USCS_Node** OutSCSNode)**: Finds the current scene root component template

### Blueprint Integration

- **GetBlueprint()**: Returns the Blueprint associated with this SCS instance
- **GetOwnerClass()**: Returns the Blueprint class associated with this SCS
- **GetParentClass()**: Returns the parent class of the SCS owner class

### Editor Support

- **BeginEditorComponentConstruction()**: Prepares for constructing editable components in the SCS editor
- **EndEditorComponentConstruction()**: Cleans up after constructing editable components
- **ClearEditorComponentReferences()**: Clears all SCS editor component references
- **SetComponentEditorActorInstance(AActor* InActor)**: Sets the actor instance for component editing
- **GetComponentEditorActorInstance()**: Gets the SCS editor actor instance
- **IsConstructingEditorComponents()**: Checks if currently constructing components in the SCS editor

### Validation

- **ValidateNodeTemplates(FCompilerResultsLog& MessageLog)**: Ensures all nodes have valid templates
- **ValidateNodeVariableNames(FCompilerResultsLog& MessageLog)**: Ensures all nodes have valid names for compilation/replication
- **ValidateSceneRootNodes()**: Checks the root node set for scene components and validates it
- **FixupRootNodeParentReferences()**: Ensures all root node parent references are valid

### Performance Optimization

- **CreateNameToSCSNodeMap()**: Creates a map from names to SCS nodes to improve FindSCSNode performance
- **RemoveNameToSCSNodeMap()**: Removes the name-to-node mapping
- **PreloadChain()**: Preloads the construction script chain

### Utility Functions

- **RegisterInstancedComponent(UActorComponent* Component)** [Static]: Helper method to register instanced components post-construction
- **SaveToTransactionBuffer()**: Saves the current state to the transaction buffer for undo/redo support

## Usage in Blueprint Construction

USimpleConstructionScript is fundamental to Blueprint component management:

1. **Component Hierarchy**: Manages the tree structure of components in a Blueprint
2. **Construction Execution**: Executes the construction script during actor instantiation
3. **Editor Integration**: Provides the foundation for the Blueprint component editor
4. **Serialization**: Handles saving/loading of component configurations
5. **Validation**: Ensures component hierarchies are valid during compilation

## Key Relationships

- **USCS_Node**: Individual nodes in the construction script tree
- **UBlueprint**: The owning Blueprint class
- **AActor**: Target actor for script execution  
- **UActorComponent**: Component templates managed by the script

## Common Use Cases

- Creating and managing Blueprint component hierarchies
- Executing construction scripts during actor spawning
- Blueprint editor component tree management
- Component template validation and compilation
- Undo/redo operations in the Blueprint editor