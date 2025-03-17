# Using the WordPress VIP Block Governance Plugin

In this lesson, we will explore the [WordPress VIP Block Governance plugin](https://docs.wpvip.com/vip-go-mu-plugins/block-governance-plugin/), a powerful tool designed to enhance editorial permissions and design governance within the WordPress Block Editor. By the end of this lesson, you will be able to identify the key features and functionalities of the plugin, understand how it enforces content governance, and learn how to modify block permissions and governance settings to tailor the editing experience for different user roles and workflows.

## Key Features and Functionalities of the VIP Block Governance Plugin

The WordPress VIP Block Governance plugin is engineered to provide organizations with granular control over the Gutenberg Block Editor by allowing administrators to:

- **Restrict Block Insertion**: Limit the types of blocks that can be added to the editor, ensuring that only approved blocks are available for content creation.

- **Control Styling Options**: Define which styling options are permissible for each block, maintaining design consistency across the site.

- **Set Rules Based on User Roles and Post Types**: Apply specific block and style restrictions depending on user roles and post types, tailoring the editing environment to suit various workflows and responsibilities.

By implementing these controls, the plugin enables organizations to encode their design systems directly into the site's block editor, reducing complexities in content creation and ensuring adherence to established design standards.

## Enabling the Plugin

For WordPress VIP sites, you can enable the Block Governance plugin by adding the following code to your application's `plugin-loader.php` in the client-mu-plugins directory:

```php
\Automattic\VIP\Integrations\activate( 'vip-governance' );
```

For any other site you can enable the plugin by downloading it from the [WordPress VIP GitHub repository](https://github.com/Automattic/vip-governance-plugin), installing it manually in your plugins directory and activating it in your WordPress installation.

## Enforcing Content Governance with the VIP Block Governance Plugin

The VIP Block Governance plugin enforces content governance through several mechanisms:

### Restricting Block Usage

Administrators can specify which blocks are available for use, preventing the insertion of unauthorized or potentially disruptive blocks. This restriction ensures that content creators adhere to the organization's content strategy and design guidelines.

### Ensuring Editorial Consistency

By controlling the blocks and styles available in the editor, the plugin helps maintain a uniform editorial voice and presentation. This consistency is crucial for brand integrity and user experience, as it prevents deviations from approved content structures and designs.

### Maintaining Design Standards

The plugin allows for the enforcement of design standards by limiting styling options to those that align with the organization's visual identity. This control minimizes the risk of styling errors and ensures that all content remains on-brand.

## Modifying Block Permissions and Governance Settings

To tailor the editing experience for different user roles and workflows, administrators can modify block permissions and governance settings through a configuration file named `governance-rules.json`.

> In the context of WordPress VIP sites this file should be placed in the `/private` directory of the application's `wpcomvip` GitHub repository. For any other site with the plugin installed this file should be placed in the root of the `vip-governance` plugin folder.

### Structure of `governance-rules.json`

The configuration file follows a specific schema and includes:

- **Versioning**: Specifies the version of the ruleset.

- **Rules Array**: Contains individual rule objects that define the governance settings.

Each rule object can include the following properties:

- **`type`** (string): Defines the scope of the rule, such as 'default', 'role', or 'postType'.

- **`allowedFeatures`** (array): Lists the editor features that are permitted, such as 'codeEditor' or 'lockBlocks'.

- **`allowedBlocks`** (array): Specifies the blocks that are allowed for insertion.

- **`allowedStyles`** (object): Defines permissible styling options for blocks.

Here is an example of a basic `governance-rules.json` configuration:

```json
{
  "$schema": "https://api.wpvip.com/schemas/plugins/governance.json",
  "version": "1.0.0",
  "rules": [
    {
      "type": "default",
      "allowedFeatures": ["codeEditor", "lockBlocks"],
      "allowedBlocks": ["*"]
    }
  ]
}
```

In this example, all blocks are allowed, and both the code editor and block locking features are enabled. Administrators can customize this configuration to restrict specific blocks or features as needed

### Applying Rules Based on User Roles and Post Types

The plugin allows for the application of specific rules based on user roles or post types by setting the `type` property to 'role' or 'postType' and providing the corresponding `roles` or `postTypes` array. For example:

```json
{
  "$schema": "https://api.wpvip.com/schemas/plugins/governance.json",
  "version": "1.0.0",
  "rules": [
    {
      "type": "role",
      "roles": ["editor"],
      "allowedFeatures": ["lockBlocks"],
      "allowedBlocks": ["core/paragraph", "core/image"]
    },
    {
      "type": "postType",
      "postTypes": ["post"],
      "allowedFeatures": ["codeEditor"],
      "allowedBlocks": ["core/paragraph", "core/heading", "core/list"]
    }
  ]
}
```

In this configuration:

- Editors are permitted to use the block locking feature and can insert only paragraph and image blocks.

- For the 'post' post type, the code editor feature is enabled, and only paragraph, heading, and list blocks are allowed.

This level of customization ensures that the editing environment aligns with the specific responsibilities and workflows of different user roles and content types.

### Example Configurations

Check out these example configurations of `governance-rules.json`:

- [Basic](https://github.com/Automattic/wpvip-learn-enterprise-block-editor/blob/trunk/examples/vip-governance-plugin-demo/basic/governance-rules.json) ([live demo](https://playground.wordpress.net/?blueprint-url=https://raw.githubusercontent.com/Automattic/wpvip-learn-enterprise-block-editor/refs/heads/trunk/examples/vip-governance-plugin-demo/basic/_playground/blueprint.json)) - A simple configuration limiting the blocks available
- [Per Role](https://github.com/Automattic/wpvip-learn-enterprise-block-editor/blob/trunk/examples/vip-governance-plugin-demo/per-role/governance-rules.json) ([live demo](https://playground.wordpress.net/?blueprint-url=https://raw.githubusercontent.com/Automattic/wpvip-learn-enterprise-block-editor/refs/heads/trunk/examples/vip-governance-plugin-demo/per-role/_playground/blueprint.json)) - A simple role-based configuration
- [Per Post Type](https://github.com/Automattic/wpvip-learn-enterprise-block-editor/blob/trunk/examples/vip-governance-plugin-demo/per-post-type/governance-rules.json) ([live demo](https://playground.wordpress.net/?blueprint-url=https://raw.githubusercontent.com/Automattic/wpvip-learn-enterprise-block-editor/refs/heads/trunk/examples/vip-governance-plugin-demo/per-post-type/_playground/blueprint.json)) - A post-type-based configuration that includes block settings for specific blocks
- [Advanced](https://github.com/Automattic/wpvip-learn-enterprise-block-editor/blob/trunk/examples/vip-governance-plugin-demo/advanced/governance-rules.json) ([live demo as author](https://playground.wordpress.net/?blueprint-url=https://raw.githubusercontent.com/Automattic/wpvip-learn-enterprise-block-editor/refs/heads/trunk/examples/vip-governance-plugin-demo/advanced/_playground/as-author/blueprint.json) - [live demo as editor](https://playground.wordpress.net/?blueprint-url=https://raw.githubusercontent.com/Automattic/wpvip-learn-enterprise-block-editor/refs/heads/trunk/examples/vip-governance-plugin-demo/advanced/_playground/as-editor/blueprint.json)) - An advanced configuration with different rule types and detailed block settings

## WordPress Admin Dashboard Settings

The plugin provides a settings page in the WordPress Admin dashboard with features for enablement and debugging:

- **Plugin Settings**: Quickly enable/disable the plugin's rules without code changes
- **Governance Rules Validation**: Validate your JSON ruleset configuration
- **View Governance Rules**: Filter and view rules by role type and post type combinations

Access to these settings requires the `manage_options` capability (Administrators on single sites, Super Admins on multisites).

## Conclusion

The WordPress VIP Block Governance plugin is a vital tool for organizations seeking to enforce editorial permissions and design governance within the Block Editor. By restricting block usage, controlling styling options, and tailoring settings based on user roles and post types, the plugin ensures consistency, maintains design standards, and streamlines the content creation process. Administrators can leverage the `governance-rules.json` configuration file to define and enforce these rules effectively.
