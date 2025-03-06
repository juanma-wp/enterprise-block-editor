# 5.1: Block Categories

In the WordPress block editor, blocks are organized into categories to help users find and manage them efficiently. As an enterprise WordPress developer, understanding how to create, modify, and manage block categories is crucial for enhancing the user experience and organizing custom blocks effectively.

## Understanding Block Categories

Block categories in WordPress serve as a way to group related blocks together, making it easier for users to locate and use specific blocks when building their content. By default, WordPress provides several pre-defined categories such as Text, Media, Design, Widgets, Theme, and Embeds.

However, as you develop custom blocks for enterprise solutions, you may find it necessary to create new categories or modify existing ones to better suit your specific needs.

## Creating Custom Block Categories

To create a custom block category, you'll need to use the `block_categories_all` filter. This filter allows you to add, modify, or reorder block categories in the WordPress block editor.

Here's an example of how to create a new block category:

```php
add_filter( 'block_categories_all', 'add_custom_block_category', 10, 2 );

function add_custom_block_category( $categories, $post ) {
    return array_merge(
        $categories,
        array(
            array(
                'slug'  => 'custom-enterprise-blocks',
                'title' => __( 'Enterprise Blocks', 'text-domain' ),
                'icon'  => 'building',
            ),
        )
    );
}
```

In this example, we're adding a new category called "Enterprise Blocks" with the slug `custom-enterprise-blocks`. The `icon` parameter is optional and can be used to specify a Dashicon or custom SVG icon for the category.

## Reordering Block Categories

You may want to change the order in which block categories appear in the editor. This can be achieved by manipulating the array of categories within the `block_categories_all` filter.

Here's an example of how to reorder categories:

```php
add_filter( 'block_categories_all', 'reorder_block_categories', 10, 2 );

function reorder_block_categories( $categories, $post ) {
    $custom_category = array(
        'slug'  => 'custom-enterprise-blocks',
        'title' => __( 'Enterprise Blocks', 'text-domain' ),
        'icon'  => 'building',
    );

    $category_slugs = array_column( $categories, 'slug' );
    $position = array_search( 'design', $category_slugs );

    if ( false !== $position ) {
        array_splice( $categories, $position, 0, array( $custom_category ) );
    } else {
        $categories[] = $custom_category;
    }

    return $categories;
}
```

In this example, we're inserting our custom "Enterprise Blocks" category just before the "Design" category. If the "Design" category isn't found, the custom category is added to the end of the list.

## Renaming Existing Block Categories

Sometimes, you might want to rename existing block categories to better fit your enterprise needs. Here's how you can modify the titles of existing categories:

```php
add_filter( 'block_categories_all', 'rename_block_categories', 10, 2 );

function rename_block_categories( $categories, $post ) {
    return array_map( function( $category ) {
        if ( 'widgets' === $category['slug'] ) {
            $category['title'] = __( 'Enterprise Widgets', 'text-domain' );
        }
        return $category;
    }, $categories );
}
```

This example renames the "Widgets" category to "Enterprise Widgets".

## Assigning Blocks to Custom Categories

Once you've created your custom categories, you'll want to assign your custom blocks to them. This is done in the block registration process, either in your `block.json` file or in the JavaScript code that registers the block.

In your `block.json` file:

```json
{
    "apiVersion": 2,
    "name": "custom-namespace/enterprise-block",
    "title": "Enterprise Block",
    "category": "custom-enterprise-blocks",
    "icon": "building",
    "description": "A custom block for enterprise solutions",
    "version": "1.0.0",
    "textdomain": "custom-namespace",
    "editorScript": "file:./build/index.js",
    "editorStyle": "file:./build/index.css",
    "style": "file:./build/style-index.css"
}
```

Or in your JavaScript code:

```javascript
registerBlockType( 'custom-namespace/enterprise-block', {
    title: __( 'Enterprise Block', 'custom-namespace' ),
    icon: 'building',
    category: 'custom-enterprise-blocks',
    // ... other block properties
} );
```

By assigning your custom blocks to specific categories, you create a more organized and user-friendly block insertion experience for your enterprise users.

## Conclusion

Managing block categories is an essential skill for enterprise WordPress developers. By creating custom categories, reordering existing ones, and assigning blocks to appropriate categories, you can significantly improve the user experience of the block editor for your enterprise clients.

Remember to consider the specific needs of your enterprise users when organizing blocks and categories. A well-structured block library can greatly enhance productivity and make content creation more intuitive for your clients.

Citations:
[1] https://github.co
[2] https://developer.wordpress.org/news/2025/01/one-hook-to-rule-them-all-the-many-faces-of-block-categories/
[3] https://gutenberghub.com/how-to-create-custom-block-category/
[4] https://rudrastyh.com/gutenberg/create-custom-block-categories.html
[5] https://www.elegantthemes.com/blog/wordpress/categories-block
[6] https://wordpress.com/support/wordpress-editor/blocks/categories-list-block/
[7] https://wordpress.org/support/topic/custom-html-block-for-specific-pages-categories/
[8] https://matty.blog/building-a-terms-list-block-for-wordpress/
[9] https://essential-blocks.com/categorize-and-filter-patterns/
[10] https://make.wordpress.org/core/2023/07/10/block-library/
[11] https://fullsiteediting.com/lessons/introduction-to-block-patterns/
[12] https://www.youtube.com/watch?v=VitOhM_05WQ
[13] https://www.godaddy.com/how-to/wordpress-for-beginners/how-to-use-the-block-library-in-wordpress
[14] https://pilotdigital.com/blog/wordpress-basics-learn-about-block-editing-in-wordpress/
[15] https://wordpress.com/forums/topic/block-of-blog-posts-to-only-show-certain-categories/
[16] https://wptavern.com/yet-another-wordpress-block-library-plugin
[17] https://wpmarmite.com/en/wordpress-block-pattern/
[18] https://www.youtube.com/watch?v=mOXODrSkgO0
[19] https://wordpress.com/support/posts/categories/
[20] https://www.joomunited.com/news/organizing-wordpress-with-categories-and-tags
[21] https://learn.wordpress.org/lesson/overview-of-wordpress-block-theme-terms-and-hierarchy/
[22] https://www.youtube.com/watch?v=fUNrID1N9s4
[23] https://wordpress.com/support/wordpress-editor/blocks/
[24] https://wordpress.org/support/topic/block-theme-and-subcategories/
[25] https://zapier.com/blog/wordpress-blocks/
[26] https://wordpress.stackexchange.com/questions/411464/template-for-posts-of-category-in-block-theme
[27] https://wordpress.stackexchange.com/questions/73587/how-to-structure-wordpress-for-an-organization-and-its-departments
[28] https://www.youtube.com/watch?v=g1H52lfgPds
[29] https://stackoverflow.com/questions/54185278/how-to-list-and-re-arrange-or-manipulate-gutenberg-block-categories-in-wordpress
[30] https://essential-blocks.com/how-to-rename-group-blocks-in-gutenberg/
[31] https://www.wpbeginner.com/plugins/how-to-change-the-category-order-in-wordpress/
[32] https://wptavern.com/gutenberg-16-9-lets-you-rename-almost-any-block-adds-experimental-form-and-input-blocks
[33] https://github.com/WordPress/developer-blog-content/issues/343
[34] https://wordpress.org/documentation/article/categories-block/
[35] https://developer.wordpress.org/reference/hooks/block_categories/
[36] https://qodeinteractive.com/magazine/how-to-reorder-categories-in-wordpress/
[37] https://wordpress.stackexchange.com/questions/373676/how-change-a-core-block-label
[38] https://wordpress.org/support/topic/how-to-change-the-categories-order/
[39] https://x.com/WordPress/status/1894474447966347705
[40] https://wpfrank.com/how-to-change-the-category-order-in-wordpress/
[41] https://www.zettahost.com/wordpress-tutorials/wordpress-block-library/
[42] https://www.youtube.com/watch?v=SsJx0szYZZk
[43] https://developer.wordpress.org/block-editor/reference-guides/packages/packages-block-library/
[44] https://weplugins.com/basics-of-block-editor-wordpress/
[45] https://taxopress.com/what-are-wordpress-categories/
[46] https://www.wpbeginner.com/beginners-guide/how-to-add-categories-and-subcategories-to-wordpress/
[47] https://solidwp.com/tutorials/using-wordpress-categories-and-tags/
[48] https://ultimateblocks.com/how-to-use-the-wordpress-categories/