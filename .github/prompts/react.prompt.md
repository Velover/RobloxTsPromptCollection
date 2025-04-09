# @rbxts/react

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

## Recommendation:

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
//AVOID setting container as PlayerGui directly
//It deletes every other Instance in PlayerGui including Roblox auto-injected ones and can break movement for mobile users
const container = new Instance("ScreenGui");

//The ScreenGui can be used as the container and even have other ScreenGuis, BillboardGuis and SurfaceGuis in it without breaking
//The Screen gui can contain other ScreenGuis/BillboardGuis/SurfaceGuis without breaking their functionality

//The most important property that has to be set
//otherwise all UI will disappear when the player dies
container.ResetOnSpawn = false;
container.Parent = Players.LocalPlayer.WaitForChild("PlayerGui");

const root = createRoot(container);
root.render(React.createElement(App, {}));
```
