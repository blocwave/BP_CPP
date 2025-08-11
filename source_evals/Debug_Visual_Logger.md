# Debug_Visual_Logger.md

## Overview
Analysis of Unreal Engine 5.2.1's Visual Logger system for Blueprint to C++ conversion, focusing on how visual debugging information is preserved and translated in generated C++ code.

## Source Files Analyzed
- `/Engine/Source/Runtime/Engine/Public/VisualLogger/VisualLogger.h` - Main visual logger interface
- `/Engine/Source/Runtime/Engine/Classes/Debug/DebugDrawService.h` - Debug drawing service
- `/Engine/Source/Runtime/Engine/Classes/Engine/DebugCameraController.h` - Debug camera system

## Visual Logger System Architecture

### 1. Core Visual Logger Class
```cpp
class ENGINE_API FVisualLogger : public FOutputDevice
{
public:
    // Main logging functions
    static void CategorizedLogf(const UObject* LogOwner, const FLogCategoryBase& Category, 
                               ELogVerbosity::Type Verbosity, const TCHAR* Fmt, ...);
    
    // Geometric shape logging
    static void GeometryShapeLogf(const UObject* LogOwner, const FName& CategoryName, 
                                 ELogVerbosity::Type Verbosity, const FVector& Start, 
                                 const FVector& End, const FColor& Color, ...);
    
    // Navigation debug data
    static void NavigationDataDump(const UObject* LogOwner, const FLogCategoryBase& Category, 
                                  const ELogVerbosity::Type Verbosity, const FBox& Box);
    
    // Event logging
    static void EventLog(const UObject* LogOwner, const FName EventTag1, 
                        const FVisualLogEventBase& Event1, ...);
};
```

### 2. Debug Draw Service
```cpp
UCLASS(config=Engine)
class ENGINE_API UDebugDrawService : public UBlueprintFunctionLibrary
{
    GENERATED_UCLASS_BODY()

public:
    // Registration system for debug delegates
    static FDelegateHandle Register(const TCHAR* Name, const FDebugDrawDelegate& NewDelegate);
    static void Unregister(FDelegateHandle HandleToRemove);

    // Drawing functions
    static void Draw(const FEngineShowFlags Flags, class UCanvas* Canvas);
    static void Draw(const FEngineShowFlags Flags, class FViewport* Viewport, 
                    FSceneView* View, FCanvas* Canvas, class UCanvas* CanvasObject = nullptr);

private:
    static TArray<TArray<FDebugDrawDelegate>> Delegates;
    static FEngineShowFlags ObservedFlags;
};
```

## Visual Logging Macros

### 1. Conditional Compilation
```cpp
#if ENABLE_VISUAL_LOG
    // Full visual logging support
    #define UE_VLOG(LogOwner, CategoryName, Verbosity, Format, ...) \
        if(FVisualLogger::IsRecording()) \
            FVisualLogger::CategorizedLogf(LogOwner, CategoryName, ELogVerbosity::Verbosity, Format, ##__VA_ARGS__)

    #define UE_VLOG_SEGMENT(LogOwner, CategoryName, Verbosity, SegmentStart, SegmentEnd, Color, Format, ...) \
        if(FVisualLogger::IsRecording()) \
            FVisualLogger::GeometryShapeLogf(LogOwner, CategoryName, ELogVerbosity::Verbosity, SegmentStart, SegmentEnd, Color, 0, Format, ##__VA_ARGS__)

    #define UE_VLOG_LOCATION(LogOwner, CategoryName, Verbosity, Location, Radius, Color, Format, ...) \
        if(FVisualLogger::IsRecording()) \
            FVisualLogger::GeometryShapeLogf(LogOwner, CategoryName, ELogVerbosity::Verbosity, Location, Radius, Color, Format, ##__VA_ARGS__)
#else
    // Disabled - no overhead
    #define UE_VLOG(Actor, CategoryName, Verbosity, Format, ...)
    #define UE_VLOG_SEGMENT(Actor, CategoryName, Verbosity, SegmentStart, SegmentEnd, Color, DescriptionFormat, ...)
    #define UE_VLOG_LOCATION(Actor, CategoryName, Verbosity, Location, Radius, Color, DescriptionFormat, ...)
#endif
```

### 2. Geometric Shape Logging
```cpp
// Box shapes
#define UE_VLOG_BOX(LogOwner, CategoryName, Verbosity, Box, Color, Format, ...) \
    if(FVisualLogger::IsRecording()) \
        FVisualLogger::GeometryBoxLogf(LogOwner, CategoryName, ELogVerbosity::Verbosity, Box, FMatrix::Identity, Color, Format, ##__VA_ARGS__)

// Cone shapes
#define UE_VLOG_CONE(LogOwner, CategoryName, Verbosity, Origin, Direction, Length, Angle, Color, Format, ...) \
    if(FVisualLogger::IsRecording()) \
        FVisualLogger::GeometryShapeLogf(LogOwner, CategoryName, ELogVerbosity::Verbosity, Origin, Direction, Length, Angle, Color, Format, ##__VA_ARGS__)

// Arrow shapes
#define UE_VLOG_ARROW(LogOwner, CategoryName, Verbosity, SegmentStart, SegmentEnd, Color, Format, ...) \
    if(FVisualLogger::IsRecording()) \
        FVisualLogger::ArrowLogf(LogOwner, CategoryName, ELogVerbosity::Verbosity, SegmentStart, SegmentEnd, Color, Format, ##__VA_ARGS__)
```

## Blueprint Node Visual Debug Conversion

### 1. Blueprint Visual Debug Nodes
Blueprint editor provides visual debug nodes that must be converted to C++ visual logger calls:

#### Debug Draw Line
```cpp
// Blueprint: "Draw Debug Line"
// Generated C++ equivalent:
void UGeneratedClass::DrawDebugLine_Implementation()
{
    #if ENABLE_VISUAL_LOG
    UE_VLOG_SEGMENT(this, LogBlueprint, Log, StartLocation, EndLocation, 
                    LineColor.ToFColor(true), TEXT("Debug Line from Blueprint"));
    #endif
    
    // Also draw in world for immediate visibility
    #if !UE_BUILD_SHIPPING
    UKismetSystemLibrary::DrawDebugLine(GetWorld(), StartLocation, EndLocation, 
                                       LineColor, Duration, Thickness);
    #endif
}
```

#### Debug Draw String
```cpp
// Blueprint: "Draw Debug String"
// Generated C++ equivalent:
void UGeneratedClass::DrawDebugString_Implementation()
{
    #if ENABLE_VISUAL_LOG
    UE_VLOG_LOCATION(this, LogBlueprint, Log, WorldLocation, 0.0f, 
                     TextColor.ToFColor(true), TEXT("%s"), *DebugText);
    #endif
    
    #if !UE_BUILD_SHIPPING
    UKismetSystemLibrary::DrawDebugString(GetWorld(), WorldLocation, DebugText, 
                                         nullptr, TextColor, Duration);
    #endif
}
```

### 2. Automatic Visual Logging Integration
```cpp
// Generated function with automatic visual logging
void UGeneratedClass::GeneratedFunction()
{
    // Log function entry
    UE_VLOG(this, LogBlueprint, Log, TEXT("Entering function: %s"), TEXT("GeneratedFunction"));
    
    // Log variable changes with visual representation
    if (SomeImportantVariable != PreviousValue)
    {
        UE_VLOG_LOCATION(this, LogBlueprint, Log, GetActorLocation(), 50.0f, 
                        FColor::Yellow, TEXT("Variable changed: %s = %d"), 
                        TEXT("SomeImportantVariable"), SomeImportantVariable);
    }
    
    // Log function exit
    UE_VLOG(this, LogBlueprint, Log, TEXT("Exiting function: %s"), TEXT("GeneratedFunction"));
}
```

## Debug Camera Controller Integration

### 1. Debug Camera Features
```cpp
class ENGINE_API ADebugCameraController : public APlayerController
{
public:
    // Camera positioning and movement
    UPROPERTY(globalconfig)
    uint32 bShowSelectedInfo:1;
    
    UPROPERTY()
    uint32 bIsFrozenRendering:1;
    
    // Object selection and inspection
    UPROPERTY()
    TWeakObjectPtr<class AActor> SelectedActor;
    
    UPROPERTY()
    TWeakObjectPtr<class UPrimitiveComponent> SelectedComponent;
    
    // Debug visualization controls
    void ToggleDisplay();
    void CycleViewMode();
    void ToggleBufferVisualizationOverviewMode();
};
```

### 2. Generated C++ Debug Camera Support
```cpp
// Add debug camera support to generated classes
#if !UE_BUILD_SHIPPING
void UGeneratedClass::OnDebugCameraSelected()
{
    // Log when this object is selected by debug camera
    UE_VLOG(this, LogBlueprint, Log, TEXT("Object selected by debug camera"));
    
    // Provide additional debug information
    UE_VLOG_BOX(this, LogBlueprint, Log, GetActorBounds(false), FColor::Green,
                TEXT("Bounds of selected Blueprint actor"));
    
    // Log important state variables
    LogDebugInformation();
}
#endif
```

## Visual Debug Data Preservation

### 1. Blueprint Visual Debug Metadata
```cpp
struct FBlueprintVisualDebugInfo
{
    FGuid NodeGuid;                    // Original Blueprint node
    EBlueprintVisualDebugType Type;    // Type of visual debug (Line, String, etc.)
    FTransform DebugTransform;         // World position/rotation
    FLinearColor DebugColor;           // Display color
    FString DebugText;                 // Associated text
    float Duration;                    // Display duration
    float Thickness;                   // Line/shape thickness
    
    // Conversion to C++ visual logger calls
    FString GenerateCppCode() const;
};
```

### 2. Runtime Visual Debug Registry
```cpp
class FBlueprintVisualDebugRegistry
{
public:
    // Register Blueprint visual debug elements
    void RegisterVisualDebugElement(const FGuid& NodeGuid, const FBlueprintVisualDebugInfo& DebugInfo);
    
    // Convert Blueprint visual debug to C++ calls
    void GenerateVisualDebugCppCode(const UBlueprint* Blueprint, FString& OutCppCode);
    
    // Runtime visual debug rendering
    void RenderVisualDebugElements(UWorld* World, const UObject* Owner);
    
private:
    TMap<FGuid, FBlueprintVisualDebugInfo> VisualDebugElements;
};
```

## Performance Considerations

### 1. Compilation Flags
```cpp
// Visual logging performance optimization
#if ENABLE_VISUAL_LOG
    #define BLUEPRINT_VLOG_OVERHEAD_CHECK() \
        if (!FVisualLogger::IsRecording()) return;
#else
    #define BLUEPRINT_VLOG_OVERHEAD_CHECK()
#endif

// Usage in generated code
void UGeneratedClass::ExpensiveDebugFunction()
{
    BLUEPRINT_VLOG_OVERHEAD_CHECK();
    
    // Expensive visual logging operations only when recording
    UE_VLOG_COMPLEX_OPERATION(this, LogBlueprint, Log, GetComplexDebugData());
}
```

### 2. Conditional Debug Code Generation
```cpp
// Generate different code based on build configuration
#if WITH_EDITOR
    // Full debug support in editor builds
    void UGeneratedClass::FullDebugSupport()
    {
        UE_VLOG(this, LogBlueprint, VeryVerbose, TEXT("Detailed debug information"));
        DrawDebugVisualizations();
        LogAllVariableStates();
    }
#elif !UE_BUILD_SHIPPING
    // Limited debug support in development builds
    void UGeneratedClass::FullDebugSupport()
    {
        UE_VLOG(this, LogBlueprint, Log, TEXT("Basic debug information"));
    }
#else
    // No debug overhead in shipping builds
    void UGeneratedClass::FullDebugSupport()
    {
        // Empty implementation
    }
#endif
```

## Integration with Blueprint Profiling

### 1. Performance Instrumentation
```cpp
// Generated code with performance logging
void UGeneratedClass::PerformanceCriticalFunction()
{
    SCOPE_CYCLE_COUNTER(STAT_Blueprint_PerformanceCriticalFunction);
    
    #if ENABLE_VISUAL_LOG
    double StartTime = FPlatformTime::Seconds();
    #endif
    
    // Function implementation
    DoPerformanceCriticalWork();
    
    #if ENABLE_VISUAL_LOG
    double EndTime = FPlatformTime::Seconds();
    UE_VLOG_HISTOGRAM(this, LogBlueprintPerf, Log, TEXT("Performance"), 
                     TEXT("FunctionExecutionTime"), 
                     FVector2D(StartTime, EndTime - StartTime));
    #endif
}
```

### 2. Memory Usage Tracking
```cpp
// Visual logging for memory allocations
void UGeneratedClass::MemoryIntensiveOperation()
{
    #if ENABLE_VISUAL_LOG
    SIZE_T InitialMemory = FPlatformMemory::GetStats().UsedPhysical;
    #endif
    
    // Memory intensive operations
    PerformMemoryIntensiveWork();
    
    #if ENABLE_VISUAL_LOG
    SIZE_T FinalMemory = FPlatformMemory::GetStats().UsedPhysical;
    UE_VLOG(this, LogBlueprintMemory, Log, TEXT("Memory delta: %d bytes"), 
            (int32)(FinalMemory - InitialMemory));
    #endif
}
```

## Critical Requirements for BP-to-C++ Conversion

### 1. Visual Debug Preservation
- All Blueprint visual debug nodes must translate to equivalent C++ visual logger calls
- Debug information must be visible in both runtime and Visual Logger tool
- Performance characteristics must be preserved (conditional compilation)

### 2. Debug Camera Integration
- Generated C++ classes must support debug camera inspection
- Object selection and property visualization must work seamlessly
- Debug information must be contextually relevant and helpful

### 3. Cross-Platform Compatibility
- Visual logging must work across all supported platforms
- Platform-specific optimizations should be applied automatically
- Fallback mechanisms for platforms with limited debug support

## Implementation Recommendations

### 1. Automated Code Generation
```cpp
// Template for generating visual debug code
class FVisualDebugCodeGenerator
{
public:
    static FString GenerateVisualLogCall(const FBlueprintVisualDebugInfo& DebugInfo);
    static FString GenerateDebugDrawCall(const FBlueprintVisualDebugInfo& DebugInfo);
    static FString GenerateConditionalWrapper(const FString& DebugCode);
};
```

### 2. Runtime Debug Configuration
```cpp
// Allow runtime configuration of debug verbosity
UCLASS(Config=Engine)
class UBlueprintDebugSettings : public UObject
{
    GENERATED_BODY()
    
public:
    UPROPERTY(Config)
    bool bEnableVisualLogging = true;
    
    UPROPERTY(Config)
    ELogVerbosity::Type DefaultBlueprintLogVerbosity = ELogVerbosity::Log;
    
    UPROPERTY(Config)
    TArray<FString> EnabledDebugCategories;
};
```

## Conclusion
The Visual Logger system provides comprehensive runtime debugging capabilities that must be preserved and enhanced in the Blueprint to C++ conversion process. Proper integration ensures developers retain full debugging capabilities while benefiting from C++ performance improvements.