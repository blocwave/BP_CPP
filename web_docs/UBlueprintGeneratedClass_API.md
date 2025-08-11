# UBlueprintGeneratedClass API Documentation

Source: https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Runtime/Engine/Engine/UBlueprintGeneratedClass?application_version=5.2

## Class Definition

```cpp
class UBlueprintGeneratedClass : public UClass, public IBlueprintPropertyGuidProvider
```

## Purpose

The UBlueprintGeneratedClass is a class that represents a blueprint generated class in Unreal Engine, allowing for the creation and management of blueprint assets and their properties. This is the runtime representation of a Blueprint that contains all the compiled data needed for instantiation and execution.

## Key Properties

### Component Management
- `ComponentTemplates` (TArray<TObjectPtr<UActorComponent>>) - Array of component template objects, used by AddComponent function
- `SimpleConstructionScript` (TObjectPtr<USimpleConstructionScript>) - 'Simple' construction script - graph of components to instance
- `InheritableComponentHandler` (TObjectPtr<UInheritableComponentHandler>) - Stores data to override components from parent classes
- `ComponentClassOverrides` (TArray<FBPComponentClassOverride>) - Array of blueprint overrides of component classes in parent classes

### Timeline Support
- `Timelines` (TArray<TObjectPtr<UTimelineTemplate>>) - Array of templates for timelines that should be created

### Delegate Binding
- `DynamicBindingObjects` (TArray<TObjectPtr<UDynamicBlueprintBinding>>) - Array of objects containing information for dynamically binding delegates to functions in this blueprint

### Cooked Data
- `bHasCookedComponentInstancingData` (uint8: 1) - Flag used to indicate if this class has data to support the component instancing fast path
- `CookedComponentInstancingData` (TMap<FName, FBlueprintCookedComponentInstancingData>) - Mapping of changed properties & data to apply when instancing components in a cooked build
- `CookedPropertyGuids` (TMap<FName, FGuid>) - Property guid map for use only when this BP is cooked

### Property Management
- `PropertyGuids` (TMap<FName, FGuid>) - Property guid map
- `NumReplicatedProperties` (int32) - Number of replicated properties
- `bIsSparseClassDataSerializable` (uint8: 1) - Used to check if this class has sparse data that can be serialized

### Field Notify System
- `FieldNotifies` (TArray<FFieldNotificationId>) - The name of the properties with FieldNotify
- `FieldNotifiesStartBitNumber` (int32) - Starting bit number for field notifies

### Internal Data
- `CalledFunctions` (TArray<TObjectPtr<UFunction>>) - Functions called by this class
- `DebugData` (FBlueprintDebugData) - Debug data for the blueprint
- `OverridenArchetypeForCDO` (TObjectPtr<UObject>) - Overridden archetype for CDO

## Key Methods

### Construction and Initialization
- `UBlueprintGeneratedClass(const FObjectInitializer& ObjectInitializer)` - Constructor

### Component Creation
- `CreateComponentsForActor(const UClass* ThisClass, AActor* Actor)` - Create Timeline objects and components for this Actor
- `CreateTimelineComponent(AActor* Actor, const UTimelineTemplate* TimelineTemplate)` - Create a specific timeline component
- `FindComponentTemplateByName(const FName& TemplateName) const` - Find the object in the TemplateObjects array with the supplied name

### Dynamic Binding
- `BindDynamicDelegates(const UClass* ThisClass, UObject* InInstance)` - Bind functions on supplied actor to delegates
- `UnbindDynamicDelegates(const UClass* ThisClass, UObject* InInstance)` - Unbind functions on supplied actor from delegates
- `GetDynamicBindingObject(const UClass* ThisClass, const UClass* BindingClass)` - Finds the desired dynamic binding object

### Fast Path Component Instancing
- `UseFastPathComponentInstancing()` - Whether or not to use 'fast path' component instancing
- `BuildCustomPropertyListForPostConstruction(...)` - Internal helper to build custom property list from array property
- `BuildCustomPropertyListForPostConstruction(...)` - Internal helper to recursively build the custom property list
- `GetCustomPropertyListForPostConstruction()` - Returns linked list of properties with default values that differ from parent
- `UpdateCustomPropertyListForPostConstruction()` - Called when custom list of properties needs to be rebuilt
- `InitPropertiesFromCustomList(...)` - Helper method to assist with initializing object properties
- `InitArrayPropertyFromCustomList(...)` - Helper method to assist with initializing from an array property

### Sparse Class Data
- `ConformSparseClassData(UObject* Object)` - Called to conform any pending sparse class data
- `PrepareToConformSparseClassData(UScriptStruct* SparseClassDataArchetypeStruct)` - Called during serialization to allow the class to stash any sparse class data

### Field Notification
- `ForEachFieldNotify(TFunctionRef<bool(const UE::FieldNotification::FFieldId FieldId)> Callback, bool bIncludeSuper)` - Called by the UE::FieldNotification::IClassDescriptor
- `InitializeFieldNotifies()` - Initialize field notifies

### Inheritance and Hierarchy
- `ForEachGeneratedClassInHierarchy(const UClass* InClass, TFunctionRef<bool(const UBlueprintGeneratedClass*)> InFunc)` - Iterate over all BPGCs used to generate this class
- `GetGeneratedClassesHierarchy(const UClass* InClass, TArray<const UBlueprintGeneratedClass*>& OutBPGClasses)` - Gets an array of all BPGeneratedClasses

### Replication
- `GetLifetimeBlueprintReplicationList(TArray<FLifetimeProperty>& OutLifetimeProps)` - Called to gather blueprint replicated properties

### Miscellaneous
- `GetInheritableComponentHandler(const bool bCreateIfNecessary)` - Get the inheritable component handler
- `GetDebugData()` - Get debug data
- `GetCookedMetaDataClass()` - Get cooked metadata class
- `GetUberGraphFrameName()` - Get uber graph frame name
- `UsePersistentUberGraphFrame()` - Check if persistent uber graph frame should be used
- `AddReferencedObjectsInUbergraphFrame(UObject* InThis, FReferenceCollector& Collector)` - Add referenced objects in uber graph frame

## Notes

- The full UBlueprintGeneratedClass API documentation is also very large (>26,000 tokens) and requires manual copying
- This class is the runtime representation of a compiled Blueprint
- It contains all the data structures needed to instantiate and execute Blueprint instances
- Works closely with UBlueprint during compilation and with instances during runtime