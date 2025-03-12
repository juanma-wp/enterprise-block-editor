# **Fetching Data with wp.data**

The WordPress Data Layer, powered by `wp.data`, provides a structured way to interact with WordPress data (available through the REST API). The `@wordpress/core-data` package simplifies querying and manipulating entities such as posts, terms, and users by offering hooks and selectors that enable fetching this type of data efficiently within the Block Editor.

By the end of this lesson, you will be able to:

- Identify key selectors and stores in `wp.data` for querying posts, terms, and users.
- Customize queries using arguments and filters.
- Fetch and display dynamic data in a custom block or plugin using `wp.data` selectors and resolvers.

## **Querying Posts, Terms, and Users with `@wordpress/core-data`**

The `@wordpress/core-data` package introduces several and actions to retrieve and update data related to of core WordPress entities.

`getEntityRecords and getEntityRecords` are two selectors available on the “core” store that allow In combination with other selectors such as `hasFinishedResolution or`

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

the `useEntityRecords` and `useEntityRecord` hooks, making it easier to fetch and manipulate WordPress entities.

### **`useEntityRecords`: Fetching Multiple Records**

The `useEntityRecords` hook is used to fetch multiple records of a given entity type, such as posts, pages, categories, or users.

#### **Example: Fetching Posts**

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

### **Comparing `useEntityRecords` with `useSelect`**

While `useEntityRecords` provides a simplified approach to fetching data, the `useSelect` hook offers more flexibility and allows direct interaction with `wp.data` selectors.

#### **Example: Fetching Posts with `useSelect`**

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

###

### **Key Differences**

| Feature        | `useEntityRecords`                                                  | `useSelect`                                    |
| -------------- | ------------------------------------------------------------------- | ---------------------------------------------- |
| API Simplicity | Easier to use, returns structured response (`records`, `isLoading`) | Requires using `select()` manually             |
| Customization  | Supports direct filtering via query arguments                       | Full flexibility using `wp.data` selectors     |
| Re-rendering   | Handles re-fetching automatically                                   | Needs dependency array to optimize performance |

## **Customizing Queries for Specific Use Cases**

The `useEntityRecords` hook allows for customization by passing query parameters.

#### **Example: Fetching Only Published Posts**

```javascript
const { records: posts } = useEntityRecords("postType", "post", {
  status: "publish",
  per_page: 5,
});
```

#### **Example: Fetching Posts by a Specific Author**

```javascript
const { records: posts } = useEntityRecords("postType", "post", {
  author: 1,
});
```

#### **Example: Fetching Posts with a Specific Category**

```javascript
const { records: posts } = useEntityRecords("postType", "post", {
  categories: [10],
});
```

## **Exploring WordPress Data Stores and Popular Selectors**

The `wp.data` module provides multiple data stores, each with its own selectors.

### **The `core` Store**

The `core` store is used for retrieving WordPress site-wide data.

#### **Popular Selectors:**

- `getEntityRecords( 'postType', 'post', query )` \- Fetches posts.
- `getEntityRecord( 'postType', 'post', id )` \- Fetches a specific post.
- `getUser( id )` \- Fetches a specific user.
- `getTaxonomy( slug )` \- Fetches a taxonomy.

### **The `core/block-directory` Store**

Used to interact with the WordPress Block Directory.

#### **Popular Selectors:**

- `getBlockTypes()` \- Fetches registered block types.
- `getCategories()` \- Fetches block categories.

### **The `core/blocks` Store**

Handles block-related data, including registered blocks and block variations.

#### **Popular Selectors:**

- `getBlockType( name )` \- Fetches a block type definition.
- `getBlockVariations( name )` \- Fetches variations of a block type.

### **The `core/block-editor` Store**

Handles editor-related data, including the block list and selection.

#### **Popular Selectors:**

- `getSelectedBlock()` \- Retrieves the currently selected block.
- `getBlocks()` \- Fetches all blocks in the editor.

## **Fetching and Displaying Data in a Custom Block**

We can integrate `wp.data` selectors into a block component.

#### **Example: Custom Block Displaying Latest Posts**

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

## **Conclusion**

The `wp.data` module and `@wordpress/core-data` package significantly simplify fetching and managing WordPress data. By leveraging hooks like `useEntityRecords`, developers can efficiently retrieve posts, terms, and users while customizing queries for specific needs. Additionally, `useSelect` provides greater flexibility when needed, allowing deeper interaction with the `wp.data` store for customized queries and performance optimization.
