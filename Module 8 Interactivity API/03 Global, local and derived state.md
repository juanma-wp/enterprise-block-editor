# Global State, Local Context, and Derived State

The [Interactivity API](https://developer.wordpress.org/block-editor/reference-guides/interactivity-api/) manages three types of state: global state, local context, and derived state. Each of these serves a different purpose in managing and sharing data across blocks:

- **[Global state](https://developer.wordpress.org/block-editor/reference-guides/interactivity-api/core-concepts/undestanding-global-state-local-context-and-derived-state/#global-state):** This data is associated to a namespace and can be accessed by any interactive block on the page. It ensures that multiple blocks remain synchronized, even if they are not directly related in the DOM.
- **[Local context](https://developer.wordpress.org/block-editor/reference-guides/interactivity-api/core-concepts/undestanding-global-state-local-context-and-derived-state/#local-context):** This is data that belongs to a specific DOM element and is accessible only to that element and its child elements. It allows independent state management for separate instances of a block.
- **[Derived state](https://developer.wordpress.org/block-editor/reference-guides/interactivity-api/core-concepts/undestanding-global-state-local-context-and-derived-state/#derived-state):** These are computed values based on global state or local context. They are dynamically calculated as needed, helping to avoid redundant data storage and ensuring consistency.

Each of these concepts plays a crucial role in managing state efficiently within interactive blocks. Let's explore them in more detail with examples.

## **Global State**

[Global state](https://developer.wordpress.org/block-editor/reference-guides/interactivity-api/core-concepts/undestanding-global-state-local-context-and-derived-state/#global-state) is a data store, with a specific namespace, that any interactive block on the page can have access and modify. This is useful when multiple blocks need to stay in sync, regardless of their position in the DOM tree.

### **Server-Side State Initialization**

The global state can be initialized on the server using the [`wp_interactivity_state()`](https://developer.wordpress.org/reference/functions/wp_interactivity_state/) function. This function takes two parameters:

- The namespace (string)
- An array of initial state values

```php
<?php
// In your PHP file (e.g. render.php)
wp_interactivity_state( 'myPlugin', [
    'counter' => 0,
    'items' => ['item1', 'item2'],
    'settings' => [
        'darkMode' => false,
        'fontSize' => 16
    ]
]);
```

This state is then automatically embedded in the page's HTML and becomes available to any element using the corresponding namespace

### **Client-Side State Enhancement**

In the client side (view.js file of each block) the developer can define both the state and the elements of the store referencing functions like actions, side effects or derived state.

The [`store`](https://developer.wordpress.org/block-editor/reference-guides/interactivity-api/api-reference/#the-store) method used to set the store in JavaScript can be imported from [@wordpress/interactivity](https://developer.wordpress.org/block-editor/reference-guides/packages/packages-interactivity/).

```javascript
const { state } = store("myPlugin", {
  // Access existing server state
  state: {
    // Add new client-side only state properties
    newClientProperty: "value",

    // Add computed properties
    get doubleCounter() {
      return state.counter * 2;
    },

    // The server-initialized properties (counter, items, settings)
    // are automatically available here
  },

  // Add actions to manipulate both server-initialized
  // and client-side state
  actions: {
    increment() {
      state.counter++;
    },
    toggleDarkMode() {
      state.settings.darkMode = !state.settings.darkMode;
    },
  },
});
```

The final state resulted form merging server-side and client-side definitions for the store, is available to any element using the corresponding namespace:

```html
<div data-wp-interactive="myPlugin">
  <span data-wp-text="state.counter"></span>
  <button data-wp-on--click="actions.increment">+</button>
</div>
```

### **Private Stores**

[Private stores](https://developer.wordpress.org/block-editor/reference-guides/interactivity-api/api-reference/#private-stores) provide a **controlled way** to manage store state within a plugin, ensuring that **certain data and actions are not exposed to external namespaces**. This helps maintain **encapsulation, security, and backward compatibility** while still allowing controlled access when necessary.

For example, an e-commerce plugin might have pricing calculation logic that should not be tampered with by third-party plugins.

For this use case we can expose `addToCart` publicly, but keep `calculateDiscountedPrice` private to prevent unauthorized modifications.

```javascript
const { state, actions } = store(
  "wooCommerce/private",
  {
    state: { discountRate: 10 },
    actions: {
      calculateDiscountedPrice: (price) => price * 0.9,
    },
  },
  { lock: true }
);
```

Private stores are not merged with any server side initalized store.

### **State Synchronization**

The Interactivity API automatically handles synchronization between server-initialized state and client-side enhancements:

The server can initialize the state of a store. When JavaScript loads, the client merges any server-initialized state with any client-side state and any other elements defined for that store (actions, side-effects…). Once the client takes control, any state changes automatically update the UI through directives.

If no store has been initialized on the server for the specified `namespace` in `store`, a new one will be automatically created.

Example: Combined Server and Client State

```php
// Server-side initialization (PHP)
wp_interactivity_state( 'todoApp', [
    'todos' => [
        ['id' => 1, 'text' => 'Server todo', 'done' => false]
    ]
]);
```

```javascript
// Client-side enhancement (JavaScript)
const { state } = store("todoApp", {
  state: {
    // New client-side property
    filter: "all",

    // Computed property using both server and client state
    get filteredTodos() {
      return state.filter === "all"
        ? state.todos
        : state.todos.filter((todo) =>
            state.filter === "done" ? todo.done : !todo.done
          );
    },
  },

  actions: {
    addTodo(text) {
      state.todos.push({
        id: Date.now(),
        text,
        done: false,
      });
    },
  },
});
```

```html
<div data-wp-interactive="todoApp">
  <!-- Access both server and client state -->
  <select data-wp-bind--value="state.filter">
    <option value="all">All</option>
    <option value="done">Done</option>
    <option value="pending">Pending</option>
  </select>

  <!-- Use computed property combining both states -->
  <ul data-wp-each="state.filteredTodos">
    <li data-wp-text="context.item.text"></li>
  </ul>
</div>
```

### **Accessing and Modifying Global State**

From directives, you can reference global state in HTML using `state`:

```html
<div data-wp-bind--hidden="!state.show">
  <span data-wp-text="state.helloText"></span>
</div>
```

In JavaScript, destructure the `state` object when using `store` and reference it from the internal elements of the store

```javascript
const { state } = store("myPlugin", {
  actions: {
    toggle() {
      state.show = !state.show;
    },
  },
});
```

## **Local Context**

[Local context](https://developer.wordpress.org/block-editor/reference-guides/interactivity-api/core-concepts/undestanding-global-state-local-context-and-derived-state/#local-context) is state that belongs to a specific element in the DOM and its children. It is ideal when you need independent state for different instances of a block.

### **Setting Local Context on the Server**

#### Using PHP Function

You can initialize local context on the server using the [`wp_interactivity_data_wp_context()`](https://developer.wordpress.org/reference/functions/wp_interactivity_data_wp_context/) function:

```php
<div
    <?php echo wp_interactivity_data_wp_context( array(
        'foods' => array( 'Pie', 'Smoothie' )
    ) ); ?>
>
    <!-- Your content here -->
</div>
```

#### Using HTML Attribute

Alternatively, you can define local context directly in the HTML using the [`data-wp-context`](https://developer.wordpress.org/block-editor/reference-guides/interactivity-api/api-reference/#wp-context) attribute:

```html
<div data-wp-context='{ "counter": 0 }'>
  <span data-wp-text="context.counter"></span>
</div>
```

### **Setting Local Context on the Client**

#### Accessing and Modifying Local Context

In JavaScript, use the [`getContext()`](https://developer.wordpress.org/block-editor/reference-guides/interactivity-api/api-reference/#store-client-methods) function to access and modify the context from any action or function in the store:

```javascript
store("myPlugin", {
  actions: {
    increment() {
      const context = getContext();
      context.counter += 1;
    },
    addItem() {
      const context = getContext();
      context.items = context.items || [];
      context.items.push("New Item");
    },
  },
});
```

#### Using Context with Derived State

You can create computed values based on local context:

```javascript
store("myPlugin", {
  state: {
    get doubleCounter() {
      const context = getContext();
      return context.counter * 2;
    },
    get itemCount() {
      const context = getContext();
      return context.items?.length || 0;
    },
  },
});
```

### **Combining Local and Global State**

Local context can be used alongside global state to create complex interactions:

```
<div
    data-wp-interactive="myPlugin"
    data-wp-context='{ "items": ["Item 1", "Item 2"] }'>

    <!-- Access global state -->
    <span data-wp-text="state.globalCounter"></span>

    <!-- Access local context -->
    <ul>
        <template data-wp-each="context.items">
            <li data-wp-text="context.item"></li>
        </template>
    </ul>

    <!-- Button that uses both states -->
    <button
        data-wp-on-click="actions.addItem"
        data-wp-bind--disabled="state.globalCounter === 0">
        Add Item
    </button>
</div>
```

## **Derived State**

**Derived state** is data computed from existing global state or local context. It ensures that values remain consistent without manually updating them.

#### Defining Derived State

In JavaScript, use a getter:

```javascript
const { state } = store("myPlugin", {
  state: {
    get doubleCounter() {
      return state.counter * 2;
    },
  },
});
```

In HTML:

```
<span data-wp-text="state.doubleCounter"></span>
```

#### Example: Derived State with Local Context

```javascript
store("myCounterPlugin", {
  state: {
    get double() {
      const { counter } = getContext();
      return counter * 2;
    },
  },
  actions: {
    increment() {
      const context = getContext();
      context.counter += 1;
    },
  },
});
```

```
<div data-wp-interactive="myCounterPlugin" data-wp-context='{ "counter": 1 }'>
    Double: <span data-wp-text="state.double"></span>
    <button data-wp-on-async--click="actions.increment">Increment</button>
</div>
```

> [!NOTE]
> Check the code and demos of the following examples that illustrate the use of global, local and derived state in different implementations:
>
> - [iapi-global-local-derived-state-block](https://github.com/Automattic/wpvip-learn-enterprise-block-editor/tree/trunk/examples/iapi-global-local-derived-state-block) ([live demo](https://playground.wordpress.net/?blueprint-url=https://raw.githubusercontent.com/Automattic/wpvip-learn-enterprise-block-editor/refs/heads/trunk/examples/iapi-global-local-derived-state-block/_playground/blueprint.json))
> - [iapi-global-local-derived-state-3-blocks](https://github.com/Automattic/wpvip-learn-enterprise-block-editor/tree/trunk/examples/iapi-global-local-derived-state-3-blocks) ([live demo](https://playground.wordpress.net/?blueprint-url=https://raw.githubusercontent.com/Automattic/wpvip-learn-enterprise-block-editor/refs/heads/trunk/examples/iapi-global-local-derived-state-3-blocks/_playground/blueprint.json))

## **Resources**

- [Interactivity API Reference](https://developer.wordpress.org/block-editor/reference-guides/interactivity-api/) | Block Editor Handbook
- Interactivity API examples:
  - [Interactivity API examples](https://wordpress.github.io/block-development-examples/?tags=interactivity-api) at `block-development-examples` repo
  - [wpmovies.dev](https://wpmovies.dev/) demo and its [wp-movies-demo](https://github.com/WordPress/wp-movies-demo) repo
- [Understanding Global State, Local Context, and Derived State](https://developer.wordpress.org/block-editor/reference-guides/interactivity-api/core-concepts/undestanding-global-state-local-context-and-derived-state/) | Block Editor Handbook
- [@wordpress/interactivity Package](https://developer.wordpress.org/block-editor/reference-guides/packages/packages-interactivity/) | Block Editor Handbook
