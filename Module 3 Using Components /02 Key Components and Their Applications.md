## Key Components and Their Applications

In this lesson, we'll explore some useful components from the @wordpress/components package, which are useful for creating rich, interactive user interfaces in the WordPress Block Editor. 

These components provide standardized UI elements that maintain consistency across the WordPress ecosystem while offering powerful functionality for block development.

This lesson only covers some of the more commonly used components. For a complete list, refer to the [Component Reference](https://developer.wordpress.org/block-editor/reference-guides/components/) in the WordPress Block Editor Handbook.

## WordPress Storybook

Storybook is a tool for developing UI components in isolation, allowing developers to build and test components independently of the application. It provides a sandbox environment to showcase components, test different states, and document their behavior.

WordPress uses Storybook to showcase UI elements for the Block Editor including all components in the components package. By exploring Storybook stories, developers can understand how components work, view their props, and see them in action.

To access the WordPress Storybook, visit [https://wordpress.github.io/gutenberg/](https://wordpress.github.io/gutenberg/). To jump directly to the components section, go to [https://wordpress.github.io/gutenberg/?path=/docs/components-introduction--docs](https://wordpress.github.io/gutenberg/?path=/docs/components-introduction--docs).

## Button Component

The Button component is a fundamental building block for creating interactive elements in your custom blocks. It allows users to trigger actions or navigate through your interface.

```javascript
import { useBlockProps } from '@wordpress/block-editor';
import { Button } from '@wordpress/components';
export default function Edit() {

    const handleClick = () => {
        console.log('Button clicked!');
    };

    return (
        <div { ...useBlockProps() }>
            <Button
                variant="primary"
                onClick={ handleClick }
            >
                Click here
            </Button>
        </div>
    );
}
```

The Button component accepts various props to customize its appearance and behavior, such as 'variant' for styling and 'onClick' for handling user interactions.

For more details on the Button component, including all available props and usage examples, visit the [Button component documentation](https://developer.wordpress.org/block-editor/reference-guides/components/button/).

## BaseControl Component

BaseControl is a wrapper component that enhances form controls by providing a consistent layout for labels and help text.

```javascript
import { useBlockProps } from '@wordpress/block-editor';
import { BaseControl } from '@wordpress/components';

export default function Edit() {
    return (
        <div { ...useBlockProps() }>
            <BaseControl
                label="Label text"
            >
                <textarea/>
            </BaseControl>
        </div>
    );
}
```

BaseControl ensures that your form fields have a uniform appearance and proper accessibility attributes.

To explore the full capabilities of the BaseControl component, check out the [BaseControl component documentation](https://developer.wordpress.org/block-editor/reference-guides/components/base-control/).

## TextControl Component

TextControl is used for capturing user input in text fields. It's a versatile component for various types of text-based data entry.

```javascript
import { useBlockProps } from '@wordpress/block-editor';
import { BaseControl, TextControl } from '@wordpress/components';

export default function Edit() {
    return (
        <div { ...useBlockProps() }>
            <BaseControl
                label="Label text"
            >
                <TextControl
                    label="Additional CSS Class"
                />
            </BaseControl>
        </div>
    );
}
```

TextControl handles both single-line and multi-line text inputs, making it suitable for a wide range of use cases.

To learn more about the TextControl component and its various props, visit the [TextControl component documentation](https://developer.wordpress.org/block-editor/reference-guides/components/text-control/).

## SelectControl Component

SelectControl creates dropdown menus for option selection, allowing users to choose from a predefined list of options.

```javascript
import { useState } from 'react';
import { useBlockProps } from '@wordpress/block-editor';
import { SelectControl } from '@wordpress/components';

export default function Edit() {
    const [ size, setSize ] = useState( '50%' );
    return (
        <div { ...useBlockProps() }>
            <SelectControl
                label="Size"
                value={ size }
                options={ [
                    { label: 'Big', value: '100%' },
                    { label: 'Medium', value: '50%' },
                    { label: 'Small', value: '25%' },
                ] }
                onChange={ ( newSize ) => setSize( newSize ) }
            />
        </div>
    );
}
```

This component is particularly useful when you need to provide users with a list of choices.

For a detailed explanation of the SelectControl component and its usage, check out the [SelectControl component documentation](https://developer.wordpress.org/block-editor/reference-guides/components/select-control/).

## ToggleControl Component

ToggleControl is used for managing boolean states with a simple on/off switch.

```javascript
import { useState } from 'react';
import { useBlockProps } from '@wordpress/block-editor';
import { ToggleControl } from '@wordpress/components';

export default function Edit() {
    const [ hasFixedBackground, setHasFixedBackground ] = useState( false );
    return (
        <div { ...useBlockProps() }>
            <ToggleControl
                label="Fixed Background"
                help={
                    hasFixedBackground
                        ? 'Has fixed background.'
                        : 'No fixed background.'
                }
                checked={ hasFixedBackground }
                onChange={ (newValue) => {
                    setHasFixedBackground( newValue );
                } }
            />
        </div>
    );
}
```

ToggleControl is ideal for settings that can be either enabled or disabled.

To dive deeper into the ToggleControl component and its implementation, refer to the [ToggleControl component documentation](https://developer.wordpress.org/block-editor/reference-guides/components/toggle-control/).

## CheckboxControl and RadioControl Components

These components are used for building form inputs that allow users to select one or multiple options.

CheckboxControl is used for multiple selections

```javascript
import { useState } from 'react';
import { useBlockProps } from '@wordpress/block-editor';
import { CheckboxControl } from '@wordpress/components';

export default function Edit() {
    const [ isChecked, setChecked ] = useState( true );
    return (
        <div { ...useBlockProps() }>
            <CheckboxControl
                label="Is author"
                help="Is the user a author or not?"
                checked={ isChecked }
                onChange={ setChecked }
            />
        </div>
    );
}
```

RadioControl is for single-option selections.

```javascript
import { useState } from 'react';
import { useBlockProps } from '@wordpress/block-editor';
import { RadioControl } from '@wordpress/components';

export default function Edit() {
    const [ option, setOption ] = useState( 'a' );
    return (
        <div { ...useBlockProps() }>
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
        </div>
    );
}
```

For more information on these form control components, visit the [CheckboxControl component documentation](https://developer.wordpress.org/block-editor/reference-guides/components/checkbox-control/) and [RadioControl component documentation](https://developer.wordpress.org/block-editor/reference-guides/components/radio-control/).

## RangeControl Component

RangeControl creates a slider for adjusting numeric values within a specified range.

```javascript
import { useState } from 'react';
import { useBlockProps } from '@wordpress/block-editor';
import { RangeControl } from '@wordpress/components';

export default function Edit() {
    const [ columns, setColumns ] = useState( 2 );
    return (
        <div { ...useBlockProps() }>
            <RangeControl
                label="Columns"
                value={ columns }
                onChange={ ( value ) => setColumns( value ) }
                min={ 2 }
                max={ 10 }
            />
        </div>
    );
}

```

This component is particularly useful for settings that require fine-tuning of numeric values.

To explore the full potential of the RangeControl component, check out the [RangeControl component documentation](https://developer.wordpress.org/block-editor/reference-guides/components/range-control/).

## Popover Component

Popover creates contextual menus or overlays that appear relative to a specific element.

```javascript
import { useState } from 'react';
import { useBlockProps } from '@wordpress/block-editor';
import { Button, Popover } from '@wordpress/components';

export default function Edit() {
    const [ isVisible, setIsVisible ] = useState( false );
    const toggleVisible = () => {
        setIsVisible( ( state ) => ! state );
    };

    return (
        <div { ...useBlockProps() }>
            <Button variant="secondary" onClick={ toggleVisible }>
                Toggle Popover!
                { isVisible && <Popover>Popover is toggled!</Popover> }
            </Button>
        </div>
    );
}
```

Popovers are useful for displaying additional information or options without cluttering the main interface.

For detailed information on the Popover component and its various use cases, check out the [Popover component documentation](https://developer.wordpress.org/block-editor/reference-guides/components/popover/).

## Notice Component and Snackbar Component

Notice and Snackbar components are used to display messages or alerts within the block editor. Notice should be used to communicate important messages to the user, while Snackbar should be used to communicate low priority, non-interruptive messages to the user.

```javascript
import { useBlockProps } from '@wordpress/block-editor';
import { Notice } from '@wordpress/components';
export default function Edit() {
    return (
        <div { ...useBlockProps() }>
            <Notice status="error">This is an error notice.</Notice>
            <h2>Block heading</h2>
            <p>Block content</p>
        </div>
    );
}
```

```javascript
import { useBlockProps } from '@wordpress/block-editor';
import { Snackbar } from '@wordpress/components';
export default function Edit() {
    return (
        <div { ...useBlockProps() }>
            <Snackbar>Post published successfully.</Snackbar>
            <h2>Block heading</h2>
            <p>Block content</p>
        </div>
    );
}
```

For detailed information on these components and their various use cases, check out the [Notice component documentation](https://developer.wordpress.org/block-editor/reference-guides/components/notice/) and the [Snackbar component documentation](https://developer.wordpress.org/block-editor/reference-guides/components/snackbar/).

### Dynamic Notices

Using these components in the Edit function adds the error message to the block in the Editor. If you're looking to add a notice to the post-editor, you can use the `useEffect` hook from the `@wordpress/data` package to add a notice to the `core/notices` store.

Here's an example of how to trigger a Snackbar notice when a button is clicked:

```javascript
import { useBlockProps } from '@wordpress/block-editor';
import { useDispatch } from '@wordpress/data';
import { Button } from '@wordpress/components';

export default function Edit() {
    const { createNotice } = useDispatch('core/notices');

    const triggerNotice = () => {
        createNotice(
            'success',
            'Action completed successfully!',
            {
                type: 'snackbar',
                isDismissible: true
            }
        );
    };

    return (
        <div { ...useBlockProps() }>
            <h2>Block heading</h2>
            <p>Block content</p>
            <Button onClick={triggerNotice}>Trigger Notice</Button>
        </div>
    );
}
```

## Block Toolbar and Settings Sidebar Components

Some components in the @wordpress/components package make more sense when used in either the block toolbar or settings sidebar. These components provide a consistent user interface for block-level settings and actions.

To implement custom block toolbar and settings sidebar components, you can use the `BlockControls` and `InspectorControls` components from the `@wordpress/block-editor` package.

```javascript
import { BlockControls, InspectorControls } from '@wordpress/block-editor';
```

You can then add Block Toolbar and Settings Sidebar components to your block's Edit function to provide additional controls and settings.

### Block Toolbar

The Block Toolbar is a set of controls that appear above the block in the editor. It provides quick access to common block actions and settings. You can add custom toolbar controls using the `BlockControls` component.

```javascript
export default function Edit() {
    return (
        <div { ...useBlockProps() }>
            <BlockControls>
                {/* Add custom toolbar controls here */}
            </BlockControls>
            <h2>Custom Components!</h2>
        </div>
    );
}
```

### Settings Sidebar

The Settings Sidebar is a panel that appears on the right side of the editor, providing additional block settings and options. You can add custom settings controls using the `InspectorControls` component.

```javascript
export default function Edit() {
    return (
        <div { ...useBlockProps() }>
            <InspectorControls>
                {/* Add custom settings controls here */}
            </InspectorControls>
            <h2>Custom Components!</h2>
        </div>
    );
}
```

## ToolbarButton Component

ToolbarButton adds buttons to the Block Toolbar, usually inside a [Toolbar](https://developer.wordpress.org/block-editor/reference-guides/components/toolbar/) or [ToolbarGroup](https://developer.wordpress.org/block-editor/reference-guides/components/toolbar-group/) component, allowing for quick access to block-specific actions.

```javascript
import { BlockControls, useBlockProps } from '@wordpress/block-editor';
import { Toolbar, ToolbarButton } from '@wordpress/components';

export default function Edit() {
    const handleStyleChange = () => {
        alert('Style changed!')
    };
    return (
        <div { ...useBlockProps() }>
            <BlockControls>
                <Toolbar label="Options">
                    <ToolbarButton
                        icon="admin-appearance"
                        label="Change Style"
                        onClick={handleStyleChange}
                    />
                </Toolbar>
            </BlockControls>
            <h2>Custom Components!</h2>
        </div>
    );
}
```

ToolbarButtons enhance the user experience by providing easy access to frequently used block functions.

To learn more about implementing ToolbarButton in your blocks, visit the [ToolbarButton component documentation](https://developer.wordpress.org/block-editor/reference-guides/components/toolbar-button/).

## Panel, PanelBody, and PanelRow Components

These components are used to structure settings panels in the Settings Sidebar. The Panel creates a container with a header. PanelBody creates collapsible sections which can be added to Panels, while PanelRow organizes content within the PanelBody sections.

```javascript
import { InspectorControls, useBlockProps } from '@wordpress/block-editor';
import { Panel, PanelBody, PanelRow, TextControl } from '@wordpress/components';

export default function Edit() {
    return (
        <div { ...useBlockProps() }>
            <InspectorControls>
                <Panel header="My Panel">
                    <PanelBody title="My Block Settings" initialOpen={ true }>
                        <PanelRow>
                            <TextControl
                                label="Block Setting"
                            />
                        </PanelRow>
                    </PanelBody>
                </Panel>
            </InspectorControls>
            <h2>Custom Components!</h2>
        </div>
    );
}
```

This structure helps organize block settings in a user-friendly, collapsible interface in the Settings Sidebar.

For a comprehensive guide on using Panel, PanelBody, and PanelRow components, refer to the [Panel component documentation](https://developer.wordpress.org/block-editor/reference-guides/components/panel/).

## ColorPicker

ColorPicker allows users to select and apply colors to various elements within your block.

```javascript
import { InspectorControls, useBlockProps } from '@wordpress/block-editor';
import { Panel, PanelBody, PanelRow, ColorPicker } from '@wordpress/components';

export default function Edit() {
    return (
        <div { ...useBlockProps() }>
            <InspectorControls>
                <Panel header="My Panel">
                    <PanelBody title="My Block Settings" initialOpen={ true }>
                        <PanelRow>
                            <ColorPicker
                                color="#000"
                            />
                        </PanelRow>
                    </PanelBody>
                </Panel>
            </InspectorControls>
            <h2>Custom Components!</h2>
        </div>
    );
}
```

This component provides a user-friendly interface for color selection, which is crucial for customization options in many blocks.

To explore the full capabilities of the ColorPicker component, refer to the [ColorPicker component documentation](https://developer.wordpress.org/block-editor/reference-guides/components/color-picker/).

## Conclusion

By mastering these key parts from the @wordpress/components package, you'll be well-equipped to create sophisticated and user-friendly interfaces for your custom blocks. Each component serves a specific purpose and can be combined to create complex UI elements that enhance the overall user experience of your WordPress blocks.