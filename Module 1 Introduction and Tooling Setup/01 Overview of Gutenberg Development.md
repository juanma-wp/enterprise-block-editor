# Overview of Gutenberg Development

## What is Gutenberg?

Gutenberg is the project name for a new WordPress site building and publishing paradigm that aims to revolutionize the entire publishing experience. Its core product is the Block Editor, a block-based editor that has transformed the WordPress content creation experience since its introduction in December 2018. As a software developer working on Enterprise WordPress projects, it's important to understand that Gutenberg is not just the new editor but a complete paradigm shift in how WordPress handles content creation and site-building.

At its core, Gutenberg introduces the concept of blocks, which are standalone, isolated, and dynamic user interface components. These blocks represent individual content elements, such as paragraphs, images, headings, or more complex structures like columns and galleries. This modular approach to content creation allows for greater flexibility and control over page layouts, moving beyond the limitations of the traditional WordPress editor.

From a technical perspective, Gutenberg uses modern web technologies, primarily JavaScript and React. This marks a significant departure from the PHP-centric development that has long been associated with WordPress. As an Enterprise WordPress developer, you must familiarize yourself with these technologies to effectively work with and extend the Block Editor.

Let's look at a simple example of how a custom block might be registered using JavaScript:

```javascript
import { registerBlockType } from '@wordpress/blocks';
import { RichText } from '@wordpress/block-editor';

registerBlockType('my-plugin/custom-block', {
    title: 'Custom Block',
    icon: 'smiley',
    category: 'common',
    attributes: {
        content: {
            type: 'string',
            source: 'html',
            selector: 'p',
        },
    },
    edit: ({ attributes, setAttributes }) => {
        return (
            <RichText
                tagName="p"
                value={ attributes.content }
                onChange={ ( content ) => setAttributes({ content }) }
                placeholder="Enter your custom content here"
            />
        );
    },
    save: ({ attributes }) => {
        return <RichText.Content tagName="p" value={ attributes.content } />;
    },
});
```

This example demonstrates a block's basic structure, including its registration, edit interface, and save function. As you delve deeper into block development, you'll encounter more complex scenarios and APIs that allow for sophisticated block behaviors and interactions.

## The Gutenberg plugin

The [Gutenberg plugin](https://wordpress.org/plugins/gutenberg/) serves as the development and testing ground for new features before they are merged into WordPress core. While the Block Editor is now part of WordPress core since version 5.0, the Gutenberg plugin continues to be actively developed, offering a glimpse into the future of WordPress.

Installing the Gutenberg plugin on your development environment allows you to:
- Test upcoming features before they reach WordPress core
- Access experimental APIs and components
- Contribute to Gutenberg development
- Stay ahead of WordPress core changes

For Enterprise WordPress developers, working with the Gutenberg plugin is essential for:
- Forward compatibility testing of themes and plugins
- Early adoption of new features in client projects
- Understanding upcoming changes that might affect existing implementations
- Testing custom blocks against future WordPress versions

The plugin is available in the WordPress plugin repository and is updated every two weeks, making it a vital tool in your WordPress development workflow.

## The Block Editor vs. the Site Editor

As we progress in our understanding of blocks, it's essential to distinguish between two key components: the Block Editor and the Site Editor. While both are part of the Gutenberg project, they serve different purposes and offer distinct capabilities.

The Block Editor, which we've primarily discussed so far, is focused on content creation for individual posts and pages. It allows users to construct and manipulate content using blocks, providing a flexible and intuitive interface for crafting rich, media-heavy content. As a developer, you'll often work with the Block Editor when creating custom blocks or extending existing functionality for content creation. Sometimes you will see the Block Editor referred to as the Post Editor or the Page Editor, depending on the context it's being used in. 

On the other hand, the Site Editor represents a more recent and ambitious evolution of the project. Introduced as part of an initiative called Full Site Editing (FSE), the Site Editor extends the block paradigm to the entire website structure. With the Site Editor, users can modify not just content but also global elements like headers, footers, and sidebars and even create custom page templates – all using the familiar block interface.

From a development perspective, the Site Editor introduces new challenges and opportunities. You'll need to consider how your custom blocks and plugins interact with site-wide editing capabilities. For instance, you might need to create block patterns or custom block templates that can be used across the entire site.

Here's an example of how you might register a custom template part in PHP for use in the Site Editor:

```
function register_custom_template_part() {
    $post_type_object = get_post_type_object( 'wp_template_part' );
    $post_type_object->template = array(
        array( 'core/group', array(), array(
            array( 'core/heading', array(
                'level' => 2,
                'content' => 'Custom Template Part',
            ) ),
            array( 'core/paragraph', array(
                'content' => 'This is a custom template part for use in the Site Editor.',
            ) ),
        ) ),
    );
}
add_action( 'init', 'register_custom_template_part' );
```

This code registers a custom template part with a predefined structure, which can then be used and customized within the Site Editor.

As an Enterprise WordPress developer, it's crucial to understand the distinctions and relationships between the Block Editor and the Site Editor. While the Block Editor focuses on content creation within posts and pages, the Site Editor expands this concept to encompass the entire site structure. This holistic approach to site building opens up new possibilities for creating dynamic, flexible WordPress sites, but also requires a shift in how we approach theme development and site customization.

In the following lessons, we'll dive deeper into the intricacies of Gutenberg development, exploring advanced block creation, custom block controls, and strategies for integrating Gutenberg with enterprise-level WordPress projects. We'll examine how to leverage the power of both the Block Editor and the Site Editor to create sophisticated, user-friendly WordPress experiences that meet the demands of modern web development.

## Further Reading  

- [Block Editor Handbook](https://developer.wordpress.org/block-editor/)
- [What is the difference between the Block Editor and Site Editor?](https://learn.wordpress.org/lesson/what-is-the-difference-between-the-block-editor-and-site-editor/)
- [Gutenberg - The WordPress Block Editor](https://wordpress.org/gutenberg/)
- [Understanding the Page Editor vs Site Editor](https://learn.wordpress.org/tutorial/understanding-the-page-editor-vs-site-editor/)   