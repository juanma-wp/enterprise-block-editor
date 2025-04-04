# Building Block Themes

Block themes represent the future of WordPress site development, leveraging the full capability of blocks and the Block Editor to create highly customizable, flexible websites. 

Unlike traditional themes that rely heavily on PHP templates and custom CSS, block themes use HTML templates, block markup and a configuration file called `theme.json` to define their appearance and functionality. 

Block themes also enable full site editing, allowing site admins to visually customize every aspect of their site directly through the editor interface. 

This lesson explores the essential components of building effective block themes, including developer tooling to get started, configuring theme.json, creating custom templates and patterns, and finally extending functionality through child themes.

## Create Block Theme

Built and maintained by the various WordPress project contributors, the [Create Block Theme](https://wordpress.org/plugins/create-block-theme/) plugin bridges the gap between visual editing in the Site Editor and generating theme files. 

This official WordPress.org plugin simplifies block theme creation through database-to-filesystem translation, enabling developers to export editor customizations directly to theme files.

### Core Features

The Create Block Theme plugin offers several key features that streamline the theme development process:

1. **Theme Creation Options**: Generate blank themes, child themes, or cloned versions of existing themes with one click.
2. **Style Variation Development**: Create and export custom color palettes and typography sets as reusable style variations
3. **Asset Management**: Automatically organizes images used in templates to an `assets` folder and ensures block markup compatibility
4. **Translation Readiness**: Converts template and pattern strings to gettext functions for localization
5. **Theme.json Inspection**: Provides read-only access to theme configuration files directly in the editor interface

### Creating Your First Blank Block Theme

With the plugin installed, either on a staging or local environment, you can create a blank block theme. 

There are two ways to create a new block theme. 

The first is from the WP Admin menu:

- Navigate to **Appearance → Create Block Theme**, and then select the **Create a new Blank Theme** option.
- Enter the theme name, and optional theme details like the description and author.
- Click **Create and Activate Blank Theme**.

The new theme will be created and activated, and you will be redirected to the Site Editor to start editing the theme visually.

Alternatively, you can create a new theme directly from the Site Editor

- Click on the **Create Block Theme** (wrench) icon in the Site Editor toolbar and select **Create Blank Theme**.
- Enter the theme name, and optional theme details like the description, author, and theme or author links.
- Click **Create Blank Theme**.

The new theme will be created and activated, and the Site Editor will open with the new theme loaded.

## Core Concepts of Block Themes

### theme.json

Open the newly created theme in your code editor, and then edit the `theme.json` file in the root of the theme directory.

```json
{
	"$schema": "https://schemas.wp.org/wp/6.8/theme.json",
	"settings": {
		"appearanceTools": true,
		"layout": {
			"contentSize": "620px",
			"wideSize": "1000px"
		},
		"spacing": {
			"units": [ "%", "px", "em", "rem", "vh", "vw" ]
		},
		"typography": {
			"fontFamilies": [
				{
					"fontFamily": "-apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen-Sans, Ubuntu, Cantarell, 'Helvetica Neue', sans-serif",
					"name": "System Font",
					"slug": "system-font"
				}
			]
		},
		"useRootPaddingAwareAlignments": true
	},
	"templateParts": [
		{
			"area": "header",
			"name": "header"
		},
		{
			"area": "footer",
			"name": "footer"
		}
	],
	"version": 3
}
```

The theme.json file serves multiple essential functions in a block theme. 

First, it replaces the proliferation of theme support flags with a standardized approach to configuring the editor. In the past, theme developers had to use various theme support flags to enable or disable features like custom colors, font sizes, and layout options. Now these settings can be enabled or disabled in a single file, making it easier to manage and understand the theme's capabilities.

Second, it provides a structured way to manage CSS, reducing specificity wars and the amount of CSS enqueued. 

Finally, it enables more granular control over which customization options are available to users on a global or per-block basis.

Notice that the theme.json includes a $schema property. Adding the `$schema` reference provides autocomplete and inline documentation while working on your `theme.json` files. 

You can use the default schema at `https://schemas.wp.org/trunk/theme.json` or specify a particular WordPress version in the schema URL, such as `https://schemas.wp.org/wp/6.8/theme.json` for version-specific features.

### Core theme.json

By default, WordPress ships with and includes a core `theme.json` file that provides a set of default settings and styles. This file is located in the `wp-includes/theme.json` directory.

```php
{
	"$schema": "https://schemas.wp.org/trunk/theme.json",
	"version": 3,
	"settings": {
		"appearanceTools": false,
		"useRootPaddingAwareAlignments": false,
		"border": {
			"color": false,
			"radius": false,
			"style": false,
			"width": false
		},
		"color": {
			"background": true,
			"button": true,
			"caption": true,
			"custom": true,
			"customDuotone": true,
			"customGradient": true,
			"defaultDuotone": true,
			"defaultGradients": true,
			"defaultPalette": true,
			"duotone": [
				{
					"name": "Dark grayscale",
					"colors": [ "#000000", "#7f7f7f" ],
					"slug": "dark-grayscale"
				},
				{
					"name": "Grayscale",
					"colors": [ "#000000", "#ffffff" ],
					"slug": "grayscale"
				},
				{
					"name": "Purple and yellow",
					"colors": [ "#8c00b7", "#fcff41" ],
					"slug": "purple-yellow"
				},
				{
					"name": "Blue and red",
					"colors": [ "#000097", "#ff4747" ],
					"slug": "blue-red"
				},
				{
					"name": "Midnight",
					"colors": [ "#000000", "#00a5ff" ],
					"slug": "midnight"
				},
				{
					"name": "Magenta and yellow",
					"colors": [ "#c7005a", "#fff278" ],
					"slug": "magenta-yellow"
				},
				{
					"name": "Purple and green",
					"colors": [ "#a60072", "#67ff66" ],
					"slug": "purple-green"
				},
				{
					"name": "Blue and orange",
					"colors": [ "#1900d8", "#ffa96b" ],
					"slug": "blue-orange"
				}
			],
			"gradients": [
				{
					"name": "Vivid cyan blue to vivid purple",
					"gradient": "linear-gradient(135deg,rgba(6,147,227,1) 0%,rgb(155,81,224) 100%)",
					"slug": "vivid-cyan-blue-to-vivid-purple"
				},
				{
					"name": "Light green cyan to vivid green cyan",
					"gradient": "linear-gradient(135deg,rgb(122,220,180) 0%,rgb(0,208,130) 100%)",
					"slug": "light-green-cyan-to-vivid-green-cyan"
				},
				{
					"name": "Luminous vivid amber to luminous vivid orange",
					"gradient": "linear-gradient(135deg,rgba(252,185,0,1) 0%,rgba(255,105,0,1) 100%)",
					"slug": "luminous-vivid-amber-to-luminous-vivid-orange"
				},
				{
					"name": "Luminous vivid orange to vivid red",
					"gradient": "linear-gradient(135deg,rgba(255,105,0,1) 0%,rgb(207,46,46) 100%)",
					"slug": "luminous-vivid-orange-to-vivid-red"
				},
				{
					"name": "Very light gray to cyan bluish gray",
					"gradient": "linear-gradient(135deg,rgb(238,238,238) 0%,rgb(169,184,195) 100%)",
					"slug": "very-light-gray-to-cyan-bluish-gray"
				},
				{
					"name": "Cool to warm spectrum",
					"gradient": "linear-gradient(135deg,rgb(74,234,220) 0%,rgb(151,120,209) 20%,rgb(207,42,186) 40%,rgb(238,44,130) 60%,rgb(251,105,98) 80%,rgb(254,248,76) 100%)",
					"slug": "cool-to-warm-spectrum"
				},
				{
					"name": "Blush light purple",
					"gradient": "linear-gradient(135deg,rgb(255,206,236) 0%,rgb(152,150,240) 100%)",
					"slug": "blush-light-purple"
				},
				{
					"name": "Blush bordeaux",
					"gradient": "linear-gradient(135deg,rgb(254,205,165) 0%,rgb(254,45,45) 50%,rgb(107,0,62) 100%)",
					"slug": "blush-bordeaux"
				},
				{
					"name": "Luminous dusk",
					"gradient": "linear-gradient(135deg,rgb(255,203,112) 0%,rgb(199,81,192) 50%,rgb(65,88,208) 100%)",
					"slug": "luminous-dusk"
				},
				{
					"name": "Pale ocean",
					"gradient": "linear-gradient(135deg,rgb(255,245,203) 0%,rgb(182,227,212) 50%,rgb(51,167,181) 100%)",
					"slug": "pale-ocean"
				},
				{
					"name": "Electric grass",
					"gradient": "linear-gradient(135deg,rgb(202,248,128) 0%,rgb(113,206,126) 100%)",
					"slug": "electric-grass"
				},
				{
					"name": "Midnight",
					"gradient": "linear-gradient(135deg,rgb(2,3,129) 0%,rgb(40,116,252) 100%)",
					"slug": "midnight"
				}
			],
			"heading": true,
			"link": false,
			"palette": [
				{
					"name": "Black",
					"slug": "black",
					"color": "#000000"
				},
				{
					"name": "Cyan bluish gray",
					"slug": "cyan-bluish-gray",
					"color": "#abb8c3"
				},
				{
					"name": "White",
					"slug": "white",
					"color": "#ffffff"
				},
				{
					"name": "Pale pink",
					"slug": "pale-pink",
					"color": "#f78da7"
				},
				{
					"name": "Vivid red",
					"slug": "vivid-red",
					"color": "#cf2e2e"
				},
				{
					"name": "Luminous vivid orange",
					"slug": "luminous-vivid-orange",
					"color": "#ff6900"
				},
				{
					"name": "Luminous vivid amber",
					"slug": "luminous-vivid-amber",
					"color": "#fcb900"
				},
				{
					"name": "Light green cyan",
					"slug": "light-green-cyan",
					"color": "#7bdcb5"
				},
				{
					"name": "Vivid green cyan",
					"slug": "vivid-green-cyan",
					"color": "#00d084"
				},
				{
					"name": "Pale cyan blue",
					"slug": "pale-cyan-blue",
					"color": "#8ed1fc"
				},
				{
					"name": "Vivid cyan blue",
					"slug": "vivid-cyan-blue",
					"color": "#0693e3"
				},
				{
					"name": "Vivid purple",
					"slug": "vivid-purple",
					"color": "#9b51e0"
				}
			],
			"text": true
		},
		"dimensions": {
			"defaultAspectRatios": true,
			"aspectRatios": [
				{
					"name": "Square - 1:1",
					"slug": "square",
					"ratio": "1"
				},
				{
					"name": "Standard - 4:3",
					"slug": "4-3",
					"ratio": "4/3"
				},
				{
					"name": "Portrait - 3:4",
					"slug": "3-4",
					"ratio": "3/4"
				},
				{
					"name": "Classic - 3:2",
					"slug": "3-2",
					"ratio": "3/2"
				},
				{
					"name": "Classic Portrait - 2:3",
					"slug": "2-3",
					"ratio": "2/3"
				},
				{
					"name": "Wide - 16:9",
					"slug": "16-9",
					"ratio": "16/9"
				},
				{
					"name": "Tall - 9:16",
					"slug": "9-16",
					"ratio": "9/16"
				}
			]
		},
		"shadow": {
			"defaultPresets": true,
			"presets": [
				{
					"name": "Natural",
					"slug": "natural",
					"shadow": "6px 6px 9px rgba(0, 0, 0, 0.2)"
				},
				{
					"name": "Deep",
					"slug": "deep",
					"shadow": "12px 12px 50px rgba(0, 0, 0, 0.4)"
				},
				{
					"name": "Sharp",
					"slug": "sharp",
					"shadow": "6px 6px 0px rgba(0, 0, 0, 0.2)"
				},
				{
					"name": "Outlined",
					"slug": "outlined",
					"shadow": "6px 6px 0px -3px rgba(255, 255, 255, 1), 6px 6px rgba(0, 0, 0, 1)"
				},
				{
					"name": "Crisp",
					"slug": "crisp",
					"shadow": "6px 6px 0px rgba(0, 0, 0, 1)"
				}
			]
		},
		"spacing": {
			"blockGap": null,
			"margin": false,
			"padding": false,
			"customSpacingSize": true,
			"defaultSpacingSizes": true,
			"units": [ "px", "em", "rem", "vh", "vw", "%" ],
			"spacingScale": {
				"operator": "*",
				"increment": 1.5,
				"steps": 7,
				"mediumStep": 1.5,
				"unit": "rem"
			}
		},
		"typography": {
			"customFontSize": true,
			"defaultFontSizes": true,
			"dropCap": true,
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
				},
				{
					"name": "Large",
					"slug": "large",
					"size": "36px"
				},
				{
					"name": "Extra Large",
					"slug": "x-large",
					"size": "42px"
				}
			],
			"fontStyle": true,
			"fontWeight": true,
			"letterSpacing": true,
			"lineHeight": false,
			"textAlign": true,
			"textDecoration": true,
			"textTransform": true,
			"writingMode": false
		},
		"blocks": {
			"core/button": {
				"border": {
					"radius": true
				}
			},
			"core/image": {
				"lightbox": {
					"allowEditing": true
				}
			},
			"core/pullquote": {
				"border": {
					"color": true,
					"radius": true,
					"style": true,
					"width": true
				}
			}
		}
	},
	"styles": {
		"blocks": {
			"core/button": {
				"variations": {
					"outline": {
						"border": {
							"width": "2px",
							"style": "solid",
							"color": "currentColor"
						},
						"color": {
							"text": "currentColor",
							"gradient": "transparent none"
						},
						"spacing": {
							"padding": {
								"top": "0.667em",
								"right": "1.33em",
								"bottom": "0.667em",
								"left": "1.33em"
							}
						}
					}
				}
			},
			"core/site-logo": {
				"variations": {
					"rounded": {
						"border": {
							"radius": "9999px"
						}
					}
				}
			}
		},
		"elements": {
			"button": {
				"color": {
					"text": "#fff",
					"background": "#32373c"
				},
				"spacing": {
					"padding": "calc(0.667em + 2px) calc(1.333em + 2px)"
				},
				"typography": {
					"fontSize": "inherit",
					"fontFamily": "inherit",
					"lineHeight": "inherit",
					"textDecoration": "none"
				},
				"border": {
					"width": "0"
				}
			},
			"link": {
				"typography": {
					"textDecoration": "underline"
				}
			}
		},
		"spacing": {
			"blockGap": "24px",
			"padding": {
				"top": "0px",
				"right": "0px",
				"bottom": "0px",
				"left": "0px"
			}
		}
	}
}

```

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