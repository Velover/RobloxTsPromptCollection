# @rbxts/react Basics

@rbxts/react is a TypeScript package that provides React-like functionality for Roblox TypeScript (roblox-ts) projects, following the syntax of regular React v17.2.3. It is important to note that @rbxts/react is NOT the same as @rbxts/roact.

## Core Features and Syntax

@rbxts/react allows you to create UI components using TSX syntax, similar to how you would in a web React application:

```tsx
function App() {
  return (
    <screengui ResetOnSpawn={false}>{/* Some other children */}</screengui>
  );
}
```

You can create components with children using the PropsWithChildren type:

```tsx
function ComponentWithChildren({ children }: PropsWithChildren<{}>) {
  return <frame>{children}</frame>;
}
```

Fragments are also supported, allowing you to group elements without adding extra nodes:

```tsx
function ComponentWithFragment() {
  return (
    <>
      <frame />
      <frame />
      <frame />

  );
}
```

## Centralized App Structure

It's preferred to have a centralized app structure with fragments containing all GUIs for better debugging. This approach organizes your UI components in a single root component:

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

  );
}
```

This centralized structure makes it easier to manage, debug, and understand the hierarchy of your UI components.

## Mounting Your App

The recommendation is to use a Folder instead of ScreenGui as the container, as shown in the npm @rbxts/react documentation:

```tsx
import React, { StrictMode } from "@rbxts/react";
import { createPortal, createRoot } from "@rbxts/react-roblox";
import { Players } from "@rbxts/services";

const playerGui = Players.LocalPlayer.FindFirstChildOfClass("PlayerGui");
const root = createRoot(new Instance("Folder"));
root.render(<StrictMode>{createPortal(<App />, playerGui)}</StrictMode>);
```

This approach is more flexible and aligns with the official documentation. Using a Folder with createPortal ensures that your UI components are properly mounted without disrupting other PlayerGui elements.

## Bindings

Bindings are the states that dont trigger the re-render, but they change the property.
Bindings allow very rapid changes.
Bindings are exlusive to @rbxts/react.

**Usage**

```tsx
const [binding, setBinding] = useBinding(initialState);

useEffect(() => {
  //allows rapid changes without re-rendering whole component
  const connection = RunService.RenderStepped.Connect((dt) => {
    const value = updateSpring(dt);
    setBinding(value);
  });

  return () => connection.Disconnect();
}, []);

const derivedBinding = binding.map((value) => {
  return value ** 2;
});

//passing binding directly into the property allowing it to subscribe to it
return (
  <frame
    Transparency={derivedBinding}
    //...
  />
);
```
