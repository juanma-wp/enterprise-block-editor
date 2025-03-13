The `InnerBlocks` component is a powerful feature in the WordPress Block Editor that allows developers to nest blocks within other blocks. This enables the creation of complex, structured layouts where a single block can act as a container for multiple child blocks. Understanding how to configure `InnerBlocks` effectively allows developers to create flexible and reusable block structures.

### **Understanding Nested Content with InnerBlocks**

The `InnerBlocks` component enables nested content by allowing child blocks inside a parent block. This is useful when creating container-based blocks, such as:

- A **section block** that holds multiple paragraphs, images, and buttons.
- A **grid block** that contains multiple column blocks.
- A **custom card block** that includes an image, a title, and a description.

Here’s an example of a basic implementation of `InnerBlocks` inside a custom block:

```javascript
import { registerBlockType } from "@wordpress/blocks";
import { InnerBlocks } from "@wordpress/block-editor";

registerBlockType("custom/parent-block", {
  title: "Parent Block",
  category: "layout",
  edit: () => {
    return (
      <div className="custom-parent-block">
        <InnerBlocks />
      </div>
    );
  },
  save: () => {
    return (
      <div className="custom-parent-block">
        <InnerBlocks.Content />
      </div>
    );
  },
});
```

In this example:

- The `edit` function renders an `InnerBlocks` component, allowing users to insert any block inside.
- The `save` function ensures that the nested blocks are stored properly when saving the post.

## Restricting Child Blocks with allowedBlocks

To control which blocks can be inserted within a parent block, the `allowedBlocks` property is utilized. This property accepts an array of block names that are permitted as children. For example, to allow only image and paragraph blocks within our custom container block:

```javascript
const ALLOWED_BLOCKS = [ 'core/image', 'core/paragraph' ];

edit: () => {
    const blockProps = useBlockProps();
    return (
        <div { ...blockProps }>
            <InnerBlocks allowedBlocks={ ALLOWED_BLOCKS } />
        </div>
    );
},
```

This configuration ensures that users can only add images and paragraphs within the container block, maintaining a structured and predictable layout.

## Enforcing Structure with templateLock

The `templateLock` property is used to enforce a specific structure within the `InnerBlocks`. It can be set to:

- `false`: No locking, allowing full customization.
- `'all'`: Prevents any addition, removal, or rearrangement of blocks.
- `'insert'`: Allows moving and deleting blocks but prevents adding new ones.

For instance, to completely lock the template:

```javascript
const TEMPLATE = [
    [ 'core/image', {} ],
    [ 'core/heading', { placeholder: 'Enter heading...' } ],
    [ 'core/paragraph', { placeholder: 'Enter content...' } ],
];

edit: () => {
    const blockProps = useBlockProps();
    return (
        <div { ...blockProps }>
            <InnerBlocks
                template={ TEMPLATE }
                templateLock="all"
            />
        </div>
    );
},
```

In this setup, the user cannot add, remove, or rearrange the predefined blocks, ensuring the layout remains consistent.

Check out a [live demo](https://playground.wordpress.net/?blueprint-url=https://raw.githubusercontent.com/WordPress/block-development-examples/trunk/plugins/inner-blocks-dcd824/_playground/blueprint.json) of this example and its [complete code](https://github.com/WordPress/block-development-examples/tree/trunk/plugins/inner-blocks-dcd824).

## Adding Custom Settings via Block Attributes

Some `InnerBlocks` properties can be defined in the `block.json` metadata file, such as `parent`, `ancestor`, and `allowed-blocks`. These attributes act as default properties for `InnerBlocks`, ensuring a predefined structure, while additional configuration can be added in `edit.js`.

Example:

**block.json**

```
{
    "$schema": "https://schemas.wp.org/trunk/block.json",
    "apiVersion": 3,
    "name": "wpvip-learn-enterprise-block-editor/inner-blocks",
    "version": "0.1.0",
    "title": "Block with Inner Blocks ",
    "allowedBlocks": [
        "core/paragraph",
        "core/image",
        "core/heading",
        "core/list",
        "core/quote"
    ],
    "category": "widgets",
    "example": {},
    "textdomain": "block-development-examples",
    "editorScript": "file:./index.js",
    "editorStyle": "file:./index.css"
}
```

**edit.js**

```javascript
import { InnerBlocks, useBlockProps } from "@wordpress/block-editor";
import "./editor.scss";

const MY_TEMPLATE = [
  ["core/image", {}],
  ["core/heading", { placeholder: "Book Title" }],
  ["core/paragraph", { placeholder: "Summary" }],
];

const Edit = () => {
  const blockProps = useBlockProps();
  return (
    <div {...blockProps}>
      <InnerBlocks template={MY_TEMPLATE} />
    </div>
  );
};
export default Edit;
```

In this example:

- The `block.json` file defines `allowedBlocks`, restricting the types of blocks that can be inserted.
- The `edit.js` file further customizes the `InnerBlocks` template, ensuring a predefined structure while allowing users to fill in specific content.

This combination of defining properties in `block.json` and configuring behavior in `edit.js` provides a powerful way to control and enhance the `InnerBlocks` experience.

Check out a [live demo](https://playground.wordpress.net/?blueprint-url=https://raw.githubusercontent.com/Automattic/wpvip-learn-enterprise-block-editor/refs/heads/trunk/examples/inner-blocks/_playground/blueprint.json) of this example and its [complete code](https://github.com/Automattic/wpvip-learn-enterprise-block-editor/tree/trunk/examples/inner-blocks).

## Conclusion

The `InnerBlocks` component is a powerful tool for creating nested and structured content within custom WordPress blocks. By effectively utilizing properties like `allowedBlocks` and `templateLock`, and enhancing functionality with custom attributes, developers can create flexible and controlled editing experiences, leading to more dynamic and user-friendly content management.

For more detailed information, refer to the official [Inner Blocks Documentation](https://developer.wordpress.org/block-editor/how-to-guides/block-tutorial/nested-blocks-inner-blocks/) or any of resources below.

## **Resources**

- [Setting up a multi-block plugin using InnerBlocks and post meta](https://developer.wordpress.org/news/2024/05/setting-up-a-multi-block-using-inner-blocks-and-post-meta/) | Developer Blog
