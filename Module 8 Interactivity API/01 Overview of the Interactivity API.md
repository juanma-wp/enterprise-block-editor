## **Overview of the Interactivity API**

The [Interactivity API](https://developer.wordpress.org/block-editor/reference-guides/interactivity-api/), introduced in WordPress 6.5, is a **standard** way to add interactivity to the **frontend** of WordPress—whether inside blocks or other parts of a page. It provides a consistent method for developers to enhance user experiences with dynamic features.

_[From the docs](https://developer.wordpress.org/block-editor/reference-guides/interactivity-api/iapi-about/):_

> The Interactivity API is **a [standard](https://developer.wordpress.org/block-editor/reference-guides/interactivity-api/iapi-about/#why-a-standard) system of [directives](https://developer.wordpress.org/block-editor/reference-guides/interactivity-api/iapi-about/#why-directives), based on declarative code, for [adding frontend interactivity to blocks](https://developer.wordpress.org/block-editor/reference-guides/interactivity-api/iapi-about/#api-goals)**.

> **Directives extend HTML with special attributes** that tell the Interactivity API to attach a specified behavior to a DOM element or even to transform it.

This API is already used in several built-in WordPress blocks, such as Search, Query, Navigation, and File. It allows developers to build anything from simple interactions like counters and pop-ups to more advanced functionality like instant navigation, live search, shopping carts, and checkouts.

Because it follows a standardized approach, blocks and other frontend elements can easily share data, actions, and callbacks. This makes interactions smoother and reduces errors. For example, clicking an "add to cart" button can instantly update a separate shopping cart block.

## **Key Features of the Interactivity API**

The [Interactivity API](https://developer.wordpress.org/block-editor/reference-guides/interactivity-api/api-reference/) is a standard system of directives for adding frontend interactivity to WordPress blocks. Its key features include:

- **[Directive-based Approach](https://developer.wordpress.org/block-editor/reference-guides/interactivity-api/iapi-about/#why-directives)**
  - Uses HTML attributes as directives to extend HTML functionality
  - Allows attaching behaviors to DOM elements or transforming them
- **[Server-side Rendering Support](https://developer.wordpress.org/block-editor/reference-guides/interactivity-api/core-concepts/server-side-rendering/)**
  - Works seamlessly with PHP and the current block system
  - Supports dynamic blocks and server-side rendering
- **[Declarative and Reactive](https://developer.wordpress.org/block-editor/reference-guides/interactivity-api/core-concepts/the-reactive-and-declarative-mindset/)**
  - Uses declarative code to describe desired outcomes
  - Automatically updates UI in response to data changes
- **[Performant Design](https://developer.wordpress.org/block-editor/reference-guides/interactivity-api/iapi-about/#performant)**
  - Runtime code for directives is only \~10 KB
  - Scripts load without blocking page rendering
- **[Compatibility](https://developer.wordpress.org/block-editor/reference-guides/interactivity-api/iapi-about/#compatible-with-the-existing-block-development-tooling)**
  - Works with WordPress hooks and internationalization
  - Compatible with existing JS libraries on the site

## **Benefits of the Interactivity API**

- **Standardization**
  - Provides a standard way to handle frontend interactivity for Gutenberg blocks
  - Enables easier creation of rich, interactive user experiences
- **Improved Developer Experience**
  - Reduces complexity for developers by handling common challenges
  - Works out-of-the-box with standard block-building tools
- **Enhanced Block Interaction**
  - Facilitates inter-block communication
  - Allows for composability and compatibility between interactive blocks
- **Performance Optimization**
  - Reduces the amount of JavaScript sent to the browser
  - Enables site-wide features like client-side navigation
- **WordPress Integration**
  - PHP-friendly and compatible with WordPress hooks
  - Works seamlessly with WordPress backend APIs like i18n
- **Security**
  - Prevents XSS attacks by not allowing JavaScript injection into directives
  - Compatible with custom security policies
- **Flexibility**
  - Can be used beyond blocks for adding interactive behaviours to any part of WordPress
  - Allows for gradual adoption and coexistence with other interactive approaches

## **Example of using Interactivity API in a block**

To illustrate the capabilities of the Interactivity API, let's explore an example using the Interactivity API

To [create this example](https://developer.wordpress.org/block-editor/reference-guides/interactivity-api/iapi-quick-start-guide/) go to the “plugins” folder of your WordPress installation and run:

```
npx @wordpress/create-block@latest my-first-interactive-block --template @wordpress/create-block-interactive-template
```

In this example, we will create a simple toggle block that expands or collapses content when a button is clicked.

### Pre-requisites of a block using the Interactivity API

In the scaffolded `my-first-interactive-block` example, we can see the following [parts that are required for any block using the Interactivity API](https://developer.wordpress.org/block-editor/reference-guides/interactivity-api/api-reference/#requirements):

- The [`@wordpress/interactivity`](https://developer.wordpress.org/block-editor/reference-guides/packages/packages-interactivity/) package is installed (and a dependency declared on package.json)
- [`interactivity`](https://developer.wordpress.org/block-editor/reference-guides/block-api/block-supports/#interactivity) support is added to `block.json`
- The Interactivity API JavaScript code is loaded with [`viewScriptModule`](https://developer.wordpress.org/block-editor/reference-guides/block-api/block-metadata/#view-script-module) in `block.json`, which loads the file in the frontend using script modules
- The [`wp-interactive`](https://developer.wordpress.org/block-editor/reference-guides/interactivity-api/api-reference/#wp-interactive) directive is added to a DOM element in the render.php file which enables the Interactivity API for that portion of the DOM

_`block.json`_

```javascript
{
	...
	"supports": {
		"interactivity": true
	},
	...
	"render": "file:./render.php",
	"viewScriptModule": "file:./view.js"
}
```

- The `wp-interactive` directive is added to a DOM element in the render.php file which enables the Interactivity API for that portion of the DOM

_`render.php`_

```php
<div
	<?php echo get_block_wrapper_attributes(); ?>
	data-wp-interactive="create-block"
	...
>
...
</div>
```

### Implementing directives in render.php

The Interactivity API usually works better blocks with "dynamic rendering" that use a `render.php` to define the frontend output for the block. In this case, directives are added to this file:

```javascript
<?php
// Adds the global state.
wp_interactivity_state(
	'create-block',
	array(
		'isDark'    => false,
		'darkText'  => esc_html__( 'Switch to Light', 'my-first-interactive-block' ),
		'lightText' => esc_html__( 'Switch to Dark', 'my-first-interactive-block' ),
		'themeText' => esc_html__( 'Switch to Dark', 'my-first-interactive-block' ),
	)
);

?>

<div
	...
	data-wp-interactive="create-block"
	<?php echo wp_interactivity_data_wp_context( array( 'isOpen' => false ) ); ?>
	data-wp-watch="callbacks.logIsOpen"
	data-wp-class--dark-theme="state.isDark"
>
	<button
		data-wp-on--click="actions.toggleTheme"
		data-wp-text="state.themeText"
	></button>

	<button
		data-wp-on--click="actions.toggleOpen"
		data-wp-bind--aria-expanded="context.isOpen"
		...
	>
		<?php esc_html_e( 'Toggle', 'my-first-interactive-block' ); ?>
	</button>

	<p
		...
		data-wp-bind--hidden="!context.isOpen"
	>
		...
	</p>
</div>
```

The code above initializes data on the server to establish the initial state used for rendering the HTML displayed on the frontend:

- The [wp_interactivity_state](https://developer.wordpress.org/reference/functions/wp_interactivity_state/) function sets the initial global state for the 'create-block' store
- The [wp_interactivity_data_wp_context](https://developer.wordpress.org/reference/functions/wp_interactivity_data_wp_context/) sets the initial local context state for a DOM element (it internally generates a [data-wp-context](https://developer.wordpress.org/block-editor/reference-guides/interactivity-api/api-reference/#wp-context) directive on the rendered HTML=)

The code above uses the following directives:

- The [wp-interactive](https://developer.wordpress.org/block-editor/reference-guides/interactivity-api/api-reference/#wp-interactive) directive is added to the parent DOM element to enable the Interactivity API in that portion of the DOM also providing access to the referenced `create-block` store and its global state
- The [data-wp-watch](https://developer.wordpress.org/block-editor/reference-guides/interactivity-api/api-reference/#wp-watch) runs the referenced callback function when the node is created and when the state (or context) used inside the callback function changes
- The [data-wp-class--dark-theme](https://developer.wordpress.org/block-editor/reference-guides/interactivity-api/api-reference/#wp-class) add the class \`dark-theme\` to the DOM element if the referenced state value is true
- The [data-wp-on--click](https://developer.wordpress.org/block-editor/reference-guides/interactivity-api/api-reference/#wp-on) attach the referenced action to the click event of the button
- The [data-wp-text](https://developer.wordpress.org/block-editor/reference-guides/interactivity-api/api-reference/#wp-text) displays the content of the referenced state value as text of the element
- The [data-wp-bind--hidden](https://developer.wordpress.org/block-editor/reference-guides/interactivity-api/api-reference/#wp-bind) adds the \`hidden\` attribute to the DOM element when the reference state value is true

### Implementing store in view.js

The Interactivity API usually loads a `view.js` javascript in the frontend where a store can be created (or updated if it has been previously initialized in the server) with actions, callbacks and more state values.

```javascript
/**
 * WordPress dependencies
 */
import { store, getContext } from "@wordpress/interactivity";

const { state } = store("create-block", {
  state: {
    get themeText() {
      return state.isDark ? state.darkText : state.lightText;
    },
  },
  actions: {
    toggleOpen() {
      const context = getContext();
      context.isOpen = !context.isOpen;
    },
    toggleTheme() {
      state.isDark = !state.isDark;
    },
  },
  callbacks: {
    logIsOpen: () => {
      const { isOpen } = getContext();
      // Log the value of `isOpen` each time it changes.
      console.log(`Is open: ${isOpen}`);
    },
  },
});
```

In the code above:

- The `store` method is used to create the 'create-block' store (merging any data that has been initialized in the server for that store's namespace )
- The `useContext` method is used inside callbacks and actions to get access to the local context of the element that triggered the call of function
- The store is defined with some state, actions and callbacks. All elements under [wp-interactive](https://developer.wordpress.org/block-editor/reference-guides/interactivity-api/api-reference/#wp-interactive) directive pointing to this store will have access to these state values, actions and callbacks
- The actions will update state values. When state values are updated the DOM elements with directives pointing to these state values "react" by updating their attributes or displayed values accordingly

In this example, when the button is clicked, it triggers the `toggle` action which updates the `isOpen` state defined as [local context](https://developer.wordpress.org/block-editor/reference-guides/interactivity-api/core-concepts/undestanding-global-state-local-context-and-derived-state/#local-context) of the `div`. The paragraph's visibility is then dynamically controlled based on this state.

—

The declarative nature of this API simplifies complex interactions into manageable code snippets while ensuring seamless integration with WordPress's architecture.
