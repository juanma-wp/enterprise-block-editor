# Building Block Themes

Block themes represent the future of WordPress site development, leveraging the full capability of blocks and the Block Editor to create highly customizable, flexible websites. 

Unlike traditional themes that rely heavily on PHP templates and custom CSS, block themes use HTML templates, block markup and a configuration file called `theme.json` to define their appearance and functionality. 

Block themes also enable full site editing, allowing site admins to visually customize every aspect of their site directly through the editor interface. 

This lesson explores the essential components of building effective block themes, including developer tooling to get started, configuring theme.json, creating custom templates and patterns, and finally extending functionality through child themes.

## Create Block Theme

Built and maintained by the WordPress project, the Create Block Theme plugin bridges the gap between visual editing in the Site Editor and generating theme files. 

This official WordPress.org plugin simplifies block theme creation through database-to-filesystem translation, enabling developers to export editor customizations directly to theme files.

### Core Features

The Create Block Theme plugin offers several key features that streamline the theme development process:

1. **Theme Creation Options**: Generate blank themes, child themes, or cloned versions of existing themes with one click.
2. **Style Variation Development**: Create and export custom color palettes and typography sets as reusable style variations
3. **Asset Management**: Automatically organizes images used in templates to an `assets` folder and ensures block markup compatibility
4. **Translation Readiness**: Converts template and pattern strings to gettext functions for localization
5. **Theme.json Inspection**: Provides read-only access to theme configuration files directly in the editor interface

## Creating Your First Blank Block Theme

### Step 1: Plugin Installation

1. Navigate to **Plugins → Add New** in WordPress admin
2. Search for "Create Block Theme"
3. Install and activate the plugin[6][8]

### Step 2: Access Creation Interface

Access the plugin through either:
- **Appearance → Create Block Theme** (traditional admin path)
- **Wrench icon** in Site Editor toolbar (recommended workflow)[5][7]

### Step 3: Generate Blank Theme
1. Select **"Create Blank Theme"** from creation options
2. Configure basic metadata:
   ```json
   {
     "name": "Enterprise Starter",
     "description": "Custom corporate theme",
     "author": "Your Agency",
     "version": "1.0"
   }
   ```
3. Click **Create Theme** to generate files[4][8]

The plugin creates this initial structure:
```
/enterprise-starter
├── style.css
├── theme.json
├── templates/
│   └── index.html
└── parts/
    └── header.html
    └── footer.html
```

## Building and Extending Block Themes with theme.json

Theme.json was introduced in WordPress 5.8 as a mechanism to configure the editor, providing finer-grained control and introducing a structured approach to managing styles for WordPress themes. This configuration file consolidates various APIs related to styles into a single point, creating a canonical way to define the settings and styles of the theme.

### Understanding the Purpose of theme.json

The theme.json file serves multiple essential functions in a block theme. 

First, it replaces the proliferation of theme support flags with a standardized approach to configuring the editor. 

Second, it provides a structured way to manage CSS, reducing specificity wars and the amount of CSS enqueued. 

Finally, it enables more granular control over which customization options are available to users on a global or per-block basis.

The file should be placed in the root directory of your theme, and as of WordPress 6.0, using the $schema property provides additional benefits:

```json
{
    "$schema": "https://schemas.wp.org/trunk/theme.json",
    "version": 2,
    "settings": {
        // Global settings defined here
        "layout": {
            "contentSize": "840px",
            "wideSize": "1100px"
        }
    },
    "styles": {
        // Global styles defined here
    }
}
```

Adding the $schema reference provides autocomplete and inline documentation while working on your theme.json files[5]. You can also specify a particular WordPress version in the schema URL, such as `https://schemas.wp.org/wp/6.5/theme.json` for version-specific features.

### Settings vs. Styles in theme.json

Theme.json is organized into two primary sections: settings and styles.

#### Settings

The settings section controls which options are available in the editor interface. You can define global settings that apply to all blocks or specify settings for individual block types:

```json
{
    "version": 2,
    "settings": {
        "color": {
            "palette": [
                {
                    "name": "Primary",
                    "slug": "primary",
                    "color": "#0073aa"
                },
                {
                    "name": "Secondary",
                    "slug": "secondary",
                    "color": "#23282d"
                }
            ],
            "gradients": [],
            "custom": true,
            "customGradient": true
        },
        "typography": {
            "fontSizes": [
                {
                    "name": "Small",
                    "slug": "small",
                    "size": "13px"
                },
                {
                    "name": "Medium",
                    "slug": "medium",
                    "size": "20px"
                }
            ],
            "customFontSize": true
        },
        "blocks": {
            "core/paragraph": {
                "color": {
                    "custom": false
                }
            }
        }
    }
}
```

This example defines a custom color palette and font sizes globally, but disables custom colors specifically for paragraph blocks. This level of granularity allows theme developers to provide a consistent design system while still offering appropriate flexibility.

#### Styles

The styles section defines the default visual appearance of blocks in both the editor and front end:

```json
{
    "version": 2,
    "styles": {
        "color": {
            "background": "#ffffff",
            "text": "#333333"
        },
        "typography": {
            "fontSize": "16px",
            "lineHeight": "1.6"
        },
        "blocks": {
            "core/heading": {
                "typography": {
                    "fontWeight": "700"
                }
            }
        }
    }
}
```

This example sets global text and background colors, default font size and line height, with specific styling for heading blocks. The styles section works with CSS custom properties under the hood, making it efficient and reducing CSS specificity issues[2].

### Layout Configuration

One of the most important settings in theme.json is the layout configuration, which defines content width and alignment options:

```json
{
    "version": 2,
    "settings": {
        "layout": {
            "contentSize": "840px",
            "wideSize": "1100px"
        }
    }
}
```

These settings define the default content width (contentSize) and the width for wide-aligned elements (wideSize). WordPress automatically handles full-width elements without requiring an additional setting[8].

## Creating Custom Templates, Template Parts, and Patterns

Block themes use a combination of templates, template parts, and patterns to create the structure and reusable elements of a website. Understanding how these components work together is essential for effective block theme development.

### Understanding Templates and Template Structure

Templates are HTML files that define the layout of different pages in your WordPress site. In a block theme, templates use block markup instead of PHP template tags. At minimum, a block theme requires an `index.html` file inside a `templates` folder[8].

For example, a basic `index.html` template might look like this:

```html
<!-- wp:template-part {"slug":"header","tagName":"header"} /-->

<!-- wp:group {"tagName":"main","layout":{"type":"constrained"}} -->
<main class="wp-block-group">
   <!-- wp:query {"queryId":0,"query":{"perPage":10,"offset":0,"postType":"post","order":"desc","orderBy":"date","author":"","search":"","sticky":"","inherit":true}} -->
   <div class="wp-block-query">
      <!-- wp:post-template -->
      <!-- wp:post-title {"isLink":true} /-->
      <!-- wp:post-excerpt /-->
      <!-- /wp:post-template -->

      <!-- wp:query-pagination -->
      <!-- wp:query-pagination-previous /-->
      <!-- wp:query-pagination-numbers /-->
      <!-- wp:query-pagination-next /-->
      <!-- /wp:query-pagination -->
   </div>
   <!-- /wp:query -->
</main>
<!-- /wp:group -->

<!-- wp:template-part {"slug":"footer","tagName":"footer"} /-->
```

This template includes a header template part, a main content area with a query for displaying posts, and a footer template part. The comments within `` delimiters represent block markup, which WordPress uses to render blocks.

### Creating Template Parts

Template parts are reusable components that can be included in multiple templates. Common template parts include headers, footers, and sidebars. In a block theme, template parts are stored in the `parts` folder.

For example, a basic `header.html` template part might look like:

```html
<!-- wp:group {"style":{"spacing":{"padding":{"top":"1rem","bottom":"1rem"}}},"layout":{"type":"constrained"}} -->
<div class="wp-block-group" style="padding-top:1rem;padding-bottom:1rem">
   <!-- wp:site-title {"textAlign":"center"} /-->

   <!-- wp:navigation {"layout":{"type":"flex","justifyContent":"center"}} /-->
</div>
<!-- /wp:group -->
```

This creates a simple header with a centered site title and navigation menu.

### Working with Block Patterns

Block patterns are reusable block layouts that can be inserted into pages or posts. They provide a way to create complex designs without having to recreate them from scratch each time.

There are two methods for registering block patterns in WordPress:

1. By placing files with block markup in the `/patterns` folder in your theme
2. By manually calling the `register_block_pattern()` function

#### Using the Patterns Directory

The simplest approach is to create HTML files in the `/patterns` directory. WordPress automatically registers these as patterns. For example, a file named `call-to-action.html` might contain:

```html
<!-- wp:group {"backgroundColor":"primary","textColor":"background","className":"cta-pattern"} -->
<div class="wp-block-group cta-pattern has-background-color has-primary-background-color has-text-color has-background">
   <!-- wp:heading {"textAlign":"center"} -->
   <h2 class="has-text-align-center">Ready to Get Started?</h2>
   <!-- /wp:heading -->

   <!-- wp:paragraph {"align":"center"} -->
   <p class="has-text-align-center">Join thousands of satisfied customers using our product</p>
   <!-- /wp:paragraph -->

   <!-- wp:buttons {"layout":{"type":"flex","justifyContent":"center"}} -->
   <div class="wp-block-buttons">
      <!-- wp:button -->
      <div class="wp-block-button"><a class="wp-block-button__link">Sign Up Now</a></div>
      <!-- /wp:button -->
   </div>
   <!-- /wp:buttons -->
</div>
<!-- /wp:group -->

```

WordPress will automatically register this as a pattern titled "Call to Action" (based on the filename).

#### Registering Patterns Programmatically

Alternatively, you can register patterns using PHP in your theme's `functions.php` file:

```php
add_action( 'init', 'mytheme_register_block_patterns' );

function mytheme_register_block_patterns() {
    register_block_pattern(
        'mytheme/cta-pattern',
        array(
            'title'       => __( 'Call to Action', 'mytheme' ),
            'description' => __( 'A call to action block with a heading, text, and a button', 'mytheme' ),
            'categories'  => array( 'cta' ),
            'keywords'    => array( 'call to action', 'cta', 'button' ),
            'content'     => '<!-- wp:group {"backgroundColor":"primary","textColor":"background","className":"cta-pattern"} -->
<div class="wp-block-group cta-pattern has-background-color has-primary-background-color has-text-color has-background">
    <!-- wp:heading {"textAlign":"center"} -->
    <h2 class="has-text-align-center">Ready to Get Started?</h2>
    <!-- /wp:heading -->
    
    <!-- wp:paragraph {"align":"center"} -->
    <p class="has-text-align-center">Join thousands of satisfied customers using our product</p>
    <!-- /wp:paragraph -->
    
    <!-- wp:buttons {"layout":{"type":"flex","justifyContent":"center"}} -->
    <div class="wp-block-buttons">
        <!-- wp:button -->
        <div class="wp-block-button"><a class="wp-block-button__link">Sign Up Now</a></div>
        <!-- /wp:button -->
    </div>
    <!-- /wp:buttons -->
</div>
<!-- /wp:group -->',
        )
    );
}

```

This approach gives you more control over pattern properties like categories and keywords, which affect how patterns appear in the inserter[9].

### Creating Patterns with Core Blocks

Many developers find that using core blocks is the most efficient way to create custom patterns. This approach leverages existing functionality without requiring custom React development. A practical workflow might look like:

1. Create the basic block structure with core blocks
2. Wrap them in a Group block
3. Give the Group a custom CSS class under the Advanced tab
4. Lock everything inside the Group to prevent unwanted edits
5. Register the result as a pattern
6. Add CSS in theme.json or style.css to customize the appearance[6]

This approach provides a good balance between design flexibility and development efficiency.

## Creating Child Themes

Child themes provide a way to customize an existing theme while preserving your changes during theme updates. With the advent of block themes, the role of child themes has evolved, but they remain useful in specific scenarios.

### When to Use a Block Child Theme

The need for child themes has diminished somewhat with block themes, as many customizations can now be made through the Site Editor interface. When you edit templates in the Site Editor, WordPress saves your changes to the database, protecting them from theme updates[4].

However, child themes are still necessary when:

1. You need to modify theme.json, HTML, CSS, PHP, or JavaScript files directly
2. You want to remove specific settings or styles that cannot be changed through the WordPress interface
3. You need to override parent theme templates with your custom versions[4][7]

### Setting Up a Block Child Theme

Creating a block child theme requires at minimum two files:

1. `style.css` with the appropriate theme header
2. `functions.php` to enqueue the parent theme's stylesheet

#### The style.css File

```css
/*
Theme Name: My Block Child Theme
Template: twentytwentyfour
Author: Your Name
Description: A child theme of Twenty Twenty-Four
Version: 1.0
Requires at least: 6.0
Tested up to: 6.5
Requires PHP: 7.4
License: GNU General Public License v2 or later
License URI: http://www.gnu.org/licenses/gpl-2.0.html
Text Domain: my-block-child
*/
```

The `Template` line is critical as it identifies the parent theme's folder name.

#### The functions.php File

```php
<!-- wp:template-part {"slug":"header","tagName":"header"} /-->

<!-- wp:group {"tagName":"main","layout":{"type":"constrained"}} -->
<main class="wp-block-group">
    <!-- wp:cover {"url":"my-hero-image.jpg","isDark":false,"dimRatio":50} -->
    <div class="wp-block-cover is-light">
        <span aria-hidden="true" class="wp-block-cover__background has-background-dim"></span>
        <img class="wp-block-cover__image-background" src="my-hero-image.jpg" alt="" />
        <div class="wp-block-cover__inner-container">
            <!-- wp:heading {"textAlign":"center","level":1,"style":{"typography":{"fontWeight":"700"}}} -->
            <h1 class="has-text-align-center" style="font-weight:700">Welcome to Our Website</h1>
            <!-- /wp:heading -->
            
            <!-- wp:buttons {"layout":{"type":"flex","justifyContent":"center"}} -->
            <div class="wp-block-buttons">
                <!-- wp:button -->
                <div class="wp-block-button"><a class="wp-block-button__link">Learn More</a></div>
                <!-- /wp:button -->
            </div>
            <!-- /wp:buttons -->
        </div>
    </div>
    <!-- /wp:cover -->
    
    <!-- wp:columns -->
    <div class="wp-block-columns">
        <!-- wp:column -->
        <div class="wp-block-column">
            <!-- wp:heading {"level":3} -->
            <h3>Our Services</h3>
            <!-- /wp:heading -->
            
            <!-- wp:paragraph -->
            <p>Learn about what we offer and how we can help your business grow.</p>
            <!-- /wp:paragraph -->
        </div>
        <!-- /wp:column -->
        
        <!-- wp:column -->
        <div class="wp-block-column">
            <!-- wp:heading {"level":3} -->
            <h3>About Us</h3>
            <!-- /wp:heading -->
            
            <!-- wp:paragraph -->
            <p>Discover our story and meet the team behind our success.</p>
            <!-- /wp:paragraph -->
        </div>
        <!-- /wp:column -->
        
        <!-- wp:column -->
        <div class="wp-block-column">
            <!-- wp:heading {"level":3} -->
            <h3>Contact</h3>
            <!-- /wp:heading -->
            
            <!-- wp:paragraph -->
            <p>Get in touch with us to discuss your project or ask questions.</p>
            <!-- /wp:paragraph -->
        </div>
        <!-- /wp:column -->
    </div>
    <!-- /wp:columns -->
</main>
<!-- /wp:group -->

<!-- wp:template-part {"slug":"footer","tagName":"footer"} /-->

```

This template creates a custom homepage with a hero section and a three-column content area while still using the header and footer from the parent theme.

## Conclusion

Building block themes represents a significant shift in WordPress theme development, placing greater emphasis on configuration over code and visual editing over template manipulation. The theme.json file provides unprecedented control over both settings and styles, enabling theme developers to create consistent design systems while still offering appropriate flexibility to users.

Custom templates, template parts, and patterns form the building blocks of modern theme development, allowing for reusable components and efficient workflows. While the need for child themes has diminished somewhat with the advent of the Site Editor, they remain an essential tool for theme developers who need to make deeper customizations while preserving compatibility with parent theme updates.

As block themes continue to evolve, mastering these fundamentals will ensure you can create sophisticated, flexible themes that take full advantage of WordPress's powerful block editor capabilities.

Citations:
[1] https://github.co
[2] https://developer.wordpress.org/block-editor/how-to-guides/themes/global-settings-and-styles/
[3] https://pressable.com/blog/creating-block-patterns-wordpress/
[4] https://fullsiteediting.com/lessons/child-themes/
[5] https://gutenberg.10up.com/reference/Themes/theme-json/
[6] https://www.reddit.com/r/ProWordPress/comments/1agytp9/using_core_blocks_for_making_custom_block_patterns/
[7] https://www.godaddy.com/resources/skills/how-to-create-a-block-child-theme-in-wordpress
[8] https://fullsiteediting.com/lessons/creating-block-based-themes/
[9] https://developer.wordpress.org/themes/features/block-patterns/
[10] https://www.reddit.com/r/Wordpress/comments/1d96n9t/do_block_themes_require_a_child/
[11] https://jetpack.com/resources/wordpress-theme-json/
[12] https://learn.wordpress.org/tutorial/the-difference-between-reusable-blocks-block-patterns-templates-and-template-parts/
[13] https://www.youtube.com/watch?v=m9A3HZ2cuZM
[14] https://developer.wordpress.org/themes/global-settings-and-styles/
[15] https://jetpack.com/resources/wordpress-block-patterns/
[16] https://learn.wordpress.org/lesson-plan/create-a-basic-child-theme-for-block-themes/
[17] https://kinsta.com/blog/theme-json/
[18] https://maxiblocks.com/wordpress-block-templates/
[19] https://developer.wordpress.org/themes/advanced-topics/child-themes/
[20] https://fullsiteediting.com/lessons/creating-theme-json/
[21] https://wordpress.stackexchange.com/questions/395528/can-you-use-block-patterns-in-block-templates-or-insert-them-programmatically

---
Answer from Perplexity: pplx.ai/share