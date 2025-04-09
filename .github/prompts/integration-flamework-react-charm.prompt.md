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

## React Integration with Flamework Controller

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
