## Block Context in Advanced Block Development

Block Context is a powerful feature in WordPress that allows blocks to share data and manage states across nested structures. This lesson will explore how to effectively use Block Context to enhance your block development skills.

## Understanding Block Context

Block Context enables parent blocks to provide values that can be consumed by descendant blocks within the same hierarchy. This feature is particularly useful in complex block structures and full-site editing scenarios.

### Key Concepts

1. **Provider Blocks**: These are typically parent blocks that supply context values.
2. **Consumer Blocks**: These are descendant blocks that can access and use the provided context values.

## Implementing Block Context

### Providing Context

To create a block that provides context:

1. Define the `providesContext` property in your block's `block.json` file:

```
{
  "apiVersion": 2,
  "name": "my-plugin/parent-block",
  "title": "Parent Block",
  "attributes": {
    "recordId": {
      "type": "number"
    }
  },
  "providesContext": {
    "my-plugin/recordId": "recordId"
  }
}
```

2. In your block's `edit` function, you can set the context value:

```javascript
export default function Edit({ attributes, setAttributes }) {
  return (
    <div>
      <TextControl
        label="Record ID"
        value={attributes.recordId}
        onChange={(value) => setAttributes({ recordId: parseInt(value) })}
      />
      <InnerBlocks />
    </div>
  );
}
```

### Consuming Context

To create a block that consumes context:

1. Define the `usesContext` property in your block's `block.json` file:

```
{
  "apiVersion": 2,
  "name": "my-plugin/child-block",
  "title": "Child Block",
  "usesContext": ["my-plugin/recordId"]
}
```

2. Access the context in your block's `edit` function:

```javascript
export default function Edit({ context }) {
  const recordId = context["my-plugin/recordId"];
  return <div>The current record ID is: {recordId}</div>;
}
```

Check out a [live demo](https://playground.wordpress.net/?blueprint-url=https://raw.githubusercontent.com/Automattic/wpvip-learn-enterprise-block-editor/refs/heads/trunk/examples/context/_playground/blueprint.json) of this example and its [complete code](https://github.com/Automattic/wpvip-learn-enterprise-block-editor/tree/trunk/examples/context).

## Practical Applications

1. **Dynamic Content**: Use Block Context to display content based on a parent block's settings.
2. **Conditional Rendering**: Show or hide certain blocks based on context values.
3. **Data Inheritance**: Pass down common data (like post ID or theme settings) to multiple nested blocks.

## Best Practices

1. **Namespace Your Context**: Use a prefix (e.g., `my-plugin/`) to avoid conflicts with other plugins or core WordPress contexts.
2. **Keep It Simple**: Only pass down necessary data to maintain performance.
3. **Consider Performance**: Be mindful that sharing large amounts of data through context can impact editor performance.

## Conclusion

Block Context is a powerful tool for creating more dynamic and interconnected block structures. By effectively using provider and consumer blocks, you can create more flexible and maintainable block-based layouts in WordPress.

Remember, Block Context flows downward from parent to child blocks, allowing for efficient data sharing without tight coupling between blocks. This approach enhances reusability and maintainability of your custom blocks.

By mastering Block Context, you'll be able to create more sophisticated and interactive block-based layouts, elevating your WordPress development skills to the next level.

For more detailed information, refer to the official [Block Context Documentation](https://developer.wordpress.org/block-editor/reference-guides/block-api/block-context/) or any of resources below.
