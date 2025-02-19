# Project Scaffolding

## Scaffolding a New Block Using Create-Block

When developing custom blocks for WordPress, it is helpful to follow a structured approach to scaffolding the project. The `@wordpress/create-block` package provides a standardized way to generate a new block plugin with all the necessary files and configurations.

It streamlines setting up the foundational structure for your custom blocks, saving time and ensuring project consistency while following best practices for block development.

To scaffold a new block, navigate to your WordPress plugins directory in the terminal. Typically, this would be:

```
cd wp-content/plugins/
```

Once there, you can initiate the block creation process by running:

```
npx @wordpress/create-block@latest my-custom-block
```

This command generates a new plugin directory named `my-custom-block` containing a fully functional block plugin. The process includes setting up the essential PHP, JavaScript, and CSS files for the new block and configuring the build process using @wordpress/scripts.

## Directory and File Structure Best Practices

A well-organized directory structure is essential for maintainability and scalability in enterprise WordPress development. The `create-block` tool provides a default structure, but developers can alter or optimize it based on project requirements.

```
my-custom-block/
├── build/
│   └── my-custom-block/
│       ├── block.json
│       ├── index-rtl.css
│       ├── index.asset.php
│       ├── index.css
│       ├── index.js
│       ├── style-index-rtl.css
│       ├── style-index.css
│       ├── view.asset.php
│       └── view.js
├── node_modules/
├── src/
│   └── my-custom-block/
│       ├── block.json
│       ├── edit.js
│       ├── editor.scss
│       ├── index.js
│       ├── save.js
│       ├── style.scss
│       └── view.js
├── .editorconfig
├── .gitignore
├── folder-structure.md
├── my-custom-block.php
├── package.json
├── package-lock.json
└── readme.txt
```

Let's break down the purpose of each directory and file:

The `build` directory contains the compiled and optimized assets that WordPress will actually use. These files are generated automatically by the build process and should not be edited directly. 

The `node_modules/` directory contains all npm dependencies but is not committed to version control.

The `src` directory is where you'll do most of your development work. 

This directory contains the files for your block.

The `block.json` file is a crucial component in modern block development. It declares the block's metadata, including its name, attributes, and dependencies. 

```
{
  "$schema": "https://schemas.wp.org/trunk/block.json",
  "apiVersion": 3,
  "name": "create-block/my-custom-block",
  "version": "0.1.0",
  "title": "My Custom Block",
  "category": "widgets",
  "icon": "smiley",
  "description": "Example block scaffolded with Create Block tool.",
  "example": {},
  "supports": {
    "html": false
  },
  "textdomain": "my-custom-block",
  "editorScript": "file:./index.js",
  "editorStyle": "file:./index.css",
  "style": "file:./style-index.css",
  "viewScript": "file:./view.js"
}
```

The rest of the files are the source code for the block:

- `edit.js`: Defines the block's behavior in the editor.  
- `editor.scss`: Styles specific to the block in the editor.  
- `index.js`: The main entry point for your block's JavaScript.  
- `save.js`: Determines how the block is saved and rendered on the frontend.  
- `style.scss`: Styles for the block on the frontend.

You will notice that create-block scaffolds the block's source code inside a sub-directory inside the `src` directory, in this case, `my-custom-block`. This allows you to build multiple blocks in the same plugin. The compiled files in the build directory match this directory structure. 

The main PHP file, `my-custom-block.php`, is responsible for registering the block with WordPress and enqueuing the necessary scripts and styles. Here's a basic example:

```
<?php
/**
 * Plugin Name:       My Custom Block
 * Description:       Example block scaffolded with Create Block tool.
 * Version:           0.1.0
 * Requires at least: 6.7
 * Requires PHP:      7.4
 * Author:            The WordPress Contributors
 * License:           GPL-2.0-or-later
 * License URI:       https://www.gnu.org/licenses/gpl-2.0.html
 * Text Domain:       my-custom-block
 *
 * @package CreateBlock
 */

if ( ! defined( 'ABSPATH' ) ) {
	exit; // Exit if accessed directly.
}

/**
 * Registers the block using the metadata loaded from the `block.json` file.
 * Behind the scenes, it registers also all assets so they can be enqueued
 * through the block editor in the corresponding context.
 *
 * @see https://developer.wordpress.org/reference/functions/register_block_type/
 */
function create_block_my_custom_block_block_init() {
	register_block_type( __DIR__ . '/build/my-custom-block' );
}
add_action( 'init', 'create_block_my_custom_block_block_init' );

```

The rest of the files scaffolded are not specific to developoing blocks but allow for additional developer-related functionality:

1. **package.json**  
     
   - NPM package configuration  
   - Lists dependencies (like @wordpress packages) and build scripts (wp-scripts)  
   - Defines metadata about the project

   

2. **.gitignore**  
     
   - Specifies which files Git should ignore  
   - Typically includes node\_modules and build directories  
   - Useful when commit code to Git revsion control platforms (eg GitHub, GitLab)

   

3. **.editorconfig**  
     
   - Defines coding style preferences  
   - Ensures consistent coding style across editors

   

4. **readme.txt**  
     
   - Plugin readme file  
   - Contains plugin description, changelog, and documentation

## The WordPress Scripts Package for Efficient Development

The @wordpress/scripts package is a collection of reusable scripts tailored for WordPress development. It provides a set of configurations and dependencies that are commonly used in WordPress projects, saving you the trouble of setting up complex build processes manually.

To leverage @wordpress/scripts in your project, you'll find that create-block has already set up the necessary npm scripts in your `package.json` file:

```
{
    "scripts": {
        "build": "wp-scripts build",
        "format": "wp-scripts format",
        "lint:css": "wp-scripts lint-style",
        "lint:js": "wp-scripts lint-js",
        "start": "wp-scripts start",
        "packages-update": "wp-scripts packages-update"
    }
}
```

These scripts provide several useful commands:

- `npm run build`: Compiles your source files for production use.  
- `npm run start`: Starts a development server with hot reloading.  
- `npm run format`: Formats your JavaScript files using Prettier.  
- `npm run lint:css`: Lints your CSS/SCSS files.  
- `npm run lint:js`: Lints your JavaScript files.

Using these scripts ensures that your development process adheres to WordPress coding standards and best practices.

The WordPress Block Editor, also known as Gutenberg, has revolutionized content creation and site building in WordPress. For enterprise software developers, understanding the intricacies of block development is crucial. In this section, we'll delve into the block build process, a fundamental aspect of block development.

## Understanding the Block Build Process

When developing custom blocks for WordPress, it's essential to grasp the distinction between the source code you write and the compiled code that WordPress ultimately uses. This understanding forms the foundation of efficient block development and maintenance.

### The Src and Build Directories

In a typical block plugin structure, you'll encounter two primary directories: `src` and `build`. These directories serve distinct purposes in the development lifecycle.

The `src` directory contains your source code. This is where you, as a developer, write and maintain your block's code. It typically includes:

1. JavaScript files (often using JSX syntax)  
2. SCSS or CSS files for styling  
3. PHP files for server-side rendering (if applicable)

The `build` directory, on the other hand, houses the compiled and optimized version of your code. This is the code that WordPress actually uses when rendering your block. The build directory typically contains:

1. Minified JavaScript files  
2. Compiled and minified CSS files  
3. Any other assets required for the block to function

### The wp-scripts Build Process

As discussed, the transformation from source code to compiled code is handled by `@wordpress/scripts`. This tool simplifies the build process by abstracting away complex configuration files and providing a standardized build pipeline.

When you run the build command (`npm run build`), several processes occur:

1. Transpilation: Modern JavaScript (ES6+) and JSX are converted into a format compatible with older browsers.  
     
2. Bundling: All JavaScript modules are combined into a single file, reducing HTTP requests.  
     
3. Minification: Both JavaScript and CSS files are compressed to reduce file size.  
     
4. Asset Optimization: Images and other assets may be optimized for web delivery.

Let's look at a simple example to illustrate this process:

In your `src/index.js` file, you might have:

```javascript
import { registerBlockType } from '@wordpress/blocks';
import { __ } from '@wordpress/i18n';

registerBlockType( 'my-plugin/my-block', {
    title: __( 'My Custom Block', 'my-plugin' ),
    icon: 'smiley',
    category: 'common',
    edit: () => <div>Hello from the editor!</div>,
    save: () => <div>Hello from the frontend!</div>,
} );
```

After running the build process, in your `build/index.js`, you'll find a minified and optimized version of this code, along with any necessary dependencies bundled together.

The build process also handles SCSS compilation. For instance, if you have a `src/style.scss` file:

```
.wp-block-my-plugin-my-block {
    background-color: #f0f0f0;
    padding: 20px;
    
    h2 {
        color: #333;
    }
}
```

This will be compiled and minified into `build/style.css`, ready for WordPress to enqueue.

Understanding this build process is useful for several reasons:

1. **Performance**: The build process optimizes your code for production, ensuring your blocks load quickly and efficiently.  
     
2. **Browser Compatibility**: By transpiling modern JavaScript, your blocks can work across a wide range of browsers.  
     
3. **Development Workflow**: You can leverage modern development tools and practices in your source code, while still producing WordPress-compatible output.  
     
4. **Version Control**: Typically, you'd commit your `src` directory to version control, but not the `build` directory, as it can be regenerated from the source.

By mastering the block build process, you'll be better equipped to develop, debug, and maintain your custom blocks effectively. Remember, while WordPress uses the compiled code in the `build` directory, your focus as a developer should be on writing clean, maintainable code in the `src` directory. The build tools will handle the rest, ensuring your blocks are optimized for production use.

## Setting Up a Block Development Environment

To set up an efficient block development environment, you'll need a few key components:

1. A local WordPress installation  
2. Node.js and npm  
3. A code editor (such as Visual Studio Code)

For the local WordPress installation, you have several options. WordPress recommends using wp-env, a Node.js-based local environment tool. To set it up, first ensure you have Docker installed on your system, then run:

```
npm -g i @wordpress/env
```

Once installed, you can start a new WordPress environment in any directory by running:

```
wp-env start
```

This will create a new WordPress installation, accessible at [http://localhost:8888](http://localhost:8888), with your current directory mapped as a plugin.

### Studio by WordPress.com

While wp-env provides a robust local development environment, Studio by WordPress.com is another option to consider, particularly suited for developers who prefer not to install Docker. Studio is a fast, free way to develop with WordPress, while keeping your local development process smooth and simple.

To get started with Studio visit [developer.wordpress.com/studio](https://developer.wordpress.com/studio/) and download the version for your operating system.

Once installed, open the app and follow the on screen instructions to create a new local development site.

### Code Editor

For your code editor, Visual Studio Code (VS Code) is a popular choice among WordPress developers. It offers excellent support for JavaScript and PHP development, and has a rich ecosystem of extensions. Some recommended extensions for WordPress development include:

- ESLint  
- Prettier  
- PHP Intelephense  
- WordPress Snippets

Remember, block development is an iterative process. As you develop your block, make use of the `npm run start` command to see your changes in real-time in the block editor. This workflow allows for rapid development and testing, ensuring that your custom blocks meet both your needs and those of your users.

## Further Reading

- [Using the Create Block Tool](https://learn.wordpress.org/tutorial/using-the-create-block-tool/)
- [Intro to Publishing with the Block Editor](https://learn.wordpress.org/tutorial/intro-to-publishing-with-the-block-editor/)
- [Advanced Layouts with the Block Editor](https://learn.wordpress.org/lesson/advanced-layouts-with-the-block-editor/)
- [WordPress Plugin Development Best Practices](https://developer.wordpress.org/plugins/plugin-basics/best-practices/)
- [Setting Up Your Development Environment](https://developer.wordpress.org/block-editor/getting-started/devenv/)
- [Get Started with wp-scripts](https://developer.wordpress.org/block-editor/getting-started/devenv/get-started-with-wp-scripts/)