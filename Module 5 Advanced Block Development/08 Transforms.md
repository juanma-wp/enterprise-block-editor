# Transforms

Block transforms are a powerful feature of the [WordPress Block Editor](https://developer.wordpress.org/block-editor/) that allow developers to enhance the editing experience by enabling seamless conversion into different block types. This lesson will explore the concept of block transforms, their practical applications, and how to implement them in your custom block development.

The WordPress Block Editor provides a robust [Block Transforms API](https://developer.wordpress.org/block-editor/reference-guides/block-api/block-transforms/) to define these transformations, enabling developers to specify how blocks can be converted both **to** and **from** other blocks. This capability not only enhances user experience but also promotes content versatility.

## **The Block Transforms API**

The [Block Transforms API](https://developer.wordpress.org/block-editor/reference-guides/block-api/block-transforms) is a powerful tool that allows developers to define how blocks can be transformed. By specifying transformation rules, developers can control the conversion process between different block types. The API primarily revolves around the `transforms` property within a block's configuration, which contains two main keys: `to` and `from`.

### **Transform Direction: To and From**

The `transforms` property defines the possible transformations for a block. It consists of two arrays:

- **to**: Specifies the block types that the current block can be transformed into.
- **from**: Specifies the block types that can be transformed into the current block.

Each transformation is an object that outlines the type of transformation and the associated logic.

### **Transformation Types**

The API supports several [transformation types](https://developer.wordpress.org/block-editor/reference-guides/block-api/block-transforms/#transformations-types), each serving a distinct purpose:

- **block**: Handles transformations between block types.
- **enter**: Triggered when specific text is entered, allowing for dynamic block insertion.
- **files**: Manages transformations when files are uploaded.
- **prefix**: Activates transformations based on a text prefix.
- **raw**: Processes raw HTML or plain text into blocks.
- **shortcode**: Converts shortcodes into blocks.

For this lesson, we'll focus on the `block` transformation type, which is commonly used for converting between different block types.

### **Defining a Block Transformation**

To define a block transformation, you'll need to specify the following [parameters](https://developer.wordpress.org/block-editor/reference-guides/block-api/block-transforms/#block):

- **type** (string): The type of transformation, for example `'block'`.
- **blocks** (array): An array of block names that the transformation applies to. It can also accept the wildcard value (`"*"`) to indicate all block types.
- **transform** (function): A callback function that receives the attributes and inner blocks of the block being processed. It should return a block object or an array of block objects.

There are [other optionals properties](https://developer.wordpress.org/block-editor/reference-guides/block-api/block-transforms/#block) such as **isMatch**, **isMultiBlock** and **priority**.

Here's an example of a transformation from a paragraph block to a heading block:

```javascript
transforms: {
    from: [
        {
            type: 'block',
            blocks: ['core/paragraph'],
            transform: ({ content }) => {
                return createBlock('core/heading', {
                    content,
                });
            },
        },
    ],
}
```

In this example, when a paragraph block is transformed into a heading block, the content is preserved and assigned to the new heading block.

## **Use Cases and Practical Examples**

Understanding the practical applications of block transformations can inspire developers to implement them effectively. Here are some common use cases:

### **1. Enhancing Content Flexibility**

Users often need to change the presentation of their content. For example, transforming a list block into separate paragraph blocks can be achieved through a custom transformation:

```javascript
transforms: {
    from: [
        {
            type: 'block',
            blocks: ['core/list'],
            transform: ({ values }) => {
                const items = values.split('</li>');
                return items.map((item) =>
                    createBlock('core/paragraph', {
                        content: item.replace('<li>', ''),
                    })
                );
            },
        },
    ],
}
```

This transformation splits each list item into individual paragraph blocks, providing greater flexibility in content editing.

### **2. Converting Legacy Content**

When migrating from older content formats, such as shortcodes, to blocks, transformations can automate the process:

```javascript
transforms: {
    from: [
        {
            type: 'shortcode',
            tag: 'gallery',
            transform: (atts) => {
                const images = atts.ids.split(',');
                return createBlock('core/gallery', {
                    images: images.map((id) => ({ id })),
                });
            },
        },
    ],
}
```

This example converts a `[gallery]` shortcode into a gallery block, mapping the shortcode attributes to the block's attributes.

## **Applying Filters for Conditional Transforms**

In some cases, you may want to conditionally enable or disable transformations based on specific criteria, such as block attributes or the editing context. WordPress provides [filters](https://developer.wordpress.org/block-editor/reference-guides/filters/block-filters/) that allow you to modify block settings during registration, enabling conditional logic for transformations.

For instance, using the [`blocks.registerBlockType`](https://developer.wordpress.org/block-editor/reference-guides/filters/block-filters/#blocks-registerblocktype) filter in JavaScript, you can conditionally add or remove transformations:

```javascript
import { addFilter } from "@wordpress/hooks";
import { createBlock } from "@wordpress/blocks";

function modifyBlockTransforms(settings, name) {
  // Remove the heading block from the paragraph block's transforms.
  if (name === "core/heading") {
    settings.transforms = {
      ...settings.transforms,
      from:
        settings.transforms?.from?.filter(
          (transform) =>
            !(
              transform.type === "block" &&
              transform.blocks?.includes("core/paragraph")
            )
        ) || [],
    };
  }

  // Add a custom transform to the paragraph block to transform to a heading block.
  // only if the paragraph has the "custom-class" class.
  if (name === "core/paragraph") {
    settings.transforms = {
      ...settings.transforms,
      to: [
        ...(settings.transforms?.to || []),
        {
          type: "block",
          blocks: ["core/heading"],
          transform: ({ content }) => createBlock("core/heading", { content }),
          isMatch: (attributes) =>
            attributes.className?.includes("custom-class"),
        },
      ],
    };
  }

  return settings;
}

addFilter(
  "blocks.registerBlockType",
  "my-plugin/modify-paragraph-transforms",
  modifyBlockTransforms
);
```

In this example, the `modifyBlockTransforms` function checks if the block being registered is a Paragraph block (`core/paragraph`). It then determines whether the block has a specific class (`custom-class`) and conditionally adds or removes a transformation to a Heading block based on that condition. This approach allows for dynamic control over available transformations, enhancing the flexibility and usability of custom blocks.

> [!NOTE]
> Check out a [live demo](https://playground.wordpress.net/?blueprint-url=https://raw.githubusercontent.com/Automattic/wpvip-learn-enterprise-block-editor/refs/heads/trunk/examples/transforms-filter/_playground/blueprint.json) of this example and the [complete code](https://github.com/Automattic/wpvip-learn-enterprise-block-editor/tree/trunk/examples/transforms-filter) of the basic and multiple deprecation examples above.

## **Summary**

By mastering block transformations, developers can provide a more intuitive and flexible content editing experience. The key takeaways from this lesson are:

- Block transformations improve content versatility and usability.
- The `transforms` property defines conversion rules using the Block Transforms API.
- Different transformation types exist, with `block` transformations being the most common.
- Custom transformations enable seamless content conversions.
- Filters allow for dynamic, context-aware transformations.

With these insights, you can now define, apply, and customize block transformations to enhance the WordPress Block Editor experience.

## **Resources**

- [Block Transforms API Reference](https://developer.wordpress.org/block-editor/reference-guides/block-api/block-transforms/) | Block Editor Handbook
- [Block Registration API Reference](https://developer.wordpress.org/block-editor/reference-guides/block-api/block-registration/) | Block Editor Handbook
- [Block Filters Reference](https://developer.wordpress.org/block-editor/reference-guides/filters/block-filters/) | Block Editor Handbook
- [Core Blocks Reference](https://developer.wordpress.org/block-editor/reference-guides/core-blocks/) | Block Editor Handbook
- [Shortcode Transforms Guide](https://developer.wordpress.org/block-editor/how-to-guides/block-transforms/shortcode-transforms/) | Block Editor Handbook
- [Block Development Examples](https://developer.wordpress.org/block-editor/how-to-guides/block-tutorial/) | Block Editor Handbook
