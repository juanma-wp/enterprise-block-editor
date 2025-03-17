# Block Bindings

## What Are Block Bindings?

[Block bindings](https://developer.wordpress.org/block-editor/reference-guides/block-api/block-bindings/) are a feature to dynamically populate block content from various sources, such as custom fields, taxonomy terms, or even user-defined sources. [Introduced in WordPress 6.5](https://make.wordpress.org/core/2024/03/06/new-feature-the-block-bindings-api/), block bindings significantly improve the development process by reducing the amount of custom code required to handle data synchronization between the editor and server-side data stores.

In traditional block development, developers often needed to manually fetch and store data using the [REST API](https://developer.wordpress.org/rest-api/), custom endpoints, or JavaScript. With block bindings, WordPress provides a declarative way to link block attributes with external data sources, simplifying block development and improving maintainability.

## Key Concepts of Block Bindings

### Sources

Sources are the origins of the data that will populate block attributes. They can include:

- **Post meta (custom fields)**: Data stored as [post metadata](https://developer.wordpress.org/plugins/metadata/managing-post-metadata/), accessible via the REST API.
- **Custom PHP functions**: Dynamically generated content.

### Bindings

Bindings are the connections between block attributes and their corresponding data sources. These define how and where the data from sources should be applied within a block. These bindings are defined in the markup if the block like the following:

```
!-- wp:paragraph {
    "metadata":{
        "bindings":{
            "content":{
                "source":"core/post-meta",
                "args":{
                    "key":"my_custom_field_key"
                }
            }
        }
    }
} -->
<p></p>
<!-- /wp:paragraph -->
```

The code above shows how the content of Paragraph block can be bound (block binding) to a custom field of the post.

- `metadata`: An object containing metadata about the block.
- `bindings`: An object that can contain one or more bindings for the block.
- `content`: The [block attribute](https://developer.wordpress.org/block-editor/reference-guides/block-api/block-attributes/) to bind to the custom field. For example, a Paragraph block's content attribute.
- `source`: The bindings source registered via the Block Bindings API that will handle the return value on the server. Setting it to `core/post-meta` enables the connection to use the built-in custom fields handling. It can also be set to a custom source.
- `args`: An object that can contain one or more arguments to pass to the block bindings source. For custom fields, the key property must always be set to the name of the meta key you want to bind to the block attribute.

---

## Enhancing Block Interactions with Bindings

### Using Custom Fields as Binding Sources

To use a custom field (post meta) as a binding source, follow these steps:

**Step 1: Register the Custom Field in WordPress**

To ensure the custom field is available via the REST API, you must register it using [`register_meta()`](https://developer.wordpress.org/reference/functions/register_meta/). This makes the data accessible to block bindings.

```
add_action( 'init', 'projectslug_register_meta' );

function projectslug_register_meta() {
    register_meta(
        'post',
        'projectslug_mood',
        array(
            'show_in_rest'      => true,
            'single'            => true,
            'type'              => 'string',
            'sanitize_callback' => 'wp_strip_all_tags'
        )
    );
}
```

**Step 2: Create a Block Binding**

Once the custom field is registered and exposed to the REST API, you can use it within a block. Below is an example of binding a paragraph block’s content to the `projectslug_mood` post meta.

```
<!-- wp:paragraph {
    "metadata":{
        "bindings":{
            "content":{
                "source":"core/post-meta",
                "args":{
                    "key":"projectslug_mood"
                }
            }
        }
    }
} -->
<p></p>
<!-- /wp:paragraph -->
```

With this binding, the paragraph block's content will automatically sync with the `projectslug_mood` custom field.

Check out a [live demo](https://playground.wordpress.net/?blueprint-url=https://raw.githubusercontent.com/Automattic/wpvip-learn-enterprise-block-editor/refs/heads/trunk/examples/block-bindings-post-meta/_playground/blueprint.json) of this example and its [complete code](https://github.com/Automattic/wpvip-learn-enterprise-block-editor/tree/trunk/examples/block-bindings-post-meta). For more detailed instructions on this example, refer to the Developer Blog post ["Introducing Block Bindings, Part 1: Connecting Custom Fields"](https://developer.wordpress.org/news/2024/02/introducing-block-bindings-part-1-connecting-custom-fields/).

---

## Using Custom PHP Functions as Binding Sources

If you want to use dynamic data that isn't stored in a custom field you can create a custom binding source using a PHP function.

**Step 1: Register the Custom Binding Source**

Use [`register_block_bindings_source()`](https://developer.wordpress.org/reference/functions/register_block_bindings_source/) to register a new binding source. Hook this function into [`init`](https://developer.wordpress.org/reference/hooks/init/) to ensure it runs when WordPress loads.

```
add_action( 'init', 'projectslug_register_block_bindings' );

/**
 * Register the block bindings source.
 */
function projectslug_register_block_bindings() {
	register_block_bindings_source(
		'projectslug/user-data',
		array(
			'label'              => __( 'User Data', 'projectslug' ),
			'get_value_callback' => 'projectslug_user_data_bindings',
		)
	);
}

/**
 * Get the user data bindings.
 *
 * @param array $source_args The source arguments.
 * @return string|null The user data bindings.
 */
function projectslug_user_data_bindings( $source_args ) {
	// If no key or user ID argument is set, bail early.
	if ( ! isset( $source_args['key'] ) || ! isset( $source_args['userId'] ) ) {
		return null;
	}

	// Get the user ID.
	$user_id = absint( $source_args['userId'] );

	// Return null if there's no user ID at all.
	if ( 0 >= $user_id ) {
		return null;
	}

	// Return the data based on the key argument.
	switch ( $source_args['key'] ) {
		case 'name':
			return esc_html( get_the_author_meta( 'display_name', $user_id ) );
		case 'description':
			return get_the_author_meta( 'description', $user_id );
		case 'avatar':
			return esc_url( get_avatar_url( $user_id ) );
		default:
			return null;
	}
}

```

The `register_block_bindings_source()` function, has the following signature:

```

register_block_bindings_source(
	string $source_name,
	array $source_properties
);
```

There are two available parameters:

- `$source_name`: A unique name for the custom binding source in the form of `namespace/slug`.

- `$source_properties`: An array of properties to define the binding source:

  - `label`: An internationalized text string to represent the binding source.

  - `get_value_callback`: A PHP callable (function, closure, etc.) that is called when a block's bound attribute source matches the `$source_name` parameter.

  - `uses_context`: (Optional) Extends the block instance with an array of [contexts](https://developer.wordpress.org/block-editor/reference-guides/block-api/block-context/) if needed for the callback.

When WordPress encounters the custom bindings source while parsing a block, it will run the `get_value_callback` function, whose signature should look like this:

```

projectslug_bindings_callback(
	array $source_args,
	WP_Block $block_instance,
	string $attribute_name
);
```

It can accept up to three parameters:

- `$source_args`: An array of arguments passed via the `metadata.bindings.$attribute.args` property from the block.
- `$block_instance`: The current instance of the block the binding is connected to as a [`WP_Block`](https://developer.wordpress.org/reference/classes/wp_block/) object.
- `$attribute_name`: The current attribute set via the `metadata.bindings.$attribute` property key on the block.

Check the [Registering a custom source](https://developer.wordpress.org/block-editor/reference-guides/block-api/block-bindings/#registering-a-custom-source) section on the Block Editor Handbook, to learn more about this.

**Step 2: Bind a Block to the Custom Source**

Modify your block's markup to use the newly registered source.

```
<!-- wp:group {"style":{"spacing":{"blockGap":"1rem"}},"layout":{"type":"default"}} -->
<div class="wp-block-group">
	<!-- wp:heading {
		"level":3,
		"metadata":{
			"bindings":{
				"content":{
					"source":"projectslug/user-data",
					"args":{
						"userId":1,
						"key":"name"
					}
				}
			}
		}
	} -->
	<h3 class="wp-block-heading"></h3>
	<!-- /wp:heading -->

	<!-- wp:image {
		"width":"96px",
		"height":"96px",
		"scale":"cover",
		"metadata":{
			"bindings":{
				"url":{
					"source":"projectslug/user-data",
					"args":{
						"userId":1,
						"key":"avatar"
					}
				}
			}
		},
		"align":"left",
		"style":{"layout":{"selfStretch":"fit","flexSize":null}}
	} -->
	<figure class="wp-block-image alignleft is-resized">
		<img src="" alt="" style="object-fit:cover;width:96px;height:96px"/>
	</figure>
	<!-- /wp:image -->

	<!-- wp:paragraph {
		"metadata":{
			"bindings":{
				"content":{
					"source":"projectslug/user-data",
					"args":{
						"userId":1,
						"key":"description"
					}
				}
			}
		}
	} -->
	<p></p>
	<!-- /wp:paragraph -->
</div>
<!-- /wp:group -->
```

With this setup, the paragraph blocks will dynamically display the different attributes of the current logged user (name, avatar or description) as defined in our custom function.

> [!NOTE]
> Check out a [live demo](https://playground.wordpress.net/?blueprint-url=https://raw.githubusercontent.com/Automattic/wpvip-learn-enterprise-block-editor/refs/heads/trunk/examples/block-bindings-custom-source/_playground/blueprint.json) of this example and its [complete code](https://github.com/Automattic/wpvip-learn-enterprise-block-editor/tree/trunk/examples/block-bindings-custom-source). For more detailed instructions on this example, refer to the Developer Blog post ["Introducing Block Bindings, part 2: Working with custom binding sources"](https://developer.wordpress.org/news/2024/03/introducing-block-bindings-part-2-working-with-custom-binding-sources/).

## Tips to Ensure Functionality of Block Bindings

1. **Ensure WordPress 6.5 or Later**: Block bindings are available from WordPress 6.5 onwards. If you are using an older version, [update your installation](https://wordpress.org/download/).
2. **Enable REST API for Custom Fields**: When registering post meta using `register_meta()`, make sure `show_in_rest` is set to `true` so that it is accessible for bindings.
3. **Use Code Editor for Block Markup**: When adding block bindings manually, switch to the Code Editor (`Options > Code Editor`) in the Block Editor.

---

Block Bindings significantly enhance the flexibility and interactivity of blocks in the WordPress Block Editor. By connecting block attributes to dynamic data sources, developers can create more dynamic and user-friendly content experiences. The continuous improvements in WordPress versions, have made it easier to implement and manage these bindings, providing a more seamless editing experience.

For more detailed information, refer to the official [Block Bindings Documentation](https://developer.wordpress.org/block-editor/reference-guides/block-api/block-bindings/) or any of resources below.

## Resources

- [Introducing Block Bindings, part 1: connecting custom fields](https://developer.wordpress.org/news/2024/02/introducing-block-bindings-part-1-connecting-custom-fields/) | Developer Blog
- [Introducing Block Bindings, part 2: Working with custom binding sources](https://developer.wordpress.org/news/2024/03/introducing-block-bindings-part-2-working-with-custom-binding-sources/) | Developer Blog
- [Getting and setting Block Binding values in the Editor](https://developer.wordpress.org/news/2024/10/getting-and-setting-block-binding-values-in-the-editor/) | Developer Blog
- [Block Bindings: Improvements to the Editor Experience in 6.7](https://make.wordpress.org/core/2024/10/21/block-bindings-improvements-to-the-editor-experience-in-6-7/) | DevNote
- [Editing custom fields from connected blocks](https://make.wordpress.org/core/2024/06/28/editing-custom-fields-from-connected-blocks/) | DevNote
- [New Feature: The Block Bindings API](https://make.wordpress.org/core/2024/03/06/new-feature-the-block-bindings-api/) | DevNote
- [Block Editor Handbook: Block Bindings](https://developer.wordpress.org/block-editor/how-to-guides/block-bindings/) | WordPress Developer Resources
- [WordPress REST API Handbook](https://developer.wordpress.org/rest-api/) | WordPress Developer Resources
- [Block API Reference](https://developer.wordpress.org/block-editor/reference-guides/block-api/) | WordPress Developer Resources
