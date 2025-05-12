# Project Requirements and Structure

## Requirements (Project structure):

- Feature-based separation for better maintainability and scalability

```
Features/
/Feature1
-/Controllers (client only - handles client-side logic)
-/Services (server only - handles server-side logic and data processing)
-/Systems (shared only - contains game logic accessible by both client and server, keep as isolated as possible)
-/Resources (Stores constants, static data, and configuration values)
-/GameData (Integration with Roblox Studio, provides interfaces and functions for processing game data)
-/UI (Contains React TSX components for feature-specific interface elements)

/Feature2
-/Controllers (client only)
//etc...
```

- Types and Interfaces should be stored in relative namespaces (Controllers, Services, Resources, GameData) to maintain clear dependencies
- The project structure MUST follow the following structure required by roblox-ts:

```
src/
/client (code that only runs on the client)
/server (code that only runs on the server)
/shared (code accessible by both client and server)
```

## Requirements (toolings):

- The project uses roblox-ts (TypeScript to Luau transpiler) with Roblox-Luau API for instances, Math, Arrays, strings, objects
- The core framework is @flamework/core with @flamework/networking for client-server communication
- UI components are built with @rbxts/react (TSX) NOT @rbxts/roact
- UI state management uses @rbxts/charm for reactive state handling

## Requirements (Project):

- DO NOT modify the package.json file, suggest commands for that instead
- Project follows the @flamework/core lifecycle:
  - Controllers/Services are implemented as singletons
  - Controllers handle client-side logic while Services handle server-side logic
  - Use dependency injection for accessing other components
  - Follow OnInit(), OnStart() pattern for initialization

## Best Practices:

- Keep feature modules as self-contained as possible for better maintainability
- Use TypeScript interfaces to clearly define data structures and API contracts
- Follow consistent naming conventions for classes, methods, and properties
- Use React's component-based approach for UI to promote reusability
- Keep UI logic separate from game logic for better code organization

## Common Patterns:

- Use Dependency Injection through Flamework for accessing services and controllers
- Implement state management using @rbxts/charm for reactive UI updates
- Follow singleton pattern for services and controllers with clear separation of concerns
- Use TypeScript's type system to create self-documenting code
