# 7.1: Introduction to the WordPress Data Layer

The WordPress Data Layer, powered by the [`@wordpress/data`](https://developer.wordpress.org/block-editor/reference-guides/packages/packages-data/) package, provides a centralized system for managing state within the Block Editor. This system allows developers to retrieve, modify, and interact with various aspects of WordPress content dynamically. This lesson focuses on how to leverage the existing WordPress data stores to access and manipulate editor data efficiently.

## **The @wordpress/data Package**

The [`@wordpress/data`](https://developer.wordpress.org/block-editor/reference-guides/packages/packages-data/) package is the foundation of state management in the WordPress Block Editor. It enables developers to:

- Retrieve data using **selectors**
- Modify state using **actions**
- Manage asynchronous operations and subscriptions

This package provides a consistent way to interact with editor-related data, such as retrieving the current post content or updating a block's attributes.

## **Core Data Stores**

WordPress provides several pre-registered data stores that manage different aspects of the Block Editor. These stores are available globally through `wp.data` methods and can be accessed without additional setup.

Most relevant data stores in the Block Editor are:

- [`core`](https://developer.wordpress.org/block-editor/reference-guides/data/data-core/) – Manages general WordPress data, including posts, users, taxonomies, and site settings.
- [`core/block-directory`](https://developer.wordpress.org/block-editor/reference-guides/data/data-core-block-directory/) - Block directory
- [`core/block-editor`](https://developer.wordpress.org/block-editor/reference-guides/data/data-core-block-editor/) – Manages the state of blocks within the editor.
- [`core/blocks`](https://developer.wordpress.org/block-editor/reference-guides/data/data-core-blocks/) - Block Types Data
- [`core/editor`](https://developer.wordpress.org/block-editor/reference-guides/data/data-core-editor/) - The Editor's Data

Each store is identified by a namespace and provides a set of selectors and actions to interact with its state. Developers can use `wp.data.select()` to retrieve data and `wp.data.dispatch()` to update data.

For example:

```javascript
> wp.data.select( 'core' ).getEntityRecord( 'postType', 'post', 1 ).title
{raw: 'Hello world!', rendered: 'Hello world!'}

```

## **Using selectors to retrieve data**

Selectors allow developers to retrieve specific pieces of state from a store. They are accessed using `wp.data.select( storeName )`.

**Example: Retrieving the Current Post Title**

To get the title of the post being edited:

```javascript
> const postTitle = wp.data.select( 'core/editor' ).getEditedPostAttribute( 'title' );
> console.log( postTitle );
```

This function call retrieves the `title` attribute of the post being edited from the `core/editor` store.

**Example: Retrieving the Current Blocks**

To get an array of blocks present in the editor:

```javascript
> const blocks = wp.data.select( 'core/block-editor' ).getBlocks();
> console.log( blocks );
```

This allows developers to dynamically inspect the content of the editor and manipulate blocks accordingly.

## **Using Actions to Modify Data**

Actions allow developers to modify the state within a store. They are accessed using `wp.data.dispatch( storeName )`.

**Example: Updating the Post Title**

To update the title of the current post:

```javascript
> wp.data.dispatch( 'core/editor' ).editPost( { title: 'New Title' } );
```

This function modifies the post's title in the `core/editor` store, which reflects immediately in the editor interface.

**Example: Inserting a New Block**

To insert a paragraph block at the end of the editor:

```javascript
> const newBlock = wp.blocks.createBlock( 'core/paragraph', { content: 'Hello World' } )> wp.data.dispatch( 'core/block-editor' ).insertBlocks( newBlock );
```

This function inserts a new paragraph block with predefined content.

## **Combining Selectors and Actions**

Developers often use selectors to retrieve data and then use actions to modify it. For example, to duplicate the first block in the editor:

```javascript
> const firstBlock = wp.data.select( 'core/block-editor' ).getBlocks()[0];
> if ( firstBlock ) wp.data.dispatch( 'core/block-editor' ).insertBlocks( firstBlock, 1 );
```

This retrieves the first block in the editor and inserts a copy of it after its original position.

---

The WordPress Data Layer provides a powerful way to interact with the Block Editor's state. By using `wp.data.select()` to retrieve data and `wp.data.dispatch()` to modify data, developers can create dynamic, responsive experiences in the editor without direct DOM manipulation.
