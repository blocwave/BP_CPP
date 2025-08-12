# Blueprints Technical Guide

Source: https://docs.unrealengine.com/5.2/en-US/technical-guide-for-blueprints-visual-scripting-in-unreal-engine/

**Blueprints** are a powerful feature introduced in Unreal Engine 4. Blueprints are a way to create new UClasses without the need for writing or compiling code. When you create a Blueprint, you can choose to extend a C++ class or another Blueprint class.

You can then add, arrange, and customize Components, implement custom logic using a visual scripting language, respond to Events and interactions, define custom Variables, handle Input, and create a fully custom object type.

Each Blueprint has a Construction Script, analogous to a constructor in C++, which is run when the object is created. This script can dynamically construct the Actor instance based on any number of factors, such as a fence that automatically sizes itself to fill a gap between buildings. In this sense, a Blueprint can be thought of as a very powerful prefab system.

## Related Topics

### Blueprint Function Libraries
Information about Blueprint Function Libraries for C++ in Unreal Engine.
- Link: https://dev.epicgames.com/documentation/en-us/unreal-engine/blueprint-function-libraries-in-unreal-engine?application_version=5.2

### Blueprint Compiler Overview
The steps of the Blueprint compilation process
- Link: https://dev.epicgames.com/documentation/en-us/unreal-engine/compiler-overview-for-blueprints-visual-scripting-in-unreal-engine?application_version=5.2

### Exposing Gameplay Elements to Blueprints
Technical guide for gameplay programmers exposing gameplay elements to Blueprints.
- Link: https://dev.epicgames.com/documentation/en-us/unreal-engine/exposing-gameplay-elements-to-blueprints-visual-scripting-in-unreal-engine?application_version=5.2

### Exposing C++ to Blueprints
Tips and tricks for how best to write a Blueprint-friendly API
- Link: https://dev.epicgames.com/documentation/en-us/unreal-engine/exposing-cplusplus-to-blueprints-visual-scripting-in-unreal-engine?application_version=5.2