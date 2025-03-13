# Setting Up Your Environment

In addition to having a local WordPress installation, [Node.js](http://Node.js) and its package manager, [npm](https://www.npmjs.com/), are at the heart of WordPress block development. These tools are fundamental for managing dependencies, running build processes, and leveraging the power of JavaScript in WordPress block projects.

This lesson covers installing Node.js and npm and provides an overview of the `@wordpress` npm packages that facilitate block development.

## Installing Node.js and npm

Node.js is a runtime that enables developers to execute JavaScript code outside a browser. It is essential for running build tools, managing dependencies, and executing scripts related to Block development. The Node Package Manager (npm) is a command line utility (CLI) bundled with Node.js and allows you to install and manage JavaScript libraries, including the Gutenberg-specific WordPress packages.

To install and use Node.js and npm, you must have access to a terminal to run commands. Windows users are recommended to install [Powershell](https://learn.microsoft.com/powershell), while MacOS and Linux users can use their default terminal applications. 

### Check If Node.js Is Installed

To check if Node.js is already installed on your system, open a terminal and run:

```
node -v
```

This command should return a version number. If Node.js is not installed, proceed with the next steps.

### Download and Install Node.js with Node Version Manager

While there are several ways to install Node.js and npm, [Node Version Manager](https://github.com/nvm-sh/nvm) (or nvm) lets you easily install and upgrade different node/npm versions. This is especially useful if you’re working with different JavaScript libraries or frameworks. 

This Learn.WordPress.org lesson on [Setting up your block development environment](https://learn.wordpress.org/lesson/setting-up-your-block-development-environment/) contains detailed installation steps for all three major operating systems.

Once you have installed Node.js and npm, you can run the following commands to verify the installation:

```
node -v  # Outputs the version of node installed 
npm -v   # Outputs the version of npm installed
```

## Overview of the WordPress npm Packages

Block development makes use of official WordPress packages that simplify creating, managing, and extending blocks. These packages are hosted on npmjs.com, an online software repository for JavaScript packages, under the `@wordpress` namespace and can be installed via the npm CLI.

These packages provide a wide range of functionality, from UI components to data management utilities, specifically designed for WordPress block development.

### Core Packages

Let's explore some of the most important `@wordpress` packages you'll likely use in your Block Editor development:

#### @wordpress/scripts

This package is the cornerstone of modern WordPress development. It provides a set of pre-configured webpack configurations and scripts that handle the build process for your JavaScript and CSS files.

To use `@wordpress/scripts` in your project, first install it as a dev dependency:

```
npm install @wordpress/scripts --save-dev
```

Then, add the following scripts to your `package.json`:

```
{
  "scripts": {
    "build": "wp-scripts build",
    "start": "wp-scripts start",
    "lint:js": "wp-scripts lint-js",
    "lint:css": "wp-scripts lint-style"
  }
}
```

These scripts allow you to build your assets, start a development server with hot reloading, and lint your JavaScript and CSS files.

#### @wordpress/components

This package provides a set of React components that adhere to WordPress design principles. These components are the building blocks for creating consistent and accessible user interfaces in the Block Editor.

To use `@wordpress/components`, install it in your project:

```
npm install @wordpress/components
```

You can then import and use components in your JavaScript files:

```javascript
import { Button, TextControl } from '@wordpress/components';

function MyCustomBlock() {
    return (
        <div>
            <TextControl
                label="Enter some text"
                value={ text }
                onChange={ ( value ) => setText( value ) }
            />
            <Button isPrimary onClick={ handleSave }>
                Save
            </Button>
        </div>
    );
}
```

#### @wordpress/data

The `@wordpress/data` package provides a robust data management system for WordPress JavaScript applications. It implements a flux-like pattern, allowing you to create and manage data stores for your blocks and plugins.

Install it using npm:

```
npm install @wordpress/data
```

Here's an example of how you might use `@wordpress/data` to register a custom store:

```javascript
import { registerStore } from '@wordpress/data';

const DEFAULT_STATE = {
    items: [],
};

const actions = {
    addItem( item ) {
        return {
            type: 'ADD_ITEM',
            item,
        };
    },
};

const reducer = ( state = DEFAULT_STATE, action ) => {
    switch ( action.type ) {
        case 'ADD_ITEM':
            return {
                ...state,
                items: [ ...state.items, action.item ],
            };
        default:
            return state;
    }
};

const selectors = {
    getItems( state ) {
        return state.items;
    },
};

registerStore( 'my-custom-store', {
    reducer,
    actions,
    selectors,
} );
```

This example demonstrates how to create a simple store with actions to add items and selectors to retrieve them.

### Additional Packages

There are many other `@wordpress` packages that you may find useful in your development process:

- `@wordpress/blocks`: Provides utilities for working with blocks.  
- `@wordpress/i18n`: Internationalization utilities for WordPress projects.  
- `@wordpress/api-fetch`: A utility to make WordPress REST API requests.  
- `@wordpress/editor`: Contains functionality for the Block Editor itself.

To explore the full range of available packages, visit the official [WordPress Block Editor Package Reference](https://developer.wordpress.org/block-editor/reference-guides/packages/).  