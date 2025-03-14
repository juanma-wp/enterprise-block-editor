# 5.1: Block Categories

In the WordPress block editor, blocks are organized into categories to help users find and manage them efficiently. As an enterprise WordPress developer, understanding how to create, modify, and manage block categories is crucial for enhancing the user experience and organizing custom blocks effectively.

## Understanding Block Categories

Block categories in WordPress serve as a way to group related blocks together, making it easier for users to locate and use specific blocks when building their content.

By default, WordPress provides several pre-defined categories such as Text, Media, Design, Widgets, Theme, and Embeds.

When a user makes use of the block inserter in the editor, the blocks are grouped by their categories.

As you develop custom blocks, you may find it necessary to create new categories or modify existing ones to better suit your specific needs.

## Creating Custom Block Categories

To create a custom block category, you use the `block_categories_all` PHP filter. This filter allows you to add, modify, or reorder block categories in the WordPress block editor. The callback function hooked into `block_categories_all` receives two parameters, an array of the existing `$categories`, and the current `$post` object.

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

In this example, you're adding a new category called "Enterprise Blocks" with the slug `custom-enterprise-blocks`. The `icon` parameter is optional and can be used to specify a Dashicon or custom SVG icon for the category.

## Assigning Blocks to Custom Categories

In your block's `block.json` file, you can now edit the `category` property, replacing it with `custom-enterprise-blocks`, to assign the block to that category.

```javascript
{
	"$schema": "https://schemas.wp.org/trunk/block.json",
	"apiVersion": 3,
	"name": "create-block/my-custom-block",
	"version": "0.1.0",
	"title": "My Custom Block",
	"category": "custom-enterprise-blocks",
	"icon": "smiley",
	"description": "Example block scaffolded with Create Block tool.",
}
```

Now, in the block inserter in the Editor, the "Enterprise Blocks" category appears at the bottom of the list, along with your custom block.

By assigning your custom blocks to specific categories, you create a more organized and user-friendly block insertion experience for your users.

## Reordering Block Categories

You may want to change the order in which block categories appear in the editor. This can be achieved by manipulating the array of categories that the `block_categories_all` filter passed to the callback function.

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

In this example, you're inserting the custom "Enterprise Blocks" category just before the "Design" category.

If the "Design" category isn't found, the custom category is added to the end of the list.

## Renaming Existing Block Categories

Sometimes, you might want to rename existing block categories to better fit your needs. Here's how you can modify the titles of existing categories:

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

## Contextual category restructuring

The fact that the current `$post` object is also passed to any callback function hooked into `block_categories_all` allows for even more fine-grained control.

For example, you could limit the changes only to a specific post type:

```php
add_filter( 'block_categories_all', 'restructure_block_categories', 10, 2 );

function restructure_block_categories( $categories, $post ) {
    if ( 'book' !== $post->post_type ){
        return $categories
    }
    // do something only in the context of the book post type
    return $categories;
}
```

## Conclusion

Managing block categories is an useful skill. Creating custom categories, reordering existing ones, assigning blocks to appropriate categories, and displaying specific categories on specific post types can significantly improve the user experience of the block editor for your users.
