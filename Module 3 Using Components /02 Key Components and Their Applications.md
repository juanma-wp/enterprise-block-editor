## 3.2 Key Components and Their Applications

In this lesson, we'll explore essential components from the @wordpress/components package, which are useful for creating rich, interactive user interfaces in the WordPress Block Editor. 

These components provide standardized UI elements that maintain consistency across the WordPress ecosystem while offering powerful functionality for block development.

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
import { CheckboxControl } from '@wordpress/components';

const MyCheckboxControl = () => {
    const [ isChecked, setChecked ] = useState( true );
    return (
        <CheckboxControl
            __nextHasNoMarginBottom
            label="Is author"
            help="Is the user a author or not?"
            checked={ isChecked }
            onChange={ setChecked }
        />
    );
};
```

RadioControl is for single-option selections.

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

For more information on these form control components, visit the [CheckboxControl component documentation](https://developer.wordpress.org/block-editor/reference-guides/components/checkbox-control/) and [RadioControl component documentation](https://developer.wordpress.org/block-editor/reference-guides/components/radio-control/).

## RangeControl Component

RangeControl creates a slider for adjusting numeric values within a specified range.

```javascript
import { RangeControl } from '@wordpress/components';

function MySlider() {
    return (
         setOpacity(value)}
            min={0}
            max={100}
        />
    );
}
```

This component is particularly useful for settings that require fine-tuning of numeric values.

Further Reading: To explore the full potential of the RangeControl component, check out the [RangeControl component documentation](https://developer.wordpress.org/block-editor/reference-guides/components/range-control/).

## Notice Component

The Notice component is used for displaying inline notifications or alerts to users.

```javascript
import { Notice } from '@wordpress/components';

function MyNotification() {
    return (
        <Notice status="error">An unknown error occurred.</Notice>
    );
}
```

Notices can be used to provide feedback, warnings, or important information to users within your block interface.

Further Reading: For a comprehensive guide on using the Notice component, refer to the [Notice component documentation](https://developer.wordpress.org/block-editor/reference-guides/components/notice/).

## ToolbarButton Component

ToolbarButton adds buttons to block toolbars, allowing for quick access to block-specific actions.

```javascript
import { ToolbarButton } from '@wordpress/components';
import { BlockControls } from '@wordpress/block-editor';

function MyBlockToolbar() {
    return (
        
             handleStyleChange()}
            />
        
    );
}
```

ToolbarButtons enhance the user experience by providing easy access to frequently used block functions.

Further Reading: To learn more about implementing ToolbarButton in your blocks, visit the [ToolbarButton component documentation](https://developer.wordpress.org/block-editor/reference-guides/components/toolbar-button/).

## Popover Component

Popover creates contextual menus or overlays that appear relative to a specific element.

```javascript
import { Popover, Button } from '@wordpress/components';

function MyPopoverMenu() {
    const [isOpen, setIsOpen] = useState(false);

    return (
        <>
             setIsOpen(true)}>Open Menu
            {isOpen && (
                 setIsOpen(false)}>
                    Popover content goes here
                
            )}
        
    );
}
```

Popovers are useful for displaying additional information or options without cluttering the main interface.

Further Reading: For detailed information on the Popover component and its various use cases, check out the [Popover component documentation](https://developer.wordpress.org/block-editor/reference-guides/components/popover/).

## ColorPicker Component

ColorPicker allows users to select and apply colors to various elements within your block.

```javascript
import { ColorPicker } from '@wordpress/components';

function MyColorSelector() {
    return (
         setSelectedColor(color.hex)}
        />
    );
}
```

This component provides a user-friendly interface for color selection, which is crucial for customization options in many blocks.

Further Reading: To explore the full capabilities of the ColorPicker component, refer to the [ColorPicker component documentation](https://developer.wordpress.org/block-editor/reference-guides/components/color-picker/).

## Panel, PanelBody and PanelRow Components

These components are used to structure settings panels in the block inspector. `Panel` creates a container with a header,   `PanelBody` creates collapsible sections, while PanelRow organizes content within those sections.

```javascript
import { Panel, PanelBody, PanelRow } from '@wordpress/components';
import { more } from '@wordpress/icons';

const MyPanel = () => (
    <Panel header="My Panel">
        <PanelBody title="My Block Settings" icon={ more } initialOpen={ true }>
            <PanelRow>My Panel Inputs and Labels</PanelRow>
        </PanelBody>
    </Panel>
);
```

This structure helps organize block settings in a user-friendly, collapsible interface.

For a comprehensive guide on using Panel, PanelBody and PanelRow components, refer to the [Panel component documentation](https://developer.wordpress.org/block-editor/reference-guides/components/panel/).

By mastering these key components from the @wordpress/components package, you'll be well-equipped to create sophisticated and user-friendly interfaces for your custom blocks. Each component serves a specific purpose and can be combined to create complex UI elements that enhance the overall user experience of your WordPress blocks.