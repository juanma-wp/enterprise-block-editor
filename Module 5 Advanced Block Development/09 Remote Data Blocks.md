# Lesson: Integrating External Data with Remote Data Blocks

## Introduction: Beyond Core Block Bindings

In a previous lesson, we explored the Block Bindings API, a helpful feature introduced in WordPress 6.5 for dynamically filling block attributes with data from sources like post meta or custom PHP functions. This core feature makes development much easier by creating direct links between block content and internal WordPress data.

However, modern enterprise applications often need to show data coming _from outside_ of WordPress entirely—product details from an e-commerce system, user profiles from a CRM, event listings from a third-party service, and more. While the core Block Bindings API provides the base, connecting these external sources usually requires custom development for each specific API and data format.

The [Remote Data Blocks plugin](https://remotedatablocks.com/), developed by Automattic/WPVIP, helps solve this problem. It acts as an extension and specialized version of the Block Bindings idea, offering a solid framework specifically designed for connecting block attributes to **remote**, external data sources. This allows content creators to build dynamic, data-filled layouts using familiar Block Editor tools, while developers manage the data connections and queries behind the scenes.

This lesson will explain the concepts and use of Remote Data Blocks, showing how it uses and builds on the Block Bindings API to handle external data integration well.

It's important to know that, as of this writing, the Remote Data Blocks plugin is in Beta. Although powerful, breaking changes might happen in future updates. Thorough testing is recommended before using updates in production. The plugin needs [PHP version 8.1 or higher and WordPress 6.7 or higher](https://github.com/Automattic/remote-data-blocks#requirements).

## Core Concepts of Remote Data Blocks

To use Remote Data Blocks well, understanding its main parts is essential. These build on the [foundational concepts of Block Bindings](https://remotedatablocks.com/docs/concepts/block-bindings/).

### Using Block Bindings for External Data

At its core, Remote Data Blocks uses the same Block Bindings system discussed in a previous lesson. It allows linking attributes of a block (like the `content` of a Paragraph block or the `url` of an Image block) to a dynamic data source.

The main difference is that Remote Data Blocks introduces special [**Data Sources**](https://remotedatablocks.com/docs/extending/data-source/) and [**Queries**](https://remotedatablocks.com/docs/extending/query/) designed just for getting external data. Instead of binding to `core/post-meta`, you bind to a source set up by Remote Data Blocks (like `remote-data-blocks/http-query`). The plugin then takes care of getting data from the external API, caching it, and giving it to the Block Bindings system to display.

### Data Sources

In Remote Data Blocks, a [Data Source](https://remotedatablocks.com/docs/extending/data-source/) is the setup needed to connect to a specific external service or API. It's similar to registering a custom binding source in the core API, but made specifically for remote connections. The plugin comes with built-in support for common sources (like [Airtable](https://remotedatablocks.com/docs/tutorials/airtable/), [Google Sheets](https://remotedatablocks.com/docs/tutorials/google-sheets-integration/), [Shopify](https://remotedatablocks.com/docs/tutorials/shopify/), [GitHub](https://github.com/Automattic/remote-data-blocks), and generic [HTTP/REST APIs](https://remotedatablocks.com/docs/tutorials/http/)) and provides a clear way for developers to add their own:

- Airtable
- Google Sheets
- Shopify
- GitHub
- Generic HTTP/REST APIs

Each source needs specific setup details, like API keys, login information, or base URLs. These are usually managed in the WordPress admin area (`Settings` > `Remote Data Blocks`). Setting up a Data Source makes that external service available to use for block bindings.

### Queries

Once a Data Source is set up, a [Query](https://remotedatablocks.com/docs/extending/query/) specifies _what_ data to get from that source and _how_. A Query links a Data Source with parameters, API paths, filters, and very importantly, a **schema** that defines the expected format of the returned data.

Think of a Query as the specific "request" sent to the external source. Remote Data Blocks lets you set up these Queries through the UI or with PHP code. When you bind a block attribute using the Block Bindings UI, you select a Query that's already been defined. The plugin then runs this Query to get the needed data.

The schema defined in the Query is key because it tells the Block Bindings UI which data fields can be used for binding (like `product_name`, `user_email`, `event_date`).

### Example: Connecting Block Bindings to Remote Data

Imagine you have set up an "Internal Product API" Data Source and a "Get Product Details" Query using Remote Data Blocks. This Query gets product data based on a [Stock Keeping Unit](https://en.wikipedia.org/wiki/Stock_keeping_unit).

1.  You add a Group block to your post.
2.  Inside the Group block, you add a Heading block and a Paragraph block.
3.  Using the Block Editor's Binding controls (the same UI used for core Block Bindings), you select the Heading block.
4.  Instead of choosing `core/post-meta`, you choose the "Get Product Details" Query provided by Remote Data Blocks.
5.  You bind the Heading block's `content` attribute to the `product_name` field defined in the Query's schema. You might also need to set up the Query block itself (or a parent block) to tell it which SKU to get data for.
6.  You select the Paragraph block and bind its `content` attribute to the `product_description` field from the same Query.

Now, the Heading and Paragraph blocks dynamically show data fetched from the external product API. This uses the core Block Bindings system, helped by the Remote Data Blocks plugin's ability to fetch and manage data.

### Caching

Working with external APIs requires thinking about performance and reliability. Remote APIs can be slow, limit how often you can call them, or become unavailable sometimes. Remote Data Blocks includes a very important [caching layer](https://github.com/Automattic/remote-data-blocks#requirements). A persistent object cache is highly recommended.

When data is fetched using a Query, the result is stored temporarily in the WordPress object cache (ideally a persistent cache like Redis or Memcached) for a set amount of time (Time-To-Live, or TTL). Future requests for the exact same Query (with the same parameters) within the TTL will get the data directly from the cache, avoiding repeated API calls. This greatly improves page load times and respects limits set by the external service. If a persistent object cache isn't set up, it uses per-request in-memory caching as a backup.

## Simplified Architecture: How It Works

The process combines the Block Bindings API with the Remote Data Blocks fetching system:

1.  **Block Rendering:** A page loads that has a block whose attribute is bound to a Remote Data Blocks Query.
2.  **Binding Identified:** WordPress sees the block binding, its source (e.g., `remote-data-blocks/http-query`), and the related Query definition.
3.  **Query Execution:** The Remote Data Blocks Query Runner service gets the Query details (Data Source, parameters, path, schema, etc.).
4.  **Cache Check:** The Query Runner checks the WordPress object cache for valid, unexpired data that matches this specific Query request.
5.  **API Call (Cache Miss):** If no valid cached data is found, the Query Runner uses the configured Data Source to make the API request to the external service.
6.  **Cache Update:** The newly fetched data is stored in the cache with the Query's defined TTL.
7.  **Data Returned:** The fetched (or cached) data, matching the Query's schema format, is returned.
8.  **Block Attribute Population:** The Block Bindings API fills the bound block attribute (e.g., `content`, `url`) with the correct piece of data.
9.  **Dynamic Rendering:** The block shows its final HTML using the dynamically filled attribute.

## Example: Registering a Custom Query via PHP for Dynamic Plugin Data

This example demonstrates registering a custom PHP Query that fetches data for _any_ plugin from the WordPress.org Plugin API, based on a `plugin_slug` parameter passed to it. This Query can then be used with Block Bindings.

```php

namespace RemoteDataBlocks\Example\WpOrgPlugin;

use RemoteDataBlocks\Config\DataSource\HttpDataSource;
use RemoteDataBlocks\Config\Query\HttpQuery;


/**
 * Registers a custom block to retrieve plugin information from WordPress.org.
 *
 * @return void
 * @uses register_remote_data_block() Function provided by the Remote Data Blocks plugin.
 */
function register_wporg_plugin_block(): void {
	$plugin_data_source = HttpDataSource::from_array(
		[

			'__version'       => 1,
			'display_name'    => 'WordPress.org Plugins',
			'endpoint'        => 'https://api.wordpress.org',
			'request_headers' => [
				'Accept' => 'application/json',
			],

		]
	);

	$get_plugin_query = HttpQuery::from_array(
		[
			'data_source'   => $plugin_data_source,
			'endpoint'      => function ( array $input_variables ) use ( $plugin_data_source ): string {
				return sprintf(
					'%s/plugins/info/1.0/%s.json',
					$plugin_data_source->get_endpoint(),
					$input_variables['plugin_slug'] ?? ''
				);
			},
			'input_schema'  => [
				'plugin_slug' => [
					'name' => 'Plugin Slug',
					'type' => 'string',
				],
			],
			'output_schema' => [
				'is_collection' => false,
				'path'          => '$',
				'type'          => [
					'name'              => [
						'name' => 'Plugin Name',
						'path' => '$.name',
						'type' => 'string',
					],
					'slug'              => [
						'name' => 'Plugin Slug',
						'path' => '$.slug',
						'type' => 'string',
					],
					'version'           => [
						'name' => 'Current Version',
						'path' => '$.version',
						'type' => 'string',
					],
					'author'            => [
						'name' => 'Plugin Author',
						'path' => '$.author',
						'type' => 'string',
					],
					'rating'            => [
						'name' => 'Average Rating',
						'path' => '$.rating',
						'type' => 'number',
					],
					'num_ratings'       => [
						'name' => 'Number of Ratings',
						'path' => '$.num_ratings',
						'type' => 'integer',
					],
					'downloaded'        => [
						'name' => 'Download Count',
						'path' => '$.downloaded',
						'type' => 'integer',
					],
					'last_updated'      => [
						'name' => 'Last Updated',
						'path' => '$.last_updated',
						'type' => 'string',
					],
					'short_description' => [
						'name' => 'Short Description',
						'path' => '$.short_description',
						'type' => 'string',
					],
				],
			],
		]
	);

	register_remote_data_block(
		[
			'title'        => 'WP.org Plugin Info',
			'render_query' => [
				'query' => $get_plugin_query,
			],
			'patterns'     => [
				[
					'html'  => file_get_contents( __DIR__ . '/pattern-dotorg-plugin.html' ),
					'title' => 'Pattern Simple',
				],
			],

		]
	);
}
add_action( 'init', __NAMESPACE__ . '\\register_wporg_plugin_block' );
```

The code above creates a WordPress plugin that fetches plugin information from WordPress.org using Remote Data Blocks. It:

1. Creates a data source pointing to `api.wordpress.org`
2. Creates a query that:
   - Accepts a plugin slug as input
   - Fetches data from WordPress.org's API
   - Returns fields like name, version, author, ratings, etc.
3. Registers a new block type called "WP.org Plugin Info" with a pattern template

The block can be used in the WordPress editor to display dynamic plugin information by simply providing a plugin slug.

> [!NOTE]
> Check out a [live demo](https://playground.wordpress.net/?blueprint-url=https://raw.githubusercontent.com/Automattic/wpvip-learn-enterprise-block-editor/refs/heads/trunk/examples/remote-data-blocks-custom-query/_playground/blueprint.json) of this example and the [complete code](https://github.com/Automattic/wpvip-learn-enterprise-block-editor/tree/trunk/examples/remote-data-blocks-custom-query) of the Remote Data Block example above.

## Extending Remote Data Blocks for Custom Solutions

The real value for enterprises often comes from [customizing Remote Data Blocks](https://remotedatablocks.com/docs/extending/):

- **[Custom Data Sources](https://remotedatablocks.com/docs/extending/data-source/):** Create PHP classes for company-specific APIs or services not supported out-of-the-box (e.g., SOAP services, complex GraphQL endpoints). These become selectable Data Sources in the UI.
- **[Custom Queries](https://remotedatablocks.com/docs/extending/query/):** Programmatically register complex or reusable queries (as shown in Example 2), making the editor experience simpler.
- **[Block Integration](https://remotedatablocks.com/docs/extending/block-registration/):** Build custom blocks that know how to interact with specific Queries or provide UI controls for selecting Queries/parameters.
- **[Hooks and Filters](https://remotedatablocks.com/docs/extending/hooks/):** Use the plugin's actions and filters to change Query behavior, transform data globally, adjust caching logic, or connect with other systems.
- **[Custom Query Runners](https://remotedatablocks.com/docs/extending/query-runner/):** For very specific needs, replace the core logic that handles running Queries.

## Enterprise Use Cases

Remote Data Blocks, building on Block Bindings, makes many enterprise situations possible:

- **Product Catalogs:** Show live product details from PIMs or platforms like Shopify/Magento within marketing content.
- **Personalized Content:** Show user-specific data (account details, recommendations) from CRMs by passing user IDs into Queries.
- **Event Listings:** Embed dynamic event calendars from external management tools.
- **Knowledge Bases:** Integrate content from systems like Confluence or GitHub directly into WordPress pages.
- **BI Dashboards:** Display key metrics or report summaries from data warehouses.
- **Staff Directories:** Fill team pages with data from HR systems or Active Directory.

## Getting Started and Resources

1.  **Installation:** [Install the Remote Data Blocks plugin](https://github.com/Automattic/remote-data-blocks#installation) from its GitHub repository (remember [PHP/WP version needs](https://github.com/Automattic/remote-data-blocks#requirements)).
2.  **Configuration:** Go to `Settings` > `Remote Data Blocks` to set up external Data Sources (API keys, URLs, etc.).
3.  **Create Queries:** Define Queries linked to your Data Sources, specifying paths, parameters, and importantly, the data **Schema**.
4.  **Bind Blocks:** Use the Block Bindings UI in the editor to connect block attributes (content, URL, etc.) to the fields defined in your Query schemas.
5.  **Explore:** Try using Query blocks and binding core blocks to see the data flow. You can also [launch the plugin in WordPress Playground](https://github.com/Automattic/remote-data-blocks#remote-data-blocks) for quick testing.

Key Resources:

- **Official Documentation:** [https://remotedatablocks.com/docs/](https://remotedatablocks.com/docs/)
- **GitHub Repository (Code, Issues, Releases):** [https://github.com/Automattic/remote-data-blocks](https://github.com/Automattic/remote-data-blocks)

## Conclusion

Remote Data Blocks is a powerful addition to the core Block Bindings API, specifically designed for bringing external data sources into the WordPress Block Editor. By setting up Data Sources and Queries, developers can create reusable connections to remote APIs. Content creators can then use the familiar Block Bindings interface to dynamically fill block attributes with external data, which is fetched and cached efficiently by the plugin. This combination provides a scalable and easy-to-maintain way to build rich, data-driven content experiences in enterprise WordPress applications.
