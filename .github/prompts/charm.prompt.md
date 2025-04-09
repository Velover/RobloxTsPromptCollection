# @rbxts/charm

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

## Recommendations:

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
