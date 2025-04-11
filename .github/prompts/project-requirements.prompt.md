# Project Requirements and Structure

## Requirements (Project structure):

- Feature-based separation

```
Features/
/Feature1
-/Controllers (client only)
-/Services (server only)
-/Systems (shared only) (keep as isolated as possible)
-/Resources (Stores constants and static data)
-/GameData (Integration with Roblox Studio, provides interfaces and functions for processing)
-/UI

/Feature2
-/Controllers (client only)
//etc...
```

- Types and Interfaces should be stored in relative namespaces (Controllers, Services, Resources, GameData)
- The project structure SHOULD follow the following structure and cannot be changed because it is required by roblox-ts

```
src/
/client
/server
/shared
```

## Requirements (toolings):

- The project is using roblox-ts which is a slightly modified TypeScript version that transpiles to Lua
- The roblox-ts uses mostly Roblox-Luau API for instances, Math, Arrays, strings, objects
- The core of the game is @flamework/core with @flamework/networking
- The UI of the game is @rbxts/react (TSX) NOT @rbxts/roact
- For UI state management, the game is using @rbxts/charm

## Requirements (Project):

- DO NOT modify the package.json file, suggest commands for that instead
- Project follows the @flamework/core lifecycle
