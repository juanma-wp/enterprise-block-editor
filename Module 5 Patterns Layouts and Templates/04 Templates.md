# 5.4: Templates

In this lesson, we'll explore the world of WordPress block templates, focusing on their application to custom post types, programmatic implementation, and the management of template parts. This knowledge is crucial for Enterprise WordPress developers looking to create sophisticated, customized layouts for various content types.

## Building Block Templates for Custom Post Types

Block templates provide a powerful way to define the initial structure of content for custom post types. They allow developers to pre-populate the block editor with a specific arrangement of blocks when creating new content.

### Creating a Block Template

To create a block template for a custom post type, you'll need to define an array that represents the blocks and their attributes. Here's an example of how to create a block template for a "Book" custom post type:

```php
function myplugin_register_book_post_type() {
    $args = array(
        'public'       => true,
        'label'        => 'Books',
        'show_in_rest' => true,
        'template'     => array(
            array( 'core/image', array(
                'align' => 'left',
            ) ),
            array( 'core/heading', array(
                'placeholder' => 'Add Author...',
            ) ),
            array( 'core/paragraph', array(
                'placeholder' => 'Add Description...',
            ) ),
        ),
    );
    register_post_type( 'book', $args );
}
add_action( 'init', 'myplugin_register_book_post_type' );
```

In this example, we've defined a template that includes an image block aligned to the left, followed by a heading block for the author's name, and a paragraph block for the book description.

### Template Locking

You can also control the flexibility of your template by implementing template locking. This feature allows you to restrict the ability to move or remove blocks within the template.

```php
function myplugin_register_book_post_type() {
    $args = array(
        'public'       => true,
        'label'        => 'Books',
        'show_in_rest' => true,
        'template'     => array(
            array( 'core/image', array(
                'align' => 'left',
            ) ),
            array( 'core/heading', array(
                'placeholder' => 'Add Author...',
            ) ),
            array( 'core/paragraph', array(
                'placeholder' => 'Add Description...',
            ) ),
        ),
        'template_lock' => 'all',
    );
    register_post_type( 'book', $args );
}
add_action( 'init', 'myplugin_register_book_post_type' );
```

The `'template_lock' => 'all'` argument prevents users from moving or removing blocks in the template. You can also use `'insert'` to allow moving but prevent adding or removing blocks.

## Programmatically Applying Templates

While we've seen how to apply templates during custom post type registration, there are scenarios where you might want to apply templates programmatically to existing post types or in specific contexts.

### Applying Templates to Existing Post Types

To apply a template to an existing post type, you can use the `register_post_type_args` filter:

```php
function myplugin_add_template_to_post( $args, $post_type ) {
    if ( 'post' === $post_type ) {
        $args['template'] = array(
            array( 'core/image' ),
            array( 'core/heading', array(
                'level' => 2,
            ) ),
            array( 'core/paragraph' ),
        );
    }
    return $args;
}
add_filter( 'register_post_type_args', 'myplugin_add_template_to_post', 20, 2 );
```

This function adds a template to the default 'post' post type, consisting of an image, a level 2 heading, and a paragraph.

### Conditional Template Application

You can also apply templates conditionally based on various factors. For example, you might want to apply a different template based on a post's category:

```php
function myplugin_conditional_template( $block_template, $post_type, $post ) {
    if ( 'post' === $post_type && has_category( 'featured', $post ) ) {
        return array(
            array( 'core/cover', array(
                'url' => get_the_post_thumbnail_url( $post->ID, 'full' ),
            ) ),
            array( 'core/heading', array(
                'level' => 1,
                'content' => get_the_title( $post->ID ),
            ) ),
            array( 'core/paragraph', array(
                'content' => get_the_excerpt( $post->ID ),
            ) ),
        );
    }
    return $block_template;
}
add_filter( 'default_block_template', 'myplugin_conditional_template', 10, 3 );
```

This function applies a special template to posts in the 'featured' category, using a cover block with the post's featured image, a level 1 heading with the post title, and a paragraph with the post excerpt.

## Editing and Managing Template Parts

Template parts are reusable components of your site's design, such as headers, footers, and sidebars. They play a crucial role in maintaining consistency across your site while allowing for modular development.

### Creating Custom Template Parts

To create a custom template part, you'll need to add a new file in your theme's `parts` directory. For example, to create a custom header, you might add a file named `header-custom.html`:

```html
<!-- wp:group {"layout":{"type":"constrained"}} -->
<div class="wp-block-group">
    <!-- wp:site-title {"level":1} /-->
    <!-- wp:site-tagline /-->
    <!-- wp:navigation {"layout":{"type":"flex","setCascadingProperties":true,"justifyContent":"right"}} /-->
</div>
<!-- /wp:group -->
```

### Registering Template Parts

To make your custom template parts available in the block editor, you need to register them in your theme's `theme.json` file:

```json
{
    "templateParts": [
        {
            "name": "header-custom",
            "title": "Custom Header",
            "area": "header"
        }
    ]
}
```

### Editing Template Parts

Template parts can be edited directly in the Site Editor. To do this:

1. Navigate to Appearance > Editor in the WordPress admin.
2. Click on "Template Parts" in the navigation sidebar.
3. Select the template part you want to edit.
4. Make your changes using the block editor interface.
5. Click "Save" to apply your changes.

### Programmatically Modifying Template Parts

For more advanced customizations, you can modify template parts programmatically. Here's an example of how to add a custom class to all header template parts:

```php
function myplugin_modify_header_template_part( $block_content, $block ) {
    if ( $block['blockName'] === 'core/template-part' && $block['attrs']['slug'] === 'header' ) {
        $block_content = str_replace( 'class="wp-block-template-part"', 'class="wp-block-template-part custom-header"', $block_content );
    }
    return $block_content;
}
add_filter( 'render_block', 'myplugin_modify_header_template_part', 10, 2 );
```

This function adds a `custom-header` class to all header template parts, allowing for targeted styling or JavaScript interactions.

## Conclusion

Templates and template parts are powerful features of the WordPress block editor that allow for sophisticated content structuring and site-wide design consistency. By mastering these tools, Enterprise WordPress developers can create highly customized and efficient content management experiences for their clients or organizations.

In this lesson, we've covered how to build block templates for custom post types, apply templates programmatically, and manage template parts. These skills will enable you to create more flexible and maintainable WordPress themes and plugins, leveraging the full power of the block editor.

Citations:
[1] https://github.co
[2] https://webberzone.com/custom-block-theme-templates-wordpress-plugins/
[3] https://gutenberg.10up.com/reference/custom-post-types/
[4] https://jetpack.com/resources/wordpress-block-templates/
[5] https://www.gravityforms.com/blog/an-intro-to-wordpress-templates-and-template-parts-in-the-site-editor/
[6] https://codewp.ai/blog/wordpress-template-parts-tutorial/
[7] https://portfoliowp.com/document/editing-template-parts/
[8] https://kinsta.com/blog/wordpress-block-templates/
[9] https://wpengine.com/builders/block-templates/
[10] https://www.inmotionhosting.com/support/edu/wordpress/full-site-editing/template-parts/
[11] https://developer.wordpress.org/themes/global-settings-and-styles/template-parts/
[12] https://learn.wordpress.org/tutorial/using-template-parts/
[13] https://fullsiteediting.com/lessons/creating-block-templates-for-custom-post-types/
[14] https://www.reddit.com/r/Wordpress/comments/1ffvwnf/post_templates_and_custom_post_types/
[15] https://wordpress.org/support/topic/block-theme-template-for-a-custom-post-type/
[16] https://crocoblock.com/blog/custom-post-types-templates-explained/
[17] https://wordpress.org/support/topic/block-template-for-custom-post-type/
[18] https://www.youtube.com/watch?v=Z6B5jnpJUFg
[19] https://kau-boys.com/3010/wordpress/using-block-patterns-for-new-entries-of-a-custom-post-type
[20] https://www.reddit.com/r/Wordpress/comments/1ff2jo7/block_themes_and_custom_post_type_templates/
[21] https://stackoverflow.com/questions/52662858/wordpress-can-you-programmatically-create-page-templates
[22] https://zingmap.com/replacing-the-current-block-template-dynamically-in-wordpress/
[23] https://wordpress.org/support/topic/apply-template-programatically/
[24] https://github.com/WordPress/gutenberg/issues/58417
[25] https://wordpress.stackexchange.com/questions/204657/apply-template-to-custom-post-type
[26] https://wordpress.stackexchange.com/questions/13378/add-custom-template-page-programmatically
[27] https://wordpress.stackexchange.com/questions/395528/can-you-use-block-patterns-in-block-templates-or-insert-them-programmatically
[28] https://tommcfarlin.com/programmatically-set-a-wordpress-template/
[29] https://developer.wordpress.org/block-editor/reference-guides/block-api/block-templates/
[30] https://motopress.com/blog/wordpress-template-parts/
[31] https://wordpress.com/support/templates/edit-the-page-template/
[32] https://www.dreamhost.com/blog/wordpress-templates-template-parts/
[33] https://motopress.com/blog/wordpress-templates-fse/
[34] https://learn.wordpress.org/lesson/using-template-parts/
[35] https://woocommerce.com/document/woocommerce-store-editing/patterns-template-parts/
[36] https://www.youtube.com/watch?v=Yzx8qaL9leU
[37] https://fullsiteediting.com/lessons/templates-and-template-parts/
[38] https://www.youtube.com/watch?v=CTn9jtvCvtc
[39] https://stackoverflow.com/questions/61002661/how-to-use-a-wordpress-gutenberg-block-programmatically-into-the-theme
[40] https://wordpress.com/support/template-part-block/
[41] https://carriedils.com/dynamically-adding-a-reusable-block-in-a-wordpress-template/
[42] https://rudrastyh.com/gutenberg/create-block-patterns-programmatically.html
[43] https://essential-blocks.com/design-wordpress-custom-post-templates/
[44] https://wpspectra.com/docs/templates-for-custom-post-types/
[45] https://wordpress.stackexchange.com/questions/407712/how-do-i-assign-a-block-template-html-to-a-custom-post-type
[46] https://gist.github.com/Maximilianos/b21020ecceaa67650d01
[47] https://pressidium.com/blog/create-your-wordpress-custom-templates-manually/
[48] https://tommcfarlin.com/how-to-programmatically-populate-a-wordpress-template/
[49] https://learn.wordpress.org/lesson-plan/template-parts/