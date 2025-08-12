# UK2Node\_Variable

## Navigation

## Inheritance Hierarchy

## References

## Syntax

## Variables

## Constructors

## Functions

## Overridden from UK2Node

## Deprecated Variables

[API](https://dev.epicgames.com/documentation/en-us/unreal-engine/API) \> [API/Editor](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Editor) \> [API/Editor/BlueprintGraph](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Editor/BlueprintGraph)

[UK2Node\_Variable](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Editor/BlueprintGraph/UK2Node_Variable/__ctor)
(
const FObjectInitializer& ObjectInitializer

)

[CanPasteHere](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Editor/BlueprintGraph/UK2Node_Variable/CanPasteHere)
(
const UEdGraph\* TargetGraph

)

[CheckForErrors](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Editor/BlueprintGraph/UK2Node_Variable/CheckForErrors)
(
const [UEdGraphSchema\_K2](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Editor/BlueprintGraph/UEdGraphSchema_K2)\\* Schema,

FCompilerResultsLog& MessageLog

)

[CreatePinForSelf](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Editor/BlueprintGraph/UK2Node_Variable/CreatePinForSelf) ()

[CreatePinForVariable](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Editor/BlueprintGraph/UK2Node_Variable/CreatePinForVariable)
(
EEdGraphPinDirection Direction,

FName InPinName

)

[DoesRenamedVariableMatch](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Editor/BlueprintGraph/UK2Node_Variable/DoesRenamedVariableMatch)
(
FName OldVariableName,

FName NewVariableName,

UStruct\* StructType

)

[FunctionParameterExists](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Editor/BlueprintGraph/UK2Node_Variable/FunctionParameterExists)
(
const UEdGraph\* InFunctionGraph,

const FName InParameterName

)

[GetActorComponent](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Editor/BlueprintGraph/UK2Node_Variable/GetActorComponent)
(
const FProperty\* VariableProperty

)

[GetBlueprintVarDescription](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Editor/BlueprintGraph/UK2Node_Variable/GetBlueprintVarDescription) ()

[GetDeprecationResponse](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Editor/BlueprintGraph/UK2Node_Variable/GetDeprecationResponse)
(
EEdGraphNodeDeprecationType DeprecationType

)

[GetDocumentationExcerptName](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Editor/BlueprintGraph/UK2Node_Variable/GetDocumentationExcerptName) ()

[GetFindReferenceSearchString\_Impl](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Editor/BlueprintGraph/UK2Node_Variable/GetFindReferenceSearchString_Imp-)
(
EGetFindReferenceSearchStringFlags InFlags

)

[GetIconAndTint](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Editor/BlueprintGraph/UK2Node_Variable/GetIconAndTint)
(
FLinearColor& OutColor

)

[GetNodeContextMenuActions](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Editor/BlueprintGraph/UK2Node_Variable/GetNodeContextMenuActions)
(
UToolMenu\* Menu,

UGraphNodeContextMenuContext\* Context

)

[GetPropertyForVariable](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Editor/BlueprintGraph/UK2Node_Variable/GetPropertyForVariable) ()

[GetPropertyForVariableFromSkeleton](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Editor/BlueprintGraph/UK2Node_Variable/GetPropertyForVariableFromSkelet-) ()

[GetValuePin](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Editor/BlueprintGraph/UK2Node_Variable/GetValuePin) ()

[GetVariableIconAndColor](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Editor/BlueprintGraph/UK2Node_Variable/GetVariableIconAndColor)
(
const UStruct\* VarScope,

FName VarName,

FLinearColor& IconColorOut

)

[GetVariableSourceClass](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Editor/BlueprintGraph/UK2Node_Variable/GetVariableSourceClass) ()

[GetVarIconFromPinType](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Editor/BlueprintGraph/UK2Node_Variable/GetVarIconFromPinType)
(
const FEdGraphPinType& InPinType,

FLinearColor& IconColorOut

)

[GetVarName](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Editor/BlueprintGraph/UK2Node_Variable/GetVarName) ()

[GetVarNameString](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Editor/BlueprintGraph/UK2Node_Variable/GetVarNameString) ()

[GetVarNameText](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Editor/BlueprintGraph/UK2Node_Variable/GetVarNameText) ()

[HasDeprecatedReference](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Editor/BlueprintGraph/UK2Node_Variable/HasDeprecatedReference) ()

[HasExternalDependencies](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Editor/BlueprintGraph/UK2Node_Variable/HasExternalDependencies)
(
TArray< class UStruct\* >\* OptionalOutput

)

[PostPasteNode](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Editor/BlueprintGraph/UK2Node_Variable/PostPasteNode) ()

[RecreatePinForVariable](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Editor/BlueprintGraph/UK2Node_Variable/RecreatePinForVariable)
(
EEdGraphPinDirection Direction,

TArray< UEdGraphPin\* >& OldPins,

FName InPinName

)

[RemapRestrictedLinkReference](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Editor/BlueprintGraph/UK2Node_Variable/RemapRestrictedLinkReference)
(
FName OldVariableName,

FName NewVariableName,

const UClass\* MatchInVariableClass,

const UClass\* RemapIfLinkedToClass,

bool bLogWarning

)

[SetFromProperty](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Editor/BlueprintGraph/UK2Node_Variable/SetFromProperty)
(
const FProperty\* Property,

bool bSelfContext,

UClass\* OwnerClass

)

[SuppressDeprecationWarning](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Editor/BlueprintGraph/UK2Node_Variable/SuppressDeprecationWarning) ()

[AutowireNewNode](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Editor/BlueprintGraph/UK2Node_Variable/AutowireNewNode)
(
UEdGraphPin\* FromPin

)

[CanJumpToDefinition](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Editor/BlueprintGraph/UK2Node_Variable/CanJumpToDefinition) ()

[DoPinsMatchForReconstruction](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Editor/BlueprintGraph/UK2Node_Variable/DoPinsMatchForReconstruction)
(
const UEdGraphPin\* NewPin,

int32 NewPinIndex,

const UEdGraphPin\* OldPin,

int32 OldPinIndex

)

[DrawNodeAsVariable](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Editor/BlueprintGraph/UK2Node_Variable/DrawNodeAsVariable) ()

[GetCornerIcon](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Editor/BlueprintGraph/UK2Node_Variable/GetCornerIcon) ()

[GetDocumentationLink](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Editor/BlueprintGraph/UK2Node_Variable/GetDocumentationLink) ()

[GetJumpTargetForDoubleClick](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Editor/BlueprintGraph/UK2Node_Variable/GetJumpTargetForDoubleClick) ()

[GetNodeAttributes](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Editor/BlueprintGraph/UK2Node_Variable/GetNodeAttributes)
(
TArray< [TKeyValuePair](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Runtime/Core/Templates/TKeyValuePair) < FString, FString > >& OutNodeAttributes

)

[GetNodeTitleColor](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Editor/BlueprintGraph/UK2Node_Variable/GetNodeTitleColor) ()

[GetPinMetaData](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Editor/BlueprintGraph/UK2Node_Variable/GetPinMetaData)
(
FName InPinName,

FName InKey

)

[GetToolTipHeading](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Editor/BlueprintGraph/UK2Node_Variable/GetToolTipHeading) ()

[HandleVariableRenamed](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Editor/BlueprintGraph/UK2Node_Variable/HandleVariableRenamed)
(
UBlueprint\* InBlueprint,

UClass\* InVariableClass,

UEdGraph\* InGraph,

const FName& InOldVarName,

const FName& InNewVarName

)

[JumpToDefinition](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Editor/BlueprintGraph/UK2Node_Variable/JumpToDefinition) ()

[ReconstructNode](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Editor/BlueprintGraph/UK2Node_Variable/ReconstructNode) ()

[ReferencesVariable](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Editor/BlueprintGraph/UK2Node_Variable/ReferencesVariable)
(
const FName& InVarName,

const UStruct\* InScope

)

[ReplaceReferences](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Editor/BlueprintGraph/UK2Node_Variable/ReplaceReferences)
(
UBlueprint\* InBlueprint,

UBlueprint\* InReplacementBlueprint,

const FMemberReference& InSource,

const FMemberReference& InReplacement

)

[Serialize](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Editor/BlueprintGraph/UK2Node_Variable/Serialize)
(
FArchive& Ar

)

[ValidateNodeDuringCompilation](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Editor/BlueprintGraph/UK2Node_Variable/ValidateNodeDuringCompilation)
(
FCompilerResultsLog& MessageLog

)

Ask questions and help your peers [Developer Forums](https://forums.unrealengine.com/categories?tag=unreal-engine)

Write your own tutorials or read those from others [Learning Library](https://dev.epicgames.com/community/unreal-engine/learning)

`
UCLASS (Abstract)
class UK2Node_Variable : public UK2Node  `