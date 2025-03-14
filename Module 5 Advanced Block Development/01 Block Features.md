# Block Features

## Dynamic Blocks

Dynamic blocks allow developers to create blocks where the front end content is generated on the server-side using PHP. This approach is particularly useful for blocks that need to display frequently updated information or rendering data that requires complex processing.

To create a dynamic block, you need to provide a one of two methods to render the block's content.

1. You can either define a PHP file in the `render` property of the block.json file.
2. You can also define a render\_callback function when you register the block in PHP.

Let's look at an example of both approaches.

### Using the render property in block.json

The first step is to add the `render` property to the block.json file, and set it to the path of the PHP file that will render the block.

```
	"editorScript": "file:./index.js",
	"editorStyle": "file:./index.css",
	"style": "file:./style-index.css",
	"viewScript": "file:./view.js",
	"render": "file:./index.php"
```

You can add the render property anywhere to the block.json file, but it's common to add it to the bottom of the file, after scripts and styles.

Next, you need to create the PHP file that will render the block. This file will be responsible for generating the HTML content that will be displayed in the block.

Here's an example of a simple PHP file that will render the block:

```
$format = isset( $attributes['format'] ) ? $attributes['format'] : 'F j, Y H:i:s';
echo '<p class="dynamic-date">' . date( $format ) . '</p>';
```

### Using the render\_callback argument

The other option is to pass a callback function to the render\_callback argument when you register the block with register\_block\_type in PHP.

```
add_action( 'init', 'register_dynamic_date_block' );

function register_dynamic_date_block() {
    register_block_type( 
        'my-plugin/dynamic-date', array(
            'render_callback' => 'render_dynamic_date_block'
        ) 
    );
}

function render_dynamic_date_block( $attributes ) {
    $format = isset( $attributes['format'] ) ? $attributes['format'] : 'F j, Y H:i:s';
    return '<p class="dynamic-date">' . date( $format ) . '</p>';
}
```

### Data passed to a dynamic block

In both cases, the following data is available to the dynamic block code:

- `$attributes`: The array of attributes for the block.
- `$content`: The markup of the block as stored in the database, if any.
- `$block`: The instance of the WP\_Block class that represents the rendered block (metadata of the block).

### The block's save function when rendering a dynamic block

When rendering a dynamic block, the block's save functionality is no longer needed. Therefore you can simply remove the all this functionality:

- Remove the save property from the registerBlockType call in the index.js file when you register the block.
- Remove the importing of the save function from the index.js file.
- Delete the save.js file

## Internationalization

Internationalization, often abbreviated as i18n, is the process of preparing your block for translation into different languages. Whenever you use text strings in your block, you need to provide a way for them to be translated. Fortunately WordPress already provides tools to make this process easier, as long as you use the correct functions.

To internationalize your block, you can use the relevant functions from the @wordpress/i18n [package](https://developer.wordpress.org/block-editor/reference-guides/packages/packages-i18n/). In the Edit component of the block scaffolded by create-block, you'll see that the strings are wrapped in the \_\_() function:

```javascript
import { __ } from '@wordpress/i18n';

export default function Edit() {
	return (
		<div { ...useBlockProps() }>
			<h2>{ __( 'My Custom Inner Blocks', 'my-custom-block' ) }</h2>
			<InnerBlocks />
		</div>
	);
}
```

For plurals, you can use the \_n() function:

```javascript
import { _n } from '@wordpress/i18n';

export default function Edit( props ) {
    const { count } = props.attributes;
    return (
        <div>
            { _n(
                'You have %d item.',
                'You have %d items.',
                count,
                'my-custom-block'
            ) }
        </div>
    );
}
```

These functions, as well as the rest of the functions in the @wordpress/i18n package, both require a text domain as a second argument, similar to how you would [internationalize strings in your PHP code](https://developer.wordpress.org/plugins/internationalization/how-to-internationalize-your-plugin/).  