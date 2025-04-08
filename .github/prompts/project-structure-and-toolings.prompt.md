Requirements (Project structure):

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

Requirements (toolings):

- The project is using roblox-ts which is a slightly modified TypeScript version that transpiles to Lua
- The roblox-ts uses mostly Roblox-Luau API for instances, Math, Arrays, strings, objects
- The core of the game is @rbxts/flamework
- The UI of the game is @rbxts/react (TSX) NOT @rbxts/roact
- For UI state management, the game is using @rbxts/charm

Requirements (Project):

- DO NOT modify the package.json file, suggest commands for that instead
- Project follows the @rbxts/flamework lifecycle

# Explanation (@rbxts/flamework):

Flamework uses items like:

- Controllers - purely for client
- Services - purely for server
- Systems - doesn't exist in flamework, but it implies that it's a Controller and Service at the same time

### Auto injection:

- Flamework has auto-injection in constructors for Controllers, Services, and Components
- ONLY Controllers, Services, and Systems can be used for auto-injection, everything else can be used by direct reference or follows other instructions if provided
- The auto-injection should be avoided in systems unless it's another system to keep the systems as isolated and as self-sustainable as possible

### Services / Controllers / Systems

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
		private readonly _someOtherSystem: SomeOtherSystem,
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
		private readonly _someOtherSystem: SomeOtherSystem,
	) {}

	onInit() {}

	onStart() {}
}
```

## Systems

Shared Singleton responsible for a specific feature

Should be as isolated as possible

Example:

```ts
@Controller({})
@Service({})
export class SomeSystem implements OnStart, OnInit {
	constructor(
		private readonly _someOtherSystem: SomeOtherSystem, //only other systems are allowed in auto-injection
	) {}

	onInit() {}

	onStart() {}
}
```

## Components

A class responsible for instance control

Includes:

- Attributes guard
- Instance guard

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
	//attributes guard
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
	//optional
	//if the instance doesn't have Attributes set, it will error, therefore defaults can be used for cases like this
	//when provided, will replace missing values in attributes with specified values
	defaults: {
		Value1: 5,
		Value2: "test",
	},
})
class CarsGameData extends BaseComponent<Attributes, InstanceGuard> implements OnStart {
	constructor(
		private readonly _someOtherController: SomeOtherController,
		private readonly _someOtherService: SomeOtherService,
		private readonly _someOtherSystem: SomeOtherSystem,
	) {
		super();
	}
	onStart() {}
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

const components_list = components.getAllComponents<SomeComponents>(instance); //gets all Components of specified type that exist on instances
```

You CANNOT use Dependency before ignition `Flamework.ignite();`, this will cause an error, therefore injection should be preferred. Avoid using it in global space. Functions can be acceptable.

# Explanation (@rbxts/react)

@rbxts/react is NOT (@rbxts/roact)

It follows the syntax of regular React v17.2.3

```tsx
//can use .tsx files
function App() {
	return (
		<screengui
			//all properties should be written in the component directly
			ResetOnSpawn={false}
		>
			{/*Some other children*/}
		</screengui>
	);
}

function ComponentWithChildren({ children }: PropsWithChildren<{}>) {
	return (
		<frame
		//...properties
		>
			{children}
		</frame>
	);
}

function ComponentWithFragment() {
	return (
		<>
			<frame />
			<frame />
			<frame />
		</>
	);
}
```

Recommendation:

- Using Centralized Root

```tsx
export function App() {
	return (
		<>
			<MountedGuis />
			<LoadutPromptGui />
			<DamageIndicatorGui />

			<SideMenuGui />

			<KillFeedbackGui />
			<InventoryGui />
			<CrosshairGui />

			<CentralMenuGui />
			<TopbarGui />
			<ToolTipGui />
		</>
	);
}
```

```ts
@Controller({})
export class UIController implements OnStart, OnInit {
	onInit() {}

	onStart() {
		//AVOID setting container as PlayerGui directly
		//It deletes every other Instance in PlayerGui including Roblox auto-injected ones and can break movement for mobile users
		const container = new Instance("ScreenGui");

		//The ScreenGui can be used as the container and even have other ScreenGuis, BillboardGuis and SurfaceGuis in it without breaking
		//The Screen gui can contain other ScreenGuis/BillboardGuis/SurfaceGuis without breaking their functionality

		//The most important property that has to be set
		//otherwise all UI will disappear when the player dies
		container.ResetOnSpawn = false;
		container.Parent = Players.LocalPlayer.WaitForChild("PlayerGui");

		//create root in the Controller only, because the UI can require other Controllers and if UI starts separately from the Flamework lifecycle, it will cause errors
		const root = createRoot(container);
		root.render(React.createElement(App, {}));
	}
}
```

# Explanation (@rbxts/charm)

Charm is used for state management and value storing.

Atoms act like observable values.

Example:

```ts
const valueAtom = atom<SomeType>(startValue);
const value = valueAtom(); //Getting the value
valueAtom(otherValue); //Setting the value
```

Utility functions:

```ts
//subscribe utility,
//will subscribe to the changes of atom
subscribe(valueAtom, (value) => {
	HandleValue(value);
});

//atom is a function, and you can detect and subscribe to atom at any depth of the function, no matter how nested it is
subscribe(valueAtom, (value) => {});
subscribe(
	() => valueAtom(),
	(value) => {},
);
subscribe(
	() => (() => valueAtom())(),
	(value) => {},
);

//effect utility
//will execute the function and will subscribe to any atoms inside (again at any depth)

//will run immediately and will re-run each time the subscribed atom has changed
effect(() => {
	const value = valueAtom();
	DoSomething(value);

	//returns cleanup function that will be executed if the atom will change and the callback will be re-run
	return () => {
		CleanUp();
	};
});

//effect will only subscribe to atoms that have been read
const condition = false;
effect(() => {
	if (condition) {
		const value1 = valueAtom1(); //will never be subscribed to, because it reads only Atom2, but it
		DoSomething(value1);
	} else {
		const value2 = valueAtom2();
		DoSomething(value2);
	}

	return CleanUpFunction;
});

//peek utility
//it will prevent atom from being subscribed to, at any depth
effect(() => {
	const value = peek(valueAtom); //will never get subscribed to and therefore the function in effect will never re-run;
});
```

Recommendations:

- Always add "Atom" postfix at the end of the atom variable to track that the variable is an atom

```ts
const valueAtom = atom(startValue);
```

- Always add "Atom" postfix at the end of function names that read directly atom and can be subscribed to (To track that the function is reading some atoms, and because atoms can be read at any depth, it can lead to undefined behavior when untracked)

```ts
function GetValueAtom() {
	//adding Atom to track
	return valueAtom();
}

function GetSomethingRelatedAtom() {
	//still add atom because it has the function that reads atom directly
	const value = GetValueAtom();
}
```

- Avoid "Atom" postfix for functions that use peek for ALL atoms and functions that read atoms inside

```ts
function DoSomething() {
	//No need to add Atom postfix because we prevented subscription to atom inside by using peek
	const value = peek(GetValueAtom); //because peek doesn't allow to subscribe to any atoms inside and therefore prevents chain of subscription
}
```

- The function that does something and is not meant to be subscribed to should use peek when reading all of the atoms inside

```ts
function DoSomething() {
	//as well, avoid Atom because it's not subscribable
	const value = peek(GetValueAtom); //should prevent any subscriptions
}
```

# Integration @rbxts/flamework @rbxts/charm and @rbxts/react

Since charm acts almost like jotai, it can be used to manage state in React

and following the Controller Pattern

- For full integration, these packages are required (@rbxts/flamework-react-utils, @rbxts/react-charm)

```ts
import { Controller, OnStart, OnInit } from "@flamework/core";
import { useAtom } from "@rbxts/react-charm";

@Controller({})
export class ShopMenuController implements OnStart, OnInit {
	private _isMenuVisibleAtom = atom(false);
	private _otherValueAtom = atom(otherStartValue);

	onInit() {}

	onStart() {}

	GetMenuVisibleAtom() {
		//allows to read the atom from outside of UI
		return _isMenuVisibleAtom();
	}

	SetMenuVisible(value: boolean) {
		//allows to set the atom within UI and outside of UI
		_isMenuVisibleAtom(value);
	}

	useMenuVisibleAtom() {
		//hook that provides functionality of subscribing to valueAtom
		const value = useAtom(_isMenuVisibleAtom);
		return value;
	}
}
```

UI Code:

```tsx
import { useFlameworkDependency } from "@rbxts/flamework-react-utils";
import { ShopMenuController } from "../PATH_TO_SHOP_MENU_CONTROLLER";

function ShopMenu() {
	//still cannot use Controllers directly and it requires getting them with macro
	//useFlameworkDependency is different from Dependency because it memorizes the controller once it got it
	const shopMenuController = useFlameworkDependency<ShopMenuController>();
	const isVisible = shopMenuController.useMenuVisibleAtom();

	return (
		<ShopFrame>
			<ToggleButton OnClick={() => shopMenuController.SetMenuVisible(!isVisible)} />
			<ShopMenu IsVisible={isVisible} />
		</ShopFrame>
	);
}
```

# Roblox-ts changes

### Sorting

```ts
const sortedArray = numberArray.sort((a, b) => a > b); //A function that defines the sort order. Returns true when the first element must come before the second. If omitted, the array is sorted according to the < operator.
```

### String char getting

```ts
//utility function for roblox-ts
function CharAt(str: string, index: number) {
	return str.sub(index + 1, index + 1);
}

const char = CharAt("test", 0); //"t"
```

### Getting values and keys of Map

```ts
//there's no Object.values function in roblox-ts
//there's no Object.entries function in roblox-ts
//there's no Object.keys function in roblox-ts
//therefore the utility should be used
//[...map] - to get the entries
const entries = [...map];
const values = [...map].map(([key, value]) => value);
const keys = [...map].map(([key, value]) => key);
```

### Iterating over objects

```ts
for (const [key, value] of pairs(someObject)) {
	//...code
}
```
