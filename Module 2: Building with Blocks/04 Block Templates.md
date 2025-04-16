# 2.3: Block Templates

In this lesson, we'll explore the world of WordPress block templates, focusing on their application to custom post types.

This knowledge is useful for WordPress developers looking to create sophisticated, customized layouts for dynamic content types.

## Custom Post Types: A refresher

Custom post types are a powerful feature of WordPress that allow developers to define their own content types beyond the default posts and pages. They provide a structured way to organize and display different types of content on a WordPress site.

You're probably already familiar with how to create custom post types, but let's do a quick refresher:

```php
add_action( 'init', 'vip_learn_register_book_post_type' );
function vip_learn_register_book_post_type(){
	$book_arguments = array(
		'labels'       => array(
			'name'          => 'Books',
			'singular_name' => 'Book',
			'menu_name'     => 'Books',
			'add_new'       => 'Add New Book',
			'add_new_item'  => 'Add New Book',
			'new_item'      => 'New Book',
			'edit_item'     => 'Edit Book',
			'view_item'     => 'View Book',
			'all_items'     => 'All Books',
		),
		'public'       => true,
		'has_archive'  => true,
		'show_in_rest' => true,
		'rest_base'    => 'books',
		'supports'     => array( 'title', 'editor', 'thumbnail', 'excerpt', 'custom-fields' ),
	);

	register_post_type( 'book', $book_arguments );
}
```

Here, the `register_post_type` [function](https://developer.wordpress.org/reference/functions/register_post_type/) is used to create a custom post type called "Books" with various labels and settings. By using the default custom post type support, WordPress will automatically generate the necessary UI elements for managing this new content type.

## Block Templates and Custom Post Types

WordPress allows you to register two types of templates for custom post types: a template that is used as the default initial state for an editor session, and a template that is used when the custom post type is rendered on the front end.

### Block template as the default initial state

If you've registered a custom post type (like the book example above) WordPress creates the UI elements for users to manage Book post types. It is possible to define a block template that will be preloaded in the editor when a user adds a new book. This can help guide users in creating content that adheres to a specific structure or layout.

To create this default block template for your custom post type, you set the `template` argument passed to `register_post_type`. This argument accepts a PHP array that represents the blocks and their attributes. This array can be configured to allow for a wide range of block configurations, including nested blocks and custom attributes.

To understand how to structure a block template array, let's look at an example of some block markup that you might create in the editor:

```html
<!-- wp:group {"layout":{"type":"constrained"}} -->
<div class="wp-block-group"><!-- wp:columns -->
    <div class="wp-block-columns"><!-- wp:column -->
        <div class="wp-block-column"><!-- wp:image {"align":"right"} -->
            <figure class="wp-block-image alignright"><img alt=""/></figure>
            <!-- /wp:image --></div>
        <!-- /wp:column -->

        <!-- wp:column -->
        <div class="wp-block-column"><!-- wp:heading {"placeholder":"Add Author..."} -->
            <h2 class="wp-block-heading"></h2>
            <!-- /wp:heading -->

            <!-- wp:paragraph {"placeholder":"Add Description..."} -->
            <p></p>
            <!-- /wp:paragraph --></div>
        <!-- /wp:column --></div>
    <!-- /wp:columns --></div>
<!-- /wp:group -->
```

Here we have the following structure:
- A group block with a constrained layout
- Inside the group block, a columns block
- Inside the columns block, two column blocks
- The first column block contains an image block aligned to the right
- The second column block contains a heading block with a placeholder for the author's name and a paragraph block with a placeholder for the book description

To represent this structure in a PHP array, you would structure the array like this:

```php
array(
    array( 'core/group', array(
        'layout' => array(
            'type' => 'constrained',
        ),
    ), array(
        array( 'core/columns', array(), array(
            array( 'core/column', array(), array(
                array( 'core/image', array(
                    'align' => 'right',
                ) ),
            ) ),
            array( 'core/column', array(), array(
                array( 'core/heading', array(
                    'placeholder' => 'Add Author...',
                ) ),
                array( 'core/paragraph', array(
                    'placeholder' => 'Add Description...',
                ) ),
            ) ),
        ) ),
    ) ),
);
```

- Notice how the block names are represented as strings as the first item in the array
- Block attributes are represented as associative arrays as the second item in the array.
- Nested blocks are represented as sub-arrays in the third item of a given array.

Using this structure, you can create this default editor template for a "Book" custom post type:

```php
add_action( 'init', 'vip_learn_register_book_post_type' );
function vip_learn_register_book_post_type(){
	$book_arguments = array(
		'labels'       => array(
			'name'          => 'Books',
			'singular_name' => 'Book',
			'menu_name'     => 'Books',
			'add_new'       => 'Add New Book',
			'add_new_item'  => 'Add New Book',
			'new_item'      => 'New Book',
			'edit_item'     => 'Edit Book',
			'view_item'     => 'View Book',
			'all_items'     => 'All Books',
		),
		'public'       => true,
		'has_archive'  => true,
		'show_in_rest' => true,
		'rest_base'    => 'books',
		'supports'     => array( 'title', 'editor', 'thumbnail', 'excerpt', 'custom-fields' ),
		'template'     => array(
            array( 'core/group', array(
                'layout' => array(
                    'type' => 'constrained',
                ),
            ), array(
                array( 'core/columns', array(), array(
                    array( 'core/column', array(), array(
                        array( 'core/image', array(
                            'align' => 'right',
                        ) ),
                    ) ),
                    array( 'core/column', array(), array(
                        array( 'core/heading', array(
                            'placeholder' => 'Add Author...',
                        ) ),
                        array( 'core/paragraph', array(
                            'placeholder' => 'Add Description...',
                        ) ),
                    ) ),
                ) ),
            ) ),
        ) 
	);
	register_post_type( 'book', $book_arguments );
}
```

In this example, we've defined a template that includes an image block aligned to the left, followed by a heading block for the author's name, and a paragraph block for the book description.

### Template Locking

You can also control the flexibility of your template by implementing template locking using the `template_lock` argument.

This feature allows you to restrict the ability to move or remove blocks within the template.

```php
add_action( 'init', 'vip_learn_register_book_post_type' );
function vip_learn_register_book_post_type(){
	$book_arguments = array(
		'labels'       => array(
			'name'          => 'Books',
			'singular_name' => 'Book',
			'menu_name'     => 'Books',
			'add_new'       => 'Add New Book',
			'add_new_item'  => 'Add New Book',
			'new_item'      => 'New Book',
			'edit_item'     => 'Edit Book',
			'view_item'     => 'View Book',
			'all_items'     => 'All Books',
		),
		'public'       => true,
		'has_archive'  => true,
		'show_in_rest' => true,
		'rest_base'    => 'books',
		'supports'     => array( 'title', 'editor', 'thumbnail', 'excerpt', 'custom-fields' ),
		'template'     => array(
            array( 'core/group', array(
                'layout' => array(
                    'type' => 'constrained',
                ),
            ), array(
                array( 'core/columns', array(), array(
                    array( 'core/column', array(), array(
                        array( 'core/image', array(
                            'align' => 'right',
                        ) ),
                    ) ),
                    array( 'core/column', array(), array(
                        array( 'core/heading', array(
                            'placeholder' => 'Add Author...',
                        ) ),
                        array( 'core/paragraph', array(
                            'placeholder' => 'Add Description...',
                        ) ),
                    ) ),
                ) ),
            ) ),
        ),
        'template_lock' => 'all', 
	);

	register_post_type( 'book', $book_arguments );
}
```

Setting `template_lock` to `all` prevents users from moving or removing blocks in the template. You can also use `insert` to allow moving but prevent adding or removing blocks.

## Programmatically Applying Templates

While we've seen how to apply templates during custom post type registration, there are scenarios where you might want to apply templates programmatically to existing post types or in specific contexts.

### Applying Templates to Existing Post Types

To apply a template to an existing post type, you can use the `register_post_type_args` filter:

```php
add_filter( 'register_post_type_args', 'vip_learn_add_template_to_post', 20, 2 );

function vip_learn_add_template_to_post( $args, $post_type ) {
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
```

This function adds a template to the default 'post' post type, consisting of an image, a level 2 heading, and a paragraph.

### Conditional Template Application

You can also apply templates conditionally based on various factors. For example, you might want to apply a different template based on a post's category:

```php
add_filter( 'default_block_template', 'vip_learn_conditional_template', 10, 3 );

function vip_learn_conditional_template( $block_template, $post_type, $post ) {
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
```

This function applies a special template to posts in the 'featured' category, using a cover block with the post's featured image, a level 1 heading with the post title, and a paragraph with the post excerpt.

## Block template for front-end rendering

In addition to the default block template for the editor, you can define a block template that will be used when rendering the custom post type on the front end of your site. This allows you to control the layout and structure of your custom post type's content when it is displayed to visitors.

Generally, the template used to render a custom post type on the front end is defined in the theme's template files. The [WordPress Template Hierarchy](https://developer.wordpress.org/themes/templates/template-hierarchy/) allows for a `single-{post_type}.html` template, as well as a `single-{post_type}-slug.html` template file to be used to render the content of a custom post type.

One of the benefits of the Site Editor is that you can create the block template for a custom post type directly in the editor, and then save the content of that template to the relevant file in the WordPress theme.

However, it is also possible to register a template that will be used to render the custom post type's content when the custom post type is registered.

To define this block template you can use the `register_block_template` function. This function works similarly to `register_block_pattern`, but it is specifically designed for custom post type templates.

`register_block_template` accepts the following parameters:
- A template name in the form of `plugin_uri//template_name`
- An array of arguments to set optional template properties like the title, description, content, and the post types it applies to.

Here's an example of how you might register a block template for a custom post type:

```php
register_block_template( 'vip_learn_block_templates//book',
	array(
		'title'       => __( 'Single Book', 'vip-learn-block-templates' ),
		'description' => __( 'The Single Book Template.', 'vip-learn-block-templates' ),
		'post_types'  => array( 'book' ),
		'content'     => '<!-- wp:template-part {"slug":"header"} /-->
<!-- wp:group {"tagName":"main","style":{"spacing":{"margin":{"top":"var:preset|spacing|60"}}},"layout":{"type":"constrained"}} -->
<main class="wp-block-group" style="margin-top:var(--wp--preset--spacing--60)"><!-- wp:group {"align":"full","style":{"spacing":{"padding":{"top":"var:preset|spacing|60"}}},"layout":{"type":"constrained"}} -->
    <div class="wp-block-group alignfull" style="padding-top:var(--wp--preset--spacing--60)"><!-- wp:post-title {"level":1} /-->

        <!-- wp:post-terms {"term":"category","style":{"typography":{"fontStyle":"normal","fontWeight":"400"}}} /-->

        <!-- wp:spacer {"height":"var:preset|spacing|50"} -->
        <div style="height:var(--wp--preset--spacing--50)" aria-hidden="true" class="wp-block-spacer"></div>
        <!-- /wp:spacer -->

        <!-- wp:post-content {"align":"full","layout":{"type":"constrained"}} /-->

        <!-- wp:group {"style":{"spacing":{"padding":{"top":"var:preset|spacing|60","bottom":"var:preset|spacing|60"}}},"layout":{"type":"constrained"}} -->
        <div class="wp-block-group" style="padding-top:var(--wp--preset--spacing--60);padding-bottom:var(--wp--preset--spacing--60)"><!-- wp:post-terms {"term":"post_tag","separator":"  ","className":"is-style-post-terms-1"} /--></div>
        <!-- /wp:group -->

        <!-- wp:group {"align":"full","style":{"spacing":{"padding":{"right":"var:preset|spacing|50","left":"var:preset|spacing|50"},"margin":{"top":"var:preset|spacing|60","bottom":"var:preset|spacing|60"}}},"layout":{"type":"constrained"}} -->
        <div class="wp-block-group alignfull" style="margin-top:var(--wp--preset--spacing--60);margin-bottom:var(--wp--preset--spacing--60);padding-right:var(--wp--preset--spacing--50);padding-left:var(--wp--preset--spacing--50)"><!-- wp:group {"tagName":"nav","align":"wide","style":{"border":{"top":{"color":"var:preset|color|accent-6","width":"1px"},"right":[],"bottom":[],"left":[]},"spacing":{"padding":{"top":"var:preset|spacing|40","bottom":"var:preset|spacing|40"}}},"layout":{"type":"flex","flexWrap":"nowrap","justifyContent":"space-between"}} -->
            <nav class="wp-block-group alignwide" aria-label="Post navigation" style="border-top-color:var(--wp--preset--color--accent-6);border-top-width:1px;padding-top:var(--wp--preset--spacing--40);padding-bottom:var(--wp--preset--spacing--40)"><!-- wp:post-navigation-link {"type":"previous","showTitle":true,"arrow":"arrow"} /-->

                <!-- wp:post-navigation-link {"showTitle":true,"arrow":"arrow"} /--></nav>
            <!-- /wp:group --></div>
        <!-- /wp:group --></div>
    <!-- /wp:group --></main>
<!-- /wp:group -->

<!-- wp:template-part {"slug":"footer"} /-->'
	)
);
```

In this example, you're registering a block template for the `book` custom post type. The template includes a header, main content area, and footer, each defined using template parts and blocks.

Just as you did with Patterns, you can create a custom function to load the template content from a file in a `templates` directory, and then call that function to get the template conte for the `content` argument of `register_block_template`.

```php
/**
 * Fetch and include the template content
 */
function vip_learn_get_template_content( $template ) {
	ob_start();
	include __DIR__ . "/templates/{$template}";
	return ob_get_clean();
}

register_block_template( 'vip_learn_block_templates//book',
	array(
		'title'       => __( 'Single Book', 'vip-learn-block-templates' ),
		'description' => __( 'The Single Book Template.', 'vip-learn-block-templates' ),
		'post_types'  => array( 'book' ),
		'content'     => vip_learn_get_template_content( 'single-book.php' )
	)
);
```

## Summary

Being able to define block templates for custom post types is a powerful feature that allows you to create structured content layouts for your WordPress site. By defining block templates, you can ensure that your content is consistently formatted and styled, providing a better user experience for site builders and visitors.