# 7.3: Fetching Data with wp.data

The WordPress Data Layer, powered by `wp.data`, provides a structured way to interact with WordPress data (available through the REST API). The [`@wordpress/core-data`](https://developer.wordpress.org/block-editor/reference-guides/packages/packages-core-data/) package simplifies querying and manipulating entities such as posts, terms, and users by offering hooks and selectors that enable fetching this type of data efficiently within the Block Editor.

## Querying Posts, Terms, and Users with `@wordpress/core-data`

The [`@wordpress/core-data`](https://developer.wordpress.org/block-editor/reference-guides/packages/packages-core-data/) package introduces several hooks and actions to retrieve and update data related to core WordPress entities.

[`getEntityRecord`](https://developer.wordpress.org/block-editor/reference-guides/data/data-core/#getentityrecord) and [`getEntityRecords`](https://developer.wordpress.org/block-editor/reference-guides/data/data-core/#getentityrecords) are two selectors available on the "core" store that allow in combination with other selectors such as [`hasFinishedResolution`](https://developer.wordpress.org/block-editor/reference-guides/packages/packages-data/#hasfinishedresolution) or [`isResolving`](https://developer.wordpress.org/block-editor/reference-guides/packages/packages-data/#isresolving) to efficiently fetch and track the loading state of WordPress entities.

```javascript
const data = useSelect((select) => {
  return {
    pages: select(coreDataStore).getEntityRecords("postType", "page", {
      per_page: 20,
    }),
    hasResolved: select(coreDataStore).hasFinishedResolution(
      "getEntityRecords",
      ["postType", "page", { per_page: 20 }]
    ),
  };
}, []);
```

The `@wordpress/core-data` package also provides the [`useEntityRecords`](https://developer.wordpress.org/block-editor/reference-guides/packages/packages-core-data/#useentityrecords) and [`useEntityRecord`](https://developer.wordpress.org/block-editor/reference-guides/packages/packages-core-data/#useentityrecord) hooks, making it easier to fetch and manipulate WordPress entities.

### `useEntityRecord`: Fetching Single Records

The [`useEntityRecord`](https://developer.wordpress.org/block-editor/reference-guides/packages/packages-core-data/#useentityrecord) hook is used to fetch a single record of a given entity type. This is useful when you need to retrieve a specific post, page, or other WordPress entity.

**Example: Fetching a Single Post**

```javascript
import { useEntityRecord } from "@wordpress/core-data";

const SinglePost = ({ postId }) => {
  const { record: post, isLoading } = useEntityRecord(
    "postType",
    "post",
    postId
  );

  if (isLoading) {
    return <p>Loading post...</p>;
  }

  if (!post) {
    return <p>Post not found</p>;
  }

  return (
    <article>
      <h2>{post.title.rendered}</h2>
      <div dangerouslySetInnerHTML={{ __html: post.content.rendered }} />
      <p>Last modified: {new Date(post.modified).toLocaleDateString()}</p>
    </article>
  );
};
```

### `useEntityRecords`: Fetching Multiple Records

The [`useEntityRecords`](https://developer.wordpress.org/block-editor/reference-guides/packages/packages-core-data/#useentityrecords) hook is used to fetch multiple records of a given entity type, such as posts, pages, categories, or users.

**Example: Fetching Posts**

```javascript
import { useEntityRecords } from "@wordpress/core-data";

const MyPostList = () => {
  const { records: posts, isLoading } = useEntityRecords("postType", "post");

  if (isLoading) {
    return <p>Loading posts...</p>;
  }

  return (
    <ul>
      {posts.map((post) => (
        <li key={post.id}>{post.title.rendered}</li>
      ))}
    </ul>
  );
};
```

This method simplifies querying posts compared to manually dispatching selectors.

### Comparing `useEntityRecords` with `useSelect`

While `useEntityRecords` provides a simplified approach to fetching data, the `useSelect` hook offers more flexibility and allows direct interaction with `wp.data` selectors. In most of the cases the use of `useEntityRecords` is recommended.

**Example: Fetching Posts with `useSelect`**

```javascript
import { useSelect } from "@wordpress/data";

const MyPostList = () => {
  const posts = useSelect(
    (select) =>
      select("core").getEntityRecords("postType", "post", { per_page: 5 }),
    []
  );

  if (!posts) {
    return <p>Loading posts...</p>;
  }

  return (
    <ul>
      {posts.map((post) => (
        <li key={post.id}>{post.title.rendered}</li>
      ))}
    </ul>
  );
};
```

**Key Differences**

| Feature        | `useEntityRecords`                                                  | `useSelect`                                    |
| -------------- | ------------------------------------------------------------------- | ---------------------------------------------- |
| API Simplicity | Easier to use, returns structured response (`records`, `isLoading`) | Requires using `select()` manually             |
| Customization  | Supports direct filtering via query arguments                       | Full flexibility using `wp.data` selectors     |
| Re-rendering   | Handles re-fetching automatically                                   | Needs dependency array to optimize performance |

## Customizing Queries for Specific Use Cases

The `useEntityRecords` hook allows for customization by passing query parameters.

**Example: Fetching Only Published Posts**

```javascript
const { records: posts } = useEntityRecords("postType", "post", {
  status: "publish",
  per_page: 5,
});
```

**Example: Fetching Posts by a Specific Author**

```javascript
const { records: posts } = useEntityRecords("postType", "post", {
  author: 1,
});
```

**Example: Fetching Posts with a Specific Category**

```javascript
const { records: posts } = useEntityRecords("postType", "post", {
  categories: [10],
});
```

## WordPress Data Stores and Popular Selectors

The `wp.data` module provides access to multiple data stores, each with its own selectors. For a complete reference, see the [WordPress Data Layer Reference](https://developer.wordpress.org/block-editor/reference-guides/data/).

### The `core` Store

The [`core` store](https://developer.wordpress.org/block-editor/reference-guides/data/data-core/) is used for retrieving WordPress site-wide data.

**Popular Selectors:**

- [`getEntityRecords`](https://developer.wordpress.org/block-editor/reference-guides/data/data-core/#getentityrecords) - Fetches posts (ex: `getEntityRecords( 'postType', 'post', query )`).
- [`getEntityRecord`](https://developer.wordpress.org/block-editor/reference-guides/data/data-core/#getentityrecord) - Fetches a specific post (ex: `getEntityRecord( 'postType', 'post', id )`).
- [`getUser`](https://developer.wordpress.org/block-editor/reference-guides/data/data-core/#getuser) - Fetches a specific user (ex: `getUser( id )`).
- [`getTaxonomy`](https://developer.wordpress.org/block-editor/reference-guides/data/data-core/#gettaxonomy) - Fetches a taxonomy (ex: `getTaxonomy( slug )` ).

### The `core/block-directory` Store

Used to interact with the WordPress Block Directory. See the [Block Directory Store Reference](https://developer.wordpress.org/block-editor/reference-guides/data/data-core-block-directory/).

**Popular Selectors:**

- [`getBlockTypes`](https://developer.wordpress.org/block-editor/reference-guides/data/data-core-blocks/#getblocktypes) - Fetches registered block types.
- [`getCategories`](https://developer.wordpress.org/block-editor/reference-guides/data/data-core-blocks/#getcategories) - Fetches block categories.

### The `core/blocks` Store

Handles block-related data, including registered blocks and block variations. See the [Blocks Store Reference](https://developer.wordpress.org/block-editor/reference-guides/data/data-core-blocks/).

**Popular Selectors:**

- [`getBlockType`](https://developer.wordpress.org/block-editor/reference-guides/data/data-core-blocks/#getblocktype) - Fetches a block type definition.
- [`getBlockVariations`](https://developer.wordpress.org/block-editor/reference-guides/data/data-core-blocks/#getblockvariations) - Fetches variations of a block type.

### The `core/block-editor` Store

Handles editor-related data, including the block list and selection. See the [Block Editor Store Reference](https://developer.wordpress.org/block-editor/reference-guides/data/data-core-block-editor/).

**Popular Selectors:**

- [`getSelectedBlock`](https://developer.wordpress.org/block-editor/reference-guides/data/data-core-block-editor/#getselectedblock) - Retrieves the currently selected block.
- [`getBlocks`](https://developer.wordpress.org/block-editor/reference-guides/data/data-core-block-editor/#getblocks) - Fetches all blocks in the editor.

## Fetching and Displaying Data in a Custom Block

We can integrate `wp.data` selectors into the `Edit` component of any block (that will be rendered for that block in the Block Editor).

**Example: Custom Block Displaying Latest Posts**

```javascript
import { useEntityRecords } from "@wordpress/core-data";
import { useBlockProps } from "@wordpress/block-editor";

const Edit = () => {
  const blockProps = useBlockProps();
  const { records: posts, isLoading } = useEntityRecords("postType", "post", {
    per_page: 5,
    order: "desc",
    orderby: "date",
  });

  return (
    <div {...blockProps}>
      {isLoading ? (
        <p>Loading...</p>
      ) : (
        <ul>
          {posts.map((post) => (
            <li key={post.id}>{post.title.rendered}</li>
          ))}
        </ul>
      )}
    </div>
  );
};

export default Edit;
```

---

The `wp.data` module and `@wordpress/core-data` package significantly simplify fetching and managing WordPress data. By leveraging hooks like `useEntityRecords`, developers can efficiently retrieve posts, terms, and users while customizing queries for specific needs. Additionally, `useSelect` provides greater flexibility when needed, allowing deeper interaction with the `wp.data` store for customized queries and performance optimization.
