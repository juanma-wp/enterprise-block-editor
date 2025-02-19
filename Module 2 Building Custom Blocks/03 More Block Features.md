# More Block Features

## Dynamic Blocks with PHP Integration

Dynamic blocks allow developers to create blocks whose content is generated on the server-side using PHP. This approach is particularly useful for blocks that need to display frequently updated information or data that requires complex processing.

To create a dynamic block, we need to register the block on the PHP side and provide a callback function that will render the block's content. Let's create a simple example of a dynamic block that displays the current date and time.

First, we'll register our block in PHP:

```
function register_dynamic_date_block() {
    register_block_type( 'my-plugin/dynamic-date', array(
        'render_callback' => 'render_dynamic_date_block',
        'attributes'      => array(
            'format' => array(
                'type'    => 'string',
                'default' => 'F j, Y H:i:s',
            ),
        ),
    ) );
}
add_action( 'init', 'register_dynamic_date_block' );
```

In this code, we're registering a block called 'my-plugin/dynamic-date' and specifying a render callback function. We're also defining an attribute 'format' that will allow us to customize the date format.

Next, let's implement the render callback function:

```
function render_dynamic_date_block( $attributes ) {
    $format = isset( $attributes['format'] ) ? $attributes['format'] : 'F j, Y H:i:s';
    return '<p class="dynamic-date">' . date( $format ) . '</p>';
}
```

This function takes the block attributes as an argument, retrieves the 'format' attribute (or uses a default if not set), and returns an HTML string with the current date formatted accordingly.

On the JavaScript side, we need to create an edit function that will render in the editor:

```javascript
import { useBlockProps } from '@wordpress/block-editor';
import { __ } from '@wordpress/i18n';
import ServerSideRender from '@wordpress/server-side-render';

export default function Edit( { attributes, setAttributes } ) {
    const blockProps = useBlockProps();

    return (
        <div { ...blockProps }>
            <ServerSideRender
                block="my-plugin/dynamic-date"
                attributes={ attributes }
            />
        </div>
    );
}
```

In this edit function, we're using the ServerSideRender component to display the server-rendered content in the editor.

By using dynamic blocks, we can leverage the full power of PHP to generate block content while still providing a seamless editing experience in the Block Editor.

## Handling Complex Attributes

As blocks become more sophisticated, they often require more complex attributes. These can include nested blocks, media fields, or other structured data. Let's explore how to handle these types of attributes.

### Nested Blocks

Nested blocks allow you to create a block that contains other blocks. This is achieved using the InnerBlocks component. Here's an example of a custom container block that can contain other blocks:

```javascript
import { useBlockProps, InnerBlocks } from '@wordpress/block-editor';

export default function Edit() {
    const blockProps = useBlockProps();

    return (
        <div { ...blockProps }>
            <InnerBlocks />
        </div>
    );
}
```

In this example, the InnerBlocks component allows the user to insert any other block inside our custom container block.

We can also restrict the types of blocks that can be inserted:

```javascript
const ALLOWED_BLOCKS = [ 'core/paragraph', 'core/image' ];

export default function Edit() {
    const blockProps = useBlockProps();

    return (
        <div { ...blockProps }>
            <InnerBlocks allowedBlocks={ ALLOWED_BLOCKS } />
        </div>
    );
}
```

This will only allow paragraph and image blocks to be inserted into our container.

### Media Fields

To handle media fields, we can use the MediaUpload component provided by WordPress. Here's an example of a block with an image attribute:

```javascript
import { useBlockProps, MediaUpload, MediaUploadCheck } from '@wordpress/block-editor';
import { Button } from '@wordpress/components';
import { __ } from '@wordpress/i18n';

export default function Edit( { attributes, setAttributes } ) {
    const blockProps = useBlockProps();
    const { imageId, imageUrl } = attributes;

    const onSelectImage = ( media ) => {
        setAttributes( {
            imageId: media.id,
            imageUrl: media.url,
        } );
    };

    return (
        <div { ...blockProps }>
            <MediaUploadCheck>
                <MediaUpload
                    onSelect={ onSelectImage }
                    allowedTypes={ [ 'image' ] }
                    value={ imageId }
                    render={ ( { open } ) => (
                        <Button onClick={ open }>
                            { ! imageId ? __( 'Choose an image', 'my-plugin' ) : __( 'Replace image', 'my-plugin' ) }
                        </Button>
                    ) }
                />
            </MediaUploadCheck>
            { imageUrl && <img src={ imageUrl } alt="" /> }
        </div>
    );
}
```

In this example, we're using the MediaUpload component to allow the user to select an image from the media library. We're storing both the image ID and URL as attributes of the block.

## Internationalization (i18n) for Blocks

Internationalization, often abbreviated as i18n, is the process of designing and preparing your block for translation into different languages. WordPress provides tools to make this process easier.

To internationalize your block, you'll need to use the \_\_ function (for single strings) or \_n function (for strings with plurals) from the @wordpress/i18n package. Here's an example:

```javascript
import { __ } from '@wordpress/i18n';

export default function Edit() {
    return (
        <div>
            <h2>{ __( 'My Block Title', 'my-plugin' ) }</h2>
            <p>{ __( 'This is the content of my block.', 'my-plugin' ) }</p>
        </div>
    );
}
```

In this example, we're wrapping our strings with the \_\_ function, which marks them for translation. The second argument to \_\_ is the text domain, which should match the text domain of your plugin or theme.

For plurals, you can use the \_n function:

```javascript
import { _n } from '@wordpress/i18n';

export default function Edit( { attributes } ) {
    const { count } = attributes;
    return (
        <div>
            { _n(
                'You have %d item.',
                'You have %d items.',
                count,
                'my-plugin'
            ) }
        </div>
    );
}
```

The \_n function takes four arguments: the singular form, the plural form, the count, and the text domain.

To extract these strings for translation, you'll need to use a tool like wp-cli or the @wordpress/scripts package. These tools will generate a .pot file containing all of your translatable strings, which can then be translated into different languages.

Remember to also internationalize any strings in your PHP code using similar functions like \_\_() and \_n().

By following these internationalization practices, you ensure that your blocks can be easily translated, making them accessible to a global audience.

In conclusion, mastering dynamic blocks, complex attributes, and internationalization will greatly enhance your ability to create sophisticated and widely usable blocks for the WordPress Block Editor. These advanced features allow you to create blocks that are not only powerful and flexible but also accessible to users around the world.

