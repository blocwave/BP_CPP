# K2Node Examples and FAQ

## Overview

This guide provides practical examples and answers to frequently asked questions about K2Nodes in Unreal Engine, based on common developer issues and community discussions. K2Nodes are powerful Blueprint node classes that can wrap advanced and dynamic functionality beyond simple UFUNCTIONs.

## Table of Contents

- [Getting Started](#getting-started)
- [Common K2Node Types](#common-k2node-types)
- [Frequently Asked Questions](#frequently-asked-questions)
- [Practical Examples](#practical-examples)
- [Troubleshooting](#troubleshooting)
- [Best Practices](#best-practices)

## Getting Started

### What is a K2Node?

A K2Node is a specialized Blueprint node that provides more advanced functionality than standard UFUNCTION-based nodes. They can:
- Create dynamic pin configurations
- Handle complex execution flow
- Generate custom compilation behavior
- Provide specialized visual representation

### Basic K2Node Structure

```cpp
UCLASS()
class MYMODULE_API UK2Node_MyCustomNode : public UK2Node
{
    GENERATED_BODY()

public:
    // Core overrides
    virtual void AllocateDefaultPins() override;
    virtual FText GetNodeTitle(ENodeTitleType::Type TitleType) const override;
    virtual void ExpandNode(class FKismetCompilerContext& CompilerContext, UEdGraph* SourceGraph) override;
    virtual FText GetTooltipText() const override;
    virtual FLinearColor GetNodeTitleColor() const override;
    virtual FSlateIcon GetIconAndTint(FLinearColor& OutColor) const override;
    virtual void GetMenuActions(FBlueprintActionDatabaseRegistrar& ActionRegistrar) const override;
    virtual FText GetMenuCategory() const override;
    virtual bool IsNodeSafeToIgnore() const override { return true; }
};
```

## Common K2Node Types

### UK2Node_CallFunction
- **Purpose**: Function call nodes
- **Use Case**: Calling UFUNCTIONs from C++ or Blueprint
- **Key Features**: Parameter pins, return value pins, execution pins

### UK2Node_Event  
- **Purpose**: Event nodes in Blueprint graphs
- **Use Case**: Handling gameplay events, input events
- **Key Features**: Execution output pins only

### UK2Node_Variable
- **Purpose**: Variable get/set nodes  
- **Use Case**: Reading and writing Blueprint variables
- **Key Features**: Data pins, optional execution pins for setters

### UK2Node_CustomEvent
- **Purpose**: User-defined custom events
- **Use Case**: Blueprint communication and organization
- **Key Features**: Custom parameters, execution pins

## Frequently Asked Questions

### Q: My custom K2Node doesn't show up in the Blueprint editor. What's wrong?

**A:** This is one of the most common issues. Check these requirements:

1. **Module Registration**: Ensure your module is properly loaded
2. **GetMenuActions Implementation**: This is crucial for node discovery
3. **Blueprint Action Database**: Register your node correctly

```cpp
void UK2Node_MyCustomNode::GetMenuActions(FBlueprintActionDatabaseRegistrar& ActionRegistrar) const
{
    UClass* ActionKey = GetClass();
    if (ActionRegistrar.IsOpenForRegistration(ActionKey))
    {
        UBlueprintNodeSpawner* NodeSpawner = UBlueprintNodeSpawner::Create(GetClass());
        check(NodeSpawner != nullptr);
        ActionRegistrar.AddBlueprintAction(ActionKey, NodeSpawner);
    }
}
```

4. **Menu Category**: Implement GetMenuCategory()
```cpp
FText UK2Node_MyCustomNode::GetMenuCategory() const
{
    return FText::FromString("My Custom Nodes");
}
```

### Q: How do I create a K2Node that returns a constant value?

**A:** For nodes that return constant values without execution:

```cpp
void UK2Node_MyConstantNode::AllocateDefaultPins()
{
    Super::AllocateDefaultPins();
    
    // Create output pin for the constant value
    CreatePin(EGPD_Output, UEdGraphSchema_K2::PC_Int, TEXT("Value"));
}

void UK2Node_MyConstantNode::ExpandNode(class FKismetCompilerContext& CompilerContext, UEdGraph* SourceGraph)
{
    Super::ExpandNode(CompilerContext, SourceGraph);
    
    // Get the output pin
    UEdGraphPin* ValuePin = FindPinChecked(TEXT("Value"), EGPD_Output);
    
    if (ValuePin->LinkedTo.Num() > 0)
    {
        // Create a literal term for the constant
        FKismetCompiledStatement& Statement = CompilerContext.AppendStatementForNode(this);
        Statement.Type = KCST_Assignment;
        Statement.LHS = CompilerContext.GetNetForPin(ValuePin);
        Statement.RHS.Add(FBlueprintCompiledStatement::StatementTerm_Literal(FString::FromInt(42))); // Your constant value
    }
    
    // Remove this node from the graph
    BreakAllNodeLinks();
}

bool UK2Node_MyConstantNode::IsNodePure() const
{
    return true; // Pure nodes don't have execution pins
}
```

### Q: Can I call a function on a constructed object inside a K2Node?

**A:** Yes, absolutely! You can spawn UK2Node_CallFunction nodes during expansion:

```cpp
void UK2Node_MyComplexNode::ExpandNode(class FKismetCompilerContext& CompilerContext, UEdGraph* SourceGraph)
{
    Super::ExpandNode(CompilerContext, SourceGraph);
    
    // Find the function you want to call
    UFunction* TargetFunction = FindField<UFunction>(UMyClass::StaticClass(), TEXT("MyMemberFunction"));
    if (!TargetFunction)
    {
        CompilerContext.MessageLog.Error(*LOCTEXT("MissingFunction", "Could not find target function").ToString(), this);
        return;
    }
    
    // Create a function call node
    UK2Node_CallFunction* CallFuncNode = CompilerContext.SpawnIntermediateNode<UK2Node_CallFunction>(this, SourceGraph);
    CallFuncNode->SetFromFunction(TargetFunction);
    CallFuncNode->AllocateDefaultPins();
    
    // Get pins from our node
    UEdGraphPin* OurExecPin = GetExecPin();
    UEdGraphPin* OurObjectPin = FindPinChecked(TEXT("Object"), EGPD_Input);
    
    // Get pins from the function call node
    UEdGraphPin* FuncExecPin = CallFuncNode->GetExecPin();
    UEdGraphPin* FuncSelfPin = CallFuncNode->FindPinChecked(TEXT("self"), EGPD_Input);
    
    // Connect the pins
    if (OurExecPin && FuncExecPin)
    {
        CompilerContext.GetSchema()->TryCreateConnection(OurExecPin, FuncExecPin);
    }
    
    if (OurObjectPin && FuncSelfPin)
    {
        CompilerContext.GetSchema()->TryCreateConnection(OurObjectPin, FuncSelfPin);
    }
    
    BreakAllNodeLinks();
}
```

### Q: How do I create K2Nodes that auto-execute without manual pin connections?

**A:** For auto-executing nodes, you typically want to:

1. **Make the node impure** (has execution pins)
2. **Use appropriate scheduling** during compilation
3. **Consider construction script placement** for initialization

```cpp
void UK2Node_AutoExecuteNode::AllocateDefaultPins()
{
    Super::AllocateDefaultPins();
    
    // For auto-execute nodes, you might want to minimize required connections
    // Create only essential pins
    CreatePin(EGPD_Input, UEdGraphSchema_K2::PC_Exec, UEdGraphSchema_K2::PN_Execute);
    CreatePin(EGPD_Output, UEdGraphSchema_K2::PC_Exec, UEdGraphSchema_K2::PN_Then);
    
    // Your custom logic pins
    CreatePin(EGPD_Input, UEdGraphSchema_K2::PC_Boolean, TEXT("AutoStart"));
}

// Override to provide default values
void UK2Node_AutoExecuteNode::PostReconstructNode()
{
    Super::PostReconstructNode();
    
    // Set default value for auto-start
    UEdGraphPin* AutoStartPin = FindPinChecked(TEXT("AutoStart"), EGPD_Input);
    if (AutoStartPin)
    {
        AutoStartPin->DefaultValue = TEXT("true");
    }
}
```

### Q: How do I create contextual K2Nodes that appear only for specific components?

**A:** Use conditional registration in GetMenuActions():

```cpp
void UK2Node_ComponentSpecificNode::GetMenuActions(FBlueprintActionDatabaseRegistrar& ActionRegistrar) const
{
    UClass* ActionKey = GetClass();
    if (ActionRegistrar.IsOpenForRegistration(ActionKey))
    {
        // Create a spawner with a custom condition
        UBlueprintNodeSpawner* NodeSpawner = UBlueprintNodeSpawner::Create(GetClass());
        
        // Add condition to check for specific component
        NodeSpawner->DefaultMenuSignature.MenuName = FText::FromString("Component Specific Action");
        
        // Custom filtering lambda
        auto IsContextuallyRelevant = [](FBlueprintActionContext const& Context) -> bool
        {
            if (Context.Blueprints.Num() == 1)
            {
                UBlueprint* Blueprint = Context.Blueprints[0];
                if (Blueprint && Blueprint->SimpleConstructionScript)
                {
                    // Check if blueprint has the required component
                    TArray<USCS_Node*> AllNodes = Blueprint->SimpleConstructionScript->GetAllNodes();
                    for (USCS_Node* Node : AllNodes)
                    {
                        if (Node && Node->ComponentClass && Node->ComponentClass->IsChildOf(UMySpecificComponent::StaticClass()))
                        {
                            return true; // Show the node
                        }
                    }
                }
            }
            return false; // Hide the node
        };
        
        NodeSpawner->DynamicUiSignatureGetter = UBlueprintNodeSpawner::FUiSpecOverrideDelegate::CreateLambda(
            [](const FBlueprintActionContext& Context, const IBlueprintNodeBinder::FBindingSet& Bindings) -> FBlueprintActionUiSpec
            {
                FBlueprintActionUiSpec UiSpec;
                UiSpec.MenuName = FText::FromString("Use Specific Component");
                UiSpec.Category = FText::FromString("Component Actions");
                return UiSpec;
            });
            
        ActionRegistrar.AddBlueprintAction(ActionKey, NodeSpawner);
    }
}
```

### Q: How do I find the corresponding C++ API of a Blueprint node?

**A:** Several methods to identify the underlying implementation:

1. **Copy-Paste Method**: Copy the Blueprint node and paste into a text editor
```
Begin Object Class=/Script/BlueprintGraph.K2Node_CallFunction Name="K2Node_CallFunction_0"
   FunctionReference=(MemberName="SetActorLocation",MemberGuid=F5AFEC4F-4CB8-B0AC-8C2E-21ACB3F6CAD4,bSelfContext=True)
   NodePosX=256
   NodePosY=144
End Object
```

2. **Source Code Search**: Look in Engine/Source for the function name
3. **API Documentation**: Use the official Unreal Engine API reference
4. **Blueprint Debugging**: Use Blueprint debugger to see the actual function calls

### Q: How do I create "raw" UFUNCTIONs with custom behavior?

**A:** Use UK2Node_CallFunction with custom expansion:

```cpp
// Create a K2Node that calls a custom generated function
void UK2Node_CustomRawFunction::ExpandNode(class FKismetCompilerContext& CompilerContext, UEdGraph* SourceGraph)
{
    Super::ExpandNode(CompilerContext, SourceGraph);
    
    // Create intermediate nodes that represent your custom logic
    UK2Node_CallFunction* InternalCall = CompilerContext.SpawnIntermediateNode<UK2Node_CallFunction>(this, SourceGraph);
    
    // Set up the function call with your custom behavior
    InternalCall->SetFromFunction(YourCustomFunction);
    InternalCall->AllocateDefaultPins();
    
    // Connect pins and handle execution flow
    // Your custom VM code generation logic here
    
    BreakAllNodeLinks();
}
```

## Practical Examples

### Example 1: Simple Counter Node

```cpp
UCLASS()
class UK2Node_Counter : public UK2Node
{
    GENERATED_BODY()

public:
    virtual void AllocateDefaultPins() override
    {
        Super::AllocateDefaultPins();
        
        // Execution pins
        CreatePin(EGPD_Input, UEdGraphSchema_K2::PC_Exec, UEdGraphSchema_K2::PN_Execute);
        CreatePin(EGPD_Output, UEdGraphSchema_K2::PC_Exec, UEdGraphSchema_K2::PN_Then);
        
        // Data pins
        CreatePin(EGPD_Input, UEdGraphSchema_K2::PC_Int, TEXT("Increment"));
        CreatePin(EGPD_Output, UEdGraphSchema_K2::PC_Int, TEXT("Current Value"));
    }
    
    virtual FText GetNodeTitle(ENodeTitleType::Type TitleType) const override
    {
        return FText::FromString("Counter");
    }
    
    virtual void ExpandNode(class FKismetCompilerContext& CompilerContext, UEdGraph* SourceGraph) override
    {
        Super::ExpandNode(CompilerContext, SourceGraph);
        
        // Create variable to store counter state
        FProperty* CounterProperty = CompilerContext.CreateVariable(TEXT("CounterValue"), UEdGraphSchema_K2::PC_Int);
        
        // Get our pins
        UEdGraphPin* ExecIn = GetExecPin();
        UEdGraphPin* ExecOut = FindPinChecked(UEdGraphSchema_K2::PN_Then, EGPD_Output);
        UEdGraphPin* IncrementPin = FindPinChecked(TEXT("Increment"), EGPD_Input);
        UEdGraphPin* CurrentValuePin = FindPinChecked(TEXT("Current Value"), EGPD_Output);
        
        // Create variable get node for current value
        UK2Node_VariableGet* GetNode = CompilerContext.SpawnIntermediateNode<UK2Node_VariableGet>(this, SourceGraph);
        GetNode->VariableReference.SetFromField<FProperty>(CounterProperty, false);
        GetNode->AllocateDefaultPins();
        
        // Create add node
        UK2Node_CallFunction* AddNode = CompilerContext.SpawnIntermediateNode<UK2Node_CallFunction>(this, SourceGraph);
        UFunction* AddFunction = UKismetMathLibrary::StaticClass()->FindFunctionByName("Add_IntInt");
        AddNode->SetFromFunction(AddFunction);
        AddNode->AllocateDefaultPins();
        
        // Create variable set node
        UK2Node_VariableSet* SetNode = CompilerContext.SpawnIntermediateNode<UK2Node_VariableSet>(this, SourceGraph);
        SetNode->VariableReference.SetFromField<FProperty>(CounterProperty, false);
        SetNode->AllocateDefaultPins();
        
        // Connect the nodes
        CompilerContext.GetSchema()->TryCreateConnection(ExecIn, SetNode->GetExecPin());
        CompilerContext.GetSchema()->TryCreateConnection(SetNode->GetThenPin(), ExecOut);
        
        CompilerContext.GetSchema()->TryCreateConnection(GetNode->FindPinChecked(TEXT("Output"), EGPD_Output), AddNode->FindPinChecked(TEXT("A"), EGPD_Input));
        CompilerContext.GetSchema()->TryCreateConnection(IncrementPin, AddNode->FindPinChecked(TEXT("B"), EGPD_Input));
        CompilerContext.GetSchema()->TryCreateConnection(AddNode->GetReturnValuePin(), SetNode->FindPinChecked(CounterProperty->GetName(), EGPD_Input));
        CompilerContext.GetSchema()->TryCreateConnection(AddNode->GetReturnValuePin(), CurrentValuePin);
        
        BreakAllNodeLinks();
    }
    
    virtual void GetMenuActions(FBlueprintActionDatabaseRegistrar& ActionRegistrar) const override
    {
        UClass* ActionKey = GetClass();
        if (ActionRegistrar.IsOpenForRegistration(ActionKey))
        {
            UBlueprintNodeSpawner* NodeSpawner = UBlueprintNodeSpawner::Create(GetClass());
            ActionRegistrar.AddBlueprintAction(ActionKey, NodeSpawner);
        }
    }
    
    virtual FText GetMenuCategory() const override
    {
        return FText::FromString("Custom Utility");
    }
    
    virtual FLinearColor GetNodeTitleColor() const override
    {
        return FLinearColor::Blue;
    }
};
```

### Example 2: Multi-Execution Pin Node

```cpp
UCLASS()
class UK2Node_MultiExec : public UK2Node
{
    GENERATED_BODY()

public:
    UPROPERTY()
    int32 NumExecInputs = 3;

    virtual void AllocateDefaultPins() override
    {
        Super::AllocateDefaultPins();
        
        // Create multiple execution input pins
        for (int32 i = 0; i < NumExecInputs; ++i)
        {
            FString PinName = FString::Printf(TEXT("Exec %d"), i + 1);
            CreatePin(EGPD_Input, UEdGraphSchema_K2::PC_Exec, *PinName);
        }
        
        // Single execution output
        CreatePin(EGPD_Output, UEdGraphSchema_K2::PC_Exec, UEdGraphSchema_K2::PN_Then);
        
        // Counter output
        CreatePin(EGPD_Output, UEdGraphSchema_K2::PC_Int, TEXT("Exec Count"));
    }
    
    virtual FText GetNodeTitle(ENodeTitleType::Type TitleType) const override
    {
        return FText::FromString("Multi Exec Counter");
    }
    
    virtual void ExpandNode(class FKismetCompilerContext& CompilerContext, UEdGraph* SourceGraph) override
    {
        Super::ExpandNode(CompilerContext, SourceGraph);
        
        // Create counter variable
        FProperty* CounterProperty = CompilerContext.CreateVariable(TEXT("ExecCounter"), UEdGraphSchema_K2::PC_Int);
        
        UEdGraphPin* OutputExec = FindPinChecked(UEdGraphSchema_K2::PN_Then, EGPD_Output);
        UEdGraphPin* CounterOutput = FindPinChecked(TEXT("Exec Count"), EGPD_Output);
        
        // For each execution input, create increment logic
        for (int32 i = 0; i < NumExecInputs; ++i)
        {
            FString PinName = FString::Printf(TEXT("Exec %d"), i + 1);
            UEdGraphPin* ExecInput = FindPinChecked(*PinName, EGPD_Input);
            
            if (ExecInput->LinkedTo.Num() > 0)
            {
                // Create increment sequence for this exec pin
                CreateIncrementSequence(CompilerContext, SourceGraph, ExecInput, OutputExec, CounterProperty, CounterOutput);
            }
        }
        
        BreakAllNodeLinks();
    }

private:
    void CreateIncrementSequence(FKismetCompilerContext& CompilerContext, UEdGraph* SourceGraph, 
                               UEdGraphPin* ExecInput, UEdGraphPin* ExecOutput, 
                               FProperty* CounterProperty, UEdGraphPin* CounterOutput)
    {
        // Create variable get
        UK2Node_VariableGet* GetNode = CompilerContext.SpawnIntermediateNode<UK2Node_VariableGet>(this, SourceGraph);
        GetNode->VariableReference.SetFromField<FProperty>(CounterProperty, false);
        GetNode->AllocateDefaultPins();
        
        // Create increment
        UK2Node_CallFunction* IncrementNode = CompilerContext.SpawnIntermediateNode<UK2Node_CallFunction>(this, SourceGraph);
        UFunction* AddFunction = UKismetMathLibrary::StaticClass()->FindFunctionByName("Add_IntInt");
        IncrementNode->SetFromFunction(AddFunction);
        IncrementNode->AllocateDefaultPins();
        
        // Set increment value to 1
        IncrementNode->FindPinChecked(TEXT("B"), EGPD_Input)->DefaultValue = TEXT("1");
        
        // Create variable set
        UK2Node_VariableSet* SetNode = CompilerContext.SpawnIntermediateNode<UK2Node_VariableSet>(this, SourceGraph);
        SetNode->VariableReference.SetFromField<FProperty>(CounterProperty, false);
        SetNode->AllocateDefaultPins();
        
        // Connect nodes
        CompilerContext.GetSchema()->TryCreateConnection(ExecInput, SetNode->GetExecPin());
        CompilerContext.GetSchema()->TryCreateConnection(SetNode->GetThenPin(), ExecOutput);
        
        CompilerContext.GetSchema()->TryCreateConnection(GetNode->FindPinChecked(TEXT("Output"), EGPD_Output), IncrementNode->FindPinChecked(TEXT("A"), EGPD_Input));
        CompilerContext.GetSchema()->TryCreateConnection(IncrementNode->GetReturnValuePin(), SetNode->FindPinChecked(CounterProperty->GetName(), EGPD_Input));
        CompilerContext.GetSchema()->TryCreateConnection(IncrementNode->GetReturnValuePin(), CounterOutput);
    }
};
```

## Troubleshooting

### Node Doesn't Appear in Blueprint Editor

**Checklist:**
1. ✅ Module is loaded and initialized
2. ✅ GetMenuActions() is implemented correctly
3. ✅ UBlueprintNodeSpawner is created and registered
4. ✅ GetMenuCategory() returns appropriate category
5. ✅ Class has UCLASS() macro
6. ✅ Module build dependencies include "BlueprintGraph"

### Compilation Errors

**Common Issues:**
- **Missing function references**: Check UFunction exists and is accessible
- **Invalid pin connections**: Verify pin types and directions match
- **Circular dependencies**: Ensure expansion doesn't create infinite loops
- **Missing break links**: Always call BreakAllNodeLinks() after expansion

### Runtime Crashes

**Debug Steps:**
1. Check pin validation in ExpandNode()
2. Verify all created nodes have proper pins allocated
3. Ensure function references are valid
4. Test with simple Blueprint graphs first

### Node Appears but Doesn't Work

**Check:**
- ExpandNode() implementation
- Proper intermediate node creation
- Pin connection logic
- Variable lifetime and scope

## Best Practices

### 1. Pin Management
- Always call Super::AllocateDefaultPins()
- Use consistent pin naming
- Validate pin existence before use
- Handle optional pins gracefully

### 2. Compilation
- Keep expansion logic simple and focused
- Use appropriate intermediate nodes
- Always break original node links
- Provide meaningful error messages

### 3. User Experience
- Implement clear node titles and tooltips
- Use appropriate node colors and icons
- Group related nodes in categories
- Provide context-sensitive menu items

### 4. Performance
- Minimize expansion complexity
- Cache function references when possible
- Use pure nodes for side-effect-free operations
- Consider compilation time impact

### 5. Debugging
- Add logging to expansion logic
- Use FCompilerResultsLog for user feedback
- Test with various Blueprint configurations
- Provide clear error messages

### 6. Maintenance
- Document node behavior clearly
- Version control expansion logic changes
- Test against Blueprint changes
- Consider backward compatibility

## Related Documentation

- [UK2Node_CallFunction API](/home/jbloch/mcp_resources/unreal-mcp/ref_docs/UK2Node_CallFunction_API.md)
- [FKismetEditorUtilities API](/home/jbloch/mcp_resources/unreal-mcp/ref_docs/FKismetEditorUtilities.md)
- [Blueprint Compiler Overview](/home/jbloch/mcp_resources/unreal-mcp/ref_docs/blueprint_compiler_overview.md)
- [Kismet Compiler Pipeline](/home/jbloch/mcp_resources/unreal-mcp/ref_docs/KismetCompiler_Pipeline_Overview.md)

## Contributing

This documentation is based on community discussions and developer experiences. If you have additional examples, solutions, or questions, please contribute to help other developers working with K2Nodes.