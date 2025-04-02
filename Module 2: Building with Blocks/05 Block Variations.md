# **Lesson: Block Variations**

## **Introduction**

In this lesson, we will explore block variations within the WordPress Block Editor. Block variations allow developers to create multiple versions of a block with different preset attributes, inner blocks, and configurations. By the end of this lesson, you will understand how to create block variations, manage reusable configurations, and modify variations dynamically using JavaScript and WordPress hooks.

## **Understanding Block Variations**

Block variations provide a mechanism for defining different versions of an existing block without creating entirely new blocks. This approach is widely used in core WordPress blocks, such as the Embed block, which has variations for different content providers (e.g., YouTube, Twitter, and Vimeo) based on the `providerNameSlug` attribute.

By defining variations, developers can enhance the usability of blocks by providing preconfigured versions suited for specific use cases.

## **Defining Block Variations**

A block variation is defined using an object with specific properties that determine its behavior and appearance within the Block Editor. Some of the key properties include:

* **name**: A unique, machine-readable identifier for the variation.
* **title**: The human-readable name of the variation displayed in the editor.
* **description**: A brief explanation of the variation's purpose.
* **icon**: The icon used to represent the variation in the editor.
* **attributes**: An object defining default attributes for the variation.
* **innerBlocks**: An array specifying the initial configuration of nested blocks.
* **scope**: An array indicating where the variation is applicable (e.g., inserter, transforms).
* **isDefault**: A boolean indicating if this variation should be the default.
* **isActive**: A function or array used to determine if the variation is active based on attributes.

## **Creating Block Variations**

To create a block variation, use the `registerBlockVariation` function from the `@wordpress/blocks` package. The following example registers a "`Spacer 180`" variation for the `'core/spacer'` block:

```javascript
wp.blocks.registerBlockVariation( 'core/spacer', {
	name: 'wpviplearn/spacer',
	title: wp.i18n.__( 'Spacer 180', 'example-block-variation' ),
	scope: [ 'block', 'inserter', 'transform' ],
	attributes: {
		height: '180px',
	},
	isActive: ( blockAttributes ) =>
		blockAttributes.height && '180px' === blockAttributes.height,
} );
```

In this example:

* The `name` uniquely identifies the variation as 'themeslug/spacer'.
* The `title` provides a user-friendly label: "Theme Name: Spacer".
* The `attributes` property sets the `height` attribute to '180px', ensuring all instances of this variation have a fixed height.
* The `isActive` function checks if the `height` attribute is set to '180px' and marks this variation as active only if this condition is true.

## **Managing Reusable Configurations**

Reusable configurations streamline the development process by allowing developers to define and register block variations that can be used across multiple projects. This improves consistency and reduces redundancy.

For example, too further optimize block variations, developers can create reusable functions for registering variations dynamically:

```javascript
const sharedAttributes = {
	align: 'wide',
	className: 'custom-block-style',
};

const createVariation = (
	blockName,
	variationName,
	title,
	customAttributes = {},
	customInnerBlocks = []
) => {
	wp.blocks.registerBlockVariation( blockName, {
		name: variationName,
		title,
		attributes: {
			...sharedAttributes,
			...customAttributes,
		},
		innerBlocks: customInnerBlocks,
	} );
};

// Apply reusable configuration
createVariation( 'core/group', 'featured-content', 'Featured Content Group', {
	backgroundColor: 'pale-pink',
} );

createVariation(
	'core/columns',
	'service-columns',
	'Service Columns',
	{
		columns: 3,
	},
	[
		[
			'core/column',
			{},
			[ [ 'core/paragraph', { content: 'Service 1' } ] ],
		],
		[
			'core/column',
			{},
			[ [ 'core/paragraph', { content: 'Service 2' } ] ],
		],
		[
			'core/column',
			{},
			[ [ 'core/paragraph', { content: 'Service 3' } ] ],
		],
	]
);

```

## **Modifying Existing Block Variations Dynamically**

Dynamic modification of block variations allows developers to adjust variations based on specific conditions or user interactions. This can be done using JavaScript and WordPress hooks.

### **Unregistering a Block Variation**

To remove a block variation, use `unregisterBlockVariation`:

```javascript
function unregisterSocialLinkVariations() {
	const allowedVariations = [
		'wordpress',
		'facebook',
		'x',
		'linkedin',
		'github',
		'gravatar',
	];

	// Get all Social Link block variations.
	const allVariations = wp.data
		.select( 'core/blocks' )
		.getBlockVariations( 'core/social-link' );

	allVariations.forEach( function ( variation ) {
		if ( allowedVariations.indexOf( variation.name ) === -1 ) {
			wp.blocks.unregisterBlockVariation(
				'core/social-link',
				variation.name
			);
		}
	} );
}

wp.domReady( () => {
	unregisterSocialLinkVariations();
} );

```

This code removes all the social link variations from the `core/social-link` block, except the ones in the array  `allowedVariations`

### **Modifying an Existing Variation**

The `blocks.registerBlockType` filter can be used to modify existing block variations definitions before they are registered:

```javascript
function customEmbedVariations( settings, name ) {
	if ( name !== 'core/embed' ) {
		return settings;
	}

	const variations = settings.variations?.map( ( variation ) => {
		if ( variation.name === 'youtube' ) {
			return {
				...variation,
				attributes: {
					...variation.attributes,
					align: 'wide',
				},
			};
		}
		return variation;
	} );

	return {
		...settings,
		variations,
	};
}
wp.hooks.addFilter(
	'blocks.registerBlockType',
	'my-plugin/custom-embed-variations',
	customEmbedVariations
);
```

This example modifies the YouTube Embed block variation to always have a `wide` alignment.

## **Practical Example: Creating a Custom Block Variation for the Quote block**

The following example registers a "Site Header" variation using the core Columns block:

```javascript
const quoteApiEditorVariationSettings = {
	name: 'quote-random-editor',
	description:
		'A "core/quote" block variation that displays a random quote from a local JSON file',
	title: 'Quote Random',
	scope: [ 'block', 'inserter', 'transform' ],
	keywords: [ 'quote' ],
	icon: 'universal-access',
	attributes: {
		namespace: 'quote-random-editor',
	},
	isActive: [ 'namespace' ],
};

registerBlockVariation( 'core/quote', quoteApiEditorVariationSettings );
```

In this example:

* The variation is named `'quote-random-editor` and labeled "`Quote Random`".
* It applies a value to the `namespace` attribute
* In the [`isActive` property](https://developer.wordpress.org/block-editor/reference-guides/block-api/block-variations/#using-isactive), It sets the value of the attribute namespace as the value to decide which variation is active.

As `namespace` is not a default attribute of `core/quote` it must be added previously as the block variation can only modify existing attributes

```javascript
function addAttributes( settings ) {
	if ( 'core/quote' !== settings.name ) {
		return settings;
	}

	const extraAttributes = {
		namespace: {
			type: 'string',
		},
	};

	const newSettings = {
		...settings,
		attributes: {
			...settings.attributes,
			...extraAttributes,
		},
	};

	return newSettings;
}

addFilter(
	'blocks.registerBlockType',
	'quote-random/add-attributes',
	addAttributes
);
```

We can use the namespace attribute to determine which variation is active to add specific Inspector Controls to specific variations:

```javascript
const isQuoteAPIEditorVariation = ( props ) => {
	const {
		attributes: { namespace },
	} = props;
	return namespace && namespace === 'quote-random-editor';
};
// The inspector controls for the "quote-random-editor" variation.const QuoteAPIEditorInspectorControls = ( props ) => { ... }function addInspectorControls( BlockEdit ) {
	return ( props ) => {
		if ( ! isQuoteAPIEditorVariation( props ) ) {
			return <BlockEdit { ...props } />;
		}
		return (
			<>
				<BlockEdit { ...props } />
				<QuoteAPIEditorInspectorControls { ...props } />
			</>
		);
	};
}addFilter(
	'editor.BlockEdit',
	'quote-random-editor/add-inspector-controls',
	addInspectorControls
);
```

The inspector controls for our Quote Block Variation could include a button to randomly insert a quote from a local json file

```javascript
import quotes from './quotes.json';

...

const QuoteAPIEditorInspectorControls = ( props ) => {
	const { clientId, setAttributes } = props;

	const onClickUpdateData = () => {
		const randomQuote =
			quotes[ Math.floor( Math.random() * quotes.length ) ];
		setAttributes( {
			citation: randomQuote.author,
		} );
		const newInnerParagraphWithQuote = [
			createBlock( 'core/paragraph', {
				content: randomQuote.content,
			} ),
		];
		dispatch( blockEditorStore ).replaceInnerBlocks(
			clientId,
			newInnerParagraphWithQuote
		);
	};

	return (
		<InspectorControls>
			<PanelBody
				title={ __( 'Quote settings', 'quote-random' ) }
				initialOpen={ true }
			>
				<PanelRow>
					<Button
						variant="primary"
						label={ __( 'Update data', 'quote-random' ) }
						onClick={ onClickUpdateData }
						icon="update"
						iconPosition="left"
					>
						{ __( 'Get random quote', '' ) }
					</Button>
				</PanelRow>
			</PanelBody>
		</InspectorControls>
	);
};

```

Check out a [live demo](https://playground.wordpress.net/?blueprint-url=https://raw.githubusercontent.com/WordPress/block-development-examples/trunk/plugins/inner-blocks-dcd824/_playground/blueprint.json) of this example and its [complete code](https://github.com/WordPress/block-development-examples/tree/trunk/plugins/inner-blocks-dcd824).

## **Conclusion**

Block variations are a powerful way to extend and customize blocks in the WordPress Block Editor. By understanding the `registerBlockVariation` API, managing reusable configurations, and dynamically modifying variations, developers can create more efficient, reusable, and flexible block configurations. Leveraging these techniques will enhance the editing experience and improve the maintainability of WordPress projects.

For further details, refer to the [WordPress Developer Documentation on Block Variations](https://developer.wordpress.org/block-editor/reference-guides/block-api/block-variations/).
