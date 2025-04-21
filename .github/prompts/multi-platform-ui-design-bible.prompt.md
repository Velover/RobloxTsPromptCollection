# Multi platform UI design

This is the recommendation guide on how to design adaptive UI for phone, tablet, and PC platforms in Roblox.

The main framework for UI is @rbxts/react, NOT @rbxts/roact.

Do NOT use scale in properties, unless it's specific values (e.g., 0.5, 1). Use Offset instead with a combination of the usePx hook.

## usePx hook

**Usage**

```tsx
function SomeComponent() {
	const px = usePx();

	return (
		<frame
			Size={UDim2.fromOffset(px(420), px(240))}
			//...other props
		>
			{/* content */}
		</frame>
	);
}
```

**API**

```ts
//baseResolution is 1080p (optional)
//dominantAxis is 1 (0 - prefer width, 1 - prefer height, defaults to 1) (optional)
const px: (value: number) => number = usePx(baseResolution?: Vector2 = new Vector2(1920, 1080), dominantAxis?: number = 1);
const newValue: number = px(value);

//---

const pxBinding: React.Binding<number> = usePxBinding(baseResolution?: Vector2 = new Vector2(1920, 1080), dominantAxis?: number = 1);
const derivedBinding: React.Binding<UDim2> = pxBinding.map((scale: number) => UDim2.fromOffset(scale * 420, scale * 240));
```

## UI design and layout

The whole UI should be designed with a height of 1080p in mind. The px function will provide the perfect scaling for the values. The main challenge you might face is that the UI is too small or the layout overlaps, therefore there is a utility hook `useDeviceLayoutType()`.

### useDeviceLayoutType hook

#### enums

NOTE: DesktopPortrait DOES NOT exist

```ts
export const enum EDeviceLayout {
	DesktopLandscape = "DesktopLandscape",
	TabletLandscape = "TabletLandscape",
	MobileLandscape = "MobileLandscape",
	TabletPortrait = "TabletPortrait",
	MobilePortrait = "MobilePortrait",
}
```

**useDeviceLayout**
Hook that returns the current device layout with optional whitelist filtering.

```ts
// useDeviceLayout: (whiteListedLayouts?: EDeviceLayout[]) => EDeviceLayout
const deviceLayout: EDeviceLayout = useDeviceLayout();
```

**useDeviceLandscapeLayout**
Hook that returns a landscape device layout with optional whitelist filtering.
KEEP IN MIND that most of the Roblox games are made for LANDSCAPE only, therefore AVOID using other useDeviceLayoutType hooks besides `deviceLandscapeLayout` unless EXPLICITLY requested that the game should have support for PORTRAIT UI.

```ts
// useDeviceLandscapeLayout: (whiteListedLayouts?: EDeviceLayout[]) => EDeviceLayout
const deviceLandscapeLayout: EDeviceLayout = useDeviceLandscapeLayout();
```

**useDevicePortraitLayout**
Hook that returns a portrait device layout with optional whitelist filtering.

```ts
// useDevicePortraitLayout: (whiteListedLayouts?: EDeviceLayout[]) => EDeviceLayout
const devicePortraitLayout: EDeviceLayout = useDevicePortraitLayout();
```

### Recommended UI sizes

Since the game has a scaling function, all UI should be designed for a resolution upscaled to 1080px in height.
Here is a cheat sheet for recommended UI sizes for Desktop, Tablet, and Phone.

NOTE: Designing with 1080px in height will allow the person to more easily work with UI sizes in Figma with 1080px height upscaled templates for phone, tablet, and PC. And it will allow one to avoid scaling elements down to put them on a specific template, and the designer can adjust the size by eye.

NOTE: `SCALING_FACTOR` - if you want to use some particular size for the layout, multiply that value by SCALING_FACTOR and scaling function will scale it down to the needed size.

NOTE: The provided sizes in cheat sheet are ALREADY upscaled by SCALING_FACTOR.

#### Desktop

`SCALING_FACTOR` = 1

**Interactive elements**

- Smallest (24px)
- Recommended (32-40px)
- Important (48px)

**Button spacing**

- Small buttons (16-24px)
- Medium buttons (16-24px)
- Big buttons (8-16px)

**Text size**

- Small (13px)
- Recommended (14-18px)
- Important (20-24px)

#### Tablet

`SCALING_FACTOR` = 1.406

**Interactive elements**

- Smallest (62px)
- Recommended (85px)
- Important (102px)

**Button spacing**

- Small buttons (51-68px)
- Medium buttons (34-65px)
- Big buttons (17-34px)

**Text size**

- Small (20px)
- Recommended (23-29px)
- Important (31-40px)

#### Phone

`SCALING_FACTOR` = 2.748

**Interactive elements**

- Smallest (116px)
- Recommended (165px)
- Important (198px)

**Button spacing**

- Small buttons (99-132px)
- Medium buttons (66-127px)
- Big buttons (33-66px)

**Text size**

- Small (39px)
- Recommended (44-55px)
- Important (61-77px)

**Exported scaling factors:**

```ts
export const DESKTOP_SCALING_FACTOR = 1.0;
export const TABLET_SCALING_FACTOR = 1.40625;
export const MOBILE_SCALING_FACTOR = 2.7480916;
```

### Adaptive usePx

Hook that provides a scaling function optimized for the current device layout.
In the original usePx, you have to provide the `dominantAxis`.
There are variations of usePx that DO NOT require the `dominantAxis` and pick it adaptively based on the current Layout.

NOTE: It detects the current layout automatically.

```ts
const px: (value: number) => number = useAdaptivePx();
//---
const pxBinding: React.Binding<number> = useAdaptivePxBinding();
```

### useSwitchLayout hook

Hook that returns a value based on the current device layout. Can be used instead of doing complex logic of switching to pick the value for layout.

Available variations:

- useSwitchLayout
- useSwitchLandscapeLayout
- useSwitchPortraitLayout

```tsx
// useSwitchLandscapeLayout: <T>(whitelist?: EDeviceLayout[]) => (defaultValue: T, options: {	[key in EDeviceLayout]?: T;}) => T
const sw: <T>(defaultValue: T, options: { [key in EDeviceLayout]?: T }) => T =
	useSwitchLandscapeLayout();

const buttonSize = sw(new UDim2(), {
	[EDeviceLayout.DesktopLandscape]: UDim2.fromOffset(48, 48),
	[EDeviceLayout.TabletLandscape]: UDim2.fromOffset(102, 102),
	[EDeviceLayout.MobileLandscape]: UDim2.fromOffset(198, 198),
});

return <frame Size={buttonSize} />;
```

### Examples

**Layout specific size**

```tsx
function Minimap() {
	const sw = useSwitchLandscapeLayout();

	return (
		<frame
			Size={sw(new UDim2(), {
				[EDeviceLayout.DesktopLandscape]: UDim2.fromOffset(200, 200),
				[EDeviceLayout.TabletLandscape]: UDim2.fromOffset(
					//the scaling factor could be used in some cases
					150 * TABLET_SCALING_FACTOR,
					150 * TABLET_SCALING_FACTOR,
				),
				[EDeviceLayout.MobileLandscape]: UDim2.fromOffset(344, 344),
			})}
			//...rest props
		>
			<textlabel
				key={"Coordinates"}
				TextSize={sw(0, {
					[EDeviceLayout.DesktopLandscape]: 16,
					[EDeviceLayout.TabletLandscape]: 25,
					[EDeviceLayout.MobileLandscape]: 50,
				})}
				//...rest props
			/>
			{/*...children*/}
		</frame>
	);
}
```

**Layout specific components**

```tsx
function Minimap() {
	const deviceLayout = useDeviceLandscapeLayout();
	const sw = useSwitchLandscapeLayout();

	return (
		<>
			{deviceLayout === EDeviceLayout.MobileLandscape && <SomeMobileLandscapeSpecificElement />}
			{sw(undefined, {
				[EDeviceLayout.DesktopLandscape]: <MinimapDesktopView />,
				[EDeviceLayout.TabletLandscape]: <MinimapTabletView />,
				//didn't provide view for phones
			})}

	);
}
```

## useInputType hook

Integration with @rbxts/input-actions, detects the current input type.

**enums**

```ts
EInputType.KeyboardAndMouse;
EInputType.Touch;
EInputType.Gamepad;
```

**Example**

```tsx
export function Controls() {
	const inputType = useInputType();
	const isTouch = inputType === EInputType.Touch;
	const isPc = inputType === EInputType.KeyboardAndMouse;

	return (
		<>
			{isTouch && <TouchControls />}
			{isPc && <KeybindsText />}
		</>
	);
}
```
