# Styling Blocks

In the world of WordPress block development, creating visually appealing and functional blocks is crucial. This lesson will delve into the intricacies of styling blocks, covering editor styles, frontend styles, CSS selectors, and responsive design techniques. By the end of this lesson, you'll have a comprehensive understanding of how to create blocks that look great both in the editor and on the frontend.

## Block style vs Editor style

In the Anatomy of a Block lesson, you learned that create-block configures two files for you: `style.scss` and `editor.scss`. These files are [Syntactically Awesome Style Sheets](https://sass-lang.com/) (or Sass) and are used to define the styles for your block. 

During the build process, these files are compiled into CSS and enqueued in the appropriate places.

### Block Styles

The block styles in `style.scss` are applied to blocks when they are rendered on the front end of the website. These styles determine how the block will appear to visitors of your site. 

```sass
.wp-block-create-block-my-custom-block {
  background-color: #21759b;
  color: #fff;
  padding: 2px;
}
```

Because these styles are applied to the block, they are also enqueued in the editor, so that that block looks the same when using it in the editor as it does on the front end.

### Editor Styles

On the other hand the editor styles in `editor.scss` are only applied to the block when it's displayed in the editor. This allows you to style the block differently in the editor than it appears on the front end. 

```sass
.wp-block-create-block-my-custom-block {
  border: 1px dotted #f00;
}
```

Generally editor styles are used to if the block has additional elements that are only visible in the editor, for example, performing some tasks that filter the data to be displayed in the final block that's saved to the database.

## CSS Selectors

When styling custom blocks, it's crucial to use appropriate CSS selectors to target specific elements within your block. WordPress provides several classes and attributes that you can leverage for styling purposes.

### Block-Specific Classes

Every block in WordPress is wrapped in a div with a class that follows the pattern `wp-block-{namespace}-{blockname}`. For example, if you've created a custom block called "my-custom-block" in the namespace "my-plugin", the wrapper div would have the class `wp-block-my-plugin-my-custom-block`.

You can target this class in your CSS to apply styles to your entire block:

```
.wp-block-my-plugin-my-custom-block {
    background-color: #f0f0f0;
    padding: 20px;
    border-radius: 5px;
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

## Managing Responsive Design in Blocks

Ensuring that your blocks are responsive is crucial for providing a consistent user experience across different devices. Here are some methodologies to achieve responsive design in your blocks:

### Using CSS Media Queries

CSS media queries are a powerful tool for applying different styles based on the viewport size. You can use them in your block's stylesheet to adjust the layout and styling for different screen sizes:

```
.wp-block-my-plugin-my-custom-block {
    padding: 20px;
}

@media (max-width: 768px) {
    .wp-block-my-plugin-my-custom-block {
        padding: 10px;
    }
}
```

### Fluid Typography

Implement fluid typography to ensure that your text scales smoothly across different screen sizes. You can use CSS `clamp()` function to set a minimum, preferred, and maximum font size:

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

Always test your blocks across various devices and screen sizes to ensure they behave as expected. Use browser developer tools to simulate different viewport sizes and identify any issues with your responsive design.

By mastering these techniques for styling blocks, adding CSS selectors, and managing responsive design, you'll be well-equipped to create blocks that look great and function seamlessly across all devices, both in the editor and on the frontend of WordPress sites.

Citations: \[1\] [https://webdevstudios.com/2023/12/19/wordpress-styling-techniques/](https://webdevstudios.com/2023/12/19/wordpress-styling-techniques/) \[2\] [https://www.boxuk.com/insight/wordpress-block-editor-gutenberg-for-enterprise-content-creators/](https://www.boxuk.com/insight/wordpress-block-editor-gutenberg-for-enterprise-content-creators/) \[3\] [https://gutenbergmarket.com/news/a-comprehensive-guide-to-building-wordpress-block-themes](https://gutenbergmarket.com/news/a-comprehensive-guide-to-building-wordpress-block-themes) \[4\] [https://wpvip.com/how-the-wordpress-gutenberg-block-editor-empowers-enterprise-content-creators/](https://wpvip.com/how-the-wordpress-gutenberg-block-editor-empowers-enterprise-content-creators/) \[5\] [https://www.aslamdoctor.com/exploring-wordpress-styling-techniques/](https://www.aslamdoctor.com/exploring-wordpress-styling-techniques/) \[6\] [https://learn.wordpress.org/tutorial/content-creation-in-wordpress-using-gutenberg/](https://learn.wordpress.org/tutorial/content-creation-in-wordpress-using-gutenberg/) \[7\] [https://learn.wordpress.org/tutorial/styling-your-wordpress-block/](https://learn.wordpress.org/tutorial/styling-your-wordpress-block/) \[8\] [https://learn.wordpress.org/tutorial/intro-to-publishing-with-the-block-editor/](https://learn.wordpress.org/tutorial/intro-to-publishing-with-the-block-editor/) \[9\] [https://css-tricks.com/getting-the-wordpress-block-editor-to-look-like-the-front-end-design/](https://css-tricks.com/getting-the-wordpress-block-editor-to-look-like-the-front-end-design/)  