# Timeline Component Analysis - Blueprint to C++ Conversion

## File Location
- **Primary**: `/Engine/Source/Runtime/Engine/Classes/Components/TimelineComponent.h`
- **Context**: Blueprint Timeline system for animation and interpolation

## Core Purpose
UTimelineComponent provides Blueprint timeline functionality with curve-based animation, event handling, and property interpolation that requires comprehensive C++ conversion for Blueprint-to-C++ migration.

## Critical Data Structures

### Timeline Core Structure (FTimeline)
```cpp
USTRUCT()
struct FTimeline
{
    // Timeline Configuration
    UPROPERTY(NotReplicated)
    TEnumAsByte<ETimelineLengthMode> LengthMode;    // TL_TimelineLength or TL_LastKeyFrame
    
    UPROPERTY()
    uint8 bLooping:1;                               // Loop when reaching end
    
    UPROPERTY()
    uint8 bReversePlayback:1;                       // Direction of playback
    
    UPROPERTY()
    uint8 bPlaying:1;                               // Current play state
    
    // Timeline Properties
    UPROPERTY(NotReplicated)
    float Length;                                   // Timeline duration
    
    UPROPERTY()
    float PlayRate;                                 // Playback speed multiplier
    
    UPROPERTY()
    float Position;                                 // Current playback position
};
```

### Track Systems

#### Float Track Structure
```cpp
USTRUCT()
struct FTimelineFloatTrack
{
    UPROPERTY()
    TObjectPtr<class UCurveFloat> FloatCurve;       // Curve asset reference
    
    UPROPERTY()
    FOnTimelineFloat InterpFunc;                    // Delegate for output
    
    UPROPERTY()
    FName TrackName;                                // Track identifier
    
    UPROPERTY()
    FName FloatPropertyName;                        // Target property name
    
    // Runtime cached data
    FFloatProperty* FloatProperty;                  // Cached property pointer
    FOnTimelineFloatStatic InterpFuncStatic;        // Static delegate version
};
```

#### Vector Track Structure
```cpp
USTRUCT()
struct FTimelineVectorTrack
{
    UPROPERTY()
    TObjectPtr<class UCurveVector> VectorCurve;     // Vector curve asset
    
    UPROPERTY()
    FOnTimelineVector InterpFunc;                   // Delegate callback
    
    UPROPERTY()
    FName TrackName;                                // Track identifier
    
    UPROPERTY()
    FName VectorPropertyName;                       // Target property name
    
    // Runtime cached data  
    FStructProperty* VectorProperty;                // Cached property pointer
    FOnTimelineVectorStatic InterpFuncStatic;       // Static delegate version
};
```

#### Linear Color Track Structure
```cpp
USTRUCT()
struct FTimelineLinearColorTrack
{
    UPROPERTY()
    TObjectPtr<class UCurveLinearColor> LinearColorCurve;  // Color curve asset
    
    UPROPERTY()
    FOnTimelineLinearColor InterpFunc;              // Delegate callback
    
    UPROPERTY()
    FName TrackName;                                // Track identifier
    
    UPROPERTY()
    FName LinearColorPropertyName;                  // Target property name
    
    // Runtime cached data
    FStructProperty* LinearColorProperty;           // Cached property pointer  
    FOnTimelineLinearColorStatic InterpFuncStatic; // Static delegate version
};
```

#### Event Track Structure
```cpp
USTRUCT()
struct FTimelineEventEntry
{
    UPROPERTY()
    float Time;                                     // Event trigger time
    
    UPROPERTY()
    FOnTimelineEvent EventFunc;                     // Event delegate
};
```

### Timeline Track Collections
```cpp
// Inside FTimeline
UPROPERTY(NotReplicated)
TArray<struct FTimelineEventEntry> Events;         // Timed events

UPROPERTY(NotReplicated)
TArray<struct FTimelineVectorTrack> InterpVectors; // Vector interpolations

UPROPERTY(NotReplicated)
TArray<struct FTimelineFloatTrack> InterpFloats;   // Float interpolations

UPROPERTY(NotReplicated)
TArray<struct FTimelineLinearColorTrack> InterpLinearColors;  // Color interpolations
```

### Timeline Delegate System
```cpp
// Timeline lifecycle delegates
UPROPERTY(NotReplicated)
FOnTimelineEvent TimelinePostUpdateFunc;           // Called each tick

UPROPERTY(NotReplicated) 
FOnTimelineEvent TimelineFinishedFunc;             // Called on completion

// Property binding target
UPROPERTY(NotReplicated)
TWeakObjectPtr<UObject> PropertySetObject;         // Object to update properties on

UPROPERTY(NotReplicated)
FName DirectionPropertyName;                       // Property for timeline direction
```

## Blueprint to C++ Conversion Requirements

### 1. Timeline Declaration and Setup
**C++ Class Member Declaration**:
```cpp
// Header file - component declaration
UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = Timeline)
class UTimelineComponent* TimelineComponent;

// Property binding target (if used)
UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = Timeline)
float InterpolatedValue;

UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = Timeline)
FVector InterpolatedVector;

UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = Timeline)
FLinearColor InterpolatedColor;
```

**Constructor Implementation**:
```cpp
AMyActor::AMyActor()
{
    // Timeline component creation
    TimelineComponent = CreateDefaultSubobject<UTimelineComponent>(TEXT("TimelineComponent"));
    
    // Timeline configuration from Blueprint data
    TimelineComponent->SetTimelineLength(5.0f);          // From Timeline.Length
    TimelineComponent->SetLooping(true);                 // From Timeline.bLooping
    TimelineComponent->SetIgnoreTimeDilation(false);     // From bIgnoreTimeDilation
    
    // Setup property binding target
    TimelineComponent->SetPropertySetObject(this);
}
```

### 2. Curve Asset Integration
**Curve References**:
```cpp
// Header - curve asset references
UPROPERTY(EditDefaultsOnly, Category = Timeline)
class UCurveFloat* FloatCurve;

UPROPERTY(EditDefaultsOnly, Category = Timeline)
class UCurveVector* VectorCurve;

UPROPERTY(EditDefaultsOnly, Category = Timeline)  
class UCurveLinearColor* ColorCurve;

// Constructor - curve loading
FloatCurve = LoadObject<UCurveFloat>(nullptr, TEXT("/Game/Curves/MyFloatCurve.MyFloatCurve"));
VectorCurve = LoadObject<UCurveVector>(nullptr, TEXT("/Game/Curves/MyVectorCurve.MyVectorCurve"));
ColorCurve = LoadObject<UCurveLinearColor>(nullptr, TEXT("/Game/Curves/MyColorCurve.MyColorCurve"));
```

### 3. Track Setup and Delegates
**BeginPlay Implementation**:
```cpp
void AMyActor::BeginPlay()
{
    Super::BeginPlay();
    
    // Float track setup
    if (FloatCurve)
    {
        FOnTimelineFloat FloatDelegate;
        FloatDelegate.BindDynamic(this, &AMyActor::OnFloatUpdate);
        TimelineComponent->AddInterpFloat(FloatCurve, FloatDelegate, TEXT("InterpolatedValue"), TEXT("FloatTrack"));
    }
    
    // Vector track setup  
    if (VectorCurve)
    {
        FOnTimelineVector VectorDelegate;
        VectorDelegate.BindDynamic(this, &AMyActor::OnVectorUpdate);
        TimelineComponent->AddInterpVector(VectorCurve, VectorDelegate, TEXT("InterpolatedVector"), TEXT("VectorTrack"));
    }
    
    // Color track setup
    if (ColorCurve)
    {
        FOnTimelineLinearColor ColorDelegate;
        ColorDelegate.BindDynamic(this, &AMyActor::OnColorUpdate);
        TimelineComponent->AddInterpLinearColor(ColorCurve, ColorDelegate, TEXT("InterpolatedColor"), TEXT("ColorTrack"));
    }
    
    // Event setup
    FOnTimelineEvent EventDelegate;
    EventDelegate.BindDynamic(this, &AMyActor::OnTimelineEvent);
    TimelineComponent->AddEvent(2.5f, EventDelegate);  // Event at 2.5 seconds
    
    // Lifecycle delegates
    FOnTimelineEvent UpdateDelegate;
    UpdateDelegate.BindDynamic(this, &AMyActor::OnTimelineUpdate);
    TimelineComponent->SetTimelinePostUpdateFunc(UpdateDelegate);
    
    FOnTimelineEvent FinishedDelegate;
    FinishedDelegate.BindDynamic(this, &AMyActor::OnTimelineFinished);
    TimelineComponent->SetTimelineFinishedFunc(FinishedDelegate);
    
    // Direction property setup
    TimelineComponent->SetDirectionPropertyName(TEXT("TimelineDirection"));
}
```

### 4. Delegate Function Implementations
**Update Callbacks**:
```cpp
// Float track callback
UFUNCTION()
void OnFloatUpdate(float Value)
{
    InterpolatedValue = Value;
    // Additional logic based on Blueprint implementation
}

// Vector track callback
UFUNCTION()
void OnVectorUpdate(FVector Value)
{
    InterpolatedVector = Value;
    // Additional logic based on Blueprint implementation
}

// Color track callback
UFUNCTION()
void OnColorUpdate(FLinearColor Value)
{
    InterpolatedColor = Value;
    // Additional logic based on Blueprint implementation
}

// Event callback
UFUNCTION()
void OnTimelineEvent()
{
    // Event-specific logic from Blueprint
}

// Lifecycle callbacks
UFUNCTION()
void OnTimelineUpdate()
{
    // Per-tick update logic
}

UFUNCTION()
void OnTimelineFinished()
{
    // Completion logic
}
```

### 5. Timeline Control Interface
**Control Functions**:
```cpp
// Timeline control functions - map directly to Blueprint pins
void PlayTimeline()
{
    if (TimelineComponent)
    {
        TimelineComponent->Play();
    }
}

void PlayTimelineFromStart()
{
    if (TimelineComponent)
    {
        TimelineComponent->PlayFromStart();
    }
}

void StopTimeline()
{
    if (TimelineComponent)
    {
        TimelineComponent->Stop();
    }
}

void ReverseTimeline()
{
    if (TimelineComponent)
    {
        TimelineComponent->Reverse();
    }
}

void SetTimelinePosition(float NewPosition)
{
    if (TimelineComponent)
    {
        TimelineComponent->SetPlaybackPosition(NewPosition, true, true);
    }
}
```

## Blueprint JSON Processing

### Expected JSON Structure
```json
{
  "TimelineComponent": {
    "TimelineName": "MyTimeline",
    "Length": 5.0,
    "bLoop": true,
    "bAutoPlay": false,
    "bIgnoreTimeDilation": false,
    "FloatTracks": [
      {
        "TrackName": "FloatTrack",
        "CurveAsset": "/Game/Curves/MyFloatCurve.MyFloatCurve",
        "PropertyName": "InterpolatedValue",
        "InterpFunction": "OnFloatUpdate"
      }
    ],
    "VectorTracks": [
      {
        "TrackName": "VectorTrack", 
        "CurveAsset": "/Game/Curves/MyVectorCurve.MyVectorCurve",
        "PropertyName": "InterpolatedVector",
        "InterpFunction": "OnVectorUpdate"
      }
    ],
    "LinearColorTracks": [
      {
        "TrackName": "ColorTrack",
        "CurveAsset": "/Game/Curves/MyColorCurve.MyColorCurve", 
        "PropertyName": "InterpolatedColor",
        "InterpFunction": "OnColorUpdate"
      }
    ],
    "Events": [
      {
        "Time": 2.5,
        "EventFunction": "OnTimelineEvent"
      }
    ],
    "UpdateFunction": "OnTimelineUpdate",
    "FinishedFunction": "OnTimelineFinished"
  }
}
```

### Generated C++ Structure
```cpp
// Generated class with timeline integration
UCLASS()
class MYPROJECT_API AMyActor : public AActor
{
    GENERATED_BODY()
    
public:
    AMyActor();
    
protected:
    virtual void BeginPlay() override;
    
    // Timeline component
    UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = Timeline)
    class UTimelineComponent* TimelineComponent;
    
    // Curve assets
    UPROPERTY(EditDefaultsOnly, Category = Timeline)
    class UCurveFloat* FloatCurve;
    
    // Interpolated properties
    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = Timeline)
    float InterpolatedValue;
    
    // Delegate functions
    UFUNCTION()
    void OnFloatUpdate(float Value);
    
    UFUNCTION()
    void OnTimelineEvent();
    
    // Control interface
    UFUNCTION(BlueprintCallable, Category = Timeline)
    void PlayTimeline();
};
```

## Runtime Execution Flow

### 1. Component Initialization
1. Create UTimelineComponent in constructor
2. Set basic timeline properties (length, looping, etc.)
3. Load curve assets via asset references

### 2. BeginPlay Setup
1. Create and bind delegate functions for tracks
2. Add interpolation tracks to timeline
3. Setup event triggers
4. Configure lifecycle callbacks

### 3. Runtime Execution
1. Timeline ticks each frame when playing
2. Evaluates curves at current position
3. Calls delegate functions with interpolated values
4. Triggers events at specified times
5. Updates bound properties if PropertySetObject set

### 4. Property Binding (Optional)
1. Timeline can directly update object properties
2. Uses reflection to set property values
3. Bypasses delegate system for simple cases

## Critical Conversion Considerations

### Delegate Binding Requirements
- All delegate functions must be UFUNCTION()
- Function signatures must match timeline delegate types
- Dynamic binding requires UCLASS reflection system

### Curve Asset Management
- Curve assets must be loaded properly in constructor or BeginPlay
- Asset paths must be resolved from Blueprint curve references
- Handle missing curve assets gracefully

### Property Reflection
- PropertySetObject and property name strings use reflection
- Property names must match actual UPROPERTY names exactly
- Type safety enforced at runtime, not compile time

### Replication Considerations
- Timeline component supports replication (bReplicated)
- Replicated timelines sync position and state across clients
- Non-replicated tracks marked with NotReplicated

## Blueprint Node Integration

### K2Node_Timeline Properties
```cpp
// From K2Node_Timeline.h analysis
FName TimelineName;              // Timeline variable name
FGuid TimelineGuid;              // Unique timeline identifier  
uint32 bAutoPlay:1;              // Auto-play on BeginPlay
uint32 bLoop:1;                  // Loop timeline
uint32 bReplicated:1;            // Network replication
uint32 bIgnoreTimeDilation:1;    // Time dilation immunity
```

### Blueprint Pin Mapping
- **Play**: Maps to `PlayTimeline()` function call
- **PlayFromStart**: Maps to `PlayTimelineFromStart()` function call
- **Stop**: Maps to `StopTimeline()` function call
- **Reverse**: Maps to `ReverseTimeline()` function call
- **SetNewTime**: Maps to `SetTimelinePosition()` function call
- **Update**: Maps to update delegate binding
- **Finished**: Maps to finished delegate binding
- **Direction**: Maps to direction property binding
- **Track Outputs**: Maps to individual track delegate bindings

## Performance Considerations

### Tick Management
- Timeline components automatically manage tick registration
- Only tick when actively playing
- Respect component activation state

### Curve Evaluation
- Curve evaluation can be expensive for complex curves
- Cache curve data when possible
- Consider curve compression for runtime optimization

### Delegate Overhead
- Dynamic delegates have performance cost
- Static delegates more efficient but lose Blueprint compatibility
- Balance between performance and functionality

## Related Systems
- **TimelineTemplate**: Design-time timeline asset structure
- **Curve Assets**: UCurveFloat, UCurveVector, UCurveLinearColor
- **K2Node_Timeline**: Blueprint node representation
- **Component System**: ActorComponent base functionality

## Conversion Priority: HIGH
Timeline components are commonly used in Blueprints for animation and must be fully supported in the initial conversion implementation. The delegate system is critical for maintaining Blueprint behavior compatibility.