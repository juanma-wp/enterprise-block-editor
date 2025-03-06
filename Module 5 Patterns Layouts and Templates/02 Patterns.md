# Lesson 5.2: Patterns

## Understanding Block Patterns

Block patterns are pre-designed layouts composed of multiple blocks that can be easily inserted into posts or pages. They provide a powerful way to create complex, reusable designs without having to manually configure individual blocks each time. Block patterns serve as a bridge between individual blocks and full page templates, offering flexibility and efficiency in content creation.

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

2. Add the necessary blocks to create your pattern. For our "Featured Product" example, we might use:
    - A Group block to contain everything
    - A Columns block with two columns
    - An Image block in the left column
    - Heading, Paragraph, and Button blocks in the right column

3. Style the blocks as desired using the block settings and any custom CSS classes.

4. Once you're satisfied with the design, switch to the Code Editor view and copy the HTML comment structure of the blocks.

### Registering the Custom Pattern

To make your custom pattern available in the block editor, you need to register it using PHP. This is typically done in your theme's `functions.php` file or in a custom plugin.

Here's an example of how to register the "Featured Product" pattern we designed:

```php
add_action( 'init', 'register_custom_pattern' );

function register_custom_pattern() {
    register_block_pattern(
        'my-theme/featured-product',
        array(
            'title'       => __( 'Featured Product', 'my-theme' ),
            'description' => _x( 'A product showcase with image, description, and call-to-action button.', 'Block pattern description', 'my-theme' ),
            'content'     => '<!-- wp:group {"layout":{"type":"constrained"}} -->
<div class="wp-block-group"><!-- wp:columns -->
<div class="wp-block-columns"><!-- wp:column -->
<div class="wp-block-column"><!-- wp:image -->
<figure class="wp-block-image"><img alt=""/></figure>
<!-- /wp:image --></div>
<!-- /wp:column -->

<!-- wp:column -->
<div class="wp-block-column"><!-- wp:heading -->
<h2 class="wp-block-heading"></h2>
<!-- /wp:heading --></div>
<!-- /wp:column --></div>
<!-- /wp:columns --></div>
<!-- /wp:group -->',
            'categories'  => array( 'featured' ),
        )
    );
}
```

In this example, we're using the `register_block_pattern()` function to register our custom pattern. The function takes two arguments:

1. A unique identifier for the pattern (namespaced with your theme or plugin name)
2. An array of pattern properties, including:
    - `title`: The name of the pattern as it appears in the editor
    - `description`: A brief explanation of the pattern's purpose
    - `content`: The HTML structure of the pattern (copied from the Code Editor)
    - `categories`: An array of category names where the pattern should appear

The `add_action( 'init', 'register_custom_pattern' );` line ensures that our pattern is registered when WordPress initializes.

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

In conclusion, block patterns are a powerful feature of the WordPress block editor that bridge the gap between individual blocks and full page templates. By understanding how to create, register, and effectively use custom patterns, you can significantly enhance your WordPress development workflow and provide a better content creation experience for your clients or team members.