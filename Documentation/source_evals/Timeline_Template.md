# Timeline Template Analysis - Blueprint to C++ Conversion

## File Location
- **Primary**: `/Engine/Source/Runtime/Engine/Classes/Engine/TimelineTemplate.h`
- **Context**: Design-time timeline asset structure for Blueprint timeline system

## Core Purpose
UTimelineTemplate represents the design-time configuration of Blueprint timelines, containing track definitions, curve references, metadata, and structural information needed for Blueprint-to-C++ timeline conversion.

## Critical Data Structures

### Timeline Configuration Properties
```cpp
UCLASS(MinimalAPI)
class UTimelineTemplate : public UObject
{
    UPROPERTY(EditAnywhere, Category=TimelineTemplate)
    float TimelineLength;                           // Timeline duration in seconds
    
    UPROPERTY(EditAnywhere, Category=TimelineTemplate)
    TEnumAsByte<ETimelineLengthMode> LengthMode;    // Length determination method
    
    UPROPERTY(EditAnywhere, Category=TimelineTemplate)
    uint8 bAutoPlay:1;                              // Auto-start on BeginPlay
    
    UPROPERTY(EditAnywhere, Category=TimelineTemplate)
    uint8 bLoop:1;                                  // Loop when finished
    
    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category=TimelineTemplate)
    uint8 bReplicated:1;                            // Network replication
    
    UPROPERTY(EditAnywhere, Category = TimelineTemplate)
    uint8 bIgnoreTimeDilation : 1;                  // Time dilation immunity
};
```

### Track Base Structure
```cpp
USTRUCT()
struct FTTTrackBase
{
    enum ETrackType
    {
        TT_Event,               // Event triggers
        TT_FloatInterp,         // Float interpolation
        TT_VectorInterp,        // Vector interpolation  
        TT_LinearColorInterp,   // Color interpolation
    };
    
private:
    UPROPERTY()
    FName TrackName;                                // User-defined track name
    
public:
    UPROPERTY()
    bool bIsExternalCurve;                          // External vs embedded curve
    
#if WITH_EDITORONLY_DATA
    UPROPERTY()
    bool bIsExpanded;                               // UI expansion state
    
    UPROPERTY()
    bool bIsCurveViewSynchronized;                  // UI curve sync state
#endif
};
```

### Event Track Structure
```cpp
USTRUCT()
struct FTTEventTrack : public FTTTrackBase
{
    GENERATED_USTRUCT_BODY()
    
private:
    UPROPERTY()
    FName FunctionName;                             // Event callback function name
    
public:
    UPROPERTY()
    TObjectPtr<class UCurveFloat> CurveKeys;        // Event timing curve
};
```

### Property Track Base
```cpp
USTRUCT()
struct FTTPropertyTrack : public FTTTrackBase
{
    GENERATED_USTRUCT_BODY()
    
private:
    UPROPERTY()
    FName PropertyName;                             // Target property for binding
};
```

### Float Track Structure
```cpp
USTRUCT()
struct FTTFloatTrack : public FTTPropertyTrack
{
    GENERATED_USTRUCT_BODY()
    
    UPROPERTY()
    TObjectPtr<class UCurveFloat> CurveFloat;       // Float interpolation curve
};
```

### Vector Track Structure  
```cpp
USTRUCT()
struct FTTVectorTrack : public FTTPropertyTrack
{
    GENERATED_USTRUCT_BODY()
    
    UPROPERTY()
    TObjectPtr<class UCurveVector> CurveVector;     // Vector interpolation curve
};
```

### Linear Color Track Structure
```cpp
USTRUCT()
struct FTTLinearColorTrack : public FTTPropertyTrack
{
    GENERATED_USTRUCT_BODY()
    
    UPROPERTY()
    TObjectPtr<class UCurveLinearColor> CurveLinearColor;  // Color interpolation curve
};
```

### Track Collections
```cpp
// Timeline track arrays
UPROPERTY()
TArray<FTTEventTrack> EventTracks;                  // Event trigger tracks

UPROPERTY()
TArray<FTTFloatTrack> FloatTracks;                  // Float interpolation tracks

UPROPERTY()
TArray<FTTVectorTrack> VectorTracks;                // Vector interpolation tracks

UPROPERTY()
TArray<FTTLinearColorTrack> LinearColorTracks;      // Color interpolation tracks
```

### Metadata and Identification
```cpp
UPROPERTY(EditAnywhere, Category=BPVariableDescription)
TArray<FBPVariableMetaDataEntry> MetaDataArray;     // Blueprint variable metadata

UPROPERTY(duplicatetransient)
FGuid TimelineGuid;                                  // Unique timeline identifier

UPROPERTY()
TEnumAsByte<ETickingGroup> TimelineTickGroup;       // Component tick group assignment
```

### Cached Name Properties
```cpp
private:
    UPROPERTY()
    FName VariableName;                              // C++ variable name
    
    UPROPERTY()
    FName DirectionPropertyName;                     // Direction property binding
    
    UPROPERTY()
    FName UpdateFunctionName;                        // Update callback function
    
    UPROPERTY()
    FName FinishedFunctionName;                      // Finished callback function
```

## Blueprint to C++ Conversion Requirements

### 1. Timeline Component Declaration
**C++ Header Generation**:
```cpp
// From TimelineTemplate configuration
UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = Timeline, meta = (/* MetaDataArray */))
class UTimelineComponent* {VariableName};

// Property bindings (if PropertyName specified)
UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = Timeline)
float {PropertyName};  // For float tracks

UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = Timeline) 
FVector {PropertyName};  // For vector tracks

UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = Timeline)
FLinearColor {PropertyName};  // For color tracks
```

### 2. Constructor Implementation
**Basic Setup**:
```cpp
AMyActor::AMyActor()
{
    // Timeline component creation
    {VariableName} = CreateDefaultSubobject<UTimelineComponent>(TEXT("{VariableName}"));
    
    // Apply template configuration
    {VariableName}->SetTimelineLength({TimelineLength});
    {VariableName}->SetLooping({bLoop});
    {VariableName}->SetIgnoreTimeDilation({bIgnoreTimeDilation});
    
    // Set tick group if specified
    {VariableName}->SetTickGroup({TimelineTickGroup});
    
    // Property binding setup
    if (PropertySetObject needed)
    {
        {VariableName}->SetPropertySetObject(this);
    }
}
```

### 3. Curve Asset Loading
**Asset Reference Resolution**:
```cpp
// Constructor or BeginPlay
void LoadTimelineCurves()
{
    // Float track curves
    for (const FTTFloatTrack& FloatTrack : FloatTracks)
    {
        if (FloatTrack.CurveFloat)
        {
            // Load curve asset or use reference
            UCurveFloat* Curve = FloatTrack.CurveFloat;
            // Store for runtime use
        }
    }
    
    // Vector track curves  
    for (const FTTVectorTrack& VectorTrack : VectorTracks)
    {
        if (VectorTrack.CurveVector)
        {
            UCurveVector* Curve = VectorTrack.CurveVector;
            // Store for runtime use
        }
    }
    
    // Color track curves
    for (const FTTLinearColorTrack& ColorTrack : LinearColorTracks)
    {
        if (ColorTrack.CurveLinearColor)
        {
            UCurveLinearColor* Curve = ColorTrack.CurveLinearColor;
            // Store for runtime use
        }
    }
}
```

### 4. BeginPlay Track Setup
**Runtime Track Configuration**:
```cpp
void AMyActor::BeginPlay()
{
    Super::BeginPlay();
    
    if (!{VariableName}) return;
    
    // Setup float tracks
    for (const FTTFloatTrack& FloatTrack : Template->FloatTracks)
    {
        if (FloatTrack.CurveFloat)
        {
            FOnTimelineFloat FloatDelegate;
            FloatDelegate.BindDynamic(this, &AMyActor::{InterpolationFunctionName});
            
            {VariableName}->AddInterpFloat(
                FloatTrack.CurveFloat,
                FloatDelegate, 
                FloatTrack.GetPropertyName(),  // PropertyName
                FloatTrack.GetTrackName()      // TrackName
            );
        }
    }
    
    // Setup vector tracks
    for (const FTTVectorTrack& VectorTrack : Template->VectorTracks)
    {
        if (VectorTrack.CurveVector)
        {
            FOnTimelineVector VectorDelegate;
            VectorDelegate.BindDynamic(this, &AMyActor::{InterpolationFunctionName});
            
            {VariableName}->AddInterpVector(
                VectorTrack.CurveVector,
                VectorDelegate,
                VectorTrack.GetPropertyName(),
                VectorTrack.GetTrackName()
            );
        }
    }
    
    // Setup color tracks
    for (const FTTLinearColorTrack& ColorTrack : Template->LinearColorTracks)
    {
        if (ColorTrack.CurveLinearColor)
        {
            FOnTimelineLinearColor ColorDelegate;
            ColorDelegate.BindDynamic(this, &AMyActor::{InterpolationFunctionName});
            
            {VariableName}->AddInterpLinearColor(
                ColorTrack.CurveLinearColor,
                ColorDelegate,
                ColorTrack.GetPropertyName(),
                ColorTrack.GetTrackName()
            );
        }
    }
    
    // Setup event tracks
    for (const FTTEventTrack& EventTrack : Template->EventTracks)
    {
        if (EventTrack.CurveKeys)
        {
            // Extract event times from curve
            const FRichCurve& Curve = EventTrack.CurveKeys->FloatCurve;
            for (auto It = Curve.GetKeyIterator(); It; ++It)
            {
                float EventTime = It->Time;
                
                FOnTimelineEvent EventDelegate;
                EventDelegate.BindDynamic(this, &AMyActor::{EventTrack.GetFunctionName()});
                
                {VariableName}->AddEvent(EventTime, EventDelegate);
            }
        }
    }
    
    // Setup lifecycle callbacks
    if (HasUpdateFunction)
    {
        FOnTimelineEvent UpdateDelegate;
        UpdateDelegate.BindDynamic(this, &AMyActor::{UpdateFunctionName});
        {VariableName}->SetTimelinePostUpdateFunc(UpdateDelegate);
    }
    
    if (HasFinishedFunction)
    {
        FOnTimelineEvent FinishedDelegate;
        FinishedDelegate.BindDynamic(this, &AMyActor::{FinishedFunctionName});
        {VariableName}->SetTimelineFinishedFunc(FinishedDelegate);
    }
    
    // Setup direction property
    if (DirectionPropertyName != NAME_None)
    {
        {VariableName}->SetDirectionPropertyName({DirectionPropertyName});
    }
    
    // Auto-play if configured
    if (Template->bAutoPlay)
    {
        {VariableName}->PlayFromStart();
    }
}
```

### 5. Delegate Function Generation
**Function Declarations and Implementations**:
```cpp
// Header declarations
UFUNCTION()
void {UpdateFunctionName}();

UFUNCTION()
void {FinishedFunctionName}();

// Float track callbacks
UFUNCTION()
void {FloatTrack.FunctionName}(float Value);

// Vector track callbacks
UFUNCTION()
void {VectorTrack.FunctionName}(FVector Value);

// Color track callbacks  
UFUNCTION()
void {ColorTrack.FunctionName}(FLinearColor Value);

// Event callbacks
UFUNCTION()
void {EventTrack.FunctionName}();

// Implementation placeholders
void AMyActor::{UpdateFunctionName}()
{
    // Per-tick update logic from Blueprint
}

void AMyActor::{FinishedFunctionName}()
{
    // Timeline completion logic from Blueprint
}
```

## Blueprint JSON Processing

### Expected JSON Structure
```json
{
  "TimelineTemplate": {
    "TimelineGuid": "12345678-1234-5678-9ABC-123456789ABC",
    "VariableName": "MyTimeline",
    "TimelineLength": 5.0,
    "LengthMode": "TL_TimelineLength",
    "bAutoPlay": false,
    "bLoop": true,
    "bReplicated": false,
    "bIgnoreTimeDilation": false,
    "TimelineTickGroup": "TG_PrePhysics",
    "UpdateFunctionName": "TimelineUpdate",
    "FinishedFunctionName": "TimelineFinished",
    "DirectionPropertyName": "TimelineDirection",
    "FloatTracks": [
      {
        "TrackName": "Alpha",
        "CurveFloat": "/Game/Curves/AlphaCurve.AlphaCurve",
        "PropertyName": "AlphaValue",
        "bIsExternalCurve": true
      }
    ],
    "VectorTracks": [
      {
        "TrackName": "Movement",
        "CurveVector": "/Game/Curves/MovementCurve.MovementCurve", 
        "PropertyName": "MovementVector"
      }
    ],
    "LinearColorTracks": [
      {
        "TrackName": "Color",
        "CurveLinearColor": "/Game/Curves/ColorCurve.ColorCurve",
        "PropertyName": "ColorValue"
      }
    ],
    "EventTracks": [
      {
        "TrackName": "Events",
        "FunctionName": "OnTimelineEvent",
        "CurveKeys": "/Game/Curves/EventCurve.EventCurve"
      }
    ],
    "MetaDataArray": [
      {"Key": "Category", "Value": "Timeline"},
      {"Key": "BlueprintReadOnly", "Value": "true"}
    ]
  }
}
```

### Generated C++ Output
```cpp
// Generated class with timeline from template
UCLASS()
class MYPROJECT_API AMyActor : public AActor
{
    GENERATED_BODY()
    
public:
    AMyActor();
    
protected:
    virtual void BeginPlay() override;
    
    // Timeline component - from VariableName
    UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = Timeline)
    class UTimelineComponent* MyTimeline;
    
    // Property bindings
    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = Timeline)
    float AlphaValue;
    
    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = Timeline)
    FVector MovementVector;
    
    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = Timeline)
    FLinearColor ColorValue;
    
    // Direction property
    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = Timeline)
    TEnumAsByte<ETimelineDirection::Type> TimelineDirection;
    
    // Delegate functions
    UFUNCTION()
    void TimelineUpdate();
    
    UFUNCTION()
    void TimelineFinished();
    
    UFUNCTION()
    void OnAlphaUpdate(float Value);
    
    UFUNCTION()
    void OnMovementUpdate(FVector Value);
    
    UFUNCTION()
    void OnColorUpdate(FLinearColor Value);
    
    UFUNCTION()
    void OnTimelineEvent();
};
```

## Track Index and Display Management

### Track Identification System
```cpp
USTRUCT()
struct ENGINE_API FTTTrackId
{
    UPROPERTY()
    int32 TrackType;     // ETrackType enumeration value
    
    UPROPERTY()
    int32 TrackIndex;    // Index within type-specific array
};

#if WITH_EDITORONLY_DATA
UPROPERTY()
TArray<FTTTrackId> TrackDisplayOrder;  // Editor display ordering
#endif
```

### Track Management Functions
```cpp
// Track lookup functions
int32 FindFloatTrackIndex(FName FloatTrackName) const;
int32 FindVectorTrackIndex(FName VectorTrackName) const;
int32 FindEventTrackIndex(FName EventTrackName) const;
int32 FindLinearColorTrackIndex(FName ColorTrackName) const;

// Display management
FTTTrackId GetDisplayTrackId(int32 DisplayTrackIndex);
int32 GetNumDisplayTracks() const;
void RemoveDisplayTrack(int32 DisplayTrackIndex);
void MoveDisplayTrack(int32 DisplayTrackIndex, int32 DirectionDelta);
void AddDisplayTrack(FTTTrackId NewTrackId);
```

## Curve Asset Management

### External vs Internal Curves
```cpp
// Track base indicates curve storage method
bool bIsExternalCurve;  // true = external asset, false = embedded

// External curve loading
if (bIsExternalCurve)
{
    // Load from asset reference
    UCurveFloat* Curve = LoadObject<UCurveFloat>(nullptr, CurveAssetPath);
}
else
{
    // Use embedded curve data
    UCurveFloat* Curve = InternalCurveData;
}
```

### Curve Collection
```cpp
void GetAllCurves(TSet<class UCurveBase*>& InOutCurves) const;
```
**Conversion Use**: Collect all curve references for asset dependency tracking

## Template Naming Conventions

### Variable Name Templates
```cpp
// Template naming helper
static FString TimelineVariableNameToTemplateName(FName Name);
static const FString TemplatePostfix;  // "_Template"

// Usage: MyTimeline → MyTimeline_Template
FString TemplateName = TimelineVariableNameToTemplateName(VariableName);
```

### Function Name Caching
```cpp
// Cached function names based on timeline variable name
FName GetUpdateFunctionName() const { return UpdateFunctionName; }      // {VariableName}__UpdateFunc
FName GetFinishedFunctionName() const { return FinishedFunctionName; }  // {VariableName}__FinishedFunc
FName GetDirectionPropertyName() const { return DirectionPropertyName; } // {VariableName}_Direction
```

## Critical Conversion Considerations

### Timeline Lifecycle Management
- Templates define design-time structure
- Runtime component requires setup in BeginPlay
- Auto-play handling needs constructor vs BeginPlay decision

### Curve Asset Dependencies  
- Track curve references must be resolved to actual assets
- Handle missing curve assets gracefully
- Support both external and embedded curves

### Property Binding Integration
- PropertyName strings must match actual UPROPERTY names
- Type safety depends on reflection system
- Direct property updates bypass delegate system

### Function Name Generation
- Delegate function names derived from timeline variable name
- Must be valid C++ identifiers
- UFUNCTION() requirement for dynamic binding

### Metadata Preservation
- MetaDataArray converted to UPROPERTY meta tags
- TimelineGuid preserved for editor integration
- Category information maintained for organization

## Related Systems
- **UTimelineComponent**: Runtime timeline execution
- **UCurveFloat/Vector/LinearColor**: Curve asset types
- **K2Node_Timeline**: Blueprint node representation  
- **FTimeline**: Core timeline execution structure

## Conversion Priority: HIGH
Timeline templates are essential for converting Blueprint timeline functionality to C++. The template defines all design-time configuration needed to generate equivalent C++ timeline code with proper curve bindings and delegate setup.