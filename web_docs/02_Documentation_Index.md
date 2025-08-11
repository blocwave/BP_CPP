# Unreal Engine Blueprint API Documentation Index

## Overview
This directory contains comprehensive documentation for the Unreal Engine Blueprint API system, compiled from official sources, forum discussions, and community resources.

## Core Blueprint Classes

### Graph Structure
- **[UEdGraph_API.md](UEdGraph_API.md)** - Base class for all Blueprint graphs
- **[UEdGraphNode_API.md](UEdGraphNode_API.md)** - Base class for all graph nodes
- **[UEdGraphPin.md](UEdGraphPin.md)** - Pin connections between nodes
- **[UEdGraphSchema_K2_API.md](UEdGraphSchema_K2_API.md)** - Blueprint graph schema and rules

### K2 Node System
- **[UK2Node_API.md](UK2Node_API.md)** - Base class for all Blueprint nodes
- **[UK2Node_CallFunction_API.md](UK2Node_CallFunction_API.md)** - Function call nodes
- **[UK2Node_Event_API.md](UK2Node_Event_API.md)** - Event nodes in Blueprints
- **[UK2Node_Variable_API.md](UK2Node_Variable_API.md)** - Variable getter/setter nodes
- **[K2Node_Examples_FAQ.md](K2Node_Examples_FAQ.md)** - Practical examples and FAQ

### Blueprint System
- **[UBlueprint_API.md](UBlueprint_API.md)** - Main Blueprint asset class
- **[UBlueprintGeneratedClass_API.md](UBlueprintGeneratedClass_API.md)** - Runtime generated classes
- **[UBlueprintFunctionLibrary.md](UBlueprintFunctionLibrary.md)** - Static function libraries

### Construction Scripts
- **[USimpleConstructionScript.md](USimpleConstructionScript.md)** - Construction script management
- **[USCS_Node.md](USCS_Node.md)** - Scene component nodes

### Compilation System
- **[FKismetCompilerContext_API.md](FKismetCompilerContext_API.md)** - Main compilation context
- **[FKismetCompiledStatement_API.md](FKismetCompiledStatement_API.md)** - Compiled bytecode statements
- **[FKismetFunctionContext_API.md](FKismetFunctionContext_API.md)** - Function compilation context
- **[FNodeHandlingFunctor_API.md](FNodeHandlingFunctor_API.md)** - Node processing functors
- **[KismetCompiler_Pipeline_Overview.md](KismetCompiler_Pipeline_Overview.md)** - Compilation pipeline overview

### Editor Utilities
- **[FKismetEditorUtilities.md](FKismetEditorUtilities.md)** - Kismet editor utilities
- **[FBlueprintEditorUtils_API.md](FBlueprintEditorUtils_API.md)** - Blueprint editor utilities

### Technical Guides
- **[blueprint_compiler_overview.md](blueprint_compiler_overview.md)** - Compiler architecture overview
- **[blueprint_technical_guide.md](blueprint_technical_guide.md)** - Technical implementation guide
- **[UE_5_2_Documentation.md](UE_5_2_Documentation.md)** - Unreal Engine 5.2 specific documentation and features

## Source Materials

### fc_extracted/
Contains 44 source files including:
- 26 API documentation files from Epic Games
- 10 forum discussions with practical solutions
- 2 technical guides
- 2 version-specific documentation files
- 2 third-party documentation resources
- 1 community resource
- 1 blog post on K2Nodes

## Documentation Coverage

### Complete Coverage (25 files)
All major Blueprint API classes have comprehensive documentation with:
- Class definitions and inheritance
- Method documentation
- Property descriptions
- Usage examples
- Integration patterns
- Related class references

### Coverage Statistics
- **40 of 44** fc_extracted files directly incorporated into documentation
- **100%** of API classes documented
- **10 forum discussions** synthesized into K2Node_Examples_FAQ.md
- **1 version-specific file** incorporated into UE_5_2_Documentation.md
- **4 files** not directly mapped (1 version-specific for 5.6, 2 third-party resources, 1 community discussion)

### Source Mapping
The **documentation_mapping.json** file provides:
- Complete mapping of each .md file to its fc_extracted sources
- Category organization of documentation
- Coverage statistics
- List of files not directly incorporated

## How to Use This Documentation

1. **For API Reference**: Start with the specific class documentation (e.g., UK2Node_API.md)
2. **For Implementation**: Check K2Node_Examples_FAQ.md for practical examples
3. **For Architecture**: Review the technical guides and compilation pipeline
4. **For Troubleshooting**: Consult the FAQ section in K2Node_Examples_FAQ.md

## Related Resources

- [Unreal Engine Official Documentation](https://docs.unrealengine.com/)
- [Unreal Engine Forums](https://forums.unrealengine.com/)
- [Unreal Engine Source Code](https://github.com/EpicGames/UnrealEngine) (requires access)