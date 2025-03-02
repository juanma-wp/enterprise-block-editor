## 3.3 Best Practices for Using Components

When working with components from the @wordpress/components package, it's essential to understand how to customize them effectively and manage their state and interactivity. This lesson will explore best practices for using these components in your custom block development process.

### Customizing and Styling Components

The @wordpress/components package provides a wide range of pre-built components that adhere to WordPress design principles. However, you may often need to customize these components to fit your specific block requirements or match your theme's design.

#### Using Component Props

Most components in the @wordpress/components package come with a variety of [props](https://developer.wordpress.org/block-editor/reference-guides/components/button/#props) that allow you to customize their appearance and behavior. Always refer to the component's documentation to understand available props.

For example, the Button component offers several props for customization:

```javascript
import { __ } from '@wordpress/i18n';
import { useBlockProps } from '@wordpress/block-editor';
import { Button } from '@wordpress/components';
import './editor.scss';
export default function Edit() {
    const handleClick = () => {
        console.log('Button clicked!');
    };
    return (
        <div { ...useBlockProps() }>
            <h2>Block Title</h2>
            <p>Block content goes here.</p>
            <Button
                icon='smiley'
                variant='primary'
                onClick={ handleClick }
            >
                Click here
            </Button>
        </div>
    );
}
```

In this example, we've customized the Button's `icon` and `variant` props, and added an `onClick` event handler. These props allow for significant customization without writing additional CSS.

The WordPress Storybook is a great resource for [exploring components and their props](https://wordpress.github.io/gutenberg/?path=/docs/components-introduction--docs), as well as seeing how they look and behave in different contexts. 

#### Applying Custom CSS

While component props offer a good starting point, you may need to apply custom CSS for more specific styling needs. You can do this by adding classes to your components via the `className` prop.

```javascript
import { __ } from '@wordpress/i18n';
import { useBlockProps } from '@wordpress/block-editor';
import { Button } from '@wordpress/components';
import './editor.scss';
export default function Edit() {
    const handleClick = () => {
        console.log('Button clicked!');
    };
    return (
        <div { ...useBlockProps() }>
            <h2>Block Title</h2>
            <p>Block content goes here.</p>
            <Button
                className='my-custom-button'
                icon='smiley'
                variant="primary"
                onClick={ handleClick }
            >
                Click here
            </Button>
        </div>
    );
}
```

Then in your block's `style.scss` file (or `editor.scss` if you want the styles to only apply in the editor), you can define styles for your custom component class:

```scss
.wp-block-create-block-my-custom-block .my-custom-button {
    background-color: #f0f0f0;
    border-radius: 10px;
    padding: 10px 20px;
    
    &:hover {
        background-color: #e0e0e0;
    }
}
```

#### Overriding Default Styles

In some cases, you may need to override the default styles of a component. While this should be done cautiously to maintain consistency across the WordPress admin interface, it's sometimes necessary for unique design requirements.

You can use more specific CSS selectors to override default styles:

```scss
.wp-block-create-block-my-custom-block .components-button.is-primary {
    background-color: #ff6b6b;
    color: #ffffff;
    
    &:hover {
        background-color: #ff8787;
    }
}
```

This CSS targets a primary Button component specifically within your custom block, overriding its default blue color with a custom red shade.

### Handling Component State and Interactivity

Managing state and implementing interactivity are useful for many aspects of creating dynamic and responsive blocks. The @wordpress/components package provides several tools to help with this.

#### Using the useState Hook

For simple state management within a component, the `useState` [hook](https://react.dev/reference/react/useState) from React is often sufficient. Here's an example of using the RadioControl component to manage local state:

```javascript
import { RadioControl } from '@wordpress/components';
import { useState } from 'react';

const MyRadioControl = () => {
    const [ option, setOption ] = useState( 'a' );

    return (
        <RadioControl
            label="User type"
            help="The type of the current user"
            selected={ option }
            options={ [
                { label: 'Author', value: 'a' },
                { label: 'Editor', value: 'e' },
            ] }
            onChange={ ( value ) => setOption( value ) }
        />
    );
};
```

#### Implementing Controlled Components

Many components in the @wordpress/components package are designed to be controlled components, meaning their state is controlled by React. This is typically done by passing a value prop and an onChange prop:

```javascript
import { useState } from 'react';
import { RangeControl } from '@wordpress/components';

const MyRangeControl = () => {
    const [ columns, setColumns ] = useState( 2 );

    return (
        <RangeControl
            __nextHasNoMarginBottom
            __next40pxDefaultSize
            label="Columns"
            value={ columns }
            onChange={ ( value ) => setColumns( value ) }
            min={ 2 }
            max={ 10 }
        />
    );
};
```

Here, the RangeControl component's state is managed by our component, allowing us to respond to changes and update other parts of our UI accordingly.

#### Handling Side Effects with useEffect

When you need to perform side effects in response to state or prop changes, the React `useEffect` [hook](https://react.dev/reference/react/useEffect) is invaluable:

```javascript
import React, { useState, useEffect } from 'react';
import { __ } from '@wordpress/i18n';
import { useBlockProps } from '@wordpress/block-editor';
import apiFetch from '@wordpress/api-fetch';
import './editor.scss';
export default function Edit() {
    const [teams, setTeams] = useState([]);

    useEffect(() => {
        apiFetch( { path: '/collaborator/v1/teams' } ).then( ( teams ) => {
            setTeams(teams);
        } );
    }, []);

    return (
        <div { ...useBlockProps() }>
            <label>
                { __(
                    'Select your item from the list below',
                    'text-domain'
                ) }
            </label>
            <select>
                {teams.map((team) => (
                    <option key={team.id} value={team.id}>
                        {team.name}
                    </option>
                ))}
            </select>
        </div>
    );
}
```

In this example, we use useEffect fetch data from the WordPress REST API when the component mounts, and update our state with the fetched data. This allows us to dynamically populate our select dropdown with options based on the data we retrieve.

## Conclusion

The fact that the Block Editor is built on React means that you can leverage the full power of React when developing custom blocks. This includes using hooks, managing state, and handling side effects. Combine this with the flexibility of the @wordpress/components package, and you have a powerful toolkit for creating custom blocks that meet your specific needs.