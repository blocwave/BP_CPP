# Automation Testing Framework for Blueprint Validation

## Overview
The AutomationCommon system provides testing infrastructure for validating Blueprint functionality, particularly important for ensuring C++ conversion accuracy and correctness. This framework enables automated testing of Blueprint-to-C++ conversion processes.

## Core Testing Infrastructure

### AutomationCommon Namespace
```cpp
namespace AutomationCommon {
#if WITH_AUTOMATION_TESTS

    // Rendering and screenshot utilities
    FString GetRenderDetailsString();
    FString GetScreenshotName(const FString& TestName);
    FString GetLocalPathForScreenshot(const FString& InScreenshotName);
    
    // Screenshot data building
    FAutomationScreenshotData BuildScreenshotData(
        const FString& MapOrContext, 
        const FString& TestName, 
        const FString& ScreenShotName, 
        int32 Width, 
        int32 Height
    );
    
    // Frame tracing for analysis
    TArray<uint8> CaptureFrameTrace(const FString& MapOrContext, const FString& TestName);
    
    // Widget finding for UI tests
    SWidget* FindWidgetByTag(const FName Tag);
    
    // World access for testing
    UWorld* GetAnyGameWorld();

#endif
}
```

### Event Delegates for Testing
```cpp
// Map loading notifications for testing
DECLARE_MULTICAST_DELEGATE_ThreeParams(FOnEditorAutomationMapLoad, const FString&, bool, FString*);

extern FOnEditorAutomationMapLoad OnEditorAutomationMapLoad;
static FOnEditorAutomationMapLoad& OnEditorAutomationMapLoadDelegate();
```

## Latent Automation Commands

### Basic Timing Commands
```cpp
// Wait for specified duration
DEFINE_ENGINE_LATENT_AUTOMATION_COMMAND_ONE_PARAMETER(FWaitLatentCommand, float, Duration);

// Engine-specific wait
DEFINE_ENGINE_LATENT_AUTOMATION_COMMAND_ONE_PARAMETER(FEngineWaitLatentCommand, float, Duration);

// Stream all resources and wait
DEFINE_ENGINE_LATENT_AUTOMATION_COMMAND_ONE_PARAMETER(FStreamAllResourcesLatentCommand, float, Duration);
```

### Map and World Commands
```cpp
// Load map in game
DEFINE_ENGINE_LATENT_AUTOMATION_COMMAND_ONE_PARAMETER(FLoadGameMapCommand, FString, MapName);

// Wait for map loading completion
DEFINE_ENGINE_LATENT_AUTOMATION_COMMAND(FWaitForMapToLoadCommand);
DEFINE_ENGINE_LATENT_AUTOMATION_COMMAND_ONE_PARAMETER(FWaitForSpecifiedMapToLoadCommand, FString, MapName);

// Exit game
DEFINE_ENGINE_LATENT_AUTOMATION_COMMAND(FExitGameCommand);
DEFINE_ENGINE_LATENT_AUTOMATION_COMMAND(FRequestExitCommand);
```

### Command Execution
```cpp
// Execute console commands
DEFINE_ENGINE_LATENT_AUTOMATION_COMMAND_ONE_PARAMETER(FExecStringLatentCommand, FString, ExecCommand);

// Execute world-specific commands
DEFINE_ENGINE_LATENT_AUTOMATION_COMMAND_ONE_PARAMETER(FExecWorldStringLatentCommand, FString, ExecCommand);
```

### Screenshot Commands
```cpp
// Screenshot parameters
struct WindowScreenshotParameters {
    FString ScreenshotName;
    TSharedPtr<SWindow> CurrentWindow;
};

// Take screenshots
DEFINE_ENGINE_LATENT_AUTOMATION_COMMAND_ONE_PARAMETER(FTakeActiveEditorScreenshotCommand, FString, ScreenshotName);
DEFINE_ENGINE_LATENT_AUTOMATION_COMMAND_ONE_PARAMETER(FTakeEditorScreenshotCommand, WindowScreenshotParameters, ScreenshotParameters);
```

### Rendering and Performance Commands
```cpp
// Wait for shaders to compile
DEFINE_ENGINE_LATENT_AUTOMATION_COMMAND(FWaitForShadersToFinishCompilingInGame);
```

## Advanced Performance Testing

### FWaitForInteractiveFrameRate
Specialized command for performance validation:
```cpp
class ENGINE_API FWaitForInteractiveFrameRate : public IAutomationLatentCommand {
public:
    FWaitForInteractiveFrameRate(
        float InDesiredFrameRate = 0, 
        float InDuration = 0, 
        float InMaxWaitTime = 0
    );
    
    bool Update() override;

private:
    // Configuration
    float DesiredFrameRate;  // Target framerate
    float Duration;          // How long to maintain framerate
    float MaxWaitTime;       // Maximum wait time
    
    // State tracking
    double StartTimeOfWait;
    double StartTimeOfAcceptableFrameRate;
    double LastReportTime;
    double LastTickTime;
    
    // Performance measurement
    TArray<double> RollingTickRateBuffer;
    int32 BufferIndex;
    
    // Constants
    const double kTickRate = 60.0;
    const int kSampleCount = (int32)kTickRate * 5;
    
    // Utility methods
    void AddTickRateSample(const double Value);
    double CurrentAverageTickRate() const;
};
```

### Performance Validation Usage
```cpp
// Wait for stable 60 FPS for 2 seconds, max wait 30 seconds
FWaitForInteractiveFrameRate WaitForPerformance(60.0f, 2.0f, 30.0f);
```

## Image Comparison and Validation

### Screenshot Comparison
```cpp
// Request image comparison for visual validation
void RequestImageComparison(
    const FString& InImageName, 
    int32 InWidth, 
    int32 InHeight, 
    const TArray<FColor>& InImageData, 
    EAutomationComparisonToleranceLevel InTolerance = EAutomationComparisonToleranceLevel::Low, 
    const FString& InContext = TEXT(""), 
    const FString& InNotes = TEXT("")
);
```

### Tolerance Levels
```cpp
enum class EAutomationComparisonToleranceLevel {
    Zero,    // Exact match required
    Low,     // Small differences allowed
    Medium,  // Moderate differences allowed  
    High     // Large differences allowed
};
```

## Testing Utilities for Map Operations

### Map Loading and Validation
```cpp
// Open map with optional force reload
bool AutomationOpenMap(const FString& MapName, bool bForceReload = false);
```

Map loading process:
1. **Editor Mode**: Opens map and starts PIE (Play In Editor)
2. **Game Mode**: Transitions to map and waits for load
3. **Validation**: Ensures map loads successfully
4. **Event Notification**: Triggers map load events for listeners

## Blueprint-Specific Testing Applications

### Blueprint Validation Testing
Integration points for Blueprint validation:

```cpp
// Example Blueprint validation test structure
class FBlueprintValidationTest : public FAutomationTestBase {
public:
    // Test Blueprint loading
    bool LoadBlueprintTest(const FString& BlueprintPath);
    
    // Test Blueprint compilation
    bool CompileBlueprintTest(UBlueprint* Blueprint);
    
    // Test Blueprint execution
    bool ExecuteBlueprintTest(UBlueprint* Blueprint);
    
    // Test Blueprint-to-C++ conversion
    bool ConvertBlueprintTest(UBlueprint* Blueprint);
};
```

### Conversion Validation Framework
```cpp
// Framework for validating Blueprint-to-C++ conversion
class FBlueprintConversionValidation {
public:
    // Validate conversion accuracy
    static bool ValidateConversion(
        const UBlueprint* OriginalBlueprint,
        const FString& GeneratedCppCode,
        const FString& TestContext
    );
    
    // Compare Blueprint and C++ execution results
    static bool CompareExecutionResults(
        const UBlueprint* Blueprint,
        const FString& CppImplementation,
        const TArray<FAutomationTestParameter>& TestParameters
    );
    
    // Performance comparison
    static bool ComparePerformance(
        const UBlueprint* Blueprint,
        const FString& CppImplementation,
        int32 IterationCount = 1000
    );
};
```

## Testing Integration Patterns

### Blueprint Loading Tests
```cpp
// Test Blueprint asset loading
IMPLEMENT_SIMPLE_AUTOMATION_TEST(FBlueprintLoadingTest, 
    "Blueprint.Loading.BasicLoad", 
    EAutomationTestFlags::ApplicationContextMask | EAutomationTestFlags::ProductFilter)

bool FBlueprintLoadingTest::RunTest(const FString& Parameters) {
    // Load Blueprint asset
    UBlueprint* Blueprint = LoadObject<UBlueprint>(nullptr, *Parameters);
    
    // Validate loading
    TestNotNull("Blueprint loaded successfully", Blueprint);
    
    // Validate Blueprint structure
    TestTrue("Blueprint has valid class", Blueprint->GeneratedClass != nullptr);
    TestTrue("Blueprint has graphs", Blueprint->UbergraphPages.Num() > 0);
    
    return true;
}
```

### Compilation Validation Tests
```cpp
// Test Blueprint compilation
IMPLEMENT_SIMPLE_AUTOMATION_TEST(FBlueprintCompilationTest, 
    "Blueprint.Compilation.Validation", 
    EAutomationTestFlags::ApplicationContextMask | EAutomationTestFlags::ProductFilter)

bool FBlueprintCompilationTest::RunTest(const FString& Parameters) {
    // Load and compile Blueprint
    UBlueprint* Blueprint = LoadObject<UBlueprint>(nullptr, *Parameters);
    FKismetEditorUtilities::CompileBlueprint(Blueprint);
    
    // Validate compilation results
    TestTrue("Blueprint compiled without errors", Blueprint->Status == BS_UpToDate);
    TestTrue("Generated class is valid", Blueprint->GeneratedClass != nullptr);
    
    return true;
}
```

### Conversion Accuracy Tests
```cpp
// Test Blueprint-to-C++ conversion accuracy
IMPLEMENT_COMPLEX_AUTOMATION_TEST(FBlueprintConversionTest, 
    "Blueprint.Conversion.Accuracy", 
    EAutomationTestFlags::ApplicationContextMask | EAutomationTestFlags::ProductFilter)

void FBlueprintConversionTest::GetTests(TArray<FString>& OutBeautifiedNames, 
                                      TArray<FString>& OutTestCommands) const {
    // Add test cases for different Blueprint types
    OutBeautifiedNames.Add(TEXT("Function Blueprint"));
    OutTestCommands.Add(TEXT("/Game/Tests/FunctionBlueprint.FunctionBlueprint"));
    
    OutBeautifiedNames.Add(TEXT("Event Blueprint"));
    OutTestCommands.Add(TEXT("/Game/Tests/EventBlueprint.EventBlueprint"));
}

bool FBlueprintConversionTest::RunTest(const FString& Parameters) {
    // Load Blueprint
    UBlueprint* Blueprint = LoadObject<UBlueprint>(nullptr, *Parameters);
    
    // Convert to C++
    FString GeneratedCode = ConvertBlueprintToCpp(Blueprint);
    
    // Validate conversion
    TestFalse("Generated code is not empty", GeneratedCode.IsEmpty());
    TestTrue("Generated code compiles", ValidateCppCompilation(GeneratedCode));
    TestTrue("Execution results match", CompareExecutionResults(Blueprint, GeneratedCode));
    
    return true;
}
```

## Performance Benchmarking

### Blueprint vs C++ Performance Tests
```cpp
// Performance comparison framework
class FBlueprintPerformanceTest {
public:
    struct FPerformanceResults {
        double BlueprintExecutionTime;
        double CppExecutionTime;
        double PerformanceGain;
        int32 IterationCount;
    };
    
    static FPerformanceResults BenchmarkConversion(
        const UBlueprint* Blueprint,
        const FString& CppCode,
        int32 Iterations = 10000
    );
};

// Usage in automation tests
IMPLEMENT_SIMPLE_AUTOMATION_TEST(FBlueprintPerformanceTest, 
    "Blueprint.Performance.Comparison", 
    EAutomationTestFlags::ApplicationContextMask | EAutomationTestFlags::ProductFilter)

bool FBlueprintPerformanceTest::RunTest(const FString& Parameters) {
    // Load Blueprint and generate C++
    UBlueprint* Blueprint = LoadObject<UBlueprint>(nullptr, *Parameters);
    FString CppCode = ConvertBlueprintToCpp(Blueprint);
    
    // Run performance benchmark
    auto Results = FBlueprintPerformanceTest::BenchmarkConversion(Blueprint, CppCode);
    
    // Validate performance improvement
    TestTrue("C++ version is faster", Results.PerformanceGain > 1.0);
    AddInfo(FString::Printf(TEXT("Performance gain: %.2fx"), Results.PerformanceGain));
    
    return true;
}
```

## Best Practices for Blueprint Testing

### Test Organization
1. **Categorization**: Organize tests by Blueprint type and functionality
2. **Parameterization**: Use parameterized tests for multiple Blueprint variants
3. **Isolation**: Ensure tests are independent and don't affect each other
4. **Cleanup**: Proper cleanup of test resources and state

### Validation Strategies
1. **Structural Validation**: Verify Blueprint structure integrity
2. **Compilation Validation**: Ensure successful compilation
3. **Execution Validation**: Compare execution results
4. **Performance Validation**: Benchmark performance improvements

### Error Handling and Reporting
1. **Detailed Logging**: Provide comprehensive error information
2. **Screenshot Evidence**: Capture visual evidence of failures
3. **Context Information**: Include relevant context in test results
4. **Reproducibility**: Ensure tests can be easily reproduced

### Integration with CI/CD
1. **Automated Execution**: Run tests automatically in build pipeline
2. **Result Reporting**: Generate comprehensive test reports
3. **Failure Investigation**: Provide tools for investigating test failures
4. **Performance Tracking**: Track performance improvements over time