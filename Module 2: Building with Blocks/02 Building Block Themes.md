## Building Block Themes

### Building and Extending Block Themes with Theme.json

The introduction of the `theme.json` file in WordPress 5.8 revolutionized theme development by providing a centralized configuration mechanism for global styles and settings. This file is essential for creating and customizing block themes, as it allows developers to define typography, colors, layout options, and block-specific settings programmatically.

#### Understanding Theme.json Structure

The `theme.json` file is located in the root directory of a block theme. It consists of two primary sections: `settings` and `styles`. These sections can be defined globally (applying to the entire site) or specifically for individual blocks.

Example of a basic `theme.json` file:

```json
{
  "$schema": "https://schemas.wp.org/trunk/theme.json",
  "version": 2,
  "settings": {
    "color": {
      "palette": [
        {
          "slug": "primary",
          "color": "#0073aa",
          "name": "Primary"
        },
        {
          "slug": "secondary",
          "color": "#005177",
          "name": "Secondary"
        }
      ]
    },
    "typography": {
      "fontSizes": [
        {
          "slug": "small",
          "size": 12,
          "name": "Small"
        },
        {
          "slug": "large",
          "size": 36,
          "name": "Large"
        }
      ]
    }
  },
  "styles": {
    "blocks": {
      "core/heading": {
        "typography": {
          "fontSize": "36px"
        }
      }
    }
  }
}
```

This example demonstrates how to define global color palettes, typography settings, and block-specific styles. The `$schema` key provides inline documentation and auto-completion support in code editors.

#### Extending Theme.json for Customization

You can use the `blocks` property within `theme.json` to apply granular control over specific blocks. For example, you can restrict customization options or enforce design consistency across your theme.

To disable text color customization globally but allow it for headings:

```json
{
  "settings": {
    "color": {
      "custom": false
    },
    "blocks": {
      "core/heading": {
        "color": {
          "custom": true
        }
      }
    }
  }
}
```

This configuration ensures that only heading blocks can have custom text colors, maintaining design consistency elsewhere.

### Creating Custom Templates, Template Parts, and Patterns

Block themes enable developers to create custom templates, template parts, and patterns that streamline site design and editing.

#### Custom Templates

Templates define the structure of specific pages or post types. To create a custom template:

1. Navigate to **Appearance → Editor → Templates** in the WordPress dashboard.
2. Click the **+** icon to add a new template.
3. Choose a template type (e.g., single post) or create a custom template.
4. Design your layout using blocks such as headers, footers, and content areas.

Example: A custom single post template might include a header block, content block, and footer block.

#### Template Parts

Template parts are reusable components like headers or footers. You can create custom template parts by:

1. Selecting **Template Parts** in the Site Editor.
2. Adding a new part (e.g., Header).
3. Designing the part with blocks such as navigation menus or logos.

Template parts can be used across multiple templates for consistency.

#### Block Patterns

Block patterns are predefined layouts that users can insert into pages or posts. To register a custom block pattern:

Add the following code to your theme's `functions.php` file:

```php
add_action('init', 'register_custom_block_pattern');

function register_custom_block_pattern() {
    register_block_pattern(
        'mytheme/hero-section',
        array(
            'title'       => __('Hero Section', 'mytheme'),
            'description' => __('A hero section with a heading and call-to-action button.', 'mytheme'),
            'content'     => 'Welcome!Click Here'
        )
    );
}
```

This registers a reusable hero section pattern that users can add via the block editor.

### Creating Child Themes

Child themes allow developers to extend parent themes without modifying their original files, ensuring updates do not overwrite customizations.

#### Steps to Create a Child Theme for Block Themes

1. **Create a Child Theme Directory**  
   In `wp-content/themes/`, create a directory named `parenttheme-child`.

2. **Add Required Files**
    - Create a `style.css` file with the following content:
      ```css
      /*
      Theme Name: ParentTheme Child
      Template: parenttheme
      */
      ```
    - Copy the parent theme's `theme.json` file into the child theme directory.

3. **Enqueue Parent Styles**  
   Add this code to the child theme's `functions.php` file:
   ```php
   function enqueue_parent_styles() {
       wp_enqueue_style('parent-style', get_template_directory_uri() . '/style.css');
   }
   add_action('wp_enqueue_scripts', 'enqueue_parent_styles');
   ```

4. **Activate the Child Theme**  
   Go to **Appearance → Themes** in the WordPress dashboard and activate your child theme.

#### Customizing Child Themes

You can modify the child theme's `theme.json` file to override or extend parent settings. For example, adding new color options:

```json
{
  "$schema": "https://schemas.wp.org/trunk/theme.json",
  "version": 2,
  "settings": {
    "color": {
      "palette": [
        {
          "slug": "tertiary",
          "color": "#ff6600",
          "name": "Tertiary"
        }
      ]
    }
  }
}
```

Child themes offer flexibility while preserving parent functionality, making them ideal for advanced customizations.

---

By mastering these techniques—leveraging `theme.json`, creating templates and patterns, and developing child themes—you can build robust block themes tailored for enterprise WordPress development.

Citations:
[1] https://github.co
[2] https://kinsta.com/blog/theme-json-blocks-property/
[3] https://jetpack.com/resources/wordpress-theme-json/
[4] https://gutenberg.10up.com/reference/Themes/theme-json/
[5] https://jetpack.com/resources/wordpress-block-templates/
[6] https://developer.wordpress.org/themes/features/block-patterns/
[7] https://crocoblock.com/blog/wordpress-child-theme-foundations/
[8] https://support.yithemes.com/hc/en-us/articles/215945088-Creating-a-WordPress-Child-Theme-Classic-and-Block
[9] https://wpvip.com/blog/using-a-design-system-with-the-wordpress-block-editor-pt-1-theme-json/
[10] https://learn.wordpress.org/lesson-plan/create-a-basic-child-theme-for-block-themes/
[11] https://jetpack.com/resources/wordpress-block-themes/
[12] https://fullsiteediting.com/lessons/theme-json-layout-and-spacing-options/
[13] https://kinsta.com/blog/theme-json/
[14] https://fullstackdigital.io/blog/styling-themes-and-blocks-with-the-wordpress-theme-json-file/
[15] https://www.youtube.com/watch?v=n7zQdmXpPwI
[16] https://learn.wordpress.org/tutorial/introduction-to-theme-json/
[17] https://www.elmastudio.de/en/theme-json-for-wordpress-block-themes-explained/
[18] https://fullsiteediting.com/lessons/creating-theme-json/
[19] https://developer.wordpress.org/block-editor/how-to-guides/themes/global-settings-and-styles/
[20] https://developer.wordpress.org/block-editor/reference-guides/theme-json-reference/
[21] https://samhermes.com/posts/slowly-backing-away-from-theme-json/
[22] https://www.youtube.com/watch?v=x7vrYmP6VTM
[23] https://seahawkmedia.com/wordpress/theme-json-guide/
[24] https://css-tricks.com/understanding-gutenberg-blocks-patterns-and-templates/
[25] https://wordpress.org/documentation/article/template-part-block/
[26] https://developer.wordpress.org/themes/patterns/
[27] https://gutenberghub.com/how-to-create-a-template-part-in-wordpress-block-editor/
[28] https://www.wpzoom.com/docs/the-difference-between-reusable-blocks-block-patterns-templates-template-parts/
[29] https://www.reddit.com/r/ProWordPress/comments/1atyvp1/creating_slightly_customizable_bespoke_block/
[30] https://maxiblocks.com/wordpress-block-themes-3/
[31] https://www.youtube.com/watch?v=ljyRP3ozcnE
[32] https://www.godaddy.com/resources/skills/wordpress-block-pattern-reusable-block-or-template-part
[33] https://wordpress.com/support/template-part-block/
[34] https://www.youtube.com/watch?v=hXq6pp7IvIE
[35] https://developer.woocommerce.com/docs/how-to-set-up-and-use-a-child-theme/
[36] https://www.linkedin.com/pulse/mini-guide-wordpress-child-themes-allie-nimmons-utquc
[37] https://www.youtube.com/watch?v=m9A3HZ2cuZM
[38] https://www.youtube.com/watch?v=VE_3wI65EIc
[39] https://developer.wordpress.org/themes/advanced-topics/child-themes/
[40] https://www.reddit.com/r/Wordpress/comments/144ecxs/how_to_create_a_child_theme_out_of_a_block_theme/
[41] https://community.localwp.com/t/how-to-create-a-child-theme-for-a-block-theme/31894
[42] https://fullsiteediting.com/lessons/creating-block-based-themes/
[43] https://developer.wordpress.org/themes/global-settings-and-styles/settings/blocks/
[44] https://torquemag.io/2023/06/modify-wordpress-block-themes/
[45] https://www.pootlepress.com/2023/06/the-complete-newbies-guide-to-theme-json-in-wordpress/
[46] https://gutenbergtimes.com/block-based-template-parts-for-classic-themes/
[47] https://motopress.com/blog/wordpress-patterns-vs-template-parts/
[48] https://developer.wordpress.org/themes/templates/template-parts/
[49] https://fullsiteediting.com/lessons/templates-and-template-parts/
[50] https://learn.wordpress.org/tutorial/using-block-template-parts-in-classic-themes/
[51] https://www.wpbeginner.com/wp-themes/how-to-create-a-wordpress-child-theme-video/
[52] https://wordpress.com/support/themes/child-themes/
[53] https://www.godaddy.com/resources/skills/how-to-create-a-block-child-theme-in-wordpress
[54] https://fullsiteediting.com/lessons/child-themes/

---
Answer from Perplexity: pplx.ai/share