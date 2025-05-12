# usePx hook

used in @rbxts/react NOT @rbxts/roact

Do NOT use scale in properties, unless it's specific values (e.g 0.5, 1). Use Offset instead with combination of use usePx hook

**Usage**

```tsx
function SomeComponent() {
	const px = usePx();

	//...
  <frame
    Size={UDim2.fromOffset(px(420), px(240))}
    //...
  >
	//...
}
```

**API**

```ts
//baseResolution is 1080p (optional)
//dominantAxis is 0.5 (0 - prefer width, 1 - prefer height, defaults to .5) (optional)
const px: (value: number) => number = usePx(baseResolution?: Vector2 = new Vector(1920), dominantAxis?: number = 0.5);
const newValue: number = px(value);

const pxBinding: React.Binding<number> = usePxBinding(baseResolution?: Vector2 = new Vector(1920), dominantAxis?: number = 0.5);
const derivedBinding: React.Binding<UDim2> = pxBinding.map((scale: number) => UDim2.fromOffset(scale * 420, scale * 240));
```
