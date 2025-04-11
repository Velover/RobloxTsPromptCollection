# @flamework/networking Basics

@flamework/networking is a powerful Roblox TypeScript module that provides type-safe communication between client and server in Roblox games. It's designed with ease of use and convenience in mind while maintaining strong type safety guarantees.

## Core Features

- **Type-checked events and functions**: Define your network interfaces once and get full type checking across your codebase
- **Automatic type guards**: Built-in validation prevents clients from sending improper data types
- **IntelliSense support**: Get autocompletion for event and function arguments
- **Promise-based API**: Clean asynchronous operations with timeout and cancellation support
- **Middleware support**: Modify, delay, or drop requests before they reach handlers

## Understanding Flamework Networking Types

Flamework networking uses four main interface types to handle different communication patterns:

### 1. ClientToServerEvents

Used when clients need to send information to the server without expecting a response.

### 2. ServerToClientEvents

Used when the server needs to send information to specific clients or broadcast to all clients.

### 3. ClientToServerFunctions

Used when clients need to request data from the server and receive a response (returns a Promise).

### 4. ServerToClientFunctions

Used when the server needs to request data from clients (use sparingly due to security concerns).

## Basic Setup

Here's a simplified version of setting up Flamework networking:

```ts
// network.ts - Shared file
import { Networking } from "@flamework/networking";
import { RunService } from "@rbxts/services";

// Define your interfaces
interface ClientToServerEvents {
  ClientToServerEvent(arg1: string, arg2: number): void;
}

interface ServerToClientEvents {
  ServerToClientEvent(data: string): void;
}

interface ClientToServerFunctions {
  ClientToServerFunction(input: string): number;
}

interface ServerToClientFunctions {
  ServerToClientFunction(query: string): boolean;
}

// Create networking handlers
const globalEvents = Networking.createEvent();
const globalFunctions = Networking.createFunction();
```

## Naming Convention

Flamework networking events are named after the receiver:

- A "Client Event" is listened to by the client
- A "Server Event" is listened to by the server

## Usage Examples

### Client Sending Events to Server

```ts
// client.ts
import { globalEvents } from "./network";

// Create client events
const ClientEvents = globalEvents.createClient({});

// Fire event to server
ClientEvents.ClientToServerEvent.fire("Hello server", 42);
```

### Server Listening for Client Events

```ts
// server.ts
import { globalEvents } from "./network";

// Create server events
const ServerEvents = globalEvents.createServer({});

// Listen for client events
ServerEvents.ClientToServerEvent.connect((player, message, count) => {
  print(`${player.Name} sent: ${message} (${count})`);
});
```

### Client Requesting Data from Server

```ts
// client.ts
import { globalFunctions } from "./network";

// Create client functions
const ClientFunctions = globalFunctions.createClient({});

// Using async/await with error handling
async function fetchDataFromServer(query: string) {
  try {
    const result = await ClientFunctions.ClientToServerFunction.invoke(query);
    return result;
  } catch (err) {
    warn("Failed to fetch data:", err);
    return -1;
  }
}
```

## Best Practices

1. **Security**: Never trust client data - always validate on the server side
2. **Performance**: Use events for one-way communication, only use functions when responses are needed
3. **Error Handling**: Always implement proper error handling with try/catch blocks or Promise chains
4. **Promise Management**: Set timeouts using `.timeout(seconds)` to prevent hanging operations
5. **Type Safety**: Leverage Flamework's built-in type guards to enforce proper data types
6. **Avoid ServerToClientFunctions**: These can be security risks and should be used sparingly
