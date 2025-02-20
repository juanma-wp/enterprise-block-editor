# Block Development Workflow

Mastering the block development workflow is useful for creating efficient and maintainable custom blocks. This lesson will delve into using `wp-scripts` for build processes, implementing hot reloading for rapid development, and cover some useful debugging techniques. 

## Leveraging wp-scripts for Build Processes

Now that you're familiar with `wp-scripts` from Module 1, let's dive deeper into how to leverage its capabilities in your block development workflow. 

Let's review the scripts that create-block set up for you in your project.

```json
	"scripts": {
		"build": "wp-scripts build",
		"format": "wp-scripts format",
		"lint:css": "wp-scripts lint-style",
		"lint:js": "wp-scripts lint-js",
		"packages-update": "wp-scripts packages-update",
		"plugin-zip": "wp-scripts plugin-zip",
		"start": "wp-scripts start"
	},
```

Here's what each script does:

- `build`: Compiles and optimizes your block's assets (JS, CSS) for production use, creating the final versions in the build directory
- `format`: Automatically formats your source code according to WordPress coding standards
- `lint:css`: Checks your CSS/SCSS files for coding standard violations and potential errors
- `lint:js`: Analyzes your JavaScript code for potential errors and coding standard violations
- `packages-update`: Updates your npm package dependencies to their latest compatible versions
- `plugin-zip`: Creates a ZIP archive of your plugin, ready for distribution
- `start`: Launches a development server that watches your source files and automatically rebuilds them when changes are detected. 

These scripts can be run from the command line using npm.

```shell
npm run build
```

### Development Process with wp-scripts

Generally during block development, you'll primarily use the `start` script. This script watches your source files for changes and automatically rebuilds your block as you work. To use this script, run:

```
npm run start
```

This command will start a [webpack development server](https://webpack.js.org/configuration/dev-server/) and compile your block's assets. As you make changes to your source files, it will automatically recompile and update your block in real-time.

The beauty of this process is that you can leave the development server running in the background while you work on your block, even putting your computer to sleep, and picking up development at a later stage.

## Implementing Hot Reloading for Rapid Development

While the development server will automatically rebuild your block as you work, you still need to refresh the post or page you might be testing your block in to see the changes. This can be time-consuming and disrupt your workflow.

Hot reloading is a powerful feature that allows you to see changes in your block immediately without refreshing the entire page. This significantly speeds up the development process and provides instant feedback on your changes. 

You make a change to your code, the dev server detects the change, rebuilds the block and updates the block editor with the new code. This process happens in milliseconds, allowing you to see the changes in real-time.

To enable hot reloading with `wp-scripts`, you might need to make a few modifications to your local development environment.

You'll need to set up a local WordPress environment to support hot reloading. `wp-env` or Studio by WordPress.com supports hot reloading by default, which is why these tools are recommended for block development.

You will also need to enable the `SCRIPT_DEBUG` [constant](https://developer.wordpress.org/advanced-administration/debug/debug-wordpress/#script_debug) in your local WordPress installations `wp-config.php` file. This constant forces WordPress to use the “dev” versions of core CSS and JavaScript files rather than the minified versions that are normally loaded.

```php
define( 'SCRIPT_DEBUG', true );
```

Finally, you need to add a new script to your block plugin's `package.json` file. This new script contains the same command as the `start` script but with the `--hot` flag.

```
"hot-reload": "wp-scripts start --hot"
```

Adding a separate script to your `package.json` file allows you to keep the default `start` script for development without hot reloading.

With this setup in place, you can run your development server with hot reloading:

```
npm run hot-reload
```

With this running, as you make changes to your block's source files, you'll see the changes reflected immediately in the block editor without needing to refresh the page.

### Production Build with wp-scripts

When you're ready to create a production-ready version of your block, you'll use the build script. This script optimizes your code for performance and creates minified versions of your JavaScript and CSS files. To create a production build, run:

```
npm run build
```

This command will generate optimized files in a build directory, which you can then use in your WordPress plugin or theme.

## Debugging Block JavaScript Code

Effective debugging is crucial for identifying and resolving issues in your block's JavaScript code. Fortunately, modern browsers provide tools to help you debug your JavaScript efficiently.

The browser's developer tools are your first line of defense when debugging block JavaScript code. Most modern browsers provide robust debugging capabilities, including console logging, setting breakpoints, and step-through debugging.

To use the browser developer tools:

1. Open the WordPress block editor in your browser.
2. Open the developer tools (usually F12, Ctrl+Shift+I on Windows/Linux or Option+Cmd+I on MacOS).

You will be presented with a set of tabs, including Elements, Console, Sources, Network, and more. 

### Leveraging console.log for Debugging

A popular way to debug code is to log variables to the Console. This can be incredibly useful for quick debugging and understanding the flow of your code. 

Here's an example of how you might use `console.log()` in your block's Edit component:

```javascript
export default function Edit( { attributes } ) {
    console.log( attributes );
    const { content } = attributes;
    console.log( content );
    const blockProps = useBlockProps();
    console.log( blockProps );
    return (
        <p { ...blockProps() }>
            { __(
                content,
                'my-custom-block'
            ) }
        </p>
    );
}
```

This will log information about the different variables at key points in the block's lifecycle, which can help you understand the data flow when your block is rendered in the editor.

### Using breakpoints and step through debugging

Another powerful debugging technique is setting breakpoints in your code. Breakpoints pause the execution of your code at a specific line, allowing you to inspect variables and step through your code line by line. To set breakpoints in your block's JavaScript code:

1. Navigate to the Sources tab in your browser's developer tools.
2. Find your block's JavaScript file in the file tree.
3. Set breakpoints by clicking on the line numbers in the `src` files where you want to pause execution.
4. Refresh the page to trigger the debugger.

When the code execution hits your breakpoint, it will pause. You can then step through your code, inspect variables, and identify issues.

Debugging this way is possible due to the work that `wp-scripts` does to generate source maps for your block's JavaScript code. Source maps map the minified JavaScript code back to the original source code, and make it easier to debug code in the browser's developer tools.

If you combine the `wp-scripts` development server with hot reloading and use the browser developer tools efficiently, you'll have a powerful setup for efficient block development and debugging.