# Creating Custom Data Stores

In the WordPress ecosystem, custom data stores play a crucial role in managing application state efficiently. By leveraging the WordPress Data Layer and wp.data, developers can create powerful and flexible state management solutions for their plugins and themes.

## Introduction to the WordPress Data Layer and wp.data

The WordPress Data Layer, facilitated by the `@wordpress/data` package, provides a centralized approach to managing state across WordPress applications. It is inspired by Redux, a predictable state container for JavaScript applications, but tailored to fit seamlessly into the WordPress ecosystem. This system allows developers to manage and share data across various components and plugins efficiently.

### State Management with Redux Principles

At its core, the `wp.data` package adopts Redux-like principles for state management, which include:

- **Store**: Holds the application state.
- **Actions**: Plain JavaScript objects that describe events affecting the state.
- **Reducers**: Pure functions that determine how the state evolves in response to actions.
- **Selectors**: Functions that retrieve specific pieces of state.
- **Resolvers**: Functions that handle asynchronous operations to fetch data and populate the state.

By adhering to these principles, developers can create predictable and maintainable state management solutions within WordPress.

## Creating and registering a Custom Store

The `register` function (from the @wordpress/data package) is the interface for registering a store with `wp.data`. By using this function, you custom store gets integrated into the centralized data registry.

In the WordPress Block Editor (Gutenberg), each data store within `wp.data` must be registered, and numerous [examples of this can be found throughout the Gutenberg codebase](https://github.com/search?q=repo%3AWordPress%2Fgutenberg+register%28+store+%29&type=code).

When registering a custom data store, you typically use `createReduxStore` and `register`. These functions are provided by the `@wordpress/data` package and help define and manage custom state in Gutenberg.

### createReduxStore

`createReduxStore` is a function used to create a Redux-like store that manages state in Gutenberg. It returns an object representing the store with actions, selectors, and a reducer.

Example:

```javascript
import { createReduxStore } from '@wordpress/data';

const DEFAULT_STATE = {
    items: [],
};

const actions = {
    addItem(item) {
        return {
            type: 'ADD_ITEM',
            item,
        };
    },
};

function reducer(state = DEFAULT_STATE, action) {
    switch (action.type) {
        case 'ADD_ITEM':
            return {
                ...state,
                items: [...state.items, action.item],
            };
        default:
            return state;
    }
}

const selectors = {
    getItems(state) {
        return state.items;
    },
};


const myStore = createReduxStore(''my-custom-store', {
    reducer,
    actions,
    selectors
});

```

### register

Once you've created the store using `createReduxStore`, you need to **register** it using `register` from `@wordpress/data`. This makes the store available throughout Gutenberg, allowing components and blocks to access and interact with its state.

```javascript
import { register } from "@wordpress/data";

register(myStore);
```

## **Using Selectors and Resolvers**

Selectors are functions that extract specific pieces of state from the stores that are registered. They help in accessing data without directly manipulating the state. Here's an example of using a selector:

After registering the store, you can use it within your React components using `useSelect`.

```javascript
import { useSelect } from '@wordpress/data';

const MyComponent = () => {
    const count = useSelect((select) => select(''my-custom-store').getItems().length, []);
    return <p>Num Items: {count}</p>;
```

Resolvers are special functions that can be used to fetch data asynchronously the first time a selector is called. They are particularly useful for handling API requests. A resolver's name should match that of its corresponding selector.

Here's an example of a resolver of the previously defined \``getItems`\` selector:

```javascript
import { fetchItemsFromAPI } from './api'; // Assuming fetchItemsFromAPI is imported from an API module
import { setItems } from './actions'; // Assuming setItems is an action creator
import { createReduxStore } from '@wordpress/data';

// actions definition
// selectors definition
// reducer definition

const resolvers = {
  getItems() {
    return async ( { dispatch } ) => {
      const items = await fetchItemsFromAPI();
      dispatch(setItems(items));
    };
  },
};

const myStore = createReduxStore(''my-custom-store', {
    reducer,
    actions,
    selectors,
   resolvers
});
```

## **Using Thunks for Async Operations**

Thunks are a powerful way to handle asynchronous operations in your custom store. They allow you to write action creators that return a function instead of an action object. This function can perform async logic and dispatch actions when needed5.

Thunks can be used in actions and resolvers.

Here's an example of using a thunk for an async operation:

```javascript
const actions = {
  fetchItems:
    () =>
    async ({ dispatch }) => {
      dispatch({ type: "FETCH_ITEMS_START" });
      try {
        const response = await fetch("/api/items");
        const items = await response.json();
        dispatch({ type: "FETCH_ITEMS_SUCCESS", items });
      } catch (error) {
        dispatch({ type: "FETCH_ITEMS_ERROR", error: error.message });
      }
    },
};
```

In this example, `fetchItems` is a thunk that performs an API request and dispatches different actions based on the result.

## **Using useRegistry Hook**

The `useRegistry` hook provides access to the \*\*data registry\*\* within a React component, allowing you to interact with custom data stores dynamically within your components.

The \*\*data registry\*\* is an object that holds references to all the registered stores. It provides methods to:

- \- Register new stores.
- \- Access existing stores.
- \- Dispatch actions to modify state.
- \- Select data from stores.
- \- Subscribe to changes in the store.

For example. If you need to register or unregister a store dynamically during the lifecycle of a component, `useRegistry` is required. `useSelect` and `useDispatch` don't provide this capability.

```javascript
import { useRegistry } from "@wordpress/data";
import { useEffect } from "react";

function MyComponent() {
  const registry = useRegistry();

  useEffect(() => {
    // Dynamically register a store
    registry.registerStore("my-dynamic-store", {
      reducer: (state = {}) => state,
      actions: {
        doSomething: () => ({ type: "DO_SOMETHING" }),
      },
    });

    // Cleanup: Unregister the store on unmount
    return () => {
      registry.unregisterStore("my-dynamic-store");
    };
  }, [registry]);

  return <div>Dynamic Store Example</div>;
}
```

The `useRegistry` hook is considered a low-level utility and is rarely needed in typical implementations. In most cases, interactions with the `@wordpress/data` API can be effectively managed using the `useSelect` or `useDispatch` hooks.

However, `useRegistry` becomes useful in scenarios requiring more advanced operations that go beyond what `useSelect` and `useDispatch` offer, such as:

- Dynamically registering a new store.
- Interacting with multiple stores in ways not supported by the simpler hooks.
- Directly accessing the registry for debugging or testing purposes.

### batch

There’s a handy method of registry called \`batch\` that can avoid unnecessary re-renders in React components for dispatch actions.[This method is widely used in the “gutenberg” repo](https://github.com/search?q=repo%3AWordPress%2Fgutenberg+registry.batch&type=code)

In WordPress data-based applications, dispatch calls trigger component updates in two steps:

1. Selectors run with the updated state.
2. If the new selector values differ (strict equality), the component re-renders.

As applications scale, frequent updates can become expensive. When multiple dispatch calls are needed in sequence, wrapping them in `batch` ensures that selectors run and components re-render only once, improving performance.

```javascript
import { useRegistry } from "@wordpress/data";

function Component() {
  const registry = useRegistry();

  function callback() {
    registry.batch(() => {
      registry.dispatch(someStore).someAction();
      registry.dispatch(someStore).someOtherAction();
    });
  }

  return <button onClick={callback}>Click me</button>;
}
```

This approach optimizes state updates by minimizing unnecessary re-renders

—

Custom data stores in WordPress provide a powerful way to manage application state. By understanding how to register stores, use selectors and resolvers, implement thunks for async operations, and leverage the `useRegistry` hook, developers can create robust and scalable state management solutions for their WordPress projects.
