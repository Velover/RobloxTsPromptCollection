## TypeScript Best Practices

### Use `const enum` Instead of Regular `enum`

```ts
// BAD: Regular enum
enum Direction {
  Up,
  Down,
  Left,
  Right,
}

// GOOD: const enum for better performance
const enum Direction {
  Up,
  Down,
  Left,
  Right,
}
```

- Always prefer `const enum` when possible as they are completely removed during compilation and replaced with literal values, resulting in zero runtime overhead
- Use regular enums only when you need runtime access to enum values or need to iterate through enum keys
- Use enums when dealing with a closed set of related values rather than string literals or boolean flags

### Avoid Inlining Interfaces

```ts
// BAD: Inlined interface
class CharacterComponent extends BaseComponent<{
  Health: number;
  Speed: number;
  CanRespawn: boolean;
}> {
  // Implementation
}

// GOOD: Separate interface definition
interface CharacterAttributes {
  Health: number;
  Speed: number;
  CanRespawn: boolean;
}

class CharacterComponent extends BaseComponent<CharacterAttributes> {
  // Implementation with better readability
}
```

- Separate interface definitions improve readability, code navigation, and maintainability
- Named interfaces can be reused across multiple components, reducing duplication
- This principle applies to all generic type parameters, not just component attributes
- Consider placing related interfaces in the same file as their implementation for better discoverability

### Consistent Naming Conventions

```ts
// For components:
interface WeaponAttributes {
  /* ... */
}
class WeaponComponent extends BaseComponent {
  /* ... */
}

// For systems:
const enum WeaponType /* ... */ {}
class WeaponSystem implements OnStart, OnInit {
  /* ... */
}

// File naming matches the primary export:
// WeaponComponent.ts
// WeaponSystem.ts
```

- Use PascalCase for class names, interfaces, and type aliases
- Maintain a consistent suffix pattern for related entities (e.g., XxxComponent, XxxSystem, XxxAttributes)
- Name files after their primary export to improve code navigation and organization
- Group related enums, interfaces, and classes in the same file when they're tightly coupled
- For components, use the pattern `{Name}Component` and `{Name}Attributes`
- For systems, use the pattern `{Name}System` and appropriate interface implementations

### Namespace Organization to Avoid Global Pollution

```ts
// BAD: Everything in global namespace
interface PlayerData {
  value: number;
}
interface WeaponData {
  damage: number;
}
export class Player {
  /* ... */
}
export class Weapon {
  /* ... */
}

// GOOD: Organized with namespaces
// Private (non-exported) interfaces can remain outside
interface PrivateUtilType {
  value: string;
}

export class Player {
  /* ... */
}
// Public interfaces coupled with their related class/namespace
export namespace Player {
  export interface PlayerData {
    value: number;
  }
}

export class Weapon {
  /* ... */
}
export namespace Weapon {
  export interface WeaponData {
    damage: number;
  }
}
```

- Couple exported interfaces, types, and enums to related namespaces or classes using declaration merging
- This prevents namespace pollution and clearly associates types with their related functionality
- Private (non-exported) types can remain uncoupled if they're used only within the file
- This approach significantly improves code organization in larger projects and reduces name collisions
- Use modules and namespaces together for hierarchical organization of complex components

### Type Safety Best Practices

```ts
// BAD: Using 'any' type
function processData(data: any): any {
  return data.value;
}

// GOOD: Using proper type definitions
interface DataObject {
  value: number;
}

function processData(data: DataObject): number {
  return data.value;
}
```

- Avoid using `any` type as it defeats TypeScript's static type checking benefits
- Use generic types when creating reusable components or functions
- Always specify return types for functions to improve code clarity and catch errors
