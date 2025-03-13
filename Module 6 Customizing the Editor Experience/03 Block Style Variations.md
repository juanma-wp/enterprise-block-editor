# 6.3: Registering Custom Block Styles Variations

Block Style Variations allow developers to define alternative visual appearances for existing blocks in the WordPress Block Editor. Unlike block variations, which modify the structure or behavior of a block, style variations focus solely on altering the block's appearance by applying custom CSS classes. This feature enhances the flexibility of the editor, enabling users to quickly switch between predefined styles without modifying the block's core functionality.

By the end of this lesson, will learn how to register block style variations using JavaScript and PHP

## **How do Block Style Variations work**

Block Styles let you apply different looks to existing blocks by adding a special class to them. This class changes the block's appearance when a specific style is selected.

For example, the following code adds a "Fancy Quote" style to the Quote block:

```javascript
wp.blocks.registerBlockStyle("core/quote", {
  name: "fancy-quote",
  label: "Fancy Quote",
});
```

When this style is chosen, the class `is-style-fancy-quote` is added to the block, applying the new design.

## **Register a Block Style Variations using JavaScript**

The [`registerBlockStyle`](https://developer.wordpress.org/block-editor/reference-guides/packages/packages-blocks/#registerblockstyle) function registers a new style variation for a specific block. It requires two parameters:

1. **blockName**: The name of the block (e.g., `core/paragraph`).
2. **styleVariation**: An object containing:
   - `name`: A unique identifier for the style (used as a CSS class).
   - `label`: A human-readable label displayed in the editor.
   - `isDefault`: Optional boolean to set this style as the default.

```javascript
registerBlockStyle("core/image", {
  name: "hand-drawn-js",
  label: __("Hand Drawn JS", "example-block-style-js"),
  isDefault: true,
});
```

This code adds a "'Hand Drawn JS" style to the Image Block. When selected, it applies the class `.is-style-'hand-drawn-js` to the block.

To remove a block style, you can use [`unregisterBlockStyle`](https://developer.wordpress.org/block-editor/reference-guides/packages/packages-blocks/#unregisterblockstyle)`()`.

To ensure this script is loaded in the editor, enqueue it using the [`enqueue_block_editor_assets`](https://developer.wordpress.org/reference/hooks/enqueue_block_editor_assets/) hook:

```php
add_action( 'enqueue_block_editor_assets', 'wpviplearn_editor_assets' );

/**
 * EnqueueEditor scripts and styles
 */
function wpviplearn_editor_assets() {

	$dir = untrailingslashit( plugin_dir_path( __FILE__ ) );
	$url = untrailingslashit( plugin_dir_url( __FILE__ ) );

	if ( file_exists( "{$dir}/build/index.asset.php" ) ) {
		$asset = include "{$dir}/build/index.asset.php";

		wp_enqueue_script(
			'wpviplearn-inspector-controls-filter-block-edit',
			"{$url}/build/index.js",
			$asset['dependencies'],
			$asset['version'],
			true
		);

	}
}
```

The styles that will be applied when the Block Style Variation is selected (and related class is added to the element) could look like this

```css
.wp-block-image.is-style-hand-drawn-js img {
  color: inherit;
  border: 3px dashed currentcolor;
  overflow: hidden;
  box-shadow: 4px 4px 8px rgba(0, 0, 0, 0.2);
  border-radius: 50px 5px 50px 5px/5px 50px 5px 50px !important;
  transform: rotate(2deg);
  padding: 10px;
}
```

To ensure that the styles are properly loaded in both the editor and the frontend, enqueue them using the [`enqueue_block_assets`](https://developer.wordpress.org/reference/hooks/enqueue_block_assets/) hook:

```php
add_action( 'enqueue_block_assets', 'wpviplearn_block_assets' );

/**
 * Enqueue Editor content scripts and styles.
 */
function wpviplearn_block_assets() {

	$dir = untrailingslashit( plugin_dir_path( __FILE__ ) );
	$url = untrailingslashit( plugin_dir_url( __FILE__ ) );

	if ( file_exists( "{$dir}/build/index.css" ) ) {

		wp_enqueue_style(
			'wpviplearn-inspector-controls-filter-block-edit',
			"{$url}/build/index.css",
			array(),
			filemtime( "{$dir}/build/index.css" )
		);
	}
}
```

## **Register a Block Style Variations using PHP**

The [`register_block_style`](https://developer.wordpress.org/reference/functions/register_block_style/) function allows server-side registration of block styles. It takes two arguments:

1. **$block**: The name of the block (e.g., `core/button`).
2. **$style_properties**: An array defining:
   - `name`: Unique identifier for the style.
   - `label`: Display name in the editor.
   - Optional properties like `inline_style` or `style_handle`.

Example:

```php
add_action( 'init', 'wpviplearn_register_block_styles' );

function wpviplearn_register_block_styles() {
	register_block_style(
		'core/image',
		array(
			'name'         => 'hand-drawn-php',
			'label'        => __( 'Hand Drawn PHP', 'example-block-style-php' ),
			'inline_style' => '.wp-block-image.is-style-hand-drawn-php img {
			color: inherit;
			border: 2px solid currentColor;
			overflow: hidden;
			box-shadow: 0 1px 2px 0 rgb( 0 0 0 / 0.05 );
			border-radius: 255px 15px 225px 15px/15px 225px 15px 255px !important;
			transform: rotate(-2deg);
			padding: 20px;
		}',
		)
	);
}
```

This registers an "`Hand Drawn PHP`" style to the image block with inline CSS applied directly

> [!NOTE]
> The [block-styles](https://github.com/Automattic/wpvip-learn-enterprise-block-editor/tree/trunk/examples/block-styles) ([live demo](https://playground.wordpress.net/?blueprint-url=https://raw.githubusercontent.com/Automattic/wpvip-learn-enterprise-block-editor/refs/heads/trunk/examples/block-styles/_playground/blueprint.json)) example illustrates how to to register block style variations via PHP and JS

## Further Reading

- [Block Style Variations – Theme Handbook](https://developer.wordpress.org/themes/features/block-style-variations/)
- [Block Styles API Reference](https://developer.wordpress.org/block-editor/reference-guides/block-api/block-styles/)
- [Section Styles - DevNote](https://make.wordpress.org/core/2024/06/24/section-styles/)
- [Styling Your WordPress Block](https://learn.wordpress.org/tutorial/styling-your-wordpress-block/)
- [`wp_enqueue_script` Reference](https://developer.wordpress.org/reference/functions/wp_enqueue_script/)
- [`wp_enqueue_style` Reference](https://developer.wordpress.org/reference/functions/wp_enqueue_style/)
