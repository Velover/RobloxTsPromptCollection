# Project Best Practices

## Project Best Practices

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

- Always prefer `const enum` when possible as they are inlined at compile time, resulting in better performance
- Use enums when dealing with a limited set of options rather than string literals

### Avoid Inlining Interfaces

```ts
// BAD: Inlined interface
class CharacterComponent extends BaseComponent<
  {
    Health: number;
    Speed: number;
    CanRespawn: boolean;
  },
  Model
> {
  // Implementation
}

// GOOD: Separate interface definition
interface CharacterAttributes {
  Health: number;
  Speed: number;
  CanRespawn: boolean;
}

class CharacterComponent extends BaseComponent<CharacterAttributes, Model> {
  // Implementation with better readability
}
```

- Separate interface definitions improve readability and reusability

### Consistent Naming Conventions

```ts
// For components:
interface WeaponAttributes {
  /* ... */
}
class WeaponComponent extends BaseComponent<WeaponAttributes, Model> {
  /* ... */
}

// For systems:
const enum WeaponType /* ... */ {}
class WeaponSystem implements OnStart, OnInit {
  /* ... */
}
```

- Use consistent naming patterns to improve code navigation and organization
- Group related enums, interfaces, and classes in the same file when they're tightly coupled

### Namespace Organization to Avoid Global Pollution

```ts
interface PrivateDataType {
  //can be outside of class, because it's not exported
  Value1: string;
}
export class SomeClass {}

export namespace SomeClass {
  export interface PublicDataType {
    //coupled to the related namespace or class when needs to be exported
    Value1: number;
  }
}
```

- Couple exported interfaces, types, and enums to related namespaces or types
- This prevents them from polluting the global namespace
- Private types can remain uncoupled if they're not exported
- This approach improves code organization and reduces name collisions
