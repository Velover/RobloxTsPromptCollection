# Flamework Core Basics

Flamework is a TypeScript game framework designed for Roblox development that emphasizes simplicity and extensibility. The `@flamework/core` package provides the foundation of the framework with several key features that help structure and organize your code.

## Core Concepts

### Singletons

Flamework organizes code through three main types of singletons:

- **Controllers**: Client-sided singletons responsible for specific features
- **Services**: Server-sided singletons responsible for specific features
- **Systems**: Shared singletons that run on both server and client simultaneously

### Lifecycle Events

Flamework provides non-obtrusive lifecycle events that are easy to implement:

- **OnInit()**: Executed before dependency injection; used for initialization
- **OnStart()**: Executed after injection when the component is ready

### Dependency Injection

One of Flamework's key features is constructor-based dependency injection:

```ts
constructor(
  private readonly _someController: SomeController, // client only
  private readonly _someService: SomeService, // server only
  private readonly _someSystem: SomeSystem // shared
){ }
```

## Controllers

Controllers handle client-side functionality:

```ts
import { Controller, OnStart, OnInit } from "@flamework/core";

@Controller({})
export class SomeController implements OnStart, OnInit {
  constructor(
    private readonly _someOtherController: SomeOtherController,
    private readonly _someOtherSystem: SomeOtherSystem
  ) {}

  onInit() {
    // Initialization code
  }

  onStart() {
    // Code that runs after injection
  }
}
```

## Services

Services handle server-side functionality:

```ts
import { Service, OnStart, OnInit } from "@flamework/core";

@Service({})
export class SomeService implements OnStart, OnInit {
  constructor(
    private readonly _someOtherService: SomeOtherService,
    private readonly _someOtherSystem: SomeOtherSystem
  ) {}

  onInit() {
    // Initialization code
  }

  onStart() {
    // Code that runs after injection
  }
}
```

## Systems

Systems run on both client and server simultaneously and should be designed to be as isolated as possible:

```ts
@Controller({})
@Service({})
export class SomeSystem implements OnStart, OnInit {
  constructor(
    private readonly _someOtherSystem: SomeOtherSystem // only other systems allowed in auto-injection
  ) {}

  onInit() {}
  onStart() {}

  // For client-only functionality within a System
  public ClientOnlyFunction() {
    assert(
      RunService.IsClient(),
      "This function should run ONLY on the client"
    );
    // Client-specific code
  }

  // For server-only functionality within a System
  public ServerOnlyFunction() {
    assert(
      RunService.IsServer(),
      "This function should run ONLY on the server"
    );
    // Server-specific code
  }
}
```

**Important notes about Systems**:

- They run on BOTH server and client simultaneously
- They should NOT rely on Controllers or Services
- System instances on server and client maintain separate states
- If state synchronization is needed, explicit networking solutions must be used

## Components

Components handle instance control with attributes and instance type validation:

```ts
import { OnStart } from "@flamework/core";
import { Component, BaseComponent } from "@flamework/components";

interface Attributes {
  Value1: number;
  Value2: string;
}

type InstanceGuard = BasePart; // Instance type validation

@Component({
  defaults: {
    Value1: 5,
    Value2: "test",
  },
  tag: "MyComponent" //Tag that should be applied to instance. The instances with that tag will be instantiated with this compomponent
})
class MyComponent extends BaseComponent implements OnStart {
  constructor(
    private readonly _someController: SomeController,
    private readonly _someService: SomeService,
    private readonly _someSystem: SomeSystem
  ) {
    super();
  }

  onStart() {}

  GetValueOne(): number {
    return this.attributes.Value1;
  }
}
```

## Accessing Singletons

To access singletons outside of constructor injection, use the Dependency function:

```ts
const someController = Dependency();
const someService = Dependency();
const someSystem = Dependency();

// For components:
const components = Dependency();
const component = components.getComponent(instance);
const componentAsync = await components.waitForComponent(instance);
const allComponents = components.getAllComponents(instance);
```

Note that you cannot use Dependency before `Flamework.ignite();` is called.

## Benefits of Flamework

- Minimizes boilerplate code
- Supports optional lifecycle events
- Automatically generates type guards for networking and components
- Designed to be extended via modding API
- Splits functionality into packages so you only install what you need

Flamework provides a structured yet flexible approach to Roblox TypeScript development, making it easier to organize and maintain complex game code while maintaining type safety.

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
    return (
      this._damageSystem.getBaseDamage(weapon) *
      this.getFalloffMultiplier(distance)
    );
  }

  // Client-only functionality with safety check
  PlayWeaponEffects(weapon: string): void {
    assert(
      RunService.IsClient(),
      "Weapon effects can only be played on the client"
    );
    // Effect playing code
  }
}

// BAD: System with improper dependencies
@Controller({})
@Service({})
export class BadWeaponSystem implements OnStart, OnInit {
  constructor(
    private readonly _playerController: PlayerController, // BAD: Depends on Controller
    private readonly _dataService: DataService // BAD: Depends on Service
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
