# UnrealEngine Source Files to Documentation Index

This document maps UnrealEngine 5.2.1 source files to their corresponding analysis documentation in `/bp_json/ref_docs/BP_CPP/source_evals/`.

## Compilation Pipeline

### KismetCompiler.md
**Source Files:**
- `/Engine/Source/Editor/KismetCompiler/Private/KismetCompiler.cpp`
- `/Engine/Source/Editor/KismetCompiler/Public/KismetCompiler.h`
- `/Engine/Source/Editor/KismetCompiler/Public/KismetCompilerModule.h`

### KismetCompilerVMBackend.md
**Source Files:**
- `/Engine/Source/Editor/KismetCompiler/Private/KismetCompilerVMBackend.cpp`

### KismetCompilerMisc.md
**Source Files:**
- `/Engine/Source/Editor/KismetCompiler/Private/KismetCompilerMisc.cpp`

### BlueprintGeneratedClass_Runtime_Analysis.md
**Source Files:**
- `/Engine/Source/Runtime/Engine/Private/BlueprintGeneratedClass.cpp`
- `/Engine/Source/Runtime/Engine/Classes/Engine/BlueprintGeneratedClass.h`

## Execution Flow System

### Execution_Flow_Compiler_Overview.md
**Source Files:**
- `/Engine/Source/Editor/KismetCompiler/Public/KismetCompilerModule.h`
- `/Engine/Source/Editor/KismetCompiler/Private/KismetCompiler.cpp`

### Execution_Flow_FKismetFunctionContext.md
**Source Files:**
- `/Engine/Source/Editor/KismetCompiler/Private/FKismetFunctionContext.h`
- `/Engine/Source/Editor/KismetCompiler/Private/KismetCompiler.cpp` (contains implementation)

### Execution_Flow_FBlueprintCompiledStatement.md
**Source Files:**
- `/Engine/Source/Editor/KismetCompiler/Public/BlueprintCompiledStatement.h`

### Execution_Flow_FBPTerminal_System.md
**Source Files:**
- `/Engine/Source/Editor/KismetCompiler/Private/FBPTerminal.h`

### Execution_Flow_Linear_Generation.md
**Source Files:**
- `/Engine/Source/Editor/KismetCompiler/Private/KismetCompiler.cpp` (LinearExecutionList generation)

## Graph Structure Core

### Graph_Structure_Core.md
**Source Files:**
- `/Engine/Source/Runtime/Engine/Classes/EdGraph/EdGraph.h`
- `/Engine/Source/Runtime/Engine/Private/EdGraph/EdGraph.cpp`

### Graph_Structure_Node.md
**Source Files:**
- `/Engine/Source/Runtime/Engine/Classes/EdGraph/EdGraphNode.h`
- `/Engine/Source/Runtime/Engine/Private/EdGraph/EdGraphNode.cpp`

### Graph_Structure_Pin.md
**Source Files:**
- `/Engine/Source/Runtime/Engine/Classes/EdGraph/EdGraphPin.h`
- `/Engine/Source/Runtime/Engine/Private/EdGraph/EdGraphPin.cpp`

### Graph_Structure_K2Node.md
**Source Files:**
- `/Engine/Source/Editor/BlueprintGraph/Classes/K2Node.h`
- `/Engine/Source/Editor/BlueprintGraph/Private/K2Node.cpp`

### Graph_Structure_MemberReference.md
**Source Files:**
- `/Engine/Source/Runtime/Engine/Classes/Engine/MemberReference.h`

### EdGraphPin_Connection_Serialization.md
**Source Files:**
- `/Engine/Source/Runtime/Engine/Private/EdGraph/EdGraphPin.cpp`

### EdGraph_Core_Infrastructure.md
**Source Files:**
- `/Engine/Source/Runtime/Engine/Classes/EdGraph/EdGraphSchema.h`
- `/Engine/Source/Runtime/Engine/Private/EdGraph/EdGraphSchema.cpp`

## K2Node Implementations

### K2Node_CallFunction.md
**Source Files:**
- `/Engine/Source/Editor/BlueprintGraph/Private/K2Node_CallFunction.cpp`
- `/Engine/Source/Editor/BlueprintGraph/Classes/K2Node_CallFunction.h`

### K2Node_Event.md
**Source Files:**
- `/Engine/Source/Editor/BlueprintGraph/Private/K2Node_Event.cpp`
- `/Engine/Source/Editor/BlueprintGraph/Classes/K2Node_Event.h`

### K2Node_Variable.md
**Source Files:**
- `/Engine/Source/Editor/BlueprintGraph/Private/K2Node_Variable.cpp`
- `/Engine/Source/Editor/BlueprintGraph/Private/K2Node_VariableGet.cpp`
- `/Engine/Source/Editor/BlueprintGraph/Private/K2Node_VariableSet.cpp`
- `/Engine/Source/Editor/BlueprintGraph/Classes/K2Node_Variable.h`

### K2Node_IfThenElse_Analysis.md
**Source Files:**
- `/Engine/Source/Editor/BlueprintGraph/Private/K2Node_IfThenElse.cpp`
- `/Engine/Source/Editor/BlueprintGraph/Classes/K2Node_IfThenElse.h`

### K2Node_ForEachElementInEnum_Analysis.md
**Source Files:**
- `/Engine/Source/Editor/BlueprintGraph/Private/K2Node_ForEachElementInEnum.cpp`
- `/Engine/Source/Editor/BlueprintGraph/Classes/K2Node_ForEachElementInEnum.h`

### K2Node_BreakStruct_Analysis.md
**Source Files:**
- `/Engine/Source/Editor/BlueprintGraph/Private/K2Node_BreakStruct.cpp`
- `/Engine/Source/Editor/BlueprintGraph/Classes/K2Node_BreakStruct.h`

### K2Node_MakeStruct_Analysis.md
**Source Files:**
- `/Engine/Source/Editor/BlueprintGraph/Private/K2Node_MakeStruct.cpp`
- `/Engine/Source/Editor/BlueprintGraph/Classes/K2Node_MakeStruct.h`

### K2Node_Timeline_Analysis.md
**Source Files:**
- `/Engine/Source/Editor/BlueprintGraph/Private/K2Node_Timeline.cpp`
- `/Engine/Source/Editor/BlueprintGraph/Classes/K2Node_Timeline.h`

### K2Node_SpawnActorFromClass_Analysis.md
**Source Files:**
- `/Engine/Source/Editor/BlueprintGraph/Private/K2Node_SpawnActorFromClass.cpp`
- `/Engine/Source/Editor/BlueprintGraph/Classes/K2Node_SpawnActorFromClass.h`
- `/Engine/Source/Editor/BlueprintGraph/Private/K2Node_SpawnActor.cpp`

### K2Node_DynamicCast_Implementation.md
**Source Files:**
- `/Engine/Source/Editor/BlueprintGraph/Private/K2Node_DynamicCast.cpp`
- `/Engine/Source/Editor/BlueprintGraph/Private/K2Node_ClassDynamicCast.cpp`
- `/Engine/Source/Editor/BlueprintGraph/Classes/K2Node_DynamicCast.h`

### K2Node_MacroInstance_Expansion.md
**Source Files:**
- `/Engine/Source/Editor/BlueprintGraph/Private/K2Node_MacroInstance.cpp`
- `/Engine/Source/Editor/BlueprintGraph/Classes/K2Node_MacroInstance.h`

### K2Node_CustomEvent.md
**Source Files:**
- `/Engine/Source/Editor/BlueprintGraph/Private/K2Node_CustomEvent.cpp`
- `/Engine/Source/Editor/BlueprintGraph/Classes/K2Node_CustomEvent.h`

### K2Node_FunctionEntry.md
**Source Files:**
- `/Engine/Source/Editor/BlueprintGraph/Private/K2Node_FunctionEntry.cpp`
- `/Engine/Source/Editor/BlueprintGraph/Classes/K2Node_FunctionEntry.h`

### K2Node_Composite.md
**Source Files:**
- `/Engine/Source/Editor/BlueprintGraph/Private/K2Node_Composite.cpp`
- `/Engine/Source/Editor/BlueprintGraph/Classes/K2Node_Composite.h`

## Component System

### Component_SimpleConstructionScript.md
**Source Files:**
- `/Engine/Source/Runtime/Engine/Private/SimpleConstructionScript.cpp`
- `/Engine/Source/Runtime/Engine/Classes/Engine/SimpleConstructionScript.h`

### Component_SCSNode.md
**Source Files:**
- `/Engine/Source/Runtime/Engine/Private/SCS_Node.cpp`
- `/Engine/Source/Runtime/Engine/Classes/Engine/SCS_Node.h`

### Component_Blueprint_Hierarchy_System.md
**Source Files:**
- `/Engine/Source/Runtime/Engine/Private/Blueprint.cpp`
- `/Engine/Source/Runtime/Engine/Classes/Engine/Blueprint.h`

## Delegate System

### Delegate_Core_System.md
**Source Files:**
- `/Engine/Source/Runtime/Core/Public/Delegates/Delegate.h`
- `/Engine/Source/Runtime/Core/Public/Delegates/DelegateSignatureImpl.inl`

### Dynamic_Delegate_Bindings.md
**Source Files:**
- `/Engine/Source/Runtime/Core/Public/Delegates/MulticastDelegateBase.h`
- `/Engine/Source/Runtime/CoreUObject/Public/UObject/ScriptDelegates.h`

## Interface System

### Interface_Implementation_System.md
**Source Files:**
- `/Engine/Source/Runtime/CoreUObject/Public/UObject/Interface.h`
- `/Engine/Source/Runtime/CoreUObject/Private/UObject/Interface.cpp`

### Interface_Events.md
**Source Files:**
- `/Engine/Source/Editor/BlueprintGraph/Private/K2Node_Event.cpp` (interface event handling)

## Property System

### FProperty_Type_System.md
**Source Files:**
- `/Engine/Source/Runtime/CoreUObject/Public/UObject/UnrealType.h`
- `/Engine/Source/Runtime/CoreUObject/Public/UObject/Field.h`

### EPropertyFlags_System.md
**Source Files:**
- `/Engine/Source/Runtime/CoreUObject/Public/UObject/PropertyPortFlags.h`
- `/Engine/Source/Runtime/CoreUObject/Public/UObject/ObjectMacros.h`

### Property_Metadata_System.md
**Source Files:**
- `/Engine/Source/Runtime/CoreUObject/Public/UObject/MetaData.h`

### UPROPERTY_Macro_Generation.md
**Source Files:**
- `/Engine/Source/Programs/UnrealHeaderTool/` (UHT source files)

## Network & Replication

### Network_RPC_System.md
**Source Files:**
- `/Engine/Source/Runtime/Engine/Classes/Engine/NetDriver.h`
- `/Engine/Source/Runtime/Engine/Private/NetDriver.cpp`

### Replication_Property_System.md
**Source Files:**
- `/Engine/Source/Runtime/Engine/Classes/Engine/ReplicationDriver.h`
- `/Engine/Source/Runtime/Engine/Private/ReplicationDriver.cpp`

### RepNotify_Mechanism.md
**Source Files:**
- `/Engine/Source/Runtime/Engine/Private/RepLayout.cpp`

## Asset References

### Asset_Reference_Types.md
**Source Files:**
- `/Engine/Source/Runtime/CoreUObject/Public/UObject/SoftObjectPtr.h`
- `/Engine/Source/Runtime/CoreUObject/Public/UObject/WeakObjectPtr.h`

### ConstructorHelpers_Asset_Loading.md
**Source Files:**
- `/Engine/Source/Runtime/Engine/Classes/Engine/EngineBaseTypes.h` (ConstructorHelpers)

### TSubclassOf_Implementation.md
**Source Files:**
- `/Engine/Source/Runtime/CoreUObject/Public/Templates/SubclassOf.h`

## Timeline System

### Timeline_Component.md
**Source Files:**
- `/Engine/Source/Runtime/Engine/Classes/Components/TimelineComponent.h`
- `/Engine/Source/Runtime/Engine/Private/Components/TimelineComponent.cpp`

### Timeline_Template.md
**Source Files:**
- `/Engine/Source/Runtime/Engine/Classes/Engine/TimelineTemplate.h`
- `/Engine/Source/Runtime/Engine/Private/TimelineTemplate.cpp`

## Node Implementation Details

### Node_Implementation_CallArrayFunction.md
**Source Files:**
- `/Engine/Source/Editor/BlueprintGraph/Private/K2Node_CallArrayFunction.cpp`
- `/Engine/Source/Editor/BlueprintGraph/Classes/K2Node_CallArrayFunction.h`

### Node_Implementation_CallFunction.md
**Source Files:**
- `/Engine/Source/Editor/BlueprintGraph/Private/K2Node_CallFunction.cpp`
- `/Engine/Source/Editor/BlueprintGraph/Private/K2Node_CallFunctionOnMember.cpp`

### Node_Implementation_CommutativeAssociativeBinaryOperator.md
**Source Files:**
- `/Engine/Source/Editor/BlueprintGraph/Private/K2Node_CommutativeAssociativeBinaryOperator.cpp`

### Node_Implementation_Event.md
**Source Files:**
- `/Engine/Source/Editor/BlueprintGraph/Private/K2Node_Event.cpp`
- `/Engine/Source/Editor/BlueprintGraph/Private/K2Node_ActorBoundEvent.cpp`
- `/Engine/Source/Editor/BlueprintGraph/Private/K2Node_ComponentBoundEvent.cpp`

### Node_Implementation_GetArrayItem.md
**Source Files:**
- `/Engine/Source/Editor/BlueprintGraph/Private/K2Node_GetArrayItem.cpp`

### Node_Implementation_Select.md
**Source Files:**
- `/Engine/Source/Editor/BlueprintGraph/Private/K2Node_Select.cpp`

### Node_Implementation_Switch.md
**Source Files:**
- `/Engine/Source/Editor/BlueprintGraph/Private/K2Node_Switch.cpp`
- `/Engine/Source/Editor/BlueprintGraph/Private/K2Node_SwitchEnum.cpp`
- `/Engine/Source/Editor/BlueprintGraph/Private/K2Node_SwitchInteger.cpp`
- `/Engine/Source/Editor/BlueprintGraph/Private/K2Node_SwitchName.cpp`
- `/Engine/Source/Editor/BlueprintGraph/Private/K2Node_SwitchString.cpp`

### Node_Implementation_Variable.md
**Source Files:**
- `/Engine/Source/Editor/BlueprintGraph/Private/K2Node_Variable.cpp`
- `/Engine/Source/Editor/BlueprintGraph/Private/K2Node_LocalVariable.cpp`
- `/Engine/Source/Editor/BlueprintGraph/Private/K2Node_TemporaryVariable.cpp`

### Node_Implementation_AddPinInterface.md
**Source Files:**
- `/Engine/Source/Editor/BlueprintGraph/Private/K2Node_AddPinInterface.cpp`

## Special Nodes

### Special_Node_Complex_Execution.md
**Source Files:**
- `/Engine/Source/Editor/BlueprintGraph/Private/K2Node_MultiGate.cpp`
- `/Engine/Source/Editor/BlueprintGraph/Private/K2Node_ExecutionSequence.cpp`

### Special_Node_DynamicCast.md
**Source Files:**
- `/Engine/Source/Editor/BlueprintGraph/Private/K2Node_DynamicCast.cpp`

### Special_Node_Knot_and_Tunnel.md
**Source Files:**
- `/Engine/Source/Editor/BlueprintGraph/Private/K2Node_Knot.cpp`
- `/Engine/Source/Editor/BlueprintGraph/Private/K2Node_Tunnel.cpp`

### Special_Node_MacroInstance.md
**Source Files:**
- `/Engine/Source/Editor/BlueprintGraph/Private/K2Node_MacroInstance.cpp`

## Latent Actions

### Latent_Actions.md
**Source Files:**
- `/Engine/Source/Runtime/Engine/Classes/Engine/LatentActionManager.h`
- `/Engine/Source/Runtime/Engine/Private/LatentActionManager.cpp`
- `/Engine/Source/Editor/BlueprintGraph/Private/K2Node_AsyncAction.cpp`
- `/Engine/Source/Editor/BlueprintGraph/Private/K2Node_BaseAsyncTask.cpp`

## Blueprint Core

### UBlueprint_Class_Structure.md
**Source Files:**
- `/Engine/Source/Runtime/Engine/Classes/Engine/Blueprint.h`
- `/Engine/Source/Runtime/Engine/Private/Blueprint.cpp`

### Blueprint_Runtime_System_Summary.md
**Source Files:**
- `/Engine/Source/Runtime/Engine/Private/BlueprintGeneratedClass.cpp`
- `/Engine/Source/Runtime/Engine/Private/KismetBytecode.cpp`

## Schema & Validation

### Schema_K2_Core_Validation_System.md
**Source Files:**
- `/Engine/Source/Editor/BlueprintGraph/Classes/EdGraphSchema_K2.h`
- `/Engine/Source/Editor/BlueprintGraph/Private/EdGraphSchema_K2.cpp`

### Schema_K2_Actions_Node_Creation.md
**Source Files:**
- `/Engine/Source/Editor/BlueprintGraph/Classes/EdGraphSchema_K2_Actions.h`
- `/Engine/Source/Editor/BlueprintGraph/Private/EdGraphSchema_K2_Actions.cpp`

## Debug System

### Debug_System_Summary.md
**Source Files:**
- `/Engine/Source/Editor/Kismet/Private/BlueprintDebugger.h`
- `/Engine/Source/Editor/Kismet/Private/BlueprintDebugger.cpp`

### Debug_Breakpoint_System.md
**Source Files:**
- `/Engine/Source/Runtime/Engine/Classes/Engine/Breakpoint.h`

### Debug_Output_Nodes.md
**Source Files:**
- `/Engine/Source/Runtime/Engine/Classes/Kismet/KismetSystemLibrary.h` (PrintString)

### Debug_Visual_Logger.md
**Source Files:**
- `/Engine/Source/Runtime/Engine/Classes/Engine/EngineTypes.h` (Visual Logger)

### Debug_Compilation_Flags.md
**Source Files:**
- Compilation flags are spread across multiple build configuration files

### Debug_Instrumentation_Profiling.md
**Source Files:**
- `/Engine/Source/Runtime/Engine/Classes/Engine/Blueprint.h` (instrumentation)

## Default Values

### Default_Values_System_Summary.md
**Source Files:**
- `/Engine/Source/Runtime/CoreUObject/Public/UObject/UnrealType.h` (ExportText/ImportText)

### Default_Values_Primitive_Types.md
**Source Files:**
- `/Engine/Source/Runtime/CoreUObject/Private/UObject/PropertyBaseObject.cpp`

### Default_Values_Container_Types.md
**Source Files:**
- `/Engine/Source/Runtime/CoreUObject/Private/UObject/PropertyArray.cpp`
- `/Engine/Source/Runtime/CoreUObject/Private/UObject/PropertySet.cpp`
- `/Engine/Source/Runtime/CoreUObject/Private/UObject/PropertyMap.cpp`

### Default_Values_Complex_Types.md
**Source Files:**
- `/Engine/Source/Runtime/CoreUObject/Private/UObject/PropertyStruct.cpp`
- `/Engine/Source/Runtime/CoreUObject/Private/UObject/PropertyText.cpp`

### Default_Values_Wildcard_Resolution.md
**Source Files:**
- `/Engine/Source/Editor/BlueprintGraph/Private/BlueprintTypePromotion.cpp`

## Edge Cases

### Edge_Case_Circular_References.md
**Source Files:**
- Blueprint circular reference handling is integrated throughout the compilation system

### Edge_Case_Hot_Reload.md
**Source Files:**
- `/Engine/Source/Developer/HotReload/` (Hot reload system)

### Edge_Case_Level_Blueprints.md
**Source Files:**
- `/Engine/Source/Runtime/Engine/Classes/Engine/LevelScriptActor.h`
- `/Engine/Source/Runtime/Engine/Private/LevelScriptActor.cpp`

### Edge_Case_Orphaned_Pins.md
**Source Files:**
- Pin orphaning handling in `/Engine/Source/Editor/BlueprintGraph/Private/K2Node.cpp`

### Edge_Case_Package_Boundaries.md
**Source Files:**
- Package management throughout `/Engine/Source/Runtime/CoreUObject/`

### Edge_Case_World_Context.md
**Source Files:**
- `/Engine/Source/Runtime/Engine/Classes/Engine/World.h`
- `/Engine/Source/Runtime/Engine/Private/World.cpp`

## Validation & Testing

### Validation_Automation_Testing_Framework.md
**Source Files:**
- `/Engine/Source/Runtime/Core/Public/Misc/AutomationTest.h`
- `/Engine/Source/Developer/AutomationController/`

### Validation_Blueprint_Editor_Utilities.md
**Source Files:**
- `/Engine/Source/Editor/Kismet/Public/BlueprintEditorUtils.h`
- `/Engine/Source/Editor/Kismet/Private/BlueprintEditorUtils.cpp`

### Validation_Material_Parameter_Collection.md
**Source Files:**
- `/Engine/Source/Editor/BlueprintGraph/Private/K2Node_CallMaterialParameterCollectionFunction.cpp`

### Validation_Math_Expression_System.md
**Source Files:**
- `/Engine/Source/Editor/BlueprintGraph/Private/K2Node_MathExpression.cpp`

### Validation_Variable_Scoping_System.md
**Source Files:**
- Variable scoping integrated throughout K2Node implementations

## Testing Framework

### Testing_Framework_Overview.md
**Source Files:**
- `/Engine/Source/Developer/AutomationController/`
- `/Engine/Source/Runtime/Core/Public/Misc/AutomationTest.h`

### Testing_Compilation_Validation.md
**Source Files:**
- Blueprint compilation tests in `/Engine/Source/Editor/UnrealEd/Private/Tests/`

### Testing_Runtime_Equivalence.md
**Source Files:**
- Runtime testing framework integrated with automation system

### Testing_Generation_Patterns.md
**Source Files:**
- Test generation patterns in automation framework

### Testing_Validation_Checkpoints.md
**Source Files:**
- Validation checkpoints throughout compilation pipeline

### Testing_Blueprint_Catalog.md
**Source Files:**
- Test Blueprint examples in `/Engine/Content/` and test projects

## Notes
- All source paths are relative to `/mnt/c/Users/Jake_Bloch/Desktop/wave25/UnrealEngine-5.2.1-release`
- Some documentation files may reference multiple source files as systems are distributed
- Header (.h) and implementation (.cpp) files are typically paired
- Some systems like edge case handling are integrated throughout multiple files rather than isolated