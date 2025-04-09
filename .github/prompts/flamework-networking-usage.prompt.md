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
const globalEvents = Networking.createEvent<ClientToServerEvents, ServerToClientEvents>();
const globalFunctions = Networking.createFunction<
	ClientToServerFunctions,
	ServerToClientFunctions
>();

// Client-side exports (undefined on server)
export const ClientEvents = RunService.IsClient() ? globalEvents.createClient({}) : undefined!;
export const ClientFunctions = RunService.IsClient()
	? globalFunctions.createClient({})
	: undefined!;

// Server-side exports (undefined on client)
export const ServerEvents = RunService.IsServer() ? globalEvents.createServer({}) : undefined!;
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
async function checkClientState(player: Player, query: string): Promise<boolean> {
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
