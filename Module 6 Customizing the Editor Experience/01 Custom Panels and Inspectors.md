## **Custom Panels and Inspectors**

In this lesson, you'll learn how to create and improve custom block panels and inspector controls in the WordPress Block Editor (Gutenberg).

After this lesson, you'll be able to:

- Understand what **InspectorControls** does in the Block Editor.
- Organize block settings using custom panels in the inspector.
- Add a sidebar panel with **InspectorControls**.
- Modify a block to include more settings.

### **Understanding InspectorControls**

[`InspectorControls`](https://github.com/WordPress/gutenberg/blob/HEAD/packages/block-editor/src/components/inspector-controls/README.md) is a WordPress Block Editor component that allows developers to add custom settings to the editor's sidebar. It helps separate content editing from configuration settings, keeping the editing area clean and focused.

Settings that affect a block's appearance, behavior, or functionality should go in `InspectorControls` rather than inline controls like toolbars. This improves clarity and makes blocks easier to use.

Grouping related settings in custom panels makes blocks easier to configure. For example, an "Alert" block could include a panel for selecting the alert type (success, warning, error) and an option to show or hide an icon.

### **Creating a Custom Sidebar Panel with InspectorControls**

To add custom settings to a block’s sidebar, use the `InspectorControls` component from `@wordpress/block-editor`. This wraps the settings in a structured panel.

For example, having the following block with the attributes “`fallbackCurrentYear”, “showStartingYear” and “startingYear”`

```javascript
{
	"$schema": "https://schemas.wp.org/trunk/block.json",
	"apiVersion": 3,
	"name": "block-development-examples/copyright-date-block-09aac3",
	"version": "0.1.0",
	"title": "Copyright Date Block 09aac3",
	"category": "widgets",
	"description": "Display your site's copyright date.",
	"example": {},
	"keywords": [ "09aac3"],
	"attributes": {
		"fallbackCurrentYear": {
			"type": "string"
		},
		"showStartingYear": {
			"type": "boolean"
		},
		"startingYear": {
			"type": "string"
		}
	},
	"supports": {
		"color": {
			"background": false,
			"text": true
		},
		"html": false,
		"typography": {
			"fontSize": true
		}
	},
	"textdomain": "block-development-examples",
	"editorScript": "file:./index.js",
	"editorStyle": "file:./index.css",
	"style": "file:./style-index.css",
	"render": "file:./render.php"
}
```

The following block's edit file include `InspectorControls` alongside other UI components such as `PanelBody, TextControl, ToggleControl` to display a custom area the block’s sidebar panel to update some of the block’s attributes

```javascript
import { __ } from "@wordpress/i18n";
import { InspectorControls, useBlockProps } from "@wordpress/block-editor";
import { PanelBody, TextControl, ToggleControl } from "@wordpress/components";
import { useEffect } from "react";

export default function Edit({ attributes, setAttributes }) {
  const { fallbackCurrentYear, showStartingYear, startingYear } = attributes;

  const currentYear = new Date().getFullYear().toString();

  useEffect(() => {
    if (currentYear !== fallbackCurrentYear) {
      setAttributes({ fallbackCurrentYear: currentYear });
    }
  }, [currentYear, fallbackCurrentYear, setAttributes]);

  let displayDate;

  if (showStartingYear && startingYear) {
    displayDate = startingYear + "–" + currentYear;
  } else {
    displayDate = currentYear;
  }

  return (
    <>
      <InspectorControls>
        <PanelBody title={__("Settings", "block-development-examples")}>
          <ToggleControl
            checked={showStartingYear}
            label={__("Show starting year", "block-development-examples")}
            onChange={() =>
              setAttributes({
                showStartingYear: !showStartingYear,
              })
            }
          />
          {showStartingYear && (
            <TextControl
              label={__("Starting year", "block-development-examples")}
              value={startingYear}
              onChange={(value) => setAttributes({ startingYear: value })}
            />
          )}
        </PanelBody>
      </InspectorControls>
      <p {...useBlockProps()}>© {displayDate}</p>
    </>
  );
}
```

Here, `PanelBody` groups `TextControl` and `ToggleControl` inside a "`'Settings`" panel. `TextControl` sets the `startingYear` attribute, while `ToggleControl` allows users to set the `showStartingYear` attribute

### **Modifying an Existing Block to Add Inspector Controls**

The [`editor.BlockEdit`](https://developer.wordpress.org/block-editor/reference-guides/filters/block-filters/#editor-blockedit) filter allows developers to enhance block editing by injecting custom components into the block editor. This filter is used with `addFilter()` to wrap existing blocks with additional controls, improving their customization options without modifying the core block code.

```javascript
addFilter(editor.BlockEdit, "namespace", callback);
```

The callback required by the [`editor.BlockEdit`](https://developer.wordpress.org/block-editor/reference-guides/filters/block-filters/#editor-blockedit) filter is a a Higher-Order Component (HOC) that takes the original `BlockEdit` component, extends it with additional UI elements, and returns the modified version. This lets developers introduce new settings within the inspector panel.

As a HOC returns a new component, the `createHigherOrderComponent` function can be be used to return a custom name for the component returned by the HOC.

For example, adding a custom CSS class input to the core "Image" block:

```
import { addFilter } from '@wordpress/hooks';
import { createHigherOrderComponent } from '@wordpress/compose';
import { InspectorControls } from '@wordpress/block-editor';
import { PanelBody, Notice, TextControl } from '@wordpress/components';
import { Fragment } from '@wordpress/element';

const withCharacterLimitWarning = createHigherOrderComponent( ( BlockEdit ) => {
	return ( props ) => {
		// Only apply to the Paragraph block.
		if ( props.name !== 'core/paragraph' ) {
			return <BlockEdit { ...props } />;
		}

		// Get the block's attributes and setAttributes function.
		const { attributes, setAttributes } = props;
		const { content, maxCharacters } = attributes;

		// Check if the content exceeds the character limit.
		const exceedsLimit =
			content && maxCharacters && content.length > maxCharacters;

		// Update the maxCharacters attribute.
		const updateMaxCharacters = ( newValue ) => {
			const parsedValue = parseInt( newValue, 10 );
			if ( ! isNaN( parsedValue ) && parsedValue > 0 ) {
				setAttributes( { maxCharacters: parsedValue } );
			}
		};

		return (
			<Fragment>
				<BlockEdit { ...props } />
				<InspectorControls>
					<PanelBody
						title="Character Limit Settings"
						initialOpen={ true }
					>
						<TextControl
							label="Maximum Characters"
							type="number"
							value={ maxCharacters || '' }
							onChange={ updateMaxCharacters }
							min={ 1 }
							help="Set the maximum number of characters allowed for this paragraph."
						/>
						{ exceedsLimit && (
							<Notice status="warning" isDismissible={ false }>
								This paragraph exceeds the recommended{ ' ' }
								{ maxCharacters }-character limit. Consider
								splitting it into smaller paragraphs for better
								readability.
							</Notice>
						) }
					</PanelBody>
				</InspectorControls>
			</Fragment>
		);
	};
}, 'withCharacterLimitWarning' );

// Hook into the BlockEdit component.
addFilter(
	'editor.BlockEdit',
	'my-plugin/with-character-limit-warning',
	withCharacterLimitWarning
);
```

This code adds character limit functionality specifically for paragraph blocks. It adds a new control in the Inspector (sidebar) panel where users can set a maximum character limit. When the paragraph's content exceeds this limit, a warning notice appears suggesting to split the text into smaller paragraphs for better readability. This is implemented as a higher-order component that wraps around the default paragraph block editor component.

Custom block panels and inspectors improve the WordPress Block Editor experience by keeping settings organized and easy to use. Use clear labels, logical grouping, and helpful instructions to create blocks that offer powerful customization options while maintaining a clean editing experience.

## Further Reading

- [InspectorControls](https://github.com/WordPress/gutenberg/blob/HEAD/packages/block-editor/src/components/inspector-controls/README.md)
