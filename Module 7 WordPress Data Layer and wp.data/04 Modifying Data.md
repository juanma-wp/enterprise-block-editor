# **WordPress Data Layer and wp.data: Modifying Data**

In this lesson, we'll explore how to modify data using the WordPress Data Layer and wp.data module, focusing on updating post settings and options, managing nonces, and security considerations.

The WordPress Data Layer provides a powerful way to modify data in the block editor environment. The `useDispatch` hook is the recommended method for dispatching actions to update the state.

## **Using useDispatch**

The `useDispatch` hook allows you to access the dispatch functions (actions) for a specific store. Here's an example of how to use it to update post title:

```javascript
import { useDispatch } from "@wordpress/data";
import { store as editorStore } from "@wordpress/editor";

function MyComponent() {
  const { editPost } = useDispatch(editorStore);

  const updateTitle = (newTitle) => {
    editPost({ title: newTitle });
  };

  return <button onClick={() => updateTitle("New Title")}>Update Title</button>;
}
```

The `useDispatch` hook is recommended because it integrates seamlessly with React's lifecycle. Every store provides its own set of actions that can be dispatched to update the state of that store. Refer to the [Data Module Reference docs](https://developer.wordpress.org/block-editor/reference-guides/data/) for a complete documentation of the actions available for each store

## **Nonces and Security Considerations**

Nonces are crucial for securing WordPress REST API requests**.** The [Block Editor stores](https://developer.wordpress.org/block-editor/reference-guides/data/) handle nonce management automatically when you use their actions. Here's an example:

```javascript
import { useDispatch } from "@wordpress/data";
import { store as coreStore } from "@wordpress/core-data";

function SaveButton() {
  const { saveEntityRecord } = useDispatch(coreStore);

  const savePost = () => {
    saveEntityRecord("postType", "post", {
      id: postId,
      title: "Updated Title",
    });
  };

  return <button onClick={savePost}>Save Post</button>;
}
```

In this case, the nonce is automatically included in the request.

For direct REST API requests, you need to include the nonce manually:

```javascript
import apiFetch from "@wordpress/api-fetch";

// Set the nonce for all subsequent requests
apiFetch.use(apiFetch.createNonceMiddleware(wpApiSettings.nonce));

// Make a request
apiFetch({
  path: "/wp/v2/posts/1",
  method: "POST",
  data: { title: "Updated Title" },
}).then((response) => {
  console.log("Post updated:", response);
});
```

## **Different Stores and Popular Actions**

## **"core" store**

The ["core" store](https://developer.wordpress.org/block-editor/reference-guides/data/data-core/) (managed by the `@wordpress/core-data` package) provides actions for interacting with WordPress core data. Some key actions include:

- `saveEntityRecord`: Save an entity record
- `deleteEntityRecord`: Delete an entity record
- `editEntityRecord`: Edit an entity record

Example:

```javascript
import { useDispatch } from "@wordpress/data";
import { store as coreStore } from "@wordpress/core-data";
import { Button } from "@wordpress/components";
import { __ } from "@wordpress/i18n";

const UpdatePostTitleButton = () => {
  const { saveEntityRecord } = useDispatch(coreStore);

  const handleUpdatePostTitle = () => {
    const updatedPost = {
      id: 1, // Replace with the actual post ID you want to update
      title: __("New Title", "text-domain"), // New title for the post
    };

    saveEntityRecord("postType", "post", updatedPost)
      .then((savedRecord) => {
        console.log("Post updated successfully:", savedRecord);
      })
      .catch((error) => {
        console.error("Failed to update post:", error);
      });
  };

  return (
    <Button variant="primary" onClick={handleUpdatePostTitle}>
      {__("Update Post Title", "text-domain")}
    </Button>
  );
};

export default UpdatePostTitleButton;
```

## **"core/block-directory" store**

The ["core/block-directory" store](https://developer.wordpress.org/block-editor/reference-guides/data/data-core-block-directory/) (managed by the [`@wordpress/block-directory` package](https://developer.wordpress.org/block-editor/reference-guides/packages/packages-block-directory/))

This store manages the block directory. Key actions include:

- `downloadBlock`: Download a block
- `installBlock`: Install a block
- `uninstallBlock`: Uninstall a block

Example:

```javascript
import { useDispatch } from "@wordpress/data";
import { store as blockDirectoryStore } from "@wordpress/block-directory";
import { Button } from "@wordpress/components";
import { __ } from "@wordpress/i18n";

const InstallBlockButton = () => {
  const { installBlock } = useDispatch(blockDirectoryStore);

  const handleInstallBlock = () => {
    const block = {
      id: "block-block",
      name: "block/block",
      title: __("Test Block", "text-domain"),
    };

    installBlock(block)
      .then(() => {
        console.log("Block installed successfully!");
      })
      .catch((error) => {
        console.error("Failed to install block:", error);
      });
  };

  return (
    <Button variant="primary" onClick={handleInstallBlock}>
      {__("Install Test Block", "text-domain")}
    </Button>
  );
};

export default InstallBlockButton;
```

## **"core/blocks" Store**

The "core/blocks" store (managed by the [`@wordpress/blocks package`](https://developer.wordpress.org/block-editor/reference-guides/packages/packages-blocks/)) manages block types. Popular actions include:

- `addBlockTypes`: Add block types
- `removeBlockTypes`: Remove block types

Example:

```javascript
import { useDispatch } from "@wordpress/data";
import { store as blocksStore } from "@wordpress/blocks";
import { Button } from "@wordpress/components";
import { __ } from "@wordpress/i18n";

const RegisterBlockButton = () => {
  const { addBlockTypes } = useDispatch(blocksStore);

  const handleRegisterBlock = () => {
    const blockType = {
      name: "my-block", // Replace with your block's name
      settings: {
        title: __("My Custom Block", "text-domain"), // Block title
        icon: "smiley", // Block icon (you can use any Dashicon or custom SVG)
        category: "widgets", // Block category (e.g., 'common', 'formatting', 'layout', etc.)
        edit: () => <p>{__("Hello from the editor!", "text-domain")}</p>, // Edit callback
        save: () => <p>{__("Hello from the frontend!", "text-domain")}</p>, // Save callback
      },
    };

    addBlockTypes([blockType])
      .then(() => {
        console.log("Block registered successfully!");
      })
      .catch((error) => {
        console.error("Failed to register block:", error);
      });
  };

  return (
    <Button variant="primary" onClick={handleRegisterBlock}>
      {__("Register My Block", "text-domain")}
    </Button>
  );
};

export default RegisterBlockButton;
```

## **"core/block-editor" Store**

This store "core/block-editor" ((managed by the [`@wordpress/block-editor package`](https://developer.wordpress.org/block-editor/reference-guides/packages/packages-block-editor/))) manages the block editor state. Key actions include:

- `insertBlocks`: Insert blocks
- `updateBlockAttributes`: Update block attributes
- `removeBlock`: Remove a block

Example:

```javascript
import { useDispatch } from "@wordpress/data";
import { store as blockEditorStore } from "@wordpress/block-editor";
import { createBlock } from "@wordpress/blocks";
import { Button } from "@wordpress/components";
import { __ } from "@wordpress/i18n";

const InsertParagraphButton = () => {
  const { insertBlocks } = useDispatch(blockEditorStore);

  const handleInsertParagraph = () => {
    const paragraphBlock = createBlock("core/paragraph", {
      content: __("This is a new paragraph.", "text-domain"),
    });

    insertBlocks([paragraphBlock]);
  };

  return (
    <Button variant="primary" onClick={handleInsertParagraph}>
      {__("Insert Paragraph", "text-domain")}
    </Button>
  );
};

export default InsertParagraphButton;
```

## **Conclusion**

Upon completing this lesson, you should be able to:

1. Explain how the WordPress Data API facilitates modifying post settings and options using the wp.data module.

2. Demonstrate how to securely update post data while managing nonces to prevent unauthorized modifications.

3. Modify post settings and options programmatically using wp.data.dispatch and validate changes through state management functions.

The WordPress Data Layer provides a powerful and secure way to modify data in the block editor environment. By using the `useDispatch` hook and understanding the different stores and their actions, you can efficiently update post settings, manage options, and ensure proper security measures are in place.
