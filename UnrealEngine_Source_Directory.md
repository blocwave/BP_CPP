# UnrealEngine 5.2.1 Blueprint System Source Files

## Compilation Pipeline Files

### Core Compiler
- `/Engine/Source/Editor/KismetCompiler/Private/KismetCompiler.cpp` - Main compilation orchestrator
- `/Engine/Source/Editor/KismetCompiler/Private/KismetCompilerVMBackend.cpp` - Bytecode generation backend
- `/Engine/Source/Editor/KismetCompiler/Private/KismetCompilerMisc.cpp` - Compilation utilities
- `/Engine/Source/Editor/KismetCompiler/Public/KismetCompiler.h` - Compiler interface

### Runtime Classes
- `/Engine/Source/Runtime/Engine/Private/BlueprintGeneratedClass.cpp` - Runtime class generation
- `/Engine/Source/Runtime/Engine/Private/Blueprint.cpp` - Blueprint asset structure
- `/Engine/Source/Runtime/Engine/Private/SimpleConstructionScript.cpp` - Component hierarchy
- `/Engine/Source/Runtime/Engine/Private/SCS_Node.cpp` - Component node structure

## Node Implementations

### Core K2Node Files
- `/Engine/Source/Editor/BlueprintGraph/Private/K2Node_CallFunction.cpp` - Function call implementation
- `/Engine/Source/Editor/BlueprintGraph/Private/K2Node_Event.cpp` - Event entry points  
- `/Engine/Source/Editor/BlueprintGraph/Private/K2Node_Variable.cpp` - Variable access
- `/Engine/Source/Editor/BlueprintGraph/Private/K2Node_VariableGet.cpp` - Variable getter
- `/Engine/Source/Editor/BlueprintGraph/Private/K2Node_VariableSet.cpp` - Variable setter
- `/Engine/Source/Editor/BlueprintGraph/Private/K2Node_IfThenElse.cpp` - Conditional branching
- `/Engine/Source/Editor/BlueprintGraph/Private/K2Node_ForEachElementInEnum.cpp` - Enum iteration
- `/Engine/Source/Editor/BlueprintGraph/Private/K2Node_BreakStruct.cpp` - Structure decomposition
- `/Engine/Source/Editor/BlueprintGraph/Private/K2Node_MakeStruct.cpp` - Structure construction
- `/Engine/Source/Editor/BlueprintGraph/Private/K2Node_Timeline.cpp` - Timeline animation
- `/Engine/Source/Editor/BlueprintGraph/Private/K2Node_SpawnActorFromClass.cpp` - Actor spawning
- `/Engine/Source/Editor/BlueprintGraph/Private/K2Node_DynamicCast.cpp` - Type casting
- `/Engine/Source/Editor/BlueprintGraph/Private/K2Node_ClassDynamicCast.cpp` - Class casting
- `/Engine/Source/Editor/BlueprintGraph/Private/K2Node_MacroInstance.cpp` - Macro expansion
- `/Engine/Source/Editor/BlueprintGraph/Private/K2Node_CustomEvent.cpp` - User-defined events
- `/Engine/Source/Editor/BlueprintGraph/Private/K2Node_FunctionEntry.cpp` - Function entry points
- `/Engine/Source/Editor/BlueprintGraph/Private/K2Node_Composite.cpp` - Graph encapsulation

### Event Node Types
- `/Engine/Source/Editor/BlueprintGraph/Private/K2Node_ActorBoundEvent.cpp` - Actor events
- `/Engine/Source/Editor/BlueprintGraph/Private/K2Node_ComponentBoundEvent.cpp` - Component events
- `/Engine/Source/Editor/BlueprintGraph/Private/K2Node_InputActionEvent.cpp` - Input actions
- `/Engine/Source/Editor/BlueprintGraph/Private/K2Node_InputAxisEvent.cpp` - Input axis
- `/Engine/Source/Editor/BlueprintGraph/Private/K2Node_InputKeyEvent.cpp` - Key input
- `/Engine/Source/Editor/BlueprintGraph/Private/K2Node_InputTouchEvent.cpp` - Touch input
- `/Engine/Source/Editor/BlueprintGraph/Private/K2Node_InputVectorAxisEvent.cpp` - Vector input

### Special Function Nodes
- `/Engine/Source/Editor/BlueprintGraph/Private/K2Node_CallArrayFunction.cpp` - Array operations
- `/Engine/Source/Editor/BlueprintGraph/Private/K2Node_CallDataTableFunction.cpp` - Data table access
- `/Engine/Source/Editor/BlueprintGraph/Private/K2Node_CallFunctionOnMember.cpp` - Member function calls
- `/Engine/Source/Editor/BlueprintGraph/Private/K2Node_CallMaterialParameterCollectionFunction.cpp` - Material parameters
- `/Engine/Source/Editor/BlueprintGraph/Private/K2Node_AsyncAction.cpp` - Async operations
- `/Engine/Source/Editor/BlueprintGraph/Private/K2Node_BaseAsyncTask.cpp` - Base async task

### Variable Management
- `/Engine/Source/Editor/BlueprintGraph/Private/K2Node_LocalVariable.cpp` - Local variables
- `/Engine/Source/Editor/BlueprintGraph/Private/K2Node_TemporaryVariable.cpp` - Temporary variables
- `/Engine/Source/Editor/BlueprintGraph/Private/K2Node_MakeVariable.cpp` - Variable creation
- `/Engine/Source/Editor/BlueprintGraph/Private/K2Node_SetVariableOnPersistentFrame.cpp` - Persistent frame variables

### Additional Nodes
- `/Engine/Source/Editor/BlueprintGraph/Private/K2Node_AddComponent.cpp` - Component addition
- `/Engine/Source/Editor/BlueprintGraph/Private/K2Node_AddComponentByClass.cpp` - Dynamic component
- `/Engine/Source/Editor/BlueprintGraph/Private/K2Node_AddPinInterface.cpp` - Dynamic pin interface
- `/Engine/Source/Editor/BlueprintGraph/Private/K2Node_AssignDelegate.cpp` - Delegate assignment
- `/Engine/Source/Editor/BlueprintGraph/Private/K2Node_AssignmentStatement.cpp` - Assignment operations
- `/Engine/Source/Editor/BlueprintGraph/Private/K2Node_BitmaskLiteral.cpp` - Bitmask literals
- `/Engine/Source/Editor/BlueprintGraph/Private/K2Node_SpawnActor.cpp` - Actor spawning

## Graph Structure Files

### Core Graph System
- `/Engine/Source/Runtime/Engine/Classes/EdGraph/EdGraphNode.h` - Base node class
- `/Engine/Source/Runtime/Engine/Private/EdGraph/EdGraphNode.cpp` - Node implementation
- `/Engine/Source/Runtime/Engine/Classes/EdGraph/EdGraphPin.h` - Pin system header
- `/Engine/Source/Runtime/Engine/Private/EdGraph/EdGraphPin.cpp` - Pin implementation
- `/Engine/Source/Runtime/Engine/Classes/EdGraph/EdGraphSchema.h` - Schema validation
- `/Engine/Source/Runtime/Engine/Classes/Engine/MemberReference.h` - Member references
- `/Engine/Source/Editor/BlueprintGraph/Classes/K2Node.h` - Blueprint node base

### Execution Flow Files
- `/Engine/Source/Editor/KismetCompiler/Public/KismetCompilerModule.h` - Compiler interface
- `/Engine/Source/Editor/KismetCompiler/Private/FKismetFunctionContext.h` - Function context
- `/Engine/Source/Editor/KismetCompiler/Public/BlueprintCompiledStatement.h` - Statement representation
- `/Engine/Source/Editor/KismetCompiler/Private/FBPTerminal.h` - Terminal/variable storage

## Delegate & Interface System
- `/Engine/Source/Runtime/Core/Public/Delegates/Delegate.h` - Core delegate system
- `/Engine/Source/Runtime/Core/Public/Delegates/MulticastDelegateBase.h` - Multicast delegates
- `/Engine/Source/Runtime/Core/Public/UObject/ScriptDelegates.h` - Script delegates
- `/Engine/Source/Runtime/CoreUObject/Public/UObject/Interface.h` - Interface base

## Property System Files
- `/Engine/Source/Runtime/CoreUObject/Public/UObject/UnrealType.h` - Property types
- `/Engine/Source/Runtime/CoreUObject/Public/UObject/PropertyPortFlags.h` - Property flags
- `/Engine/Source/Runtime/Engine/Classes/Engine/BlueprintGeneratedClass.h` - Generated class header

## Animation System Integration
- `/Engine/Source/Editor/AnimGraph/Private/K2Node_AnimGetter.cpp` - Animation getters
- `/Engine/Source/Editor/AnimGraph/Private/K2Node_AnimNodeReference.cpp` - Anim node references
- `/Engine/Source/Editor/AnimGraph/Private/K2Node_PlayMontage.cpp` - Montage playback
- `/Engine/Source/Editor/AnimGraph/Private/K2Node_TransitionRuleGetter.cpp` - Transition rules

## AI System Integration
- `/Engine/Source/Editor/AIGraph/Private/K2Node_AIMoveTo.cpp` - AI movement nodes

## Directory Structure Summary

```
UnrealEngine-5.2.1-release/Engine/Source/
├── Editor/
│   ├── KismetCompiler/
│   │   ├── Private/
│   │   │   ├── KismetCompiler.cpp
│   │   │   ├── KismetCompilerVMBackend.cpp
│   │   │   ├── KismetCompilerMisc.cpp
│   │   │   └── FKismetFunctionContext.h
│   │   └── Public/
│   │       ├── KismetCompiler.h
│   │       ├── KismetCompilerModule.h
│   │       └── BlueprintCompiledStatement.h
│   ├── BlueprintGraph/
│   │   ├── Private/
│   │   │   └── [K2Node_*.cpp files - 50+ implementations]
│   │   └── Classes/
│   │       └── K2Node.h
│   ├── AnimGraph/
│   │   └── Private/
│   │       └── [Animation K2Node files]
│   └── AIGraph/
│       └── Private/
│           └── [AI K2Node files]
└── Runtime/
    ├── Engine/
    │   ├── Private/
    │   │   ├── Blueprint.cpp
    │   │   ├── BlueprintGeneratedClass.cpp
    │   │   ├── SimpleConstructionScript.cpp
    │   │   └── EdGraph/
    │   │       ├── EdGraphNode.cpp
    │   │       └── EdGraphPin.cpp
    │   └── Classes/
    │       ├── Engine/
    │       │   ├── Blueprint.h
    │       │   ├── BlueprintGeneratedClass.h
    │       │   └── MemberReference.h
    │       └── EdGraph/
    │           ├── EdGraphNode.h
    │           ├── EdGraphPin.h
    │           └── EdGraphSchema.h
    ├── Core/
    │   └── Public/
    │       └── Delegates/
    │           ├── Delegate.h
    │           └── MulticastDelegateBase.h
    └── CoreUObject/
        └── Public/
            └── UObject/
                ├── UnrealType.h
                ├── PropertyPortFlags.h
                ├── ScriptDelegates.h
                └── Interface.h
```

## Notes
- All paths are relative to `/mnt/c/Users/Jake_Bloch/Desktop/wave25/UnrealEngine-5.2.1-release`
- The BlueprintGraph folder contains 50+ K2Node implementations
- Header (.h) and implementation (.cpp) files are typically paired
- The KismetCompiler handles the Blueprint to bytecode/C++ conversion
- K2Node classes represent individual Blueprint node types