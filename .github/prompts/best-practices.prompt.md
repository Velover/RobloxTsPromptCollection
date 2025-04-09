# Project Best Practices

## System Isolation Example

```ts
// GOOD: Properly isolated system
@Controller({})
@Service({})
export class WeaponSystem implements OnStart, OnInit {
	// Only depends on other systems
	constructor(private readonly _damageSystem: DamageSystem) {}

	// Shared logic used by both client and server
	CalculateDamage(weapon: string, distance: number): number {
		return this._damageSystem.getBaseDamage(weapon) * this.getFalloffMultiplier(distance);
	}

	// Client-only functionality with safety check
	PlayWeaponEffects(weapon: string): void {
		assert(RunService.IsClient(), "Weapon effects can only be played on the client");
		// Effect playing code
	}
}

// BAD: System with improper dependencies
@Controller({})
@Service({})
export class BadWeaponSystem implements OnStart, OnInit {
	constructor(
		private readonly _playerController: PlayerController, // BAD: Depends on Controller
		private readonly _dataService: DataService, // BAD: Depends on Service
	) {}

	// This system will have issues as it attempts to use client/server specific
	// components without proper isolation
}
```

## Component Attributes Best Practices

```ts
interface Attributes {
	Health: number;
	Speed: number;
	CanRespawn: boolean;
}
// GOOD: Component with clear attribute handling
@Component({
	defaults: {
		Health: 100,
		Speed: 16,
		CanRespawn: true,
	},
})
class CharacterComponent extends BaseComponent<Attributes, Model> {
	// Component implementation

	TakeDamage(amount: number): void {
		// Attributes are automatically synchronized with Roblox instance attributes
		this.attributes.Health -= amount;

		// You can update an attribute like this
		if (this.attributes.Health <= 0) {
			this.attributes.CanRespawn = false;
		}
	}
}
```

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
