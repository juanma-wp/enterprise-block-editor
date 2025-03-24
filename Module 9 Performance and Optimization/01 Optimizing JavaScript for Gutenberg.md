# Optimizing JavaScript for Gutenberg

## The Block Rendering Cycle in the Block Editor

Gutenberg, the WordPress Block Editor, follows a structured rendering cycle for blocks. Understanding this cycle is crucial for identifying opportunities to optimize performance. The rendering cycle involves the following stages:

1. **Block Registration** - [Blocks are registered](https://developer.wordpress.org/block-editor/getting-started/fundamentals/registration-of-a-block/) in the client-side using `registerBlockType`, defining their attributes, edit behavior, and save behavior.
2. **Editing Lifecycle** - When a block is added or modified, the `edit` function renders the React component responsible for the UI. State updates can cause re-renders.
3. **Saving and Serializing** - The `save` function determines [the markup stored](https://developer.wordpress.org/block-editor/getting-started/fundamentals/markup-representation-block/) in post content.
4. **Frontend Rendering** - When the post is viewed, WordPress parses the saved markup and [renders it accordingly](https://developer.wordpress.org/block-editor/getting-started/fundamentals/static-dynamic-rendering/):
   - **Static Blocks**: These store their output directly in post content, meaning no additional processing in the server is needed when the post is displayed.
   - **Dynamic Blocks**: These rely on server-side rendering, where the content is generated each time the post is requested, typically via a PHP `render_callback` function.

## Code Splitting

Code splitting allows developers to break their JavaScript code into smaller, more manageable chunks that can be loaded on demand. Instead of loading a single large JavaScript file, code splitting produces multiple smaller files that are requested only when needed.

### Customizing `@wordpress/scripts` for code splitting

The [`@wordpress/scripts` package internally uses webpack](https://developer.wordpress.org/block-editor/reference-guides/packages/packages-scripts/#advanced-usage) with a pre-configured setup optimized for WordPress development. While this default configuration handles most common use cases, you can extend and customize it by creating your own `webpack.config.js` file. This allows you to define specific settings like custom entry points while still leveraging the base configuration (see [`gutenberg/packages/scripts/config/webpack.config.js`](https://github.com/WordPress/gutenberg/blob/f90f754367e222b34dfebfaea559dcc4125af3b8/packages/scripts/config/webpack.config.js)).

Here's an example of how to customize the webpack configuration in your `webpack.config.js`:

```javascript
const {
  defaultConfig,
  plugins,
} = require("@wordpress/scripts/config/webpack.config");
const path = require("path");

module.exports = {
  ...defaultConfig,
  entry: {
    main: path.resolve(process.cwd(), "resources/js/module.js"),
    editor: path.resolve(process.cwd(), "resources/js/module.js"),
  },
};
```

After running `@wordpress/scripts` with this webpach configuration, you might get the following output files:

- `build/module.js`: Contains code from `resources/js/module.js` meant to be loaded as a Javascript modules using the Script Modules API.
- `build/editor.js`: Contains code from `resources/js/editor.js` meant to be loaded in the block editor to extend the functionality of some core blocks.

## Lazy Loading

Lazy loading defers the loading of non-critical resources until they are needed.

While there is interest and some experimentation, Lazy loading is not yet fully implemented in the Block Editor . The core team is exploring ways to make this work without breaking existing functionality or compromising user experience. See [#2768](https://github.com/WordPress/gutenberg/issues/2768) Issue and [#53260](https://github.com/WordPress/gutenberg/discussions/53260) discussion for more info

For the frontend though, there's a recent API we can leverage to load dynamically JavaScript modules on demand: The Script Modules API

### Script Modules API

WordPress 6.5 introduced the Script Modules API, providing native support for ECMAScript Modules (ESM). This API enables better code splitting and dependency management.

The Script Modules API provides two main functions for working with modules: `wp_register_script_module()` for registering modules that can be used later, and `wp_enqueue_script_module()` for both registering and immediately loading modules. Here's how to use them:

```php
// Register a module for later use
wp_register_script_module(
    '@my-plugin/core-functionality',
    plugin_dir_url( __FILE__ ) . 'build/core.js'
);

// Register and enqueue a module with dependencies
wp_enqueue_script_module(
    '@my-plugin/editor-enhancements',
    plugin_dir_url( __FILE__ ) . 'build/editor.js',
    array( '@my-plugin/core-functionality' )
);
```

Once a script has been registered as a Script Module, we can load it dynamically allows blocks to load features on demand.

```javascript
// In your main module
document.getElementById("load-feature").addEventListener("click", async () => {
  // Dynamically import the module only when needed
  const { initializeFeature } = await import("@my-plugin/advanced-feature");
  initializeFeature();
});
```

To specify dynamic dependencies in PHP:

```php
wp_enqueue_script_module(
    '@my-plugin/main',
    plugin_dir_url( __FILE__ ) . 'build/main.js',
    array(
        array(
            'id' => '@my-plugin/advanced-feature',
            'import' => 'dynamic' // Marks as dynamically imported
        )
    )
);
```

The `import` parameter accepts two values that determine how dependencies are loaded:

- `static` (default): Loads the module immediately with the main script, similar to standard ES6 imports. Use for critical functionality needed right away.
- `dynamic`: Loads the module only when requested by code (lazy-loading), similar to `import()` in JavaScript. Use for large features or specialized functionality that isn't always needed.

Using the appropriate import type can significantly improve performance by reducing initial load time while ensuring optimal user experience.

#### Script Modules API in blocks

- Use of `viewScriptModule`
- custom `webpack.config.js` to handle imports as JS modules
- Using scripts inside script modules

## Reducing Bundle Size

Tree shaking is a key optimization technique in modern JavaScript bundling that removes unused code from the final output. When building custom blocks or extending Gutenberg functionality, applying best practices can significantly reduce your bundle size.

### Use Modular Imports

Always import only the specific functions or components you need, rather than entire packages:

```javascript
// Less efficient - imports entire modules
import * as blocks from "@wordpress/blocks";
import * as components from "@wordpress/components";

// More efficient - imports only what's needed
import { createBlock } from "@wordpress/blocks";
import { Button, Panel } from "@wordpress/components";
```

### Eliminate Module-Level Side Effects

Avoid introducing [side effects](https://github.com/WordPress/gutenberg/blob/trunk/packages/side-effects.md) at the top level of your modules. Move logic like block registration into functions:

```javascript
// Has side effects
import { registerBlockType } from "@wordpress/blocks";

registerBlockType("my-plugin/my-block", {
  /* block config */
});

// Side-effect free
import { registerBlockType } from "@wordpress/blocks";

export function initializeBlock() {
  registerBlockType("my-plugin/my-block", {
    /* block config */
  });
}
```

This structure enables bundlers to apply [tree shaking](https://webpack.js.org/guides/tree-shaking/) more effectively. To further assist bundlers, you can also declare side-effect-free files explicitly in your `package.json` using the `sideEffects` field. This helps avoid unintentional exclusion of needed code.

### Inspect Bundle Contents

Use visualization tools like `webpack-bundle-analyzer` to identify large modules and optimize your dependency graph:

```javascript
// webpack.config.js
const { BundleAnalyzerPlugin } = require("webpack-bundle-analyzer");
const defaultConfig = require("@wordpress/scripts/config/webpack.config");

module.exports = {
  ...defaultConfig,
  plugins: [...defaultConfig.plugins, new BundleAnalyzerPlugin()],
};
```

### Reuse Core WordPress Scripts

Avoid bundling libraries that WordPress already provides. Instead, rely on the script dependency system:

```php
wp_enqueue_script(
    'my-block-editor',
    plugins_url( 'build/index.js', __FILE__ ),
    array( 'wp-blocks', 'wp-element', 'wp-components' ),
    filemtime( plugin_dir_path( __FILE__ ) . 'build/index.js' )
);
```

### Code Splitting and Dynamic Imports

For larger plugins, use code splitting and dynamic imports to load parts of your UI only when needed:

```javascript
// Dynamic import example
const AdvancedBlockControls = () => {
  const [Controls, setControls] = useState(null);

  useEffect(() => {
    import("./advanced-controls").then((module) => {
      setControls(() => module.default);
    });
  }, []);

  if (!Controls) {
    return <Placeholder>Loading controls...</Placeholder>;
  }

  return <Controls />;
};
```

By combining modular imports, avoiding side effects, leveraging WordPress dependencies, and applying dynamic loading, you can ensure your Gutenberg extensions remain lightweight and performant, enhancing the editing experience for all users.

## Caching Strategies for Block Data and REST API Responses

Caching is a critical performance optimization technique for WordPress applications, particularly important in the context of the Block Editor where multiple REST API requests can impact user experience. Implementing effective caching strategies can significantly reduce load times and improve editor responsiveness.

### Caching Layers in WordPress and the Block Editor

The Block Editor leverages WordPress's multi-layered caching system to optimize performance and responsiveness when working with blocks:

#### Server-Side Caching

1. **Object Cache**

   - **Definition**: Persistent storage for frequently accessed data across multiple requests
   - **Implementation**: Uses `wp_cache_get()` and `wp_cache_set()` functions
   - **Storage Options**: Memory (Memcached, Redis) or database (transients in wp_options table)
   - **Block Editor Context**: Server-side caching of REST API responses that blocks might request

#### Client-Side Caching

4. **WordPress Data Layer Cache**

   - **Definition**: In-memory JavaScript state management specifically for WordPress data
   - **Implementation**: Through the `@wordpress/data` package's selector and resolver system
   - **Example**: When `useSelect` retrieves post data, it's stored in memory and reused
   - **Block Editor Context**: Primary caching mechanism for block data within the editor

5. **React Component Cache**

   - **Definition**: Memoization of components, callbacks, and computed values
   - **Implementation**: Using React's `useMemo` and `useCallback` hooks
   - **Example**: Preventing expensive calculations when props haven't changed
   - **Block Editor Context**: Optimizing block rendering performance

6. **Browser HTTP Cache**

   - **Definition**: Browser's built-in storage of network responses
   - **Implementation**: Controlled via HTTP headers (Cache-Control, ETag)
   - **Example**: Caching of JavaScript, CSS, images, and API responses
   - **Block Editor Context**: Caching of block assets and REST API responses

7. **Browser Storage APIs**
   - **Definition**: Programmatic storage in the browser
   - **Implementation**: LocalStorage, SessionStorage, IndexedDB
   - **Example**: Storing user preferences or draft content
   - **Block Editor Context**: Custom persistent storage for block data that should survive page refreshes

This layered approach provides multiple opportunities to optimize performance at different levels of the application stack, with each layer having distinct characteristics in terms of persistence, scope, and implementation approach.

### Leveraging the WordPress Data Layer

The WordPress Data Layer provides a robust framework for managing application state and data persistence. It's specifically designed to handle REST API interactions efficiently within the Block Editor.

```javascript
import { useSelect, useDispatch } from "@wordpress/data";
import { store as coreDataStore } from "@wordpress/core-data";

function MyBlockEditor() {
  // Retrieve posts from the store with automatic caching
  const { posts, isLoading } = useSelect((select) => {
    return {
      posts: select(coreDataStore).getEntityRecords("postType", "post", {
        per_page: 5,
      }),
      isLoading: select(coreDataStore).isResolving("getEntityRecords", [
        "postType",
        "post",
        { per_page: 5 },
      ]),
    };
  }, []);

  // Component rendering logic
}
```

The Data Layer automatically implements caching, avoiding duplicate network requests for the same data. When you use selectors like `getEntityRecords`, the Data Layer:

1. Checks if the data already exists in its store
2. Makes the API request only if necessary
3. Caches the response for future use
4. Provides invalidation mechanisms when data changes

#### Strategies for Optimizing Block Data Caching with WordPress Data Layer

##### 1. Selective Data Loading

Load only the data that blocks immediately need, deferring other data retrieval.

```javascript
// Instead of loading all post meta at once
const { meta } = useSelect((select) => ({
  meta: select(coreDataStore).getEditedEntityRecord(
    "postType",
    "post",
    currentPostId
  ).meta,
}));

// Selectively load specific meta fields
const { specialBlockData } = useSelect((select) => ({
  specialBlockData: select(coreDataStore).getEditedEntityRecord(
    "postType",
    "post",
    currentPostId
  ).meta?.special_block_data,
}));
```

In the first `useSelect` whenever any property of meta changes, the cache is automatically invalidated by WordPress. In the second `useSelect` only the specific meta field `special_block_data` is tracked, so the cache is only invalidated when that particular field changes, leading to better performance.

##### 2. Batch Processing API Requests

Combine multiple data requests to reduce API calls:

```javascript
// Using batch processing via the WordPress data API
const { batchingResolver } = dispatch(coreDataStore);

batchingResolver.batch(() => {
  // These will be combined into a single API request
  select(coreDataStore).getEntityRecords("postType", "post");
  select(coreDataStore).getEntityRecords("taxonomy", "category");
  select(coreDataStore).getEntityRecords("taxonomy", "tag");
});
```

### Implementing Custom REST API Response Caching

For custom REST API endpoints that your blocks might interact with, implement server-side caching:

```php
function my_custom_block_data_endpoint() {
    // Cache key based on request parameters
    $cache_key = 'my_block_data_' . md5(serialize($_GET));

    // Try to get cached response
    $cached_data = wp_cache_get($cache_key, 'block_data');

    if (false !== $cached_data) {
        return rest_ensure_response(json_decode($cached_data, true));
    }

    // Generate the response if not cached
    $data = compute_expensive_block_data();

    // Cache the response (JSON encoded)
    wp_cache_set($cache_key, json_encode($data), 'block_data', 5 * MINUTE_IN_SECONDS);

    return rest_ensure_response($data);
}
```

### Browser-Side Caching for REST API

To enhance client-side caching of REST API responses:

1. **Implement proper HTTP caching headers** on your REST API endpoints:

```php
function add_cache_headers_to_rest_api($response) {
    if (!is_wp_error($response)) {
        $response->header('Cache-Control', 'public, max-age=60');
        $response->header('ETag', md5($response->get_data()));
    }
    return $response;
}
add_filter('rest_post_dispatch', 'add_cache_headers_to_rest_api');
```

2. **Use the browser's Cache API** for client-side caching outside of the block editor:

```javascript
// Example of using the Cache API with the WordPress Data Layer
const fetchWithCache = async (path) => {
  // Check if browser supports Cache API
  if (!("caches" in window)) {
    return apiFetch({ path });
  }

  const cache = await caches.open("wp-block-editor-cache");
  const cachedResponse = await cache.match(path);

  if (cachedResponse && cachedResponse.ok) {
    return cachedResponse.json();
  }

  const response = await apiFetch({ path });

  // Cache a clone of the response
  const responseToCache = new Response(JSON.stringify(response), {
    headers: {
      "Content-Type": "application/json",
    },
  });

  cache.put(path, responseToCache);
  return response;
};
```

### Performance Monitoring and Cache Optimization

Monitor your caching strategy effectiveness using browser developer tools:

1. Check network requests in Chrome DevTools to identify redundant API calls
2. Use the Performance tab to identify slow API responses that could benefit from caching
3. Implement telemetry to track cache hit rates and response times

### Best Practices for Block Data Caching

1. **Cache Invalidation**: Implement proper cache invalidation when data changes through actions or mutations
2. **Progressive Enhancement**: Ensure blocks function without cached data, using caching as performance enhancement
3. **Appropriate Timeouts**: Set cache expiration based on data volatility and user expectations
4. **Versioning**: Include version information in cache keys to invalidate caches when your block implementation changes

Implementing these caching strategies will significantly improve Block Editor performance and provide a smoother editing experience, especially for complex blocks that interact extensively with the WordPress REST API.
