# Security Best Practices for Blocks

When developing WordPress blocks, you are responsible for ensuring that they handle data securely and only allow access to block functionality to users with the relevant permissions.

This lesson will explore essential security best practices that should be implemented in every block you develop.

\[Note\] The official WordPress developer handbook has an entire [section dedicated to developing with security in mind](https://developer.wordpress.org/apis/security/), which is recommended reading for block developers.

## Understanding Security in Block Development

Blocks often handle user input, store data in the database, and render dynamic content on the front end. Without proper security measures, blocks can become vectors for various attacks, including Cross-Site Scripting (XSS), SQL injection, and unauthorized access to functionality.

During development, you are encouraged to follow these security practices:

**Don’t trust any data**.

Don’t trust user input, third-party APIs, or data in your database without verification. Protecting your WordPress blocks begins with ensuring the data entering and leaving your block is as intended. Always validate and sanitize user input before using it and escape all data on output.

**Rely on WordPress APIs**.

Many core WordPress APIs provide built-in functionality for validating and sanitizing data. Rely on the WordPress-provided APIs whenever possible.

**Keep your code up to date**.

As technology evolves, the potential for new security holes in your blocks increases. Stay vigilant by maintaining your code and updating it as needed.

## Relying on WordPress APIs

As a primarily JavaScript-powered environment, the Block Editor, and more specifically, the [WordPress Data Layer](https://developer.wordpress.org/block-editor/reference-guides/data/) powered by the `@wordpress/data` package, makes use of the default [WordPress REST API](https://developer.wordpress.org/rest-api/) endpoints and the `@wordpress/api-fetch` [package](https://developer.wordpress.org/block-editor/reference-guides/packages/packages-api-fetch/) to send and receive data from the WordPress database.

You should rely on these APIs to store and retrieve any data needed to power your block's functionality as much as possible. By extension, this means using core WordPress data APIs like Custom Post Types, Post Meta, and Taxonomies for your block's data storage needs.

This will ensure that data is validated and sanitized before being saved to the database and escaped before being returned to your block and that any requests made include proper security checks.

However, some additional security measures can be implemented in your block development workflow to ensure the security of your block code.

## Validating Block Attributes

While block attributes will already be validated before being processed by the data layer, validating them before saving is still a good idea.

In your block's `edit` function, you can implement basic validation before updating an attribute:

```javascript
import { useBlockProps } from '@wordpress/block-editor';
import { TextControl } from '@wordpress/components';

export default function Edit({ attributes, setAttributes }) {
    const { title } = attributes;

    const updateTitle = (newTitle) => {
        // Basic validation
        if (newTitle.length > 100) {
            // Notify user of error
            return;
        }
        setAttributes({ title: newTitle });
    };

    return (
        <div { ...useBlockProps() }>
            <TextControl
                label="Title"
                value={title}
                onChange={updateTitle}
            />
        </div>
    );
};
```

This helps by checking that data is within expected bounds before requesting to update the block's attribute.

## Escaping in Content

When working with dynamic data in your block's save functions, use the appropriate escaping methods to prevent XSS attacks.

For example, there's a `@wordpress/escape-html` [package](https://developer.wordpress.org/block-editor/reference-guides/packages/packages-escape-html/) that provides functions for escaping HTML and attributes:

```javascript
import { useBlockProps } from '@wordpress/block-editor';
import { escapeHTML, escapeAttribute } from '@wordpress/escape-html';

export default function save() {
    const { title, description } = attributes;

    return (
        <div { ...useBlockProps.save() }>
            <h2>{escapeHTML(title)}</h2>
            <div dangerouslySetInnerHTML={{ __html: escapeHTML(description) }} />
        </div>
    );
};
```

Notice the use of the React `dangerouslySetInnerHTML` [property](https://react.dev/reference/react-dom/components/common#dangerously-setting-the-inner-html) to render the escaped HTML content.

`dangerouslySetInnerHTML` allows you to render HTML markup as HTML instead of as text. The name makes it sound dangerous, but in this example, it’s safe because the data is being escaped.

WordPress core also uses this property in the Block Editor to render the content of the post in the editor.

## Escaping Dynamic Output

When rendering block content for a dynamic block, always use the appropriate escaping function:

```
/**
 * Render callback for my custom block.
 *
 * @param array    $attributes Block attributes.
 * @param string   $content    Block content.
 * @param WP_Block $block      Block instance.
 * @return string  Rendered block output.
 */
function render_my_block( $attributes, $content, $block ) {
    $title = isset( $attributes['title'] ) ? $attributes['title'] : '';
    $url = isset( $attributes['url'] ) ? $attributes['url'] : '';
    
    $output = '';
    $output .= '' . esc_html( $title ) . '';
    $output .= '' . esc_html__( 'Learn More', 'my-text-domain' ) . '';
    $output .= '';
    
    return $output;
}
```

WordPress provides several [escaping functions](https://developer.wordpress.org/apis/security/escaping/) for different contexts:

- `esc_html()` \- Use when outputting plain text with no HTML allowed
- `esc_attr()` \- Use when outputting data within HTML attributes
- `esc_url()` \- Use when outputting URLs (links, image sources, etc.)
- `esc_js()` \- Use when outputting data within JavaScript
- `wp_kses()` and `wp_kses_post()` \- Use these when you need to allow specific HTML tags

## Validating User Roles and Capabilities

WordPress ships with a robust system of roles and capabilities that determine what actions users can perform:

- **Roles** are collections of capabilities (e.g., Administrator, Editor, Author)
- **Capabilities** are specific permissions (e.g., can edit posts, can publish posts, can manage options)

These roles and capabilities control access to the Block Editor and, therefore, block functionality. For example, only Administrators can access the Site Editor or edit Pages, while Editors can only edit Posts.

However, it can also be useful to check for specific capabilities within your block code to restrict access to blocks.

### Checking User Capabilities in PHP

One of the benefits of using WordPress is that you can check user capabilities in any PHP code:

```
add_action( 'init', 'vip_learn_custom_block_init' );

function vip_learn_custom_block_init() {
	if ( current_user_can( 'manage_options' ) ) {
		register_block_type( __DIR__ . '/build' );
	}
}
```

Here, the `current_user_can()` [function](https://developer.wordpress.org/reference/functions/current_user_can/) checks if the current user has the `manage_options` capability, which is typically reserved for Administrators. If the user has this capability, the custom block is registered.

### Checking Capabilities in JavaScript

In the Block Editor, you can use the `@wordpress/core-data` package to check current user capabilities:

```javascript
import { useBlockProps } from '@wordpress/block-editor';
import { Button } from '@wordpress/components';
import { useSelect } from '@wordpress/data';
import { store as coreStore } from '@wordpress/core-data';

export default function Edit({ attributes, setAttributes }) {    
    const canPublishPosts = useSelect(select => {
        return select(coreStore).canUser('create', 'posts');
    }, []);

    return (
        <div { ...useBlockProps() }>
            {canPublishPosts ? (
                <Button
                    onClick={() => {
                        // Perform action that requires publish_posts capability
                    }}
                >
                    Publish
                </Button>
            ) : (
                <p>You don't have permission to publish posts.</p>
            )}
        </div>
    );
}

```

### Restricting Block Features Based on Capabilities

You can conditionally render block controls or features based on user capabilities:

```javascript
import { useBlockProps } from '@wordpress/block-editor';
import { TextControl, ToggleControl } from '@wordpress/components';
import { useSelect } from '@wordpress/data';
import { store as coreStore } from '@wordpress/core-data';

export default function Edit({ attributes, setAttributes }) {
    const { canEditPosts, canManageOptions } = useSelect(select => {
        return {
            canEditPosts: select(coreStore).canUser('update', 'posts'),
            canManageOptions: select(coreStore).canUser('update', 'settings')
        };
    }, []);

    return (
        <div { ...useBlockProps() }>
            <TextControl
                label="Title"
                value={attributes.title}
                onChange={(title) => setAttributes({ title })}
            />

            {canEditPosts && (
                <TextControl
                    label="Advanced Setting"
                    value={attributes.advancedSetting}
                    onChange={(advancedSetting) => setAttributes({ advancedSetting })}
                />
            )}

            {canManageOptions && (
                <ToggleControl
                    label="Admin-only Setting"
                    checked={attributes.adminSetting}
                    onChange={(adminSetting) => setAttributes({ adminSetting })}
                />
            )}
        </div>
    );
}
```

### Custom Capabilities for Blocks

For more granular control, you can add custom capabilities to specific roles to perform specific capability checks:

```
register_activation_hook( __FILE__, 'add_block_capabilities' );

/**
 * Add custom capability for managing specific blocks.
 */
function add_block_capabilities() {
    // Add the capability to the administrator role
    $role = get_role( 'administrator' );
    if ( $role ) {
        $role->add_cap( 'manage_custom_blocks' );
    }
    
    // Optionally add to editor role
    $editor_role = get_role( 'editor' );
    if ( $editor_role ) {
        $editor_role->add_cap( 'manage_custom_blocks' );
    }
}


/**
 * Check for custom block capability.
 */
function user_can_manage_custom_blocks() {
    return current_user_can( 'manage_custom_blocks' );
}
```

This function is then used to conditionally register blocks.

```
add_action( 'init', 'vip_learn_custom_block_init' );

function vip_learn_custom_block_init() {
	if ( user_can_manage_custom_blocks() ) {
		register_block_type( __DIR__ . '/build' );
	}
}
```

## Conclusion

Security is a critical aspect of block development that should never be an afterthought. By implementing proper data validation, output escaping, and capability checks, you can create functional and secure blocks.

Remember the key principles:

- Validate early, escape late
- Never trust user input
- Use the appropriate sanitization and escaping functions for each context
- Always check user capabilities before allowing actions

These practices will help protect your WordPress sites from common security vulnerabilities and ensure that your blocks contribute positively to the overall security posture of the sites where they're installed.  