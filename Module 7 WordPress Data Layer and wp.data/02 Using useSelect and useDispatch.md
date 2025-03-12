# 7.2: Using useSelect and useDispatch

The WordPress Data Layer provides a powerful way to manage state within block editor components. Two essential hooks, [`useSelect`](https://developer.wordpress.org/block-editor/reference-guides/packages/packages-data/#useselect) and [`useDispatch`](https://developer.wordpress.org/block-editor/reference-guides/packages/packages-data/#usedispatch), play a crucial role in fetching, managing, and updating WordPress data. This lesson will explore how to effectively use these hooks to interact with the WordPress data store.

## **`useSelect`**

The [`useSelect`](https://developer.wordpress.org/block-editor/reference-guides/packages/packages-data/#useselect) hook is a React Hook that allows you to retrieve data from the WordPress data store and subscribe to changes in that data. It's important to note that `useSelect` can only be used inside React components.

### **Fetching Data with `useSelect`**

Here's a basic example of how to use `useSelect` to fetch post data:

```javascript
import { useSelect } from "@wordpress/data";
import { store as coreDataStore } from "@wordpress/core-data";

function PostList() {
  const posts = useSelect(
    (select) => select(coreDataStore).getEntityRecords("postType", "post"),
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
}
```

In this example, `useSelect` automatically subscribes to the core data store. Any changes to the posts in the state will trigger a re-render of the component.

### **Optimizing Performance with Dependencies**

`useSelect` can receive an array of dependencies as its second argument. When any of these dependencies change, `useSelect` will re-run and return updated values:

```javascript
const { postType, postId } = props;

const post = useSelect(
  (select) =>
    select(coreDataStore).getEntityRecord("postType", postType, postId),
  [postType, postId]
);
```

This ensures that the component only re-renders when necessary, improving performance.

## **`useDispatch`**

While `useSelect` retrieves data, [`useDispatch`](https://developer.wordpress.org/block-editor/reference-guides/packages/packages-data/#usedispatch) allows you to update the data store. Like `useSelect`, `useDispatch` is a React Hook and should only be used within React components.

### **Updating Data with `useDispatch`**

Every store has some actions that can be "dispatched" to update its state. Here's an example of how to use `useDispatch` to update a post's title:

```javascript
import { useDispatch } from "@wordpress/data";
import { store as coreDataStore } from "@wordpress/core-data";

function PostEditor({ postId }) {
  const { editEntityRecord } = useDispatch(coreDataStore);

  const updateTitle = (newTitle) => {
    editEntityRecord("postType", "post", postId, { title: newTitle });
  };

  return <input type="text" onChange={(e) => updateTitle(e.target.value)} />;
}
```

In this example, `useDispatch` provides the `editEntityRecord` action, which we use to update the post title.

## **Combining `useSelect` and `useDispatch`**

Often, you'll need to both fetch and update data within the same component. Here's an example that demonstrates this:

```javascript
import { useSelect, useDispatch } from "@wordpress/data";
import { store as coreDataStore } from "@wordpress/core-data";

function PostEditor({ postId }) {
  const post = useSelect(
    (select) =>
      select(coreDataStore).getEntityRecord("postType", "post", postId),
    [postId]
  );

  const { editEntityRecord } = useDispatch(coreDataStore);

  if (!post) {
    return <p>Loading post...</p>;
  }

  const updateTitle = (newTitle) => {
    editEntityRecord("postType", "post", postId, { title: newTitle });
  };

  return (
    <div>
      <h1>{post.title.rendered}</h1>
      <input
        type="text"
        value={post.title.raw}
        onChange={(e) => updateTitle(e.target.value)}
      />
    </div>
  );
}
```

This component fetches a post using `useSelect`, displays its title, and provides an input to edit the title using `useDispatch`.

## **Subscribing to Store Changes**

The `useSelect` hook in `@wordpress/data` automatically subscribes to relevant state changes, ensuring the returned results stay updated.

Alternatively, the [`subscribe`](https://developer.wordpress.org/block-editor/reference-guides/packages/packages-data/#subscribe) function allows manual state change tracking by registering a listener, which is triggered whenever a registered store's state updates. If a specific store is provided (as a second argument), the listener fires only for that store.

The function returns an `unsubscribe` method to stop tracking when needed.

```javascript
import { subscribe } from "@wordpress/data";
import { store as coreDataStore } from "@wordpress/core-data";

const unsubscribeRegistry = subscribe(() => {
  // Any change in the state on any registered store will trigger this callback
});

const unsubscribeCoreDataStore = subscribe(() => {
  // Only changes in the state of coreDataStore will trigger this callback
}, coreDataStore);

// Later, if necessary...
unsubscribeRegistry();
unsubscribeCoreDataStore();
```

---

`useSelect` and `useDispatch` are widely used tools for managing state in WordPress block editor components:

- `useSelect` automatically subscribes to relevant stores and triggers re-renders when data changes.
- Use the dependency array in `useSelect` to optimize performance and control when data is re-fetched.
- `useDispatch` provides actions to update the data store.
- Both hooks are for use within React components only.
- Combining `useSelect` and `useDispatch` allows for powerful, state-aware components.
