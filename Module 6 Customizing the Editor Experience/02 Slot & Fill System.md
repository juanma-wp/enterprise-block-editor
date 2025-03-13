# 6.2: Slot & Fill System

The [Slot & Fill system](https://developer.wordpress.org/block-editor/reference-guides/slotfills/) is a powerful feature in WordPress that allows developers to inject custom UI elements into predefined areas of the Block Editor. This system provides a flexible way to extend and customize the editor experience without modifying core code.

The Slot & Fill system consists of two main components:

1. **Slot**: A designated area in the editor where custom content can be inserted.
2. **Fill**: The custom content that is injected into a specific Slot.

These components work together to enable developers to add new functionality and UI elements to various parts of the Block Editor.

## **How Slot & Fill Works in WordPress**

The Slot & Fill system operates on a simple principle:

1. WordPress defines Slots in specific locations within the editor interface.
2. Developers create Fills that target these Slots.
3. When the editor loads, the Fills are automatically inserted into their corresponding Slots.

This mechanism allows for seamless integration of custom elements without the need to modify core editor components directly.

WordPress provides several predefined SlotFills for extending different parts of the editor. Some of the most commonly used ones include:

1. [`MainDashboardButton`](https://developer.wordpress.org/block-editor/reference-guides/slotfills/main-dashboard-button/): Adds a button to the main dashboard header.
2. [`PluginBlockSettingsMenuItem`](https://developer.wordpress.org/block-editor/reference-guides/slotfills/plugin-block-settings-menu-item/): Adds an item to the block settings menu.
3. [`PluginDocumentSettingPanel`](https://developer.wordpress.org/block-editor/reference-guides/slotfills/plugin-document-setting-panel/): Adds a panel to the Document sidebar.
4. [`PluginMoreMenuItem`](https://developer.wordpress.org/block-editor/reference-guides/slotfills/plugin-more-menu-item/): Adds an item to the main ellipsis menu.
5. [`PluginPostPublishPanel`](https://developer.wordpress.org/block-editor/reference-guides/slotfills/plugin-post-publish-panel/): Adds a panel to the post-publish area.
6. [`PluginPostStatusInfo`](https://developer.wordpress.org/block-editor/reference-guides/slotfills/plugin-post-status-info/): Adds information to the post-status section of the sidebar.
7. [`PluginPrePublishPanel`](https://developer.wordpress.org/block-editor/reference-guides/slotfills/plugin-pre-publish-panel/): Adds a panel to the pre-publish area.
8. [`PluginSidebar`](https://developer.wordpress.org/block-editor/reference-guides/slotfills/plugin-sidebar/): Adds a sidebar to the editor.
9. [`PluginSidebarMoreMenuItem`](https://developer.wordpress.org/block-editor/reference-guides/slotfills/plugin-sidebar-more-menu-item/): Adds a menu item to access a custom sidebar.

## **Implementing Slot & Fill**

To use the Slot & Fill system, you'll need to follow these steps:

1. Import the necessary components from the WordPress packages.
2. Define a custom component that will serve as your Fill.
3. Register a plugin that renders your Fill component.

Here's a basic example of how to implement a custom Fill:

```js
import { registerPlugin } from "@wordpress/plugins";
import { PluginSidebar } from "@wordpress/edit-post";
import { __ } from "@wordpress/i18n";

const MyCustomSidebar = () => (
  <PluginSidebar name="my-custom-sidebar" title={__("My Custom Sidebar")}>
    <p>{__("This is my custom sidebar content.")}</p>
  </PluginSidebar>
);

registerPlugin("my-custom-sidebar-plugin", {
  render: MyCustomSidebar,
});
```

In this example, we're creating a custom sidebar using the `PluginSidebar` component, which is a pre-defined Slot in the editor

## **Some Slot & Fill Examples**

### Adding Controls to the Toolbar

You can add custom controls to the editor toolbar for a block using the [`PluginBlockSettingsMenuItem`](https://developer.wordpress.org/block-editor/reference-guides/slotfills/plugin-block-settings-menu-item/) Slot. Here's an example:

```javascript
/* global navigator */
/* eslint-disable no-console */

import { registerPlugin } from "@wordpress/plugins";
import { PluginBlockSettingsMenuItem } from "@wordpress/editor";
import { __ } from "@wordpress/i18n";
import { useSelect } from "@wordpress/data";
import { store as blockEditorStore } from "@wordpress/block-editor";

const CopyBlockAsJsonMenuItem = () => {
  // Get the selected block
  const selectedBlock = useSelect((select) => {
    return select(blockEditorStore).getSelectedBlock();
  }, []);

  // Handle copying to clipboard
  const handleCopy = () => {
    if (selectedBlock) {
      // Create a JSON string of the block's attributes and inner blocks
      const blockData = {
        name: selectedBlock.name,
        attributes: selectedBlock.attributes,
        innerBlocks: selectedBlock.innerBlocks,
      };

      // Copy the JSON string to clipboard
      navigator.clipboard
        .writeText(JSON.stringify(blockData, null, 2))
        .then(() => {
          // Could add a notice here to confirm copy
          console.log("Block JSON copied to clipboard");
        })
        .catch((err) => {
          console.error("Failed to copy block JSON:", err);
        });
    }
  };

  return (
    <PluginBlockSettingsMenuItem
      icon={
        <svg xmlns="http://www.w3.org/2000/svg">
          <circle r="10" cx="10" cy="10" fill="#FFA500" />
        </svg>
      }
      label={__("Copy Block as JSON", "wpviplearn")}
      onClick={handleCopy}
    />
  );
};

export const registerCopyBlockAsJsonMenuItem = () => {
  registerPlugin("wpviplearn-copy-json", {
    render: CopyBlockAsJsonMenuItem,
  });
};
```

This code adds a custom menu item to the block settings dropdown to copy a block's data as JSON format.

> [!NOTE]
> See [full code](https://github.com/Automattic/wpvip-learn-enterprise-block-editor/blob/trunk/examples/slot-fills/src/CopyBlockAsJsonMenuItem.js) of the example above and a [live demo](https://playground.wordpress.net/?blueprint-url=https://raw.githubusercontent.com/Automattic/wpvip-learn-enterprise-block-editor/refs/heads/trunk/examples/slot-fills/_playground/blueprint.json)

### Creating Sidebar Panels

To add a custom section to the edit Document settings area, you can use the `PluginDocumentSettingPanel` component:

```javascript
import { registerPlugin } from "@wordpress/plugins";
import { PluginDocumentSettingPanel } from "@wordpress/editor";
import { useSelect } from "@wordpress/data";
import { __ } from "@wordpress/i18n";

const ReadingTimePanel = () => {
  const postContent = useSelect(
    (select) => select("core/editor").getEditedPostContent(),
    []
  );

  const calculateReadingTime = (content) => {
    const wordsPerMinute = 200;
    const text = content.replace(/<[^>]*>/g, ""); // Remove HTML tags
    const wordCount = text.trim().split(/\s+/).length;
    return Math.ceil(wordCount / wordsPerMinute);
  };

  const readingTime = calculateReadingTime(postContent);

  return (
    <PluginDocumentSettingPanel
      name="reading-time-panel"
      title={__("Reading Time", "reading-time-panel")}
      icon={
        <svg xmlns="http://www.w3.org/2000/svg">
          <circle r="10" cx="10" cy="10" fill="#800080" />
        </svg>
      }
    >
      <p>
        {__("Estimated Reading Time:", "reading-time-panel")}{" "}
        <strong>
          {readingTime} {__("minutes", "reading-time-panel")}
        </strong>
      </p>
    </PluginDocumentSettingPanel>
  );
};

export const registerReadingTimePanel = () => {
  registerPlugin("reading-time-panel", { render: ReadingTimePanel });
};
```

> [!NOTE]
> See [full code](https://github.com/Automattic/wpvip-learn-enterprise-block-editor/blob/trunk/examples/slot-fills/src/ReadingTimePanel.js) of the example above and a [live demo](https://playground.wordpress.net/?blueprint-url=https://raw.githubusercontent.com/Automattic/wpvip-learn-enterprise-block-editor/refs/heads/trunk/examples/slot-fills/_playground/blueprint.json)

## **Best Practices for Using Slot & Fill**

1. **Use Appropriate Slots**: Choose the correct Slot for your custom UI to ensure it appears in the intended location.
2. **Keep Performance in Mind**: Avoid adding too many Fills, as they can impact editor performance.
3. **Follow WordPress Coding Standards**: Adhere to WordPress coding standards and best practices when creating your custom components.
4. **Use Conditional Rendering**: By default Slot & Fills are imported from `@wordpress/editor` package and they will be applied in both the Post Editor and the Site Editor. You can implement custom logic to show your custom UI only when relevant to the current context.
5. **Consider the Plugin API**: Use the [`registerPlugin`](https://developer.wordpress.org/block-editor/reference-guides/packages/packages-plugins) function to ensure proper registration of your SlotFills.

## Further reading

- [SlotFill Reference Guide](https://developer.wordpress.org/block-editor/reference-guides/slotfills/)
- [WordPress Plugins Package Documentation](https://developer.wordpress.org/block-editor/reference-guides/packages/packages-plugins/)
- [Creating a Plugin for the Block Editor](https://developer.wordpress.org/block-editor/how-to-guides/plugin-sidebar-0/)
- [Block Editor Architecture Key Concepts](https://developer.wordpress.org/block-editor/explanations/architecture/key-concepts/)
- [Block Editor Components Reference](https://developer.wordpress.org/block-editor/reference-guides/components/)
- [Adding a Custom Sidebar to the Block Editor](https://developer.wordpress.org/block-editor/how-to-guides/plugin-sidebar-0/)
- [Block Editor Accessibility](https://developer.wordpress.org/block-editor/explanations/a11y/)
