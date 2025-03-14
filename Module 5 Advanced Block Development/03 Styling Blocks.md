# Styling Blocks

In the world of WordPress block development, creating visually appealing and functional blocks is crucial. This lesson will delve into the intricacies of styling blocks, covering editor styles, frontend styles, CSS selectors, and responsive design techniques. By the end of this lesson, you'll have a comprehensive understanding of how to create blocks that look great both in the editor and on the front end.

## Block style vs. Editor style

In the Anatomy of a Block lesson, you learned that create-block configures two files for you: `style.scss` and `editor.scss`. These files are [Syntactically Awesome Style Sheets](https://sass-lang.com/) (or Sass) and are used to define the styles for your block.

During the build process, these files are compiled into CSS and enqueued in the appropriate places.

### Block Styles

The block styles in `style.scss` are applied to blocks when they are rendered on the front end of the website. These styles determine how the block will appear to visitors of your site.

```
.wp-block-create-block-my-custom-block {
  background-color: #21759b;
  color: #fff;
  padding: 2px;
}
```

Because these styles are applied to the block, they are also enqueued in the editor so that that block looks the same when using it in the editor as it does on the front end.

### Editor Styles

On the other hand, the editor styles in `editor.scss` are only applied to the block when it's displayed in the editor. This allows you to style the block differently in the editor than it appears on the front end.

```
.wp-block-create-block-my-custom-block {
  border: 1px dotted #f00;
}
```

Generally, editor styles are used if the block has additional elements that are only visible in the editor, for example, performing some tasks that filter the data to be displayed in the final block that's saved to the database.

## CSS Selectors

When styling custom blocks, it's useful to know which block CSS selectors are available.

### Block-Specific Classes

When using `useBlockProps` on the block's container element, it automatically adds a class to the block's wrapper div. This class follows the pattern `wp-block-{namespace}-{blockname}`.

For example, if you've scaffolded a custom block called `my-custom-block` with create-block, the default namespace is `create-block`, and so the wrapper element would have the class `wp-block-create-block-my-custom-block`.

You can, therefore target this class in your CSS to apply styles to your entire block:

```
.wp-block-create-block-my-custom-block {
	background-color: #21759b;
	color: #fff;
	padding: 2px;
}
```

### Adding classes to useBlockProps

You can also add custom classes to the block's container element by passing them to the `className` prop of `useBlockProps`. This allows you greater control over the styling of your block:

```
import { useBlockProps } from '@wordpress/block-editor';

export default function Edit() {
    const blockProps = useBlockProps({ className: 'custom-block-container' });

    return (
        <div { ...blockProps }>
            <h2>Block Title</h2>
            <p>Block content goes here.</p>
        </div>
    );
}
```

Then, in your CSS, you can target this custom class:

```
.wp-block-create-block-my-custom-block .custom-block-container {
    // Custom styles for this block
}
```

### Element-Specific Classes

For more granular control, you can add custom classes to specific elements within your block. In your block's `edit` and `save` functions, you can use the `className` prop to add these classes:

```
import { useBlockProps } from '@wordpress/block-editor';

export default function Edit() {
    const blockProps = useBlockProps();

    return (
        <div { ...blockProps }>
            <h2 className="my-custom-block__title">Block Title</h2>
            <p className="my-custom-block__content">Block content goes here.</p>
        </div>
    );
}
```

Then, in your CSS, you can target these specific elements:

```
.my-custom-block__title {
    font-size: 24px;
    color: #333;
}

.my-custom-block__content {
    font-size: 16px;
    line-height: 1.5;
}
```

### Using Attributes for Dynamic Styling

You can also use block attributes to apply dynamic styles. For example, if your block has a color attribute, you can use it to style elements dynamically:

```
import { useBlockProps } from '@wordpress/block-editor';

export default function Edit({ attributes }) {
    const blockProps = useBlockProps();
    const { textColor } = attributes;

    return (
        <div { ...blockProps }>
            <p style={{ color: textColor }}>This text color is dynamic!</p>
        </div>
    );
}
```

## Design considerations for block developers

The design of a WordPress site the responsibility of the specific theme being used. This means that as a block developer, you generally only need to consider design elements if you are defining a specific set of dimensions for the block.

If you allow your block to use the built-in layout supports for things like fonts, alignment, pagination, padding, etc, you probably won't need to worry about making your block responsive.

If you do need to define specific design styles for your block, it's a good idea to follow modern web design best practices like fluid typography and flexible layouts.

### Fluid Typography

Implement fluid typography to ensure that your text scales smoothly across different screen sizes. You can use the CSS `clamp()` function to set a minimum, preferred, and maximum font size:

```
.wp-block-my-plugin-my-custom-block h2 {
    font-size: clamp(18px, 4vw, 24px);
}
```

This will ensure that the font size is never smaller than 18px, never larger than 24px, and scales fluidly between these values based on the viewport width.

### Flexible Layouts

Use flexible layout techniques like CSS Flexbox or Grid to create layouts that adapt to different screen sizes:

```
.wp-block-my-plugin-my-custom-block {
    display: flex;
    flex-wrap: wrap;
    gap: 20px;
}

.wp-block-my-plugin-my-custom-block > * {
    flex: 1 1 300px;
}
```

This creates a flexible layout where child elements will wrap to the next line on smaller screens, maintaining a minimum width of 300px.

### Testing Responsive Behavior

Always test your blocks across various devices and screen sizes to ensure they behave as expected.

Most modern browsers have a responsive design mode built into the developer tools. This can be used to simulate different viewport sizes and identify any issues with your block's design.

# Further reading

- [How the WordPress Gutenberg Block Editor Empowers Enterprise Content Creators](https://wpvip.com/how-the-wordpress-gutenberg-block-editor-empowers-enterprise-content-creators/)
- [Content Creation in WordPress Using Gutenberg](https://learn.wordpress.org/tutorial/content-creation-in-wordpress-using-gutenberg/)
- [Styling Your WordPress Block](https://learn.wordpress.org/tutorial/styling-your-wordpress-block/)
- [Intrinsic Design, Theming, and Rethinking How to Design with WordPress](https://developer.wordpress.org/news/2023/02/intrinsic-design-theming-and-rethinking-how-to-design-with-wordpress/)
