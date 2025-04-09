# @rbxts/flamework Networking Guide

## Overview

Flamework networking provides a type-safe way to communicate between client and server in Roblox games. It offers:

- Type-checked events and remote functions
- Built-in type guards that prevent clients from sending improper data types
- IntelliSense support for arguments
- Promise-based API for asynchronous operations

## Basic Interface Definition

```ts
// Example of defining network interfaces
interface ClientToServerEvents {
  TestEvent(value1: number, value2: string): void; // Define parameters in the function signature
}
```

> **Important**: Flamework networking events and functions must be defined at compile time only. This is because Flamework uses reflection to create type guards for these interfaces. You cannot dynamically add new events after the interfaces are defined.

## Complete Setup Example

```ts
// network.ts - Shared file
import { Networking } from "@flamework/networking";
import { RunService } from "@rbxts/services";

// Client to server communication
interface ClientToServerEvents {
  ClientToServerEvent(arg1: string, arg2: number): void;
}

// Server to client communication
interface ServerToClientEvents {
  ServerToClientEvent(data: string): void;
}

// Client requesting data from server
interface ClientToServerFunctions {
  ClientToServerFunction(input: string): number; // Return type indicates what server sends back
}

// Server requesting data from client (use sparingly)
interface ServerToClientFunctions {
  ServerToClientFunction(query: string): boolean;
}

// Create networking handlers
const globalEvents = Networking.createEvent<
  ClientToServerEvents,
  ServerToClientEvents
>();
const globalFunctions = Networking.createFunction<
  ClientToServerFunctions,
  ServerToClientFunctions
>();

// Client-side exports (undefined on server)
export const ClientEvents = RunService.IsClient()
  ? globalEvents.createClient({})
  : undefined!;
export const ClientFunctions = RunService.IsClient()
  ? globalFunctions.createClient({})
  : undefined!;

// Server-side exports (undefined on client)
export const ServerEvents = RunService.IsServer()
  ? globalEvents.createServer({})
  : undefined!;
export const ServerFunctions = RunService.IsServer()
  ? globalFunctions.createServer({})
  : undefined!;
```

## Networking Types Guide

### 1. ClientToServerEvents

**Purpose**: Client sends events to server

**Client-side usage**:

```ts
// client.ts
import { ClientEvents } from "./network";

// Fire event to server
function sendToServer(message: string, count: number) {
  ClientEvents.ClientToServerEvent.fire(message, count);
}
```

**Server-side usage**:

```ts
// server.ts
import { ServerEvents } from "./network";

// Listen for client events
ServerEvents.ClientToServerEvent.connect((player, message, count) => {
  print(`${player.Name} sent: ${message} (${count} times)`);
});
```

### 2. ServerToClientEvents

**Purpose**: Server sends events to specific clients or broadcasts to all

**Client-side usage**:

```ts
// client.ts
import { ClientEvents } from "./network";

// Listen for server events
ClientEvents.ServerToClientEvent.connect((data) => {
  print(`Server sent: ${data}`);
});
```

**Server-side usage**:

```ts
// server.ts
import { ServerEvents } from "./network";
import { Players } from "@rbxts/services";

// Send to specific player
function notifyPlayer(player: Player, message: string) {
  ServerEvents.ServerToClientEvent.fire(player, message);
}

// Broadcast to all players
function broadcastAnnouncement(message: string) {
  ServerEvents.ServerToClientEvent.broadcast(message);
}
```

### 3. ClientToServerFunctions

**Purpose**: Client requests data from server and awaits response

**Client-side usage**:

```ts
// client.ts
import { ClientFunctions } from "./network";

// Option 1: Using async/await with try/catch
async function getDataFromServer(query: string): Promise<number> {
  try {
    const result = await ClientFunctions.ClientToServerFunction.invoke(query);
    return result;
  } catch (err) {
    warn("Server request failed:", err);
    return -1;
  }
}

// Option 2: Using Promise chaining with timeout and cancellation
function fetchDataWithTimeout(query: string) {
  const promise = ClientFunctions.ClientToServerFunction.invoke(query)
    .timeout(5) // Set timeout in seconds to prevent long waiting
    .then((result) => {
      print("Received result:", result);
      return result;
    })
    .catch((err) => {
      warn("Server request failed:", err);
      return -1;
    });

  // Store the promise if you need to cancel it later
  return promise;
}

// Cancel the promise if needed
function cancelRequest(promise: Promise<unknown>) {
  promise.cancel();
  print("Request cancelled");
}
```

**Server-side usage**:

```ts
// server.ts
import { ServerFunctions } from "./network";

// Handle client requests
ServerFunctions.ClientToServerFunction.setCallback((player, query) => {
  print(`${player.Name} requested data with query: ${query}`);

  // Process request and return result
  return query.size() * 10;
});
```

### 4. ServerToClientFunctions (Use Sparingly)

**Purpose**: Server requests data from client (NOTE: Avoid when possible due to security concerns)

**Client-side usage**:

```ts
// client.ts
import { ClientFunctions } from "./network";

// Respond to server requests
ClientFunctions.ServerToClientFunction.setCallback((query) => {
  print(`Server requested: ${query}`);
  return query === "authorized";
});
```

**Server-side usage**:

```ts
// server.ts
import { ServerFunctions } from "./network";

// Request data from client
async function checkClientState(
  player: Player,
  query: string
): Promise<boolean> {
  try {
    return await ServerFunctions.ServerToClientFunction.invoke(player, query);
  } catch (err) {
    warn(`Failed to get data from ${player.Name}:`, err);
    return false;
  }
}
```

## Best Practices

1. **Security**: Never trust client data - always validate on server
2. **Performance**: Use events for one-way communication, functions only when a response is needed
3. **Error Handling**:
   - Use try/catch with async/await for cleaner code
   - Alternatively, use Promise chaining with .then() and .catch() for more complex flows
4. **Promise Management**:
   - Set timeouts using .timeout(seconds) to prevent hanging operations
   - Cancel long-running promises when they're no longer needed
5. **Types**: Define interfaces in a shared file for consistency across client and server
6. **Type Safety**: Rely on Flamework's built-in type guards to enforce proper data types
7. **Avoid ServerToClientFunctions**: These can be easily exploited by malicious clients

## Common Patterns

- Use ClientToServerFunctions for data fetching and authentication
- Use ServerToClientEvents for notifications and game state updates
- Use ClientToServerEvents for player actions and input

## Advanced Promise Handling

```ts
// Example of comprehensive Promise handling
function complexServerRequest(data: string) {
  const promise = ClientFunctions.SomeFunction.invoke(data)
    .timeout(3) // Set 3-second timeout
    .then((result) => {
      return processResult(result);
    })
    .catch((err) => {
      if (err.message.includes("timeout")) {
        warn("Request timed out - server might be busy");
      } else {
        warn("Error in request:", err);
      }
      return getDefaultValue();
    })
    .finally(() => {
      // Clean up resources or UI elements
      hideLoadingIndicator();
    });

  // Store promise for potential cancellation
  activeFunctionCalls.set(data, promise);

  return promise;
}

// Later, if needed:
function cleanupRequests() {
  for (const [key, promise] of activeFunctionCalls) {
    promise.cancel();
  }
  activeFunctionCalls.clear();
}
```

## Dynamic Remote Events

If you need to create remote events dynamically (outside of the Flamework networking system), you can still leverage Flamework's type guards for runtime type safety. Here's how to approach this:

### Utility Function

```ts
// utility.ts
import { RunService } from "@rbxts/services";

export function DefineRemoteEvent(
  parent: Instance,
  eventName: string
): RemoteEvent {
  if (RunService.IsServer()) {
    assert(
      parent.FindFirstChild(eventName) === undefined,
      `Event ${eventName} already exists`
    );
    const event = new Instance("RemoteEvent", parent);
    event.Name = eventName;
    return event;
  }

  // Client-side: wait for the event to be created by the server
  return parent.WaitForChild(eventName) as RemoteEvent;
}
```

### Server Implementation

```ts
// server.ts
import { ReplicatedStorage } from "@rbxts/services";
import { Flamework } from "@flamework/core";
import { DefineRemoteEvent } from "./utility";

// Define the expected argument types
type Args = [number, string];

// Create a type guard using Flamework
const guard: (value: unknown) => value is Args = Flamework.createGuard<Args>();

// Create the RemoteEvent at a known location with a unique name
const serverEvent = DefineRemoteEvent(ReplicatedStorage, "TestEvent");

// Fire event to a specific client
function notifyClient(player: Player, arg1: number, arg2: string) {
  serverEvent.FireClient(player, arg1, arg2);
}

// Broadcast to all clients
function broadcastToClients(arg1: number, arg2: string) {
  serverEvent.FireAllClients(arg1, arg2);
}

// Connect to client events with type validation
serverEvent.OnServerEvent.Connect((player: Player, ...args: unknown[]) => {
  if (!guard(args)) {
    warn(
      `Received incorrect arguments from player ${
        player.Name
      }, Event: ${serverEvent.GetFullName()}`
    );
    return;
  }

  const [arg1, arg2] = args;
  print(`Player ${player.Name} sent: ${arg1}, ${arg2}`);
  // Process validated arguments here
});
```

### Client Implementation

```ts
// client.ts
import { ReplicatedStorage } from "@rbxts/services";
import { Flamework } from "@flamework/core";
import { DefineRemoteEvent } from "./utility";

// Define the expected argument types (must match server-side)
type Args = [number, string];

// Create a type guard using Flamework
const guard: (value: unknown) => value is Args = Flamework.createGuard<Args>();

// Get reference to the same RemoteEvent created by the server
const clientEvent = DefineRemoteEvent(ReplicatedStorage, "TestEvent");

// Send data to the server
function sendToServer(value1: number, value2: string) {
  clientEvent.FireServer(value1, value2);
}

// Listen for server events with type validation
clientEvent.OnClientEvent.Connect((...args: unknown[]) => {
  if (!guard(args)) {
    warn(
      `Received incorrect arguments from server. Event: ${clientEvent.GetFullName()}`
    );
    return;
  }

  const [arg1, arg2] = args;
  print(`Server sent: ${arg1}, ${arg2}`);
  // Process validated arguments here
});
```

### Best Practices for Dynamic Events

1. **Consistent Naming**: Use the same event names and parent instances on both client and server
2. **Type Validation**: Always use Flamework guards to validate types at runtime
3. **Error Handling**: Gracefully handle cases where type validation fails
4. **Documentation**: Document the expected argument types for each dynamic event
5. **Organization**: Group related dynamic events in dedicated folders/instances
6. **Avoid Overuse**: Use dynamic events only when necessary; prefer Flamework's standard networking when possible
