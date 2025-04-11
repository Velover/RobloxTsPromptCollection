# @flamework/core

Flamework uses items like:

- Controllers - purely for client
- Services - purely for server
- Systems - doesn't exist in flamework, but it implies that it's a Controller and Service at the same time

## Auto injection:

- Flamework has auto-injection in constructors for Controllers, Services, and Components
- ONLY Controllers, Services, and Systems can be used for auto-injection, everything else can be used by direct reference or follows other instructions if provided
- The auto-injection should be avoided in systems unless it's another system to keep the systems as isolated and as self-sustainable as possible

## Services / Controllers / Systems

- OnInit()

```ts
//is executed before injection
//it is not recommended to use any other controllers / services / systems here
//it can be used for initialization of the controller / service / system
```

- OnStart()

```ts
//is executed after injection when the Controller / service / system is ready
```

- constructor()

```ts
//used for injection of the Controllers / Services / Systems
//syntax:
constructor(
  private readonly _someController: SomeController, //client only
  private readonly _someService: SomeService, //server only
  private readonly _someSystem: SomeSystem, //shared, server and client
){ }
```

## Controller

Client-sided Singleton responsible for a specific feature

Example:

```ts
import { Controller, OnStart, OnInit } from "@flamework/core";

@Controller({})
export class SomeController implements OnStart, OnInit {
  constructor(
    private readonly _someOtherController: SomeOtherController,
    private readonly _someOtherSystem: SomeOtherSystem
  ) {}

  onInit() {}

  onStart() {}
}
```

## Service

Server-sided Singleton responsible for a specific feature

Example:

```ts
import { Service, OnStart, OnInit } from "@flamework/core";

@Service({})
export class SomeService implements OnStart, OnInit {
  constructor(
    private readonly _someOtherService: SomeOtherService,
    private readonly _someOtherSystem: SomeOtherSystem
  ) {}

  onInit() {}

  onStart() {}
}
```

## Systems

Shared Singleton responsible for a specific feature

Should be as isolated as possible

**Important clarification on Systems isolation**:

- Systems run on BOTH server and client simultaneously
- Systems should NOT rely on Controllers or Services, but CAN rely on other Systems
- Only use Systems when functionality is required by both server and client and doesn't have a clear separation
- For functionality that should run exclusively on client or server, use Controllers or Services instead
- When including limited server-only or client-only functionality in Systems, use runtime assertions:

```ts
assert(RunService.IsClient(), "This function should run ONLY on the client");
assert(RunService.IsServer(), "This function should run ONLY on the server");
```

- System instances on server and client maintain separate states and do not automatically synchronize

- If state synchronization between server and client is needed, you must use explicit networking solutions:
  - @flamework/core networking (recommended)
  - Roblox RemoteEvents/RemoteFunctions
  - Any other provided networking library

Example:

```ts
@Controller({})
@Service({})
export class SomeSystem implements OnStart, OnInit {
  constructor(
    private readonly _someOtherSystem: SomeOtherSystem //only other systems are allowed in auto-injection
  ) {}

  onInit() {}

  onStart() {}

  // Example of a client-only function within a System
  public ClientOnlyFunction() {
    assert(
      RunService.IsClient(),
      "This function should run ONLY on the client"
    );
    // Client-specific code
  }

  // Example of a server-only function within a System
  public ServerOnlyFunction() {
    assert(
      RunService.IsServer(),
      "This function should run ONLY on the server"
    );
    // Server-specific code
  }
}
```

## Components

A class responsible for instance control

Includes:

- Attributes guard - validates attributes on Roblox instances
- Instance guard - ensures the instance is of correct type

Uses auto-injection as well for Services / Controllers / Systems

```ts
constructor(
  private readonly _someOtherController: SomeOtherController, //client only components
  private readonly _someOtherService: SomeOtherService, //server only components,
  private readonly _someOtherSystem: SomeOtherSystem //server, client and shared components
){
  super(); //requires super, because it extends the BaseComponent class
}
```

Example:

```ts
import { OnStart } from "@flamework/core";
import { Component, BaseComponent } from "@flamework/components";

interface Attributes {
  //attributes guard - defines expected attributes and their types
  Value1: number;
  Value2: string;
  //etc.
}

type InstanceGuard = BasePart; //or any other Instance, that will ensure that the instance has that class

type InstanceGuardWithChildren = BasePart & { ChildName: Model }; //can be used in combination with object that has Instance Children names and types

// Cannot be used with the Instance Property Names
type InstanceGuardWithIncorrectChildName = BasePart & { Name: TextLabel }; //INCORRECT, because BasePart has a property Name on it, and it can lead to undefined behavior

// Cannot be used to validate Properties of the Instance
type InstanceGuardWithIllegalTypes = TextLabel & { Text: "Hello" }; //INCORRECT, Will cause an error

@Component({
  //optional but important for proper attribute handling
  //if the instance doesn't have Attributes set, it will error, therefore defaults can be used for cases like this
  //when provided, will replace missing values in attributes with specified values
  defaults: {
    Value1: 5,
    Value2: "test",
  },
})
class CarsGameData
  extends BaseComponent<Attributes, InstanceGuard>
  implements OnStart
{
  constructor(
    private readonly _someOtherController: SomeOtherController,
    private readonly _someOtherService: SomeOtherService,
    private readonly _someOtherSystem: SomeOtherSystem
  ) {
    super();
  }
  onStart() {}

  // You can access attributes via this.attributes
  GetValueOne(): number {
    return this.attributes.Value1;
  }
}
```

The components don't require a special decorator to separate on Server, Client, and Shared.

## Getting Services, Controllers, Systems

To get Services, Controllers, Systems, and Components, use the Dependency() function.

Dependency is a macro that uses generics to get the instance of the requested Service, Controller, and System.

You cannot get the instance of a Component without special handling. If not used correctly, it can lead to undefined behavior.

Syntax:

```ts
const someController = Dependency<SomeController>();
const someService = Dependency<SomeService>();
const someSystem = Dependency<SomeSystem>();

//Separate handling for instances
const components = Dependency<Components>(); //Getting special System
const component1 = components.getComponent<SomeComponent>(instance); //will try to get the component from the instance, can be undefined

const component2 = await components.waitForComponent<SomeComponent>(instance); //returns async that awaits the component in instance

const componentsList = components.getAllComponents<SomeComponents>(instance); //gets all Components of specified type that exist on instances
```

You CANNOT use Dependency before ignition `Flamework.ignite();`, this will cause an error, therefore injection should be preferred. Avoid using it in global space. Functions can be acceptable.
