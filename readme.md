Small and performant implementation of the React state management idea,
inspired by well-known Zustand library (https://github.com/pmndrs/zustand) approach.

Basic usage: create and export a hook, which is returned by `create` function.
Store creator function, which takes the built-in setter `set` and returns an object
with data and actions. Async actions are handled in the same way as synchronous.
```
export const useDrinkStore = create<DrinkStore>(set => ({
    tea: 1,
    coffee: 12,
    moreTea: () => set(state => ({ tea: state.tea + 1 })),
    removeTea: () => set({ tea: 0 }),
    moreCoffee: () => set(state => ({ coffee: state.coffee + 1 })),
    fetch: async () => {
		const response = await fetch('/data.json');
		const data = await response.json();
		set({ coffee: data.coffee });
	},
}));
```
You can than use this hook in React components, getting store content either using
destructuring of the hook return value:

`const { tea, coffee } = useDrinkStore();`

or using selector function like:

`const coffee = useDrinkStore(store => store.coffee);`

You can create as many stores as you need to separate data and logic.

And finally, to persist store values to browser local storage, first `create` argument
can be wrapped in `persist` function, also provided by Jaunte. The `persist` function takes
the store creator and unique key to save and retrieve persisted store from local storage:
```
export const useDrinkStore = create<DrinkStore>(persist((set) => ({
        tea: 1,
        coffee: 12,
        moreTea: () => set((state) => ({tea: state.tea + 1})),
        removeTea: () => set({ tea: 0 }),
        moreCoffee: () => set((state) => ({coffee: state.coffee + 1})),
    }), 'drinks'),
        (state) => ({
        allDrinks: state.tea + state.coffee,
    })
)
```
Optional functionality, isn't necessarily worth to be implemented - is in 
a branch `usecomputed_overload`:

To handle a computed value, `create` function can take a second optional
argument - a function, taking the store as argument and returning object with computed values:
```
interface DrinkStore {
    tea: number;
    coffee: number;
    moreTea: () => void;
    removeTea: () => void;
    moreCoffee: () => void;
}

interface Computed {
    allDrinks: number;
}

export const [useDrinkStore, useComputed] = create<DrinkStore, Computed>((set) => ({
        tea: 1,
        coffee: 12,
        moreTea: () => set((state) => ({tea: state.tea + 1})),
        removeTea: () => set({ tea: 0 }),
        moreCoffee: () => set((state) => ({coffee: state.coffee + 1})),
    }),
        (state) => ({
        allDrinks: state.tea + state.coffee,
    })
)
```
In that case `create` function consumes two types for store and computed values
respectively and returns a tuple with hooks that return corresponding data.
