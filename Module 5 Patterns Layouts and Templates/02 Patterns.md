# Lesson 5.2: Patterns

## Understanding Block Patterns

Block patterns are pre-designed layouts composed of multiple blocks that can be easily inserted into posts or pages. They provide a powerful way to create complex, reusable designs without having to manually configure individual blocks each time. Block patterns serve as a bridge between individual blocks and full page templates, offering flexibility and efficiency in content creation.

Block patterns are added via the block inserter in the WordPress editor. They are organized into categories to help users find the desired layout quickly. 

Block patterns play an important role in streamlining the page building process for site editors and theme developers alike, allowing them to:

1. Quickly insert complex layouts
2. Maintain design consistency across a site
3. Provide users with pre-built content structures
4. Enhance the block editor's capabilities without relying on custom blocks

Block patterns are particularly useful for common design elements such as headers, footers, call-to-action sections, or product showcases. They can be as simple as a few paragraphs with a heading or as complex as a full-page layout with multiple columns, images, and buttons.

## Creating and Registering Custom Patterns

Creating custom block patterns involves two main steps: designing the pattern using the block editor and registering it programmatically. Let's walk through this process step by step.

### Designing a Custom Pattern

To design a custom pattern, you can use the WordPress block editor to create the desired layout. Here's an example of how you might create a simple "Featured Product" pattern:

1. Create a new post or page in the WordPress admin.

2. Add the necessary blocks to create your pattern. For our "Featured Product" example, you might use:
    - A Group block to contain everything
    - A Columns block with two columns
    - An Image block in the left column
    - Heading, Paragraph, and Button blocks in the right column

![Creating the pattern in the editor](images/patterns/featured-product-pattern.png)

3. Style the blocks as desired using the block settings and any custom CSS classes.

4. Once you're satisfied with the design, switch to the Code Editor view and copy the block markup that makes up the structure of the blocks.

Below is an example of what the block markup for this pattern might look like. 

```html
<!-- wp:group {"layout":{"type":"constrained"}} -->
<div class="wp-block-group"><!-- wp:columns {"style":{"spacing":{"padding":{"top":"var:preset|spacing|20","bottom":"var:preset|spacing|20","left":"var:preset|spacing|20","right":"var:preset|spacing|20"}}},"borderColor":"accent-4"} -->
   <div class="wp-block-columns has-border-color has-accent-4-border-color" style="padding-top:var(--wp--preset--spacing--20);padding-right:var(--wp--preset--spacing--20);padding-bottom:var(--wp--preset--spacing--20);padding-left:var(--wp--preset--spacing--20)"><!-- wp:column -->
      <div class="wp-block-column"><!-- wp:image -->
         <figure class="wp-block-image"><img alt=""/></figure>
         <!-- /wp:image --></div>
      <!-- /wp:column -->

      <!-- wp:column -->
      <div class="wp-block-column"><!-- wp:heading -->
         <h2 class="wp-block-heading"></h2>
         <!-- /wp:heading -->

         <!-- wp:paragraph -->
         <p></p>
         <!-- /wp:paragraph -->

         <!-- wp:buttons {"layout":{"type":"flex","orientation":"horizontal","justifyContent":"right"}} -->
         <div class="wp-block-buttons"><!-- wp:button -->
            <div class="wp-block-button"><a class="wp-block-button__link wp-element-button"></a></div>
            <!-- /wp:button --></div>
         <!-- /wp:buttons --></div>
      <!-- /wp:column --></div>
   <!-- /wp:columns --></div>
<!-- /wp:group -->

<!-- wp:paragraph -->
<p></p>
<!-- /wp:paragraph -->
```

### Registering the Custom Pattern

To make your custom pattern available in the block editor, you need to register it in PHP. 

This can be done in either a theme or child theme's `functions.php` file or in a custom plugin file, using the `register_block_pattern` function.

Here's an example of how to register the "Featured Product" pattern we designed:

```php
add_action( 'init', 'vip_learn_register_custom_pattern' );

function vip_learn_register_custom_pattern() {
    register_block_pattern(
        'vip-learn/featured-product',
        array(
            'title'       => __( 'Featured Product', 'vip-custom-block' ),
            'description' => _x( 'A product showcase with image, description, and call-to-action button.', 'Block pattern description', 'my-theme' ),
            'content'     => '<!-- wp:group {"layout":{"type":"constrained"}} -->
<div class="wp-block-group"><!-- wp:columns {"style":{"spacing":{"padding":{"top":"var:preset|spacing|20","bottom":"var:preset|spacing|20","left":"var:preset|spacing|20","right":"var:preset|spacing|20"}}},"borderColor":"accent-4"} -->
    <div class="wp-block-columns has-border-color has-accent-4-border-color" style="padding-top:var(--wp--preset--spacing--20);padding-right:var(--wp--preset--spacing--20);padding-bottom:var(--wp--preset--spacing--20);padding-left:var(--wp--preset--spacing--20)"><!-- wp:column -->
        <div class="wp-block-column"><!-- wp:image -->
            <figure class="wp-block-image"><img alt=""/></figure>
            <!-- /wp:image --></div>
        <!-- /wp:column -->

        <!-- wp:column -->
        <div class="wp-block-column"><!-- wp:heading -->
            <h2 class="wp-block-heading"></h2>
            <!-- /wp:heading -->

            <!-- wp:paragraph -->
            <p></p>
            <!-- /wp:paragraph -->

            <!-- wp:buttons {"layout":{"type":"flex","orientation":"horizontal","justifyContent":"right"}} -->
            <div class="wp-block-buttons"><!-- wp:button -->
                <div class="wp-block-button"><a class="wp-block-button__link wp-element-button"></a></div>
                <!-- /wp:button --></div>
            <!-- /wp:buttons --></div>
        <!-- /wp:column --></div>
    <!-- /wp:columns --></div>
<!-- /wp:group -->

<!-- wp:paragraph -->
<p></p>
<!-- /wp:paragraph -->',
            'categories'  => array( 'featured' ),
        )
    );
}
```

The `register_block_pattern()` function takes two arguments:

1. A unique identifier for the pattern (namespaced with your theme or plugin name)
2. An array of pattern properties, including:
    - `title`: The name of the pattern as it appears in the editor
    - `description`: A brief explanation of the pattern's purpose
    - `content`: The HTML structure of the pattern (copied from the Code Editor)
    - `categories`: An array of category names where the pattern should appear

Now, when the user selects Patterns from the Block Inserter, the "Featured" Product pattern will be available in the "Featured" category.

### Cleaner pattern content with template files

Many developers prefer to keep the pattern markup separate from the registration code for better organization and maintainability. You can achieve this by storing the pattern content in a separate file and loading it dynamically in the with a custom function.

To start, create a `patterns` directory in your theme or plugin folder and save the pattern content in an PHP file, such as `featured-product.php`.

Next, create a custom function to load the pattern content from the file:

```php
/**
 * Fetch and include the pattern content
 */
function vip_learn_get_pattern_content( $pattern ) {
	ob_start();
	include __DIR__ . "/patterns/{$pattern}";
	return ob_get_clean();
}
```

Finally, update the `content` property in the `register_block_pattern()` function to call the custom function, passing the file name of the pattern file to be included:

```php
'content' => vip_learn_get_pattern_content( 'featured-product.php' ),
```

## Creating new Pattern Categories

WordPress Core registers a set of default pattern categories you can use to organize your patterns, such as "About", "Banners", "Call to Action", and "Featured".

However, you can create custom pattern categories to group related patterns together and provide a more organized experience for content creators.

To create a custom pattern category, you use the `register_block_pattern_category()` function in your theme or plugin file. 

The function takes two arguments: the category slug and an array of category properties, including the label and optional icon.

Here's an example of registering the new "VIP Featured" category for our "Featured Product" pattern:

```php
add_action( 'init', 'vip_learn_register_custom_pattern' );

function vip_learn_register_custom_pattern() {
	register_block_pattern_category(
		'vip-featured',
		array( 'label' => __( 'VIP Featured', 'vip-learn' ) )
	);

	register_block_pattern(
		'vip-learn/featured-product',
		array(
			'title'       => __( 'Featured Product', 'vip-learn' ),
			'description' => _x( 'A product showcase with image, description, and call-to-action button.', 'Block pattern description', 'vip-learn' ),
			'content'     => '<!-- wp:group {"layout":{"type":"constrained"}} -->
<div class="wp-block-group"><!-- wp:columns {"style":{"spacing":{"padding":{"top":"var:preset|spacing|20","bottom":"var:preset|spacing|20","left":"var:preset|spacing|20","right":"var:preset|spacing|20"}}},"borderColor":"accent-4"} -->
    <div class="wp-block-columns has-border-color has-accent-4-border-color" style="padding-top:var(--wp--preset--spacing--20);padding-right:var(--wp--preset--spacing--20);padding-bottom:var(--wp--preset--spacing--20);padding-left:var(--wp--preset--spacing--20)"><!-- wp:column -->
        <div class="wp-block-column"><!-- wp:image -->
            <figure class="wp-block-image"><img alt=""/></figure>
            <!-- /wp:image --></div>
        <!-- /wp:column -->

        <!-- wp:column -->
        <div class="wp-block-column"><!-- wp:heading -->
            <h2 class="wp-block-heading"></h2>
            <!-- /wp:heading -->

            <!-- wp:paragraph -->
            <p></p>
            <!-- /wp:paragraph -->

            <!-- wp:buttons {"layout":{"type":"flex","orientation":"horizontal","justifyContent":"right"}} -->
            <div class="wp-block-buttons"><!-- wp:button -->
                <div class="wp-block-button"><a class="wp-block-button__link wp-element-button"></a></div>
                <!-- /wp:button --></div>
            <!-- /wp:buttons --></div>
        <!-- /wp:column --></div>
    <!-- /wp:columns --></div>
<!-- /wp:group -->

<!-- wp:paragraph -->
<p></p>
<!-- /wp:paragraph -->',
			'categories'  => array( 'vip-featured' ),
		)
	);
}
```

## Using Patterns for Reusable Content

Block patterns excel at creating reusable content structures across a WordPress site. They offer several advantages over traditional methods of content reuse:

1. Flexibility: Unlike reusable blocks, patterns can be customized each time they're used without affecting other instances.
2. Consistency: Patterns ensure design consistency while allowing for content variation.
3. Efficiency: They speed up content creation by providing ready-made layouts.
4. User-Friendly: Non-technical users can easily insert complex layouts without needing to understand block structure.

Here are some scenarios where block patterns are particularly effective:

### Page Headers

Create a consistent header structure across different pages of your site. For example, a "Page Header" pattern might include:

- A full-width cover block with a background image
- A heading for the page title
- A paragraph for a brief description
- A separator block

### Call-to-Action Sections

Design various call-to-action (CTA) patterns for different purposes. For instance:

- A "Newsletter Signup" pattern with a heading, paragraph, and email input
- A "Download eBook" pattern with an image, description, and download button
- A "Contact Us" pattern with a brief message and a button linking to a contact form

### Testimonial Layouts

Create different layouts for displaying customer testimonials:

- A single-column layout for featured testimonials
- A multi-column layout for displaying multiple testimonials side-by-side
- A slider pattern for cycling through testimonials

### Product Features

For e-commerce or SaaS sites, create patterns to showcase product features consistently:

- An "Icon Feature" pattern with an icon, heading, and description
- A "Feature Comparison" pattern with a table layout
- An "Image Feature" pattern with an image, heading, and description on alternating sides

By leveraging block patterns effectively, you can create a library of reusable content structures that enhance consistency, speed up development, and improve the overall user experience of your WordPress site.