# 6.4: Editor Block Filters

The WordPress Block Editor provides powerful hooks that allow developers to customize and extend the editing experience. In this lesson, we'll explore two key filters: [`editor.BlockEdit`](https://developer.wordpress.org/block-editor/reference-guides/filters/block-filters/#editor-blockedit) and [`blocks.registerBlockType`](https://developer.wordpress.org/block-editor/reference-guides/filters/block-filters/#blocks-registerblocktype). These filters enable us to modify block behavior, appearance, and settings within the Block Editor.

## Hooking into editor.BlockEdit

The [`editor.BlockEdit`](https://developer.wordpress.org/block-editor/reference-guides/filters/block-filters/#editor-blockedit) filter allows developers to customize the editing interface of blocks. This filter is applied to the edit function of every block, providing an opportunity to wrap or modify the default edit component.

### How it works

The `editor.BlockEdit` filter receives the original `BlockEdit` component as an argument and should return a new component. This new component can add additional UI elements, modify existing ones, or alter the block's behavior during editing.

Here's an example of how to use the `editor.BlockEdit` filter:

```javascript
import { createHigherOrderComponent } from "@wordpress/compose";
import { Fragment } from "@wordpress/element";
import { InspectorControls } from "@wordpress/block-editor";
import { PanelBody, RangeControl } from "@wordpress/components";
import { addFilter } from "@wordpress/hooks";

const withImageSizeControls = createHigherOrderComponent((BlockEdit) => {
  return (props) => {
    // Only apply to the Core Image block
    if (props.name !== "core/image") {
      return <BlockEdit {...props} />;
    }

    // Destructure attributes and setAttributes
    const { attributes, setAttributes } = props;
    const { width, height } = attributes;

    return (
      <Fragment>
        {/* Render the original BlockEdit component */}
        <BlockEdit {...props} />

        {/* Add custom InspectorControls */}
        <InspectorControls>
          <div
            style={{
              backgroundColor: "#FFFDCD",
            }}
          >
            <PanelBody title="Image Size Settings" initialOpen={true}>
              <RangeControl
                label="Width"
                value={width || ""}
                onChange={(value) => setAttributes({ width: value })}
                min={100}
                max={800}
                step={10}
              />
              <RangeControl
                label="Height"
                value={height || ""}
                onChange={(value) => setAttributes({ height: value })}
                min={100}
                max={800}
                step={10}
              />
            </PanelBody>
          </div>
        </InspectorControls>
      </Fragment>
    );
  };
}, "withImageSizeControls");

// Hook into the editor.BlockEdit filter
addFilter(
  "editor.BlockEdit",
  "my-plugin/with-image-size-controls",
  withImageSizeControls
);
```

> [!NOTE]
> Check out a [live demo](https://playground.wordpress.net/?blueprint-url=https://raw.githubusercontent.com/WordPress/block-development-examples/trunk/plugins/inner-blocks-dcd824/_playground/blueprint.json) of this example and its [complete code](https://github.com/WordPress/block-development-examples/tree/trunk/plugins/inner-blocks-dcd824).

In this example, we're adding width and height slider controls to the Image block's sidebar settings panel. It uses a Higher Order Component (created with [`createHigherOrderComponent`](https://developer.wordpress.org/block-editor/reference-guides/packages/packages-compose/#createhigherordercomponent)) to wrap the core Image block and add custom size controls without modifying the original block code. The function is used to create a new component that wraps the original `BlockEdit` component.

For more information on using components in the Block Editor, see the [WordPress Components package documentation](https://developer.wordpress.org/block-editor/reference-guides/packages/packages-components/).

## Modifying Block Settings with blocks.registerBlockType

The [`blocks.registerBlockType`](https://developer.wordpress.org/block-editor/reference-guides/filters/block-filters/#blocks-registerblocktype) filter allows developers to modify block settings during the block registration process. This filter can be used to alter various aspects of a block, including its attributes, supports, and other configuration options.

### How it works

The `blocks.registerBlockType` filter receives the block settings object and the block name as arguments. It should return the modified settings object.

Here's an example of how to use the `blocks.registerBlockType` filter:

```javascript
function modifyParagraphBlock(settings, name) {
  if (name !== "core/paragraph") {
    return settings;
  }

  return {
    ...settings,
    attributes: {
      ...settings.attributes,
      customAttribute: {
        type: "string",
        default: "",
      },
    },
    supports: {
      ...settings.supports,
      align: false,
    },
  };
}

wp.hooks.addFilter(
  "blocks.registerBlockType",
  "my-plugin/modify-paragraph-block",
  modifyParagraphBlock
);
```

In this example, we're modifying the Paragraph block by adding a new attribute called `customAttribute` and disabling alignment support. Learn more about [block attributes](https://developer.wordpress.org/block-editor/reference-guides/block-api/block-attributes/) and [block supports](https://developer.wordpress.org/block-editor/reference-guides/block-api/block-supports/) in the official documentation.

## Practical Application

Let's combine both filters to create a more comprehensive modification of the Paragraph block.

First we add a custom `maxCharacters` attribute to the `'core/paragraph'` block using the `blocks.registerBlockType',` filter

```javascript
import { addFilter } from "@wordpress/hooks";

/**
 * Add the `maxCharacters` attribute to the Paragraph block.
 *
 * @param {Object} settings Block settings.
 * @param {string} name     Block name.
 * @return {Object} Modified block settings.
 */
const addMaxCharactersAttribute = (settings, name) => {
  // Only apply to the Paragraph block.
  if (name !== "core/paragraph") {
    return settings;
  }

  // Add the `maxCharacters` attribute to the block's attributes.
  settings.attributes = {
    ...settings.attributes,
    maxCharacters: {
      type: "number",
      default: 100, // Default character limit.
    },
  };

  return settings;
};

// Hook into the blocks.registerBlockType filter.

addFilter(
  "blocks.registerBlockType",
  "my-plugin/add-max-characters-attribute",
  addMaxCharactersAttribute
);
```

Then we enhance the `Edit` component of `'core/paragraph'` to include a custom panel ([InspectorControls](https://github.com/WordPress/gutenberg/blob/HEAD/packages/block-editor/src/components/inspector-controls/README.md)) that provides UI elements to view/set the `maxCharacters` attribute (we added with the code above) and display a message when the number of characters exceed the value set for `maxCharacters`

```javascript
import { addFilter } from "@wordpress/hooks";
import { createHigherOrderComponent } from "@wordpress/compose";
import { InspectorControls } from "@wordpress/block-editor";
import { PanelBody, Notice, TextControl } from "@wordpress/components";
import { Fragment } from "@wordpress/element";

const withCharacterLimitWarning = createHigherOrderComponent((BlockEdit) => {
  return (props) => {
    // Only apply to the Paragraph block.
    if (props.name !== "core/paragraph") {
      return <BlockEdit {...props} />;
    }

    // Get the block's attributes and setAttributes function.
    const { attributes, setAttributes } = props;
    const { content, maxCharacters } = attributes;

    // Check if the content exceeds the character limit.
    const exceedsLimit =
      content && maxCharacters && content.length > maxCharacters;

    // Update the maxCharacters attribute.
    const updateMaxCharacters = (newValue) => {
      const parsedValue = parseInt(newValue, 10);
      if (!isNaN(parsedValue) && parsedValue > 0) {
        setAttributes({ maxCharacters: parsedValue });
      }
    };

    return (
      <Fragment>
        <BlockEdit {...props} />
        <InspectorControls>
          <PanelBody title="Character Limit Settings" initialOpen={true}>
            <TextControl
              label="Maximum Characters"
              type="number"
              value={maxCharacters || ""}
              onChange={updateMaxCharacters}
              min={1}
              help="Set the maximum number of characters allowed for this paragraph."
            />
            {exceedsLimit && (
              <Notice status="warning" isDismissible={false}>
                This paragraph exceeds the recommended {maxCharacters}-character
                limit. Consider splitting it into smaller paragraphs for better
                readability.
              </Notice>
            )}
          </PanelBody>
        </InspectorControls>
      </Fragment>
    );
  };
}, "withCharacterLimitWarning");

// Hook into the BlockEdit component.
addFilter(
  "editor.BlockEdit",
  "my-plugin/with-character-limit-warning",
  withCharacterLimitWarning
);
```

In this comprehensive example, we've added a new attribute to the Paragraph block using `blocks.registerBlockType` and created a custom control in the sidebar using `editor.BlockEdit`. This demonstrates how these two filters can work together to create a more customized editing experience. For more on WordPress hooks system, refer to the [WordPress Hooks API documentation](https://developer.wordpress.org/plugins/hooks/).

> [!NOTE]
> Check out a [live demo](https://playground.wordpress.net/?blueprint-url=https://raw.githubusercontent.com/WordPress/block-development-examples/trunk/plugins/inner-blocks-dcd824/_playground/blueprint.json) of this example and its [complete code](https://github.com/WordPress/block-development-examples/tree/trunk/plugins/inner-blocks-dcd824).

## Conclusion

By leveraging the `editor.BlockEdit` and `blocks.registerBlockType` filters, developers can significantly enhance and customize the Block Editor experience. These filters provide powerful tools for modifying block behavior, adding new functionality, and tailoring the editing interface to specific needs.

Remember to always consider the user experience when making modifications, and ensure that your changes align with [WordPress coding standards](https://developer.wordpress.org/coding-standards/) and best practices.

For more information on block filters and other block editor customization options, visit the [Block Editor Handbook](https://developer.wordpress.org/block-editor/). Additional block development resources can be found in the [Block Development Examples repository](https://github.com/WordPress/block-development-examples).
