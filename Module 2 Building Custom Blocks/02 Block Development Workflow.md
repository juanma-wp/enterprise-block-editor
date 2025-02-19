# Block Development Workflow

## Introduction to Block Development Workflow

In the ever-evolving landscape of WordPress development, mastering the block development workflow is crucial for creating efficient and maintainable custom blocks. This lesson will delve into the intricacies of using wp-scripts for build processes, implementing hot reloading for rapid development, and employing effective debugging techniques. By the end of this lesson, you'll be equipped with the knowledge and skills to streamline your block development process and create high-quality, performant blocks for the WordPress ecosystem.

## Leveraging wp-scripts for Build Processes

Now that you're familiar with wp-scripts from Module 1, let's dive deeper into how to leverage its capabilities in your block development workflow. We'll explore advanced configurations and best practices that will enhance your development process.

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

During the development phase, you'll primarily use the start script. This script watches your source files for changes and automatically rebuilds your block as you work. To begin development, run:

```
npm run start
```

This command will start a development server and compile your block's assets. As you make changes to your source files, wp-scripts will automatically recompile and update your block in real-time.

### Production Build with wp-scripts

When you're ready to create a production-ready version of your block, you'll use the build script. This script optimizes your code for performance and creates minified versions of your JavaScript and CSS files. To create a production build, run:

```
npm run build
```

This command will generate optimized files in a build directory, which you can then use in your WordPress plugin or theme.

## Implementing Hot Reloading for Rapid Development

Hot reloading is a powerful feature that allows you to see changes in your block immediately without refreshing the entire page. This significantly speeds up the development process and provides instant feedback on your changes.

To enable hot reloading with wp-scripts, you'll need to make a few modifications to your development environment.

You'll need to set up a local WordPress environment to support hot reloading. One way to do this is by using the @wordpress/env (aka wp-env) package, which provides a no configuration local WordPress development environment for developing WordPress block plugins.

To start, install @wordpress/env as a development dependency in your block plugin by running the following command from the terminal:

```
npm install @wordpress/env --save-dev
```

Then, update your `package.json` to include the `wp-env` script:

```
"scripts": {
    "wp-env": "wp-env"
},
```

Now, you can start wp-env with the following command:

```
npm run wp-env start
```

This will start the local WordPress environment and your block editor will be available at `http://localhost:8888`.

Next, you need to edit the `start` script in your `package.json` to enable hot reloading.

```
"start": "wp-scripts start --hot"
```

With this setup in place, you can run your development server with hot reloading:

```
npm run start
```

As you make changes to your block's source files, you'll see the changes reflected immediately in the block editor without needing to refresh the page.

## Debugging Block JavaScript Code

Effective debugging is crucial for identifying and resolving issues in your block's JavaScript code. WordPress provides several tools and techniques to help you debug your blocks efficiently.

### Using the Browser Developer Tools

The browser's developer tools are your first line of defense when debugging block JavaScript code. Most modern browsers provide robust debugging capabilities, including breakpoints, step-through debugging, and console logging.

To use the browser developer tools effectively:

1. Open the WordPress block editor in your browser.  
2. Open the developer tools (usually F12 or Ctrl+Shift+I).  
3. Navigate to the Sources tab.  
4. Find your block's JavaScript file in the file tree.  
5. Set breakpoints by clicking on the line numbers where you want to pause execution.  
6. Refresh the page to trigger the debugger.

You can then step through your code, inspect variables, and identify issues.

### Leveraging console.log for Debugging

While not as powerful as breakpoint debugging, console.log statements can be incredibly useful for quick debugging and understanding the flow of your code. Here's an example of how you might use console.log in your block's JavaScript:

```javascript
registerBlockType( 'my-plugin/my-block', {
    edit: ( props ) => {
        console.log( 'Edit function called with props:', props );
        return (
            <div>
                <RichText
                    value={ props.attributes.content }
                    onChange={ ( content ) => {
                        console.log( 'Content changed:', content );
                        props.setAttributes( { content } );
                    } }
                />
            </div>
        );
    },
    save: ( props ) => {
        console.log( 'Save function called with props:', props );
        return (
            <div>
                <RichText.Content value={ props.attributes.content } />
            </div>
        );
    },
} );
```

In this example, we're logging information at key points in the block's lifecycle, which can help us understand when and how our block is being rendered and updated.