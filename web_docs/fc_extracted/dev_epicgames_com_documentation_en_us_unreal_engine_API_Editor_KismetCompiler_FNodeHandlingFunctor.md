Unreal Engine
5.2

- [Unreal Engine\\
5.6](https://dev.epicgames.com/documentation/en-us/unreal-engine/API?application_version=5.6)
- [Unreal Engine\\
5.5](https://dev.epicgames.com/documentation/en-us/unreal-engine/API?application_version=5.5)
- [Unreal Engine\\
5.4](https://dev.epicgames.com/documentation/en-us/unreal-engine/API?application_version=5.4)
- [Unreal Engine\\
5.3](https://dev.epicgames.com/documentation/en-us/unreal-engine/API?application_version=5.3)
- [Unreal Engine\\
5.2](https://dev.epicgames.com/documentation/en-us/unreal-engine/API?application_version=5.2)
- [Unreal Engine\\
5.1](https://dev.epicgames.com/documentation/en-us/unreal-engine/API?application_version=5.1)
- [Unreal Engine\\
5.0](https://dev.epicgames.com/documentation/en-us/unreal-engine/API?application_version=5.0)
- [Unreal Engine\\
4.27](https://dev.epicgames.com/documentation/en-us/unreal-engine/API?application_version=4.27)

Table of Contents

## Navigation

[API](https://dev.epicgames.com/documentation/en-us/unreal-engine/API?application_version=5.2) \> [API/Editor](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Editor?application_version=5.2) \> [API/Editor/KismetCompiler](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Editor/KismetCompiler?application_version=5.2)

## Inheritance Hierarchy

- FNodeHandlingFunctor
- [FKCHandler\_CallFunction](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Editor/BlueprintGraph/FKCHandler_CallFunction?application_version=5.2)
- [FKCHandler\_EventEntry](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Editor/BlueprintGraph/FKCHandler_EventEntry?application_version=5.2)
- [FKCHandler\_MakeContainer](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Editor/BlueprintGraph/FKCHandler_MakeContainer?application_version=5.2)
- [FKCHandler\_MathExpression](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Editor/BlueprintGraph/FKCHandler_MathExpression?application_version=5.2)
- [FKCHandler\_Passthru](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Editor/KismetCompiler/FKCHandler_Passthru?application_version=5.2)
- [FKCHandler\_VariableSet](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Editor/BlueprintGraph/FKCHandler_VariableSet?application_version=5.2)

## References

|  |  |
| --- | --- |
| Module | [KismetCompiler](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Editor/KismetCompiler?application_version=5.2) |
| Header | /Engine/Source/Editor/KismetCompiler/Public/KismetCompilerMisc.h |
| Include | #include "KismetCompilerMisc.h" |

## Syntax

```cpp
class FNodeHandlingFunctor
Copy full snippet
```

## Variables

|  | Type | Name | Description |
| --- | --- | --- | --- |
| ![Public variable](https://d1iv7db44yhgxn.cloudfront.net/documentation/images/2750215b-7732-452c-b08d-a9ae36e14fa9/api_variable_public.png) | [FKismetCompilerContext](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Editor/KismetCompiler/FKismetCompilerContext?application_version=5.2) & | CompilerContext |  |

## Constructors

|  | Type | Name | Description |
| --- | --- | --- | --- |
| ![Public function](https://d1iv7db44yhgxn.cloudfront.net/documentation/images/93ef6df6-8f1d-4951-b951-9dbfd0d34764/api_function_public.png) |  | [FNodeHandlingFunctor](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Editor/KismetCompiler/FNodeHandlingFunctor/__ctor?application_version=5.2)<br>(<br>[FKismetCompilerContext](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Editor/KismetCompiler/FKismetCompilerContext?application_version=5.2)& InCompilerContext<br>) |  |

## Destructors

|  | Type | Name | Description |
| --- | --- | --- | --- |
| ![Public function](https://d1iv7db44yhgxn.cloudfront.net/documentation/images/b0fb84ce-fb7a-4a35-ad9e-dd798a42cd7b/api_function_public.png)![Virtual](https://d1iv7db44yhgxn.cloudfront.net/documentation/images/f172080b-8d4d-4bab-b8c7-13cfab11f1c1/api_function_virtual.png) |  | [~FNodeHandlingFunctor](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Editor/KismetCompiler/FNodeHandlingFunctor/__dtor?application_version=5.2) () |  |

## Functions

|  | Type | Name | Description |
| --- | --- | --- | --- |
| ![Public function](https://d1iv7db44yhgxn.cloudfront.net/documentation/images/17700481-0b2e-409f-90f0-0a67eee06552/api_function_public.png)![Virtual](https://d1iv7db44yhgxn.cloudfront.net/documentation/images/d7d0937a-1a96-4d51-9229-a7983409ad93/api_function_virtual.png) | void | [Compile](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Editor/KismetCompiler/FNodeHandlingFunctor/Compile?application_version=5.2)<br>(<br>[FKismetFunctionContext](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Editor/KismetCompiler/FKismetFunctionContext?application_version=5.2)& Context,<br>[UEdGraphNode](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Runtime/Engine/EdGraph/UEdGraphNode?application_version=5.2)\\* Node<br>) |  |
| ![Protected function](https://d1iv7db44yhgxn.cloudfront.net/documentation/images/39922a9c-3480-4e2f-be0b-ce46fd434b77/api_function_protected.png) | [FBlueprintCompiledStatement](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Editor/KismetCompiler/FBlueprintCompiledStatement?application_version=5.2) & | [GenerateSimpleThenGoto](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Editor/KismetCompiler/FNodeHandlingFunctor/GenerateSimpleThenGoto/1?application_version=5.2)<br>(<br>[FKismetFunctionContext](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Editor/KismetCompiler/FKismetFunctionContext?application_version=5.2)& Context,<br>[UEdGraphNode](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Runtime/Engine/EdGraph/UEdGraphNode?application_version=5.2)& Node<br>) | Generate a goto corresponding to the then pin(s) |
| ![Protected function](https://d1iv7db44yhgxn.cloudfront.net/documentation/images/5cfd0f3e-be76-44ea-82a9-5e002ce25bf5/api_function_protected.png) | [FBlueprintCompiledStatement](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Editor/KismetCompiler/FBlueprintCompiledStatement?application_version=5.2) & | [GenerateSimpleThenGoto](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Editor/KismetCompiler/FNodeHandlingFunctor/GenerateSimpleThenGoto/2?application_version=5.2)<br>(<br>[FKismetFunctionContext](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Editor/KismetCompiler/FKismetFunctionContext?application_version=5.2)& Context,<br>[UEdGraphNode](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Runtime/Engine/EdGraph/UEdGraphNode?application_version=5.2)& Node,<br>[UEdGraphPin](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Runtime/Engine/EdGraph/UEdGraphPin?application_version=5.2)\\* ThenExecPin<br>) | Generate a goto on the corresponding exec pin. |
| ![Protected function](https://d1iv7db44yhgxn.cloudfront.net/documentation/images/ba22009a-99a0-410a-a043-c9743f6858bb/api_function_protected.png)![Virtual](https://d1iv7db44yhgxn.cloudfront.net/documentation/images/8711dbac-9f9c-43fc-851f-d2f155f31fae/api_function_virtual.png) | [FBPTerminal](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Editor/KismetCompiler/FBPTerminal?application_version=5.2) \* | [RegisterLiteral](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Editor/KismetCompiler/FNodeHandlingFunctor/RegisterLiteral?application_version=5.2)<br>(<br>[FKismetFunctionContext](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Editor/KismetCompiler/FKismetFunctionContext?application_version=5.2)& Context,<br>[UEdGraphPin](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Runtime/Engine/EdGraph/UEdGraphPin?application_version=5.2)\\* Net<br>) | Helper to register literal term. |
| ![Public function](https://d1iv7db44yhgxn.cloudfront.net/documentation/images/4e879cfb-7f30-48ed-a8a9-ddfddcc35694/api_function_public.png)![Virtual](https://d1iv7db44yhgxn.cloudfront.net/documentation/images/c5cd04d8-9e40-4ff4-a3f6-09f784c45eb4/api_function_virtual.png) | void | [RegisterNet](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Editor/KismetCompiler/FNodeHandlingFunctor/RegisterNet?application_version=5.2)<br>(<br>[FKismetFunctionContext](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Editor/KismetCompiler/FKismetFunctionContext?application_version=5.2)& Context,<br>[UEdGraphPin](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Runtime/Engine/EdGraph/UEdGraphPin?application_version=5.2)\\* Pin<br>) |  |
| ![Public function](https://d1iv7db44yhgxn.cloudfront.net/documentation/images/5395fb54-6e11-4fa6-94b6-1a157b4572ff/api_function_public.png)![Virtual](https://d1iv7db44yhgxn.cloudfront.net/documentation/images/1e6964b9-cf31-4b30-ac5d-dcc8a774de81/api_function_virtual.png) | void | [RegisterNets](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Editor/KismetCompiler/FNodeHandlingFunctor/RegisterNets?application_version=5.2)<br>(<br>[FKismetFunctionContext](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Editor/KismetCompiler/FKismetFunctionContext?application_version=5.2)& Context,<br>[UEdGraphNode](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Runtime/Engine/EdGraph/UEdGraphNode?application_version=5.2)\\* Node<br>) |  |
| ![Public function](https://d1iv7db44yhgxn.cloudfront.net/documentation/images/fc6af6e3-32d6-4658-a1ae-5addf14e8533/api_function_public.png)![Virtual](https://d1iv7db44yhgxn.cloudfront.net/documentation/images/ab3f11d6-6c90-4db8-b4ef-b9d331a69a32/api_function_virtual.png)![Const](https://d1iv7db44yhgxn.cloudfront.net/documentation/images/db40bd4f-4817-4939-aca2-9a539e364ab8/api_function_const.png) | bool | [RequiresRegisterNetsBeforeScheduling](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Editor/KismetCompiler/FNodeHandlingFunctor/RequiresRegister-?application_version=5.2) () | Returns true if this kind of node needs to be handled in the first pass, prior to execution scheduling (this is only used for function entry and return nodes) |
| ![Protected function](https://d1iv7db44yhgxn.cloudfront.net/documentation/images/9b56895b-9b14-4f02-8994-f45e938d274f/api_function_protected.png) | void | [ResolveAndRegisterScopedTerm](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Editor/KismetCompiler/FNodeHandlingFunctor/ResolveAndRegisterScopedTerm?application_version=5.2)<br>(<br>[FKismetFunctionContext](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Editor/KismetCompiler/FKismetFunctionContext?application_version=5.2)& Context,<br>[UEdGraphPin](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Runtime/Engine/EdGraph/UEdGraphPin?application_version=5.2)\\* Net,<br>[TIndirectArray](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Runtime/Core/Containers/TIndirectArray?application_version=5.2) < [FBPTerminal](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Editor/KismetCompiler/FBPTerminal?application_version=5.2) >& NetArray<br>) | Helper function that verifies the variable name referenced by the net exists in the associated scope (either the class being compiled or via an object reference on the Self pin), and then creates/registers a term for that variable access. |
| ![Public function](https://d1iv7db44yhgxn.cloudfront.net/documentation/images/cad01918-d421-4b0b-91c6-4c901866e433/api_function_public.png)![Static](https://d1iv7db44yhgxn.cloudfront.net/documentation/images/10730faa-7261-46e8-8d63-fa1320bd2077/api_function_static.png) | void | [SanitizeName](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Editor/KismetCompiler/FNodeHandlingFunctor/SanitizeName?application_version=5.2)<br>(<br>[FString](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Runtime/Core/Containers/FString?application_version=5.2)& Name<br>) | Creates a sanitized name. |
| ![Public function](https://d1iv7db44yhgxn.cloudfront.net/documentation/images/e7810438-4435-4261-813d-9ca62d2e37cb/api_function_public.png)![Virtual](https://d1iv7db44yhgxn.cloudfront.net/documentation/images/688b3df7-6bb6-49b5-b141-e191a852e35b/api_function_virtual.png) | void | [Transform](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Editor/KismetCompiler/FNodeHandlingFunctor/Transform?application_version=5.2)<br>(<br>[FKismetFunctionContext](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Editor/KismetCompiler/FKismetFunctionContext?application_version=5.2)& Context,<br>[UEdGraphNode](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Runtime/Engine/EdGraph/UEdGraphNode?application_version=5.2)\\* Node<br>) |  |
| ![Protected function](https://d1iv7db44yhgxn.cloudfront.net/documentation/images/7c7dd73e-ca49-48e3-b070-9202289af49c/api_function_protected.png) | bool | [ValidateAndRegisterNetIfLiteral](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Editor/KismetCompiler/FNodeHandlingFunctor/ValidateAndRegisterNetIfLiteral?application_version=5.2)<br>(<br>[FKismetFunctionContext](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Editor/KismetCompiler/FKismetFunctionContext?application_version=5.2)& Context,<br>[UEdGraphPin](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Runtime/Engine/EdGraph/UEdGraphPin?application_version=5.2)\\* Net<br>) | If the net is a literal, it validates the default value and registers it. |

* * *

Ask questions and help your peers [Developer Forums](https://forums.unrealengine.com/categories?tag=unreal-engine)

Write your own tutorials or read those from others [Learning Library](https://dev.epicgames.com/community/unreal-engine/learning)

- [Navigation](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Editor/KismetCompiler/FNodeHandlingFunctor?application_version=5.2#navigation)
- [Inheritance Hierarchy](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Editor/KismetCompiler/FNodeHandlingFunctor?application_version=5.2#inheritancehierarchy)
- [References](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Editor/KismetCompiler/FNodeHandlingFunctor?application_version=5.2#references)
- [Syntax](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Editor/KismetCompiler/FNodeHandlingFunctor?application_version=5.2#syntax)
- [Variables](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Editor/KismetCompiler/FNodeHandlingFunctor?application_version=5.2#variables)
- [Constructors](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Editor/KismetCompiler/FNodeHandlingFunctor?application_version=5.2#constructors)
- [Destructors](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Editor/KismetCompiler/FNodeHandlingFunctor?application_version=5.2#destructors)
- [Functions](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Editor/KismetCompiler/FNodeHandlingFunctor?application_version=5.2#functions)