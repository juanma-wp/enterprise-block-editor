# Managing Editor Performance

In this lesson, we will explore the critical aspects of maintaining a smooth and responsive editing experience in the WordPress block editor (Gutenberg). This includes both frontend techniques and backend architectural decisions that directly affect the performance perceived by editorial users.

## Assuring a Good Experience for Editorial Users

Creating a performant editor interface requires careful consideration of both frontend interactivity and backend data handling. Editorial performance should be evaluated holistically, taking into account:

- Load times for the post editor.
- Interaction latency (e.g., typing delays, block insertion speed).
- Memory usage and rendering cycles.
- Network requests made by custom blocks or plugin integrations.

### Best Practices

- **Limit the number of dynamic blocks** on a page when possible, especially those making remote API calls.
- **Avoid synchronous blocking operations**, like large data computations inside a block's render method.
- **Use selectors and memoization** from `@wordpress/data` to avoid re-computation of derived state.
- **Defer expensive computations** to `useEffect` or `requestIdleCallback` when suitable.

## Profiling React Components

React is the underlying library powering the block editor. To optimize performance, developers should become proficient in profiling React components.

### Using the React Profiler

The React DevTools extension provides a **Profiler** tab, allowing developers to record interactions and visualize rendering behavior. For more information, see the [WordPress Block Editor Handbook's Performance section](https://developer.wordpress.org/block-editor/explanations/architecture/performance/).

Steps:

1. Install the React DevTools browser extension.
2. Open the editor and navigate to the Profiler tab.
3. Start profiling and interact with your custom block.
4. Analyze the recorded flamegraph to identify expensive renders.

### Interpreting Results

Look for:

- Components that re-render more than expected.
- Blocks with large render durations.
- Props or state changes that trigger unnecessary re-renders.

## Avoiding Unnecessary Renders

React components in the editor may re-render frequently if not properly optimized. This leads to sluggishness, especially for complex or interactive blocks.

### Optimization Techniques

#### 1. Component Memoization with React.memo()

React.memo() is a higher-order component that prevents unnecessary re-renders when a component's props haven't changed. This is particularly useful for expensive components or those that receive the same props frequently. Learn more about React optimization in the [WordPress Block Editor Handbook](https://developer.wordpress.org/block-editor/reference-guides/packages/packages-performance/).

```jsx
// Without memoization
const BlockControls = ({ attributes, setAttributes }) => {
  // Component re-renders on every parent render
  return (
    <div className="block-controls">
      <RichText
        value={attributes.content}
        onChange={(content) => setAttributes({ content })}
      />
    </div>
  );
};

// With memoization
const BlockControls = React.memo(({ attributes, setAttributes }) => {
  // Component only re-renders when attributes or setAttributes change
  return (
    <div className="block-controls">
      <RichText
        value={attributes.content}
        onChange={(content) => setAttributes({ content })}
      />
    </div>
  );
});
```

#### 2. Efficient Selector Usage with useSelect

The `useSelect` hook from `@wordpress/data` provides optimized state selection with built-in memoization. Use it to access WordPress data store values efficiently. For more details, see the [@wordpress/data documentation](https://developer.wordpress.org/block-editor/reference-guides/packages/packages-data/).

```jsx
// Problematic: Selector runs on every render
const MyBlock = ({ postId }) => {
  const post = useSelect((select) => {
    // This selector runs on every render
    return select("core").getEntityRecord("postType", "post", postId);
  });

  return <div>{post?.title}</div>;
};

// Optimized: Memoized selector with dependency array
const MyBlock = ({ postId }) => {
  // Get only the needed post data
  const postTitle = useSelect(
    // Renamed to reflect what it contains
    (select) => {
      const post = select("core").getEntityRecord("postType", "post", postId);
      return post?.title.rendered ?? null; // Return just the string
    },
    [postId]
  );

  if (postTitle === null) return <Spinner />;

  return (
    <div>
      <h2>{postTitle}</h2>
    </div>
  );
};
```

The key points about `useSelect`:

1. **Automatic Memoization**: By default, `useSelect` memoizes based on the selector function and its arguments.

2. **Dependency Array**: You only need to provide a dependency array when:
   - The selector depends on values from outside the selector function (like props)
   - You want to force re-computation when certain values change
   - You're using closure variables that aren't selector arguments

#### 3. Component Splitting for Better Performance

Breaking down large components into smaller, focused units helps isolate rendering and improves maintainability.

```jsx
// Before: Large component with multiple responsibilities
const ComplexBlock = ({ attributes, setAttributes }) => {
  return (
    <div className="complex-block">
      <RichText
        value={attributes.title}
        onChange={(title) => setAttributes({ title })}
      />
      <MediaUpload
        onSelect={(media) => setAttributes({ mediaId: media.id })}
        allowedTypes={["image"]}
      />
      <BlockControls>
        <AlignmentToolbar
          value={attributes.alignment}
          onChange={(alignment) => setAttributes({ alignment })}
        />
      </BlockControls>
    </div>
  );
};

// After: Split into focused components
const BlockTitle = React.memo(({ value, onChange }) => (
  <RichText value={value} onChange={onChange} />
));

const BlockMedia = React.memo(({ onSelect }) => (
  <MediaUpload onSelect={onSelect} allowedTypes={["image"]} />
));

const BlockAlignment = React.memo(({ value, onChange }) => (
  <AlignmentToolbar value={value} onChange={onChange} />
));

const ComplexBlock = ({ attributes, setAttributes }) => {
  return (
    <div className="complex-block">
      <BlockTitle
        value={attributes.title}
        onChange={(title) => setAttributes({ title })}
      />
      <BlockMedia onSelect={(media) => setAttributes({ mediaId: media.id })} />
      <BlockControls>
        <BlockAlignment
          value={attributes.alignment}
          onChange={(alignment) => setAttributes({ alignment })}
        />
      </BlockControls>
    </div>
  );
};
```

#### 4. Avoiding Inline Functions and Object Literals

Inline functions and object literals create new references on every render, potentially triggering unnecessary re-renders in child components.

```jsx
// Problematic: Creates new function reference on every render
const MyBlock = () => {
  return (
    <ChildComponent
      onClick={() => console.log("clicked")} // New function every render
      style={{ color: "red" }} // New object every render
    />
  );
};

// Optimized: Stable references
const MyBlock = () => {
  const handleClick = useCallback(() => {
    console.log("clicked");
  }, []); // Stable function reference

  const styles = useMemo(
    () => ({
      color: "red",
    }),
    []
  ); // Stable object reference

  return <ChildComponent onClick={handleClick} style={styles} />;
};
```

These optimization techniques work together to create a more performant block editor experience. Remember to:

- Use React DevTools Profiler to measure the impact of these optimizations
- Apply optimizations strategically where they provide the most benefit
- Consider the trade-off between optimization and code complexity
- Test performance with realistic content and user interactions

## Efficient Use of WordPress Queries Within Dynamic Blocks

Server-side rendered blocks that use inefficient WordPress queries can significantly impact editor performance. These issues often become apparent only when dealing with large content datasets or high-traffic sites. Let's explore best practices and optimization techniques in detail.

### Understanding Query Performance Impact

When blocks perform database queries, they can affect the editor in several ways:

- **Initial Load Time**: Heavy queries during block rendering can slow down the editor's initial load
- **Memory Usage**: Retrieving full post objects instead of specific fields increases memory consumption
- **Database Load**: Unoptimized queries can put unnecessary strain on the database
- **Caching Opportunities**: Missed caching opportunities lead to repeated expensive queries

### Best Practices for Query Optimization

#### 1. Limiting Query Results

Always set reasonable limits on query results to prevent performance issues with large datasets. Learn more about WordPress query optimization in the [WordPress Performance Handbook](https://developer.wordpress.org/advanced-administration/performance/optimization/).

```php
// Problematic: Unbounded query
function problematic_block_render() {
    $posts = get_posts([
        'post_type' => 'product',
        // No limit specified - could return thousands of posts
    ]);
    return render_posts($posts);
}

// Optimized: Limited results with pagination support
function optimized_block_render($attributes) {
    $posts_per_page = isset($attributes['postsPerPage'])
        ? min((int) $attributes['postsPerPage'], 20) // Cap at 20 posts
        : 5;

    $posts = get_posts([
        'post_type' => 'product',
        'posts_per_page' => $posts_per_page,
        'paged' => isset($attributes['page']) ? (int) $attributes['page'] : 1,
    ]);

    return render_posts($posts);
}
```

#### 2. Selective Field Retrieval

Use the `fields` parameter to retrieve only the data you need, significantly reducing memory usage and query time. For more information about WordPress queries, see the [WordPress Database API](https://developer.wordpress.org/reference/classes/wpdb/).

```php
// Problematic: Retrieving full post objects
function heavy_block_render() {
    $posts = get_posts([
        'post_type' => 'post',
        'posts_per_page' => 10,
    ]);

    $output = '<ul>';
    foreach ($posts as $post) {
        // Using only title and ID, but we fetched everything
        $output .= sprintf(
            '<li><a href="%s">%s</a></li>',
            get_permalink($post->ID),
            esc_html($post->post_title)
        );
    }
    return $output . '</ul>';
}

// Optimized: Retrieving only needed fields
function optimized_block_render() {
    $posts = get_posts([
        'post_type' => 'post',
        'posts_per_page' => 10,
        'fields' => 'ids', // Only get post IDs
    ]);

    $output = '<ul>';
    foreach ($posts as $post_id) {
        $title = get_the_title($post_id);
        $output .= sprintf(
            '<li><a href="%s">%s</a></li>',
            get_permalink($post_id),
            esc_html($title)
        );
    }
    return $output . '</ul>';
}
```

#### 3. Implementing Caching

Use WordPress transients and object caching to store frequently accessed data. Learn more about WordPress caching in the [WordPress Transients API](https://developer.wordpress.org/apis/handbook/transients/).

```php
function cached_block_render($attributes) {
    $cache_key = 'block_posts_' . md5(serialize($attributes));

    // Try to get cached output
    $cached_output = get_transient($cache_key);
    if (false !== $cached_output) {
        return $cached_output;
    }

    // If not cached, generate output
    $posts = get_posts([
        'post_type' => 'post',
        'posts_per_page' => 5,
        'fields' => 'ids',
    ]);

    $output = render_posts($posts);

    // Cache for 1 hour
    set_transient($cache_key, $output, HOUR_IN_SECONDS);

    return $output;
}
```

#### 4. Using REST API for Dynamic Content

For blocks that need to update frequently or handle user interactions, use the REST API instead of server-side rendering. Learn more about the WordPress REST API in the [REST API Handbook](https://developer.wordpress.org/rest-api/).

```php
// Register REST API endpoint
add_action('rest_api_init', function () {
    register_rest_route('my-plugin/v1', '/posts', [
        'methods' => 'GET',
        'callback' => 'get_block_posts',
        'permission_callback' => '__return_true',
    ]);
});

function get_block_posts($request) {
    $posts = get_posts([
        'post_type' => 'post',
        'posts_per_page' => 5,
        'fields' => 'ids',
    ]);

    return rest_ensure_response([
        'posts' => array_map(function($post_id) {
            return [
                'id' => $post_id,
                'title' => get_the_title($post_id),
                'permalink' => get_permalink($post_id),
            ];
        }, $posts)
    ]);
}
```

```jsx
// Block Editor Component
const DynamicPostsBlock = () => {
  const [posts, setPosts] = useState([]);
  const [isLoading, setIsLoading] = useState(true);

  useEffect(() => {
    apiFetch({ path: "/my-plugin/v1/posts" }).then((response) => {
      setPosts(response.posts);
      setIsLoading(false);
    });
  }, []);

  if (isLoading) {
    return <Spinner />;
  }

  return (
    <div className="dynamic-posts">
      {posts.map((post) => (
        <div key={post.id}>
          <a href={post.permalink}>{post.title}</a>
        </div>
      ))}
    </div>
  );
};
```

### Additional Considerations

1. **Query Timing**: Consider when queries are executed:

   - Use `init` hook for early data preparation
   - Defer non-critical queries using `wp_schedule_single_event()`
   - Implement lazy loading for below-the-fold content

2. **Database Optimization**:

   - Add appropriate indexes to custom tables
   - Use `wp_cache_set()` and `wp_cache_get()` for custom caching
   - Consider using persistent object caching (Redis/Memcached)

3. **Monitoring and Debugging**:
   - Use [Query Monitor](https://wordpress.org/plugins/query-monitor/) plugin to identify slow queries
   - Implement logging for performance-critical operations
   - Monitor memory usage with `memory_get_usage()`

These optimization techniques work together to create efficient, scalable blocks that perform well even with large datasets. Remember to:

- Profile your blocks with realistic data volumes
- Monitor query performance in production
- Implement progressive enhancement for better user experience
- Consider implementing infinite scroll or pagination for large datasets

## Conclusion

Managing editor performance is a foundational skill for WordPress enterprise developers. Through techniques like React profiling, render optimization, and efficient querying, developers can significantly improve the authoring experience for content creators. This lesson provides a basis for understanding and applying these strategies in real-world block development.

For more information about WordPress performance optimization, visit the [WordPress Performance Handbook](https://developer.wordpress.org/advanced-administration/performance/optimization/).
