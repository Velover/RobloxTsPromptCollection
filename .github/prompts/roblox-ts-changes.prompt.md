# Roblox-ts Changes

## Sorting

```ts
const sortedArray = numberArray.sort((a, b) => a > b); //A function that defines the sort order. Returns true when the first element must come before the second. If omitted, the array is sorted according to the < operator. IT IS CORRECT WAY, because the roblox-ts uses luau sort
```

## String char getting

```ts
//utility function for roblox-ts
function CharAt(str: string, index: number) {
	return str.sub(index + 1, index + 1);
}

const char = CharAt("test", 0); //"t"
```

## Getting values and keys of Map

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

## Iterating over objects

```ts
for (const [key, value] of pairs(someObject)) {
	//...code
}
```
