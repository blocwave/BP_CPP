# UBlueprintFunctionLibrary API Reference

**Module:** Engine  
**Header:** /Engine/Source/Runtime/Engine/Classes/Kismet/BlueprintFunctionLibrary.h  
**Include:** `#include "Kismet/BlueprintFunctionLibrary.h"`

## Class Definition

```cpp
class UBlueprintFunctionLibrary : public UObject
```

## Overview

UBlueprintFunctionLibrary serves as the base class for all function libraries that expose static utility functions to Blueprints. This class provides a framework for creating collections of related static functions that can be called from Blueprint graphs without requiring an instance of the class.

## Inheritance Hierarchy

- UObjectBase
- UObjectBaseUtility  
- UObject
- **UBlueprintFunctionLibrary**

## Key Characteristics

### Static Function Design
- All methods in subclasses should be static functions
- No instance methods should be added to this base class
- Functions are callable directly in Blueprints without instantiation
- Provides a clean way to group related utility functions

### Blueprint Integration
- Functions automatically become available in Blueprint graphs
- Supports standard Blueprint data types and parameters
- Enables custom Blueprint function categories and keywords
- Functions can be marked with UFUNCTION() macros for Blueprint exposure

## Primary Functions

### Core Functionality

- **GetFunctionCallspace(UFunction* Function, FFrame* Stack)**: Virtual function that determines the execution context for function calls

## Usage Patterns

### Creating Function Libraries

To create a custom Blueprint function library:

```cpp
UCLASS()
class MYGAME_API UMyCustomLibrary : public UBlueprintFunctionLibrary
{
    GENERATED_BODY()

public:
    UFUNCTION(BlueprintCallable, Category = "My Custom Library")
    static int32 AddTwoNumbers(int32 A, int32 B);
    
    UFUNCTION(BlueprintCallable, Category = "My Custom Library")
    static FString ConcatenateStrings(const FString& A, const FString& B);
};
```

### Function Attributes

Common UFUNCTION attributes for Blueprint function libraries:

- **BlueprintCallable**: Makes the function callable from Blueprints
- **BlueprintPure**: Marks the function as pure (no side effects)
- **Category**: Groups functions in Blueprint palette
- **Keywords**: Adds searchable keywords
- **CallInEditor**: Allows execution in the editor

## Common Examples in Unreal Engine

The engine includes numerous Blueprint function libraries:

- **UGameplayStatics**: Core gameplay utility functions
- **UKismetMathLibrary**: Mathematical operations and calculations  
- **UKismetStringLibrary**: String manipulation functions
- **UKismetArrayLibrary**: Array handling utilities
- **UKismetSystemLibrary**: System-level utility functions

## Key Advantages

### Organization
- Groups related functionality logically
- Keeps Blueprint palette organized with categories
- Provides searchable function collections

### Performance
- Static functions avoid instance creation overhead
- Direct function calls without object instantiation
- Efficient execution in both C++ and Blueprints

### Maintainability
- Centralized location for utility functions
- Easy to extend with new functionality
- Clear separation of concerns

## Best Practices

### Function Design
- Keep functions pure when possible (no side effects)
- Use clear, descriptive parameter names
- Provide appropriate default values
- Handle edge cases gracefully

### Blueprint Exposure
- Use meaningful category names
- Add helpful keywords for discoverability
- Provide clear function descriptions
- Consider execution context (editor vs runtime)

### Error Handling
- Validate input parameters
- Return meaningful error states
- Use appropriate logging levels
- Fail gracefully with useful feedback

## Common Use Cases

- Mathematical utility functions
- String processing operations
- File and directory operations
- Game-specific helper functions
- Editor automation scripts
- Asset manipulation utilities
- Debug and development tools

## Integration with Blueprint System

UBlueprintFunctionLibrary integrates seamlessly with the Blueprint system:

1. **Automatic Discovery**: Functions are automatically discovered by the Blueprint compiler
2. **Category Organization**: Functions appear organized by category in the Blueprint palette
3. **Type Safety**: Full type checking and conversion support
4. **Documentation**: Function tooltips and descriptions appear in the editor
5. **Debugging**: Full debugging support in Blueprint execution

## Performance Considerations

- Static functions provide optimal performance
- No object instantiation overhead
- Direct C++ execution when called from Blueprints
- Minimal Blueprint VM overhead for simple operations
- Efficient parameter passing and return value handling