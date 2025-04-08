//Still ignores the values() guidance

```ts
Functions.getTreasureLocations.setCallback(() => {
  return [...this.treasures.values()];
});
```

Fails to sort

```ts
this.leaderboard.sort((a, b) => b.score - a.score);
```

Array slicing

```ts
// Keep only top 10 scores
if (this.leaderboard.size() > 10) {
  this.leaderboard = this.leaderboard.slice(0, 10);
}
```

Incorrect math library

```ts
// Time bonus (faster completion = higher score)
const timeBonus = Math.max(0, 3000 - timeElapsed * 10);
```

toString() doesnt exist

```ts
generateTreasureId(): string {
	return `treasure_${math.random().toString().sub(3, 10)}`;
}
```

Character tracking fail?

```ts
// Character tracking
private character?: Character;
```

- Fails to setup and use Flamework events
- Overuses Systems
- still uses binding for the function assigment

```ts
connect(this.SomeFunction.bind(this));
```
