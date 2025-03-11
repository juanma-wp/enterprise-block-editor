# 1.2: Overview of Block Themes

One of the new technologies that the Block Editor has brought to WordPress is the concept of Block themes.

Block themes in WordPress represent a significant shift towards a more modular and flexible approach to theme development. They leverage the power of the block editor, allowing developers to build entire sites using blocks. 

This lesson will delve into the core components of block themes, including templates and template parts, patterns, global settings and styles, and how to add custom functionality and styling.

## Classic Themes

Traditionally, WordPress themes have been built using PHP templates to render content. WordPress supports a [template hierarchy](https://developer.wordpress.org/themes/basics/template-hierarchy/) that determines which template file to use based on the type of content being displayed.

Here's an example of a classic theme template file, `single.php`, which displays a single post:

```php
<?php
/**
 * The template for displaying all single posts and attachments.
 *
 * @package WordPress
 * @subpackage Twenty_Fifteen
 * @since Twenty Fifteen 1.0
 */

get_header(); ?>

<div id="primary" class="content-area">
    <main id="main" class="site-main" role="main">
        <?php
        // Start the Loop.
        while ( have_posts() ) : the_post();

            // Include the post format-specific template for the content.
            get_template_part( 'content', get_post_format() );

            // If comments are open or we have at least one comment, load up the comment template.
            if ( comments_open() || get_comments_number() ) :
                comments_template();
            endif;

        endwhile;
        ?>
    </main><!-- #main -->
</div><!-- #primary -->

<?php get_footer(); ?>
```

Template functions called tags like `get_header()`, `get_footer()`, and `get_template_part()` are used to include other template files and display content. For example, the `get_header()` function includes the `header.php` file from the theme, which contains the header markup for the site.

```php
<?php
/**
 * The template for displaying the header
 *
 * Displays all of the head element and everything up until the "site-content" div.
 *
 * @package WordPress
 * @subpackage Twenty_Fifteen
 * @since Twenty Fifteen 1.0
 */
?><!DOCTYPE html>
<html <?php language_attributes(); ?> class="no-js">
<head>
	<meta charset="<?php bloginfo( 'charset' ); ?>">
	<meta name="viewport" content="width=device-width, initial-scale=1.0">
	<link rel="profile" href="https://gmpg.org/xfn/11">
	<link rel="pingback" href="<?php echo esc_url( get_bloginfo( 'pingback_url' ) ); ?>">
	<?php wp_head(); ?>
</head>

<body <?php body_class(); ?>>
<?php wp_body_open(); ?>
<div id="page" class="hfeed site">
	<a class="skip-link screen-reader-text" href="#content">
		<?php
		/* translators: Hidden accessibility text. */
		_e( 'Skip to content', 'twentyfifteen' );
		?>
	</a>

	<div id="sidebar" class="sidebar">
		<header id="masthead" class="site-header">
			<div class="site-branding">
				<?php
					twentyfifteen_the_custom_logo();

				if ( is_front_page() && is_home() ) :
					?>
						<h1 class="site-title"><a href="<?php echo esc_url( home_url( '/' ) ); ?>" rel="home"><?php bloginfo( 'name' ); ?></a></h1>
					<?php else : ?>
						<p class="site-title"><a href="<?php echo esc_url( home_url( '/' ) ); ?>" rel="home"><?php bloginfo( 'name' ); ?></a></p>
						<?php
					endif;

					$description = get_bloginfo( 'description', 'display' );
					if ( $description || is_customize_preview() ) :
						?>
						<p class="site-description"><?php echo $description; ?></p>
						<?php
					endif;
					?>
				<button class="secondary-toggle"><?php _e( 'Menu and widgets', 'twentyfifteen' ); ?></button>
			</div><!-- .site-branding -->
		</header><!-- .site-header -->

		<?php get_sidebar(); ?>
	</div><!-- .sidebar -->

	<div id="content" class="site-content">
```

The main stylesheet for the theme is typically stored in a `style.css` file, which contains the CSS rules for styling the site. This is enqueued in the theme's `functions.php` file using the `wp_enqueue_style()` function. Any additional custom functionality is also added to the `functions.php` file.

## Block Themes

With the introduction of the Block Editor, WordPress themes have evolved to embrace a block paradigm approach to build themes. Block themes leverage the block editor's capabilities to create dynamic and customizable layouts using blocks.

Instead of using PHP templates, block themes use HTML files with block markup to define the structure of the site. Block markup is a combination of HTML and block-specific attributes that is generated when a block is inserted into the editor.

Here's an example of the `single.html` template file from a block theme:

```html
<!-- wp:template-part {"slug":"header"} /-->

<!-- wp:group {"tagName":"main","style":{"spacing":{"margin":{"top":"var:preset|spacing|60"}}},"layout":{"type":"constrained"}} -->
<main class="wp-block-group" style="margin-top:var(--wp--preset--spacing--60)">
    <!-- wp:group {"align":"full","style":{"spacing":{"padding":{"top":"var:preset|spacing|60","bottom":"var:preset|spacing|60"}}},"layout":{"type":"constrained"}} -->
    <div class="wp-block-group alignfull" style="padding-top:var(--wp--preset--spacing--60);padding-bottom:var(--wp--preset--spacing--60)">
        <!-- wp:post-title {"level":1} /-->
        <!-- wp:post-featured-image {"aspectRatio":"3/2"} /-->
        <!-- wp:pattern {"slug":"twentytwentyfive/hidden-written-by"} /-->
        <!-- wp:post-content {"align":"full","layout":{"type":"constrained"}} /-->
        <!-- wp:group {"style":{"spacing":{"padding":{"top":"var:preset|spacing|60","bottom":"var:preset|spacing|60"}}},"layout":{"type":"constrained"}} -->
        <div class="wp-block-group" style="padding-top:var(--wp--preset--spacing--60);padding-bottom:var(--wp--preset--spacing--60)">
            <!-- wp:post-terms {"term":"post_tag","separator":"  ","className":"is-style-post-terms-1"} /-->
        </div>
        <!-- /wp:group -->

        <!-- wp:pattern {"slug":"twentytwentyfive/post-navigation"} /-->
        <!-- wp:pattern {"slug":"twentytwentyfive/comments"} /-->
    </div>
    <!-- /wp:group -->
    <!-- wp:pattern {"slug":"twentytwentyfive/more-posts"} /-->
</main>
<!-- /wp:group -->

<!-- wp:template-part {"slug":"footer"} /-->
```

One of the benefits of this approach is that theme developer can design the site's layout inside the Site Editor, and then export the block markup to the relevant theme file. Plugins like [Create Block Theme](https://wordpress.org/plugins/create-block-theme/) help in creating block themes, and turn the WordPress Site Editor into a theme development tool.

### Core Differences

- **Technology**: Classic themes rely on PHP to render pages, while block themes use HTML files with block markup. This makes block themes potentially more lightweight and faster[1][2].

- **Customization**: Block themes provide more granular control over site customization, allowing users to edit headers, footers, and other page parts directly. Classic themes often require plugins or custom code for similar flexibility[1][6].

- **Template Files**: Classic themes store template files in the theme's root directory, following a PHP-based template hierarchy (e.g., `index.php`, `single.php`). In contrast, block themes store templates in a `templates` directory using HTML files (e.g., `index.html`, `single.html`)[2][4].

### Example Templates

To illustrate the difference, consider a `single.php` file from a classic theme versus a `single.html` file from a block theme.

#### Classic Theme: `single.php`

```php
<?php
/**
 * The template for displaying all single posts and attachments.
 *
 * @package WordPress
 * @subpackage Twenty_Fifteen
 * @since Twenty Fifteen 1.0
 */

get_header(); ?>

<div id="primary" class="content-area">
    <main id="main" class="site-main" role="main">
        <?php
        // Start the Loop.
        while ( have_posts() ) : the_post();

            // Include the post format-specific template for the content.
            get_template_part( 'content', get_post_format() );

            // If comments are open or we have at least one comment, load up the comment template.
            if ( comments_open() || get_comments_number() ) :
                comments_template();
            endif;

        endwhile;
        ?>
    </main><!-- #main -->
</div><!-- #primary -->

<?php get_footer(); ?>
```

#### Block Theme: `single.html`

```html
<!-- wp:template-part {"slug":"header"} /-->

<!-- wp:group {"tagName":"main","style":{"spacing":{"margin":{"top":"var:preset|spacing|60"}}},"layout":{"type":"constrained"}} -->
<main class="wp-block-group" style="margin-top:var(--wp--preset--spacing--60)">
    <!-- wp:group {"align":"full","style":{"spacing":{"padding":{"top":"var:preset|spacing|60","bottom":"var:preset|spacing|60"}}},"layout":{"type":"constrained"}} -->
    <div class="wp-block-group alignfull" style="padding-top:var(--wp--preset--spacing--60);padding-bottom:var(--wp--preset--spacing--60)">
        <!-- wp:post-title {"level":1} /-->
        <!-- wp:post-featured-image {"aspectRatio":"3/2"} /-->
        <!-- wp:pattern {"slug":"twentytwentyfive/hidden-written-by"} /-->
        <!-- wp:post-content {"align":"full","layout":{"type":"constrained"}} /-->
        <!-- wp:group {"style":{"spacing":{"padding":{"top":"var:preset|spacing|60","bottom":"var:preset|spacing|60"}}},"layout":{"type":"constrained"}} -->
        <div class="wp-block-group" style="padding-top:var(--wp--preset--spacing--60);padding-bottom:var(--wp--preset--spacing--60)">
            <!-- wp:post-terms {"term":"post_tag","separator":"  ","className":"is-style-post-terms-1"} /-->
        </div>
        <!-- /wp:group -->

        <!-- wp:pattern {"slug":"twentytwentyfive/post-navigation"} /-->
        <!-- wp:pattern {"slug":"twentytwentyfive/comments"} /-->
    </div>
    <!-- /wp:group -->
    <!-- wp:pattern {"slug":"twentytwentyfive/more-posts"} /-->
</main>
<!-- /wp:group -->

<!-- wp:template-part {"slug":"footer"} /-->
```

### Directory Structure

Classic themes typically store all template files directly in the theme's root directory. Block themes, however, organize their files into specific folders:

- **Classic Themes**: Template files like `index.php`, `single.php`, and `page.php` are stored in the root directory.
- **Block Themes**:
    - **templates**: Contains main template files like `index.html`, `single.html`, and `page.html`.
    - **parts**: Holds reusable template parts such as `header.html` and `footer.html`.
    - **patterns**: Stores custom block patterns.

This structured approach in block themes makes them more organized and easier to maintain compared to classic themes[6][8].

## Templates and Template Parts

In block themes, **templates** are the foundation upon which pages are built. They are composed of blocks and can be thought of as the blueprints for different types of content, such as single posts, archives, or the front page. Templates follow a specific hierarchy similar to traditional WordPress themes but are defined using HTML files (e.g., `index.html`, `single.html`, `archive.html`) instead of PHP files.

**Template parts**, on the other hand, are reusable blocks of content that can be included within templates. These parts can be anything from headers to footers or sidebars. They are stored in a `parts` directory within the theme and are typically named after their function (e.g., `header.html`, `footer.html`). Template parts allow for easy maintenance and consistency across the site, as changes made to a part are reflected everywhere it is used.

For example, if you want to create a custom header template part, you would create a file named `header.html` in the `parts` directory. This file would contain the necessary blocks to define your header, such as a site title, navigation menu, or logo.

### Patterns

**Block patterns** are pre-designed arrangements of blocks that can be easily inserted into pages or posts. They provide a quick way to add complex layouts without needing to manually configure each block. Patterns can be registered in themes or plugins using the `register_block_pattern` function in PHP. This allows developers to offer users a variety of layouts that fit the theme's design and functionality.

To register a custom pattern, you would add code similar to the following in your theme's `functions.php` file:

```php
add_action( 'init', 'themeslug_register_patterns' );

function themeslug_register_patterns() {
    register_block_pattern(
        'themeslug/hero',
        array(
            'title'       => __( 'Hero', 'themeslug' ),
            'categories'  => array( 'featured' ),
            'source'      => 'theme',
            'content'     => '',
        )
    );
}
```

### Global Settings and Styles

**Global Settings** and **Global Styles** are crucial components of block themes, allowing developers to manage site-wide customization options efficiently. Global Settings define which customization options are available for blocks, while Global Styles provide a centralized way to apply consistent styling across the site.

These settings and styles are managed through the `theme.json` file, which is used to configure the block editor's interface and define default styles for blocks. For instance, you can set default typography or color schemes that apply to all blocks unless overridden at the block level.

Here's an example of how you might define a global font size style in `theme.json`:

```json
{
    "version": 2,
    "styles": {
        "typography": {
            "fontSize": "var(--wp--preset--font-size--xx-large)"
        }
    }
}
```

### Custom Functionality and Styling

#### Custom Functionality with `functions.php`

The `functions.php` file in a block theme serves a similar role to its counterpart in classic themes. It allows developers to add custom PHP code to extend the theme's functionality. This can include registering custom block patterns, adding support for specific features, or defining custom hooks and filters.

For example, to add support for block template parts in a classic theme, you would use the following code:

```php
add_action( 'after_setup_theme', function() {
    add_theme_support( 'block-template-parts' );
} );
```

#### Styling with `style.css`

While block themes rely heavily on the block editor for layout and design, the `style.css` file is still used for additional styling that isn't covered by the block editor's settings. This can include custom CSS rules for specific elements or overriding default block styles.

For instance, if you want to add a custom background color to all paragraphs, you could add the following CSS to your `style.css` file:

```css
.wp-block-paragraph {
    background-color: #f0f0f0;
}
```

In summary, block themes offer a powerful and flexible way to build WordPress sites using blocks. By understanding templates, template parts, patterns, global settings, and styles, developers can create robust and maintainable themes. Additionally, custom functionality can be added via `functions.php`, and styling can be enhanced with `style.css`. This foundation is crucial for mastering the Advanced Block Editor and developing sophisticated Enterprise WordPress solutions.