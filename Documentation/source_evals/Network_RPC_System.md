# Network RPC System Analysis

## Overview
The Network RPC (Remote Procedure Call) system in Unreal Engine enables function calls to be executed across network connections. RPCs allow servers and clients to communicate by calling functions on remote machines.

## Core Files
- `/Engine/Source/Runtime/Engine/Public/Net/UnrealNetwork.h`
- `/Engine/Source/Runtime/Engine/Classes/GameFramework/PlayerController.h`
- `/Engine/Source/Runtime/Engine/Classes/GameFramework/Pawn.h`
- `/Engine/Source/Runtime/Engine/Classes/GameFramework/GameState.h`

## RPC Types and Specifiers

### 1. Server RPCs
Functions called on clients that execute on the server:

```cpp
UFUNCTION(Server, Reliable)
void ServerFunction();

UFUNCTION(Server, Unreliable)  
void ServerFunctionUnreliable();

UFUNCTION(Server, Reliable, WithValidation)
void ServerFunctionWithValidation(int32 Value);
```

### 2. Client RPCs
Functions called on server that execute on specific clients:

```cpp
UFUNCTION(Client, Reliable)
void ClientFunction();

UFUNCTION(Client, Unreliable)
void ClientFunctionUnreliable();
```

### 3. Multicast RPCs
Functions called on server that execute on all connected clients:

```cpp
UFUNCTION(NetMulticast, Reliable)
void MulticastFunction();

UFUNCTION(NetMulticast, Unreliable)  
void MulticastFunctionUnreliable();
```

## RPC Reliability

### Reliable RPCs
Guaranteed delivery and execution order:
- Use TCP-like reliability
- Automatically retransmitted if lost
- Higher overhead but guaranteed execution
- Should be used for critical game state changes

```cpp
UFUNCTION(Server, Reliable)
void ServerCriticalAction();

UFUNCTION(Client, Reliable)  
void ClientImportantUpdate();
```

### Unreliable RPCs
Best-effort delivery:
- Use UDP-like unreliability
- Lower overhead but no delivery guarantee
- Good for frequent, non-critical updates
- Can be dropped under high network load

```cpp
UFUNCTION(Server, Unreliable)
void ServerMovementUpdate();

UFUNCTION(NetMulticast, Unreliable)
void MulticastVisualEffect();
```

## RPC Implementation Examples

### Server RPC Examples
From PlayerController.h:

```cpp
/** RPC used by ServerExec. Not intended to be called directly */
UFUNCTION(Reliable, Server, WithValidation)
void ServerExecRPC(const FString& Msg);

/** Implementation and validation functions */
void ServerExecRPC_Implementation(const FString& Msg);
bool ServerExecRPC_Validate(const FString& Msg);
```

### Client RPC Examples
From PlayerController.h:

```cpp
/** Return the client to the main menu gracefully */
UFUNCTION(Reliable, Client)
virtual void ClientReturnToMainMenuWithTextReason(const FText& ReturnReason);

/** Development RPC for testing object reference replication */
UFUNCTION(Reliable, Client)
virtual void ClientRepObjRef(UObject* Object);

/** Called when the client adds/removes a streamed level */
UFUNCTION(Reliable, Client)
void ClientAddStreamingLevel(FName LevelPackageName, bool bShouldBeLoaded, bool bShouldBeVisible);
```

### Multicast RPC Example
```cpp
/** Multicast RPC to notify all clients of an explosion */
UFUNCTION(NetMulticast, Reliable)
void MulticastExplosion(FVector Location, float Radius);

void MulticastExplosion_Implementation(FVector Location, float Radius)
{
    // This runs on all clients
    SpawnExplosionEffect(Location, Radius);
    PlayExplosionSound(Location);
}
```

## RPC Validation System

### WithValidation Specifier
Server RPCs can include validation to prevent cheating:

```cpp
UFUNCTION(Server, Reliable, WithValidation)
void ServerPurchaseItem(int32 ItemID, int32 Cost);

// Implementation function
void ServerPurchaseItem_Implementation(int32 ItemID, int32 Cost)
{
    // Server logic here
    if (PlayerCurrency >= Cost)
    {
        PlayerCurrency -= Cost;
        AddItemToInventory(ItemID);
    }
}

// Validation function
bool ServerPurchaseItem_Validate(int32 ItemID, int32 Cost)
{
    // Validate parameters
    if (ItemID < 0 || ItemID >= MaxItemID)
    {
        return false;
    }
    
    if (Cost < 0 || Cost > MaxItemCost)
    {
        return false;
    }
    
    return true;
}
```

### RPC_VALIDATE Macro
Helper macro for parameter validation within functions:

```cpp
#define RPC_VALIDATE( expression )						\
	if ( !( expression ) )								\
	{													\
		UE_LOG( LogNet, Warning,						\
		TEXT("RPC_VALIDATE Failed: ")					\
		TEXT( PREPROCESSOR_TO_STRING( expression ) )	\
		TEXT(" File: ")									\
		TEXT( PREPROCESSOR_TO_STRING( __FILE__ ) )		\
		TEXT(" Line: ")									\
		TEXT( PREPROCESSOR_TO_STRING( __LINE__ ) ) );	\
		return false;									\
	}

// Usage example:
bool ServerAction_Validate(float Value)
{
    RPC_VALIDATE(Value >= 0.0f && Value <= 100.0f);
    return true;
}
```

## RPC Function Naming Conventions

### Automatic Function Generation
Unreal's code generation creates multiple functions for each RPC:

```cpp
// Original declaration
UFUNCTION(Server, Reliable, WithValidation)
void ServerDoSomething(int32 Value);

// Generated functions:
void ServerDoSomething(int32 Value);                    // Public interface
void ServerDoSomething_Implementation(int32 Value);     // Actual implementation
bool ServerDoSomething_Validate(int32 Value);          // Validation (if WithValidation)
```

### Function Signature Requirements
- Implementation function: Same signature with `_Implementation` suffix
- Validation function: Same parameters, returns bool, with `_Validate` suffix
- Public function: Automatically generated, handles networking

## RPC Execution Context and Timing

### 1. Authority and Execution
```cpp
UFUNCTION(Server, Reliable)
void ServerFunction()
{
    // Only executes on server
    check(HasAuthority());
    
    // Server-only logic
    ModifyGameState();
}

UFUNCTION(Client, Reliable)
void ClientFunction()
{
    // Only executes on clients
    check(!HasAuthority() || GetNetMode() == NM_Standalone);
    
    // Client-only logic
    UpdateUI();
}
```

### 2. Ownership Requirements
RPCs respect actor ownership rules:
- Server RPCs: Can be called by owning client
- Client RPCs: Server can call on any client
- Multicast RPCs: Only server can call

```cpp
// This will only work if called by the owning client
UFUNCTION(Server, Reliable)
void ServerPlayerAction();

// Server can call this on any client
UFUNCTION(Client, Reliable) 
void ClientNotification();
```

## Advanced RPC Features

### 1. Conditional RPCs
RPCs can be conditionally sent based on network conditions:

```cpp
UFUNCTION(Server, Reliable)
void ServerConditionalAction();

void ServerConditionalAction_Implementation()
{
    // Only process if certain conditions are met
    if (IsValid(GetPawn()) && GetPawn()->IsAlive())
    {
        ProcessAction();
    }
}
```

### 2. RPC Queueing and Buffering
Engine automatically handles:
- RPC queuing when client is not ready
- Reliable RPC buffering and retransmission
- RPC ordering for reliable calls

### 3. Object Reference Parameters
RPCs can pass object references:

```cpp
UFUNCTION(Client, Reliable)
void ClientObjectUpdate(AActor* TargetActor);

void ClientObjectUpdate_Implementation(AActor* TargetActor)
{
    if (IsValid(TargetActor))
    {
        // Safe to use the actor
        TargetActor->DoSomething();
    }
}
```

## Performance and Best Practices

### 1. RPC Frequency Considerations
```cpp
// Good: Infrequent, important events
UFUNCTION(Server, Reliable)
void ServerPlayerDied();

// Bad: High frequency, continuous updates
UFUNCTION(Server, Reliable)
void ServerUpdatePosition(FVector Position); // Use replication instead
```

### 2. Data Size Optimization
```cpp
// Efficient: Minimal data
UFUNCTION(Client, Reliable)
void ClientPlaySound(USoundCue* Sound);

// Inefficient: Large data structures
UFUNCTION(Client, Reliable)
void ClientReceiveComplexData(FLargeStruct Data); // Consider replication
```

### 3. Reliability Choice Guidelines
```cpp
// Use Reliable for:
UFUNCTION(Server, Reliable)
void ServerPurchaseItem(); // Critical transactions

UFUNCTION(Client, Reliable)
void ClientMatchEnded(); // Important state changes

// Use Unreliable for:
UFUNCTION(NetMulticast, Unreliable)
void MulticastPlayEffect(); // Visual effects

UFUNCTION(Server, Unreliable)
void ServerUpdateAim(); // High-frequency updates
```

## Common RPC Patterns

### 1. Server Validation Pattern
```cpp
UFUNCTION(Server, Reliable, WithValidation)
void ServerUseItem(int32 ItemIndex);

bool ServerUseItem_Validate(int32 ItemIndex)
{
    // Validate item index
    return ItemIndex >= 0 && ItemIndex < Inventory.Num();
}

void ServerUseItem_Implementation(int32 ItemIndex)
{
    // Additional server-side validation
    if (Inventory.IsValidIndex(ItemIndex) && Inventory[ItemIndex].IsUsable())
    {
        Inventory[ItemIndex].Use();
        
        // Notify all clients of the result
        MulticastItemUsed(ItemIndex);
    }
}

UFUNCTION(NetMulticast, Reliable)
void MulticastItemUsed(int32 ItemIndex);
```

### 2. Client Feedback Pattern
```cpp
UFUNCTION(Server, Reliable)
void ServerRequestAction();

void ServerRequestAction_Implementation()
{
    if (CanPerformAction())
    {
        PerformAction();
        ClientActionSuccess();
    }
    else
    {
        ClientActionFailed(GetActionFailureReason());
    }
}

UFUNCTION(Client, Reliable)
void ClientActionSuccess();

UFUNCTION(Client, Reliable)
void ClientActionFailed(const FString& Reason);
```

### 3. Broadcast Notification Pattern
```cpp
// Server function to trigger global notification
void NotifyAllPlayers(const FString& Message)
{
    MulticastNotification(Message);
}

UFUNCTION(NetMulticast, Reliable)
void MulticastNotification(const FString& Message);

void MulticastNotification_Implementation(const FString& Message)
{
    // Show notification on all clients
    ShowNotificationToPlayer(Message);
}
```

## Integration with Blueprint System

### 1. Blueprint RPC Events
RPCs can be implemented in Blueprint:
```cpp
UFUNCTION(BlueprintImplementableEvent, Server, Reliable)
void ServerBlueprintAction();

UFUNCTION(BlueprintImplementableEvent, Client, Reliable)
void ClientBlueprintNotification();
```

### 2. C++ to Blueprint RPC Communication
```cpp
// C++ RPC that calls Blueprint event
UFUNCTION(Client, Reliable)
void ClientTriggerBlueprintEvent();

void ClientTriggerBlueprintEvent_Implementation()
{
    // Call Blueprint-implementable event
    OnClientEvent();
}

UFUNCTION(BlueprintImplementableEvent)
void OnClientEvent();
```

## Debugging and Troubleshooting

### 1. RPC Logging
```cpp
void ServerDebugAction_Implementation()
{
    UE_LOG(LogNet, Log, TEXT("ServerDebugAction called by %s"), 
           *GetNameSafe(GetOwner()));
    
    // Your RPC logic here
}
```

### 2. Network Debugging Commands
Console commands for RPC debugging:
- `net.PackageMap.DebugObject` - Debug object replication
- `net.DormancyDraw` - Show dormant actors
- `showdebug net` - Show network statistics

### 3. Common Issues and Solutions

#### RPC Not Executing
- Check ownership (Server RPCs need owner to call)
- Verify network role (Client RPCs need server authority)
- Ensure actor has NetConnection

#### Validation Failures
- Check parameter bounds in validation function
- Log validation failures for debugging
- Ensure validation logic matches gameplay rules

## Security Considerations

### 1. Server Authority
Always validate client requests on server:
```cpp
bool ServerAction_Validate(int32 Action)
{
    // Server-side validation
    return IsValidAction(Action) && CanPlayerPerformAction(Action);
}
```

### 2. Rate Limiting
Prevent RPC spam:
```cpp
UFUNCTION(Server, Reliable, WithValidation)
void ServerFrequentAction();

bool ServerFrequentAction_Validate()
{
    // Check if player is calling this too frequently
    float CurrentTime = GetWorld()->GetTimeSeconds();
    if (CurrentTime - LastActionTime < MinActionInterval)
    {
        return false;
    }
    LastActionTime = CurrentTime;
    return true;
}
```

### 3. Data Sanitization
```cpp
bool ServerChatMessage_Validate(const FString& Message)
{
    // Sanitize input
    return Message.Len() <= MaxChatLength && 
           !Message.Contains(TEXT("<script>")) &&
           IsValidChatMessage(Message);
}
```

## Conclusion
The Network RPC system provides powerful tools for client-server communication in Unreal Engine. Understanding the different RPC types, reliability options, and best practices is essential for creating robust networked gameplay systems. Proper use of validation, appropriate reliability choices, and security considerations ensure both performance and integrity in networked games.