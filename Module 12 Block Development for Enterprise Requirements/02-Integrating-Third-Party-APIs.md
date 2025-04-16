# Integrating Third-Party APIs

In enterprise WordPress development, the ability to integrate third-party APIs in custom Gutenberg blocks is a powerful way to extend block functionality, improve user experiences, and connect your WordPress applications with external services and data sources. This lesson explores the technical aspects of consuming external APIs within the Block Editor framework, addressing both the front-end and back-end rendering challenges, as well as implementing robust authentication strategies essential for enterprise applications.

## Fetching and Displaying External Data in Blocks

Modern enterprise applications frequently need to connect with external services—from CRM systems and marketing automation platforms to data visualization services and internal microservices. The WordPress Block Editor provides a sophisticated framework for implementing these integrations in a way that maintains the visual editing experience while ensuring optimal performance.

### Client-Side vs. Server-Side API Requests

When integrating external APIs into Gutenberg blocks, developers must decide whether to fetch data on the client side (JavaScript) or the server side (PHP). Each approach offers distinct advantages and challenges.

#### Client-Side Approach

The client-side approach leverages JavaScript within the Block Editor environment to fetch and render external data, using the `api-fetch` package:

```javascript
import { useEffect, useState } from '@wordpress/element';
import apiFetch from '@wordpress/api-fetch';

export default function Edit( attributes ) {
    const apiSecretKey = attributes.apiSecretKey;
    const [data, setData] = useState(null);
    const [isLoading, setIsLoading] = useState(true);
    const [error, setError] = useState(null);

    useEffect(() => {
        const fetchData = async () => {
            try {
                // Using WordPress's apiFetch for consistent handling
                // passing an Authorization header
                const response = await apiFetch({
                    path: '/custom-namespace/v1/external-data',
                    headers : {
                        Authorization: 'Bearer ' + apiSecretKey
                    }
                });
                setData(response);
                setIsLoading(false);
            } catch (error) {
                console.error('Error fetching data:', error);
                setError(error);
                setIsLoading(false);
            }
        };

        fetchData();
    }, []);

    if (isLoading) {
        return <p>Loading data...</p>;
    }

    if (error) {
        return <p>Error loading data. Please try again.</p>;
    }

    return (
        <div className="api-data-block">
            <h3>{data.title}</h3>
            <p>{data.description}</p>
        </div>
    );
}
```

This approach is particularly effective for:

- Real-time data display
- Interactive blocks that need to respond to user actions
- When you want to show a preview of external data in the editor

However, client-side fetching introduces challenges:

- Exposing credentials in client-side code
- Cross-Origin Resource Sharing (CORS) restrictions
- Managing API rate limits when multiple editors are active
- Performance impact on the editing experience

#### Server-Side Approach

The server-side approach employs PHP to fetch data and then passes it to the block, using the WordPress HTTP API:

```php
<?php
/**
 * Server-side rendering for the API data block.
 *
 * @param array $attributes The block attributes.
 * @return string The block HTML.
 */
function my_custom_api_block_render_callback( $attributes ) {
    // Fetch data from external API
    $api_url = 'https://api.example.com/data';
    $response = wp_remote_get( $api_url, array(
        'headers' => array(
            'Authorization' => 'Bearer ' . get_option( 'api_secret_key' ),
        ),
    ) );

    if ( is_wp_error( $response ) ) {
        return '<p>Error fetching API data. Please try again later.</p>';
    }

    $body = wp_remote_retrieve_body( $response );
    $data = json_decode( $body, true );

    if ( empty( $data ) ) {
        return '<p>No data available.</p>';
    }

    // Build the block HTML
    $output = '<div class="api-data-block">';
    $output .= '<h3>' . esc_html( $data['title'] ) . '</h3>';
    $output .= '<p>' . esc_html( $data['description'] ) . '</p>';
    $output .= '</div>';

    return $output;
}
```

You would register this block with:

```php
register_block_type( 'my-plugin/api-data', array(
    'attributes'      => array(
        'apiEndpoint' => array(
            'type' => 'string',
            'default' => 'default-endpoint',
        ),
    ),
    'render_callback' => 'my_custom_api_block_render_callback',
) );
```

The server-side approach offers:

- More secure handling of API credentials
- Better caching capabilities
- Reduced JavaScript payload in the editor
- Ability to process and transform data before rendering

The trade-off is that the Block Editor doesn't display actual API data until the page is rendered, limiting the visual editing experience.

### Rendering External API Data in Both Environments

For an optimal enterprise solution, you'll often want to show API data both in the editor (back-end) and on the published page (front-end). This requires a hybrid approach:

```javascript
// Block registration
registerBlockType('my-plugin/api-data-block', {
    title: 'External API Data',
    icon: 'database',
    category: 'widgets',
    attributes: {
        endpoint: {
            type: 'string',
            default: 'default-endpoint'
        },
        cachedData: {
            type: 'object',
            default: {}
        }
    },
    edit: Edit,
    save: () => null // Using server-side rendering
});

// Edit component with data fetching
function Edit({ attributes, setAttributes }) {
    const { endpoint, cachedData } = attributes;
    const [data, setData] = useState(cachedData);
    const [isLoading, setIsLoading] = useState(Object.keys(cachedData).length === 0);

    useEffect(() => {
        if (isLoading) {
            wp.apiFetch({
                path: `/my-plugin/v1/proxy-api?endpoint=${endpoint}`
            }).then(response => {
                setData(response);
                // Cache the data in block attributes
                setAttributes({ cachedData: response });
                setIsLoading(false);
            }).catch(error => {
                console.error('Error fetching API data:', error);
                setIsLoading(false);
            });
        }
    }, [endpoint, isLoading]);

    // Editor representation
    return (
        <div className="api-data-block editor-view">
            {isLoading ? (
                <p>Loading API data...</p>
            ) : (
                <>
                    <h3>{data.title}</h3>
                    <p>{data.description}</p>
                </>
            )}
        </div>
    );
}
```

This approach creates a custom REST API endpoint in WordPress to proxy requests to the external API, improving security by keeping credentials server-side while enabling data preview in the editor[^7].

```php
// Register custom REST API endpoint to proxy external API requests
add_action('rest_api_init', function () {
    register_rest_route('my-plugin/v1', '/proxy-api', array(
        'methods' => 'GET',
        'callback' => 'my_plugin_proxy_api_request',
        'permission_callback' => function() {
            return current_user_can('edit_posts');
        }
    ));
});

/**
 * Proxy API request to external service.
 *
 * @param WP_REST_Request $request Full details about the request.
 * @return WP_REST_Response|WP_Error Response object or error.
 */
function my_plugin_proxy_api_request($request) {
    $endpoint = $request->get_param('endpoint');
    $api_url = 'https://api.example.com/' . $endpoint;
    
    $response = wp_remote_get($api_url, array(
        'headers' => array(
            'Authorization' => 'Bearer ' . get_option('api_secret_key'),
        ),
    ));
    
    if (is_wp_error($response)) {
        return new WP_Error('api_error', 'Failed to fetch external data');
    }
    
    $body = wp_remote_retrieve_body($response);
    return json_decode($body, true);
}
```


### Error Handling and Loading States

For enterprise applications, robust error handling is critical. Your blocks should gracefully handle API outages, rate limiting, and authentication failures.

```javascript
// Enhanced error handling in Edit component
function Edit({ attributes, setAttributes }) {
    const [data, setData] = useState(null);
    const [status, setStatus] = useState({
        loading: true,
        error: null,
        lastUpdated: null
    });

    useEffect(() => {
        fetchData();
    }, []);

    const fetchData = async () => {
        setStatus({ ...status, loading: true });
        try {
            const response = await wp.apiFetch({
                path: '/my-plugin/v1/external-data'
            });

            setData(response);
            setStatus({
                loading: false,
                error: null,
                lastUpdated: new Date()
            });
        } catch (error) {
            console.error('API Error:', error);
            setStatus({
                loading: false,
                error: {
                    code: error.code || 'unknown',
                    message: error.message || 'Unknown error occurred'
                },
                lastUpdated: status.lastUpdated
            });
        }
    };

    // Display different UI based on status
    if (status.loading) {
        return (
            <div className="api-block-loading">
                <Spinner />
                <p>Fetching latest data...</p>
            </div>
        );
    }

    if (status.error) {
        return (
            <div className="api-block-error">
                <h3>Error: {status.error.code}</h3>
                <p>{status.error.message}</p>
                <button onClick={fetchData}>Retry</button>
                {status.lastUpdated && (
                    <p className="data-timestamp">
                        Showing data from: {status.lastUpdated.toLocaleString()}
                    </p>
                )}
            </div>
        );
    }

    return (
        <div className="api-data-block">
            <div className="block-controls">
                <button onClick={fetchData}>Refresh Data</button>
                <span>Last updated: {status.lastUpdated.toLocaleString()}</span>
            </div>
            <div className="block-content">
                {/* Render your data here */}
            </div>
        </div>
    );
}

```


### Performance Optimization Strategies

In enterprise environments, performance is paramount. Consider these strategies for optimizing API integration:

1. **Implement caching**: Cache API responses using transients or a dedicated caching plugin to reduce API calls:
```php
function get_external_api_data($endpoint) {
    $cache_key = 'api_data_' . md5($endpoint);
    $cached_data = get_transient($cache_key);
    
    if (false !== $cached_data) {
        return $cached_data;
    }
    
    // Make API request
    $response = wp_remote_get('https://api.example.com/' . $endpoint);
    
    if (is_wp_error($response)) {
        return null;
    }
    
    $data = json_decode(wp_remote_retrieve_body($response), true);
    
    // Cache for 1 hour
    set_transient($cache_key, $data, HOUR_IN_SECONDS);
    
    return $data;
}

```

2. **Implement caching reset on content updates**:
```php
add_action('save_post', function($post_id) {
    // Clear relevant API caches when content is updated
    if (!wp_is_post_revision($post_id)) {
        delete_transient('api_data_specific_cache');
    }
});

```

3. **Use batch requests**: If you're making multiple API calls, consider using batch requests when the API supports them to reduce HTTP overhead.
4. **Implement pagination and lazy loading**: For large datasets, load only what's immediately visible and implement "load more" functionality.
5. **Background processing**: For intensive API operations, consider using WordPress background processing libraries like Action Scheduler for asynchronous operations.

## Authentication Strategies

Secure authentication is critical when integrating third-party APIs, especially in enterprise environments where data sensitivity and compliance are significant concerns[^4][^8].

### Common Authentication Methods

#### Basic Authentication

Basic Authentication is the simplest form of API authentication, consisting of a username and password or client ID and client secret:

```php
$api_url = 'https://api.example.com/data';
$username = 'api_user';
$password = 'api_password';

$response = wp_remote_get($api_url, array(
    'headers' => array(
        'Authorization' => 'Basic ' . base64_encode("$username:$password"),
    ),
));

```

While straightforward, this method is less secure since credentials must be stored and transmitted with each request[^4].

#### API Key Authentication

API Key Authentication uses a single key to identify and authenticate the client:

```php
$api_url = 'https://api.example.com/data';
$api_key = get_option('my_plugin_api_key');

$response = wp_remote_get($api_url, array(
    'headers' => array(
        'X-API-Key' => $api_key,
    ),
));

```

This method is relatively simple but still requires secure storage and transmission of the API key[^4].

#### Bearer Token / JWT Authentication

JSON Web Tokens (JWT) provide a more secure mechanism for authentication, particularly for OAuth flows:

```php
// Assuming we've already obtained a JWT token
$api_url = 'https://api.example.com/data';
$token = get_option('my_plugin_jwt_token');

$response = wp_remote_get($api_url, array(
    'headers' => array(
        'Authorization' => 'Bearer ' . $token,
    ),
));

```

JWT authentication provides enhanced security through signed tokens and can include expiration times and other security features[^4].

#### OAuth 2.0 Authentication

For enterprise applications, OAuth 2.0 is often the preferred authentication method due to its security and flexibility:

```php
/**
 * Implements OAuth 2.0 Password Grant flow.
 *
 * @return string|WP_Error Access token or error.
 */
function get_oauth_access_token() {
    $cached_token = get_transient('oauth_access_token');
    
    if (false !== $cached_token) {
        return $cached_token;
    }
    
    $oauth_url = 'https://auth.example.com/oauth/token';
    $client_id = get_option('oauth_client_id');
    $client_secret = get_option('oauth_client_secret');
    $username = get_option('oauth_username');
    $password = get_option('oauth_password');
    
    $response = wp_remote_post($oauth_url, array(
        'body' => array(
            'grant_type' => 'password',
            'client_id' => $client_id,
            'client_secret' => $client_secret,
            'username' => $username,
            'password' => $password,
        ),
    ));
    
    if (is_wp_error($response)) {
        return $response;
    }
    
    $data = json_decode(wp_remote_retrieve_body($response), true);
    
    if (empty($data['access_token'])) {
        return new WP_Error('oauth_error', 'Failed to obtain access token');
    }
    
    // Cache token (subtract buffer time from expiration)
    $expiration = isset($data['expires_in']) ? $data['expires_in'] - 60 : 3600;
    set_transient('oauth_access_token', $data['access_token'], $expiration);
    
    return $data['access_token'];
}

// Using the token
function make_authenticated_request($endpoint) {
    $token = get_oauth_access_token();
    
    if (is_wp_error($token)) {
        return $token;
    }
    
    $api_url = 'https://api.example.com/' . $endpoint;
    
    $response = wp_remote_get($api_url, array(
        'headers' => array(
            'Authorization' => 'Bearer ' . $token,
        ),
    ));
    
    return $response;
}

```

OAuth 2.0 supports multiple grant types, with Password Grant and Client Credentials being most common for server-to-server communication in enterprise applications[^4][^8].

### Securing API Credentials in WordPress

For enterprise applications, securely storing API credentials is crucial. WordPress provides several approaches:

#### 1. Using wp-config.php Constants

For maximum security, define credentials as constants in your wp-config.php file:

```php
// In wp-config.php
define('MY_API_KEY', 'your-secret-api-key');
define('MY_CLIENT_ID', 'your-client-id');
define('MY_CLIENT_SECRET', 'your-client-secret');

// In your plugin
function get_api_credentials() {
    return array(
        'api_key' => defined('MY_API_KEY') ? MY_API_KEY : '',
        'client_id' => defined('MY_CLIENT_ID') ? MY_CLIENT_ID : '',
        'client_secret' => defined('MY_CLIENT_SECRET') ? MY_CLIENT_SECRET : '',
    );
}

```

This approach keeps credentials out of the database and version control systems.

#### 2. Encrypted Options

If you need to store credentials in the database, encrypt them:

```php
/**
 * Encrypts sensitive data before storing in the database.
 *
 * @param string $data Data to encrypt.
 * @return string Encrypted data.
 */
function encrypt_api_credential($data) {
    if (empty($data)) {
        return '';
    }
    
    $salt = wp_salt('auth');
    $method = 'aes-256-cbc';
    $iv = openssl_random_pseudo_bytes(openssl_cipher_iv_length($method));
    
    $encrypted = openssl_encrypt($data, $method, $salt, 0, $iv);
    
    return base64_encode($iv . $encrypted);
}

/**
 * Decrypts sensitive data from the database.
 *
 * @param string $encrypted_data Encrypted data.
 * @return string Decrypted data.
 */
function decrypt_api_credential($encrypted_data) {
    if (empty($encrypted_data)) {
        return '';
    }
    
    $salt = wp_salt('auth');
    $method = 'aes-256-cbc';
    $iv_length = openssl_cipher_iv_length($method);
    
    $decoded = base64_decode($encrypted_data);
    $iv = substr($decoded, 0, $iv_length);
    $encrypted = substr($decoded, $iv_length);
    
    return openssl_decrypt($encrypted, $method, $salt, 0, $iv);
}

// Storing encrypted credentials
function save_api_key($api_key) {
    $encrypted = encrypt_api_credential($api_key);
    update_option('my_plugin_api_key', $encrypted);
}

// Retrieving decrypted credentials
function get_api_key() {
    $encrypted = get_option('my_plugin_api_key', '');
    return decrypt_api_credential($encrypted);
}
```


#### 3. Using WordPress Application Passwords

For WordPress-to-WordPress API communication, consider using WordPress's built-in Application Passwords feature:

```php
// Making authenticated requests to another WordPress site
function make_wp_api_request($endpoint) {
    $username = 'api_user';
    $app_password = 'XXXX XXXX XXXX XXXX';
    
    $response = wp_remote_get('https://example.com/wp-json/' . $endpoint, array(
        'headers' => array(
            'Authorization' => 'Basic ' . base64_encode("$username:$app_password"),
        ),
    ));
    
    return $response;
}
```


### Implementing Authentication in Block Code

When working with authenticated APIs in Gutenberg blocks, you need to consider both the editor experience and the front-end rendering.

#### Server-Side Authentication

For security, implement authentication logic on the server side:

```php
// Register REST API endpoint to proxy authenticated requests
add_action('rest_api_init', function () {
    register_rest_route('my-plugin/v1', '/proxy-api', array(
        'methods' => 'GET',
        'callback' => 'my_plugin_proxy_authenticated_api',
        'permission_callback' => function() {
            return current_user_can('edit_posts');
        }
    ));
});

/**
 * Proxy authenticated API request.
 *
 * @param WP_REST_Request $request Full details about the request.
 * @return WP_REST_Response|WP_Error Response object or error.
 */
function my_plugin_proxy_authenticated_api($request) {
    $endpoint = $request->get_param('endpoint');
    
    // Get OAuth token
    $token = get_oauth_access_token();
    
    if (is_wp_error($token)) {
        return $token;
    }
    
    // Make authenticated request
    $api_url = 'https://api.example.com/' . $endpoint;
    $response = wp_remote_get($api_url, array(
        'headers' => array(
            'Authorization' => 'Bearer ' . $token,
        ),
    ));
    
    if (is_wp_error($response)) {
        return new WP_Error(
            'api_error',
            'Failed to fetch data: ' . $response->get_error_message()
        );
    }
    
    return json_decode(wp_remote_retrieve_body($response), true);
}
```


#### Connecting the Block to Authenticated Endpoints

In your block's Edit component, use the proxy endpoint:

```javascript
function Edit({ attributes, setAttributes }) {
    const [data, setData] = useState(null);
    const [isLoading, setIsLoading] = useState(true);

    useEffect(() => {
        const fetchData = async () => {
            try {
                // Using WordPress internal apiFetch to handle authentication
                const response = await wp.apiFetch({
                    path: '/my-plugin/v1/proxy-api?endpoint=data/customer-metrics',
                });

                setData(response);
                setIsLoading(false);
            } catch (error) {
                console.error('Failed to fetch API data:', error);
                setIsLoading(false);
            }
        };

        fetchData();
    }, []);

    // Render block UI
}
```

This approach keeps sensitive authentication details server-side while still allowing the Block Editor to display live data[^2][^7].

### Best Practices for Enterprise-Grade API Security

For enterprise applications, consider these additional security practices:

1. **Implement rate limiting** to prevent abuse of your proxy endpoints.
2. **Validate and sanitize all input and output** to prevent injection attacks:
```php
function my_plugin_proxy_api($request) {
    // Validate endpoint parameter
    $endpoint = sanitize_text_field($request-&gt;get_param('endpoint'));
    
    if (empty($endpoint) || !in_array($endpoint, array('allowed-endpoint-1', 'allowed-endpoint-2'))) {
        return new WP_Error('invalid_endpoint', 'Invalid endpoint specified', array('status' =&gt; 400));
    }
    
    // Proceed with authenticated request
}
```

3. **Use capability checks** to restrict access to proxy endpoints:
```php
'permission_callback' =&gt; function() {
    return current_user_can('manage_options');
}
```

4. **Implement proper error handling** without leaking sensitive information:
```php
function handle_api_error($response) {
    $error_code = wp_remote_retrieve_response_code($response);
    
    // Log detailed error internally
    error_log('API Error: ' . wp_remote_retrieve_body($response));
    
    // Return generic error to user
    return new WP_Error(
        'api_error',
        'The external service returned an error. Please try again later.',
        array('status' =&gt; 500)
    );
}
```

5. **Set up monitoring and alerting** for API failures in enterprise environments.
6. **Rotate credentials regularly** and implement a secure process for credential rotation.

## Complete Example: Weather Forecast Block

Let's bring everything together with a complete example of a weather forecast block that integrates with a third-party weather API:

```php
 'weather-block-editor',
        'editor_style' =&gt; 'weather-block-editor-style',
        'style' =&gt; 'weather-block-style',
        'attributes' =&gt; array(
            'location' =&gt; array(
                'type' =&gt; 'string',
                'default' =&gt; 'New York'
            ),
            'days' =&gt; array(
                'type' =&gt; 'number',
                'default' =&gt; 3
            ),
            'cachedForecast' =&gt; array(
                'type' =&gt; 'object',
                'default' =&gt; array()
            ),
            'lastUpdated' =&gt; array(
                'type' =&gt; 'number',
                'default' =&gt; 0
            )
        ),
        'render_callback' =&gt; 'render_weather_block',
    ));
}
add_action('init', 'register_weather_block');

// Enqueue editor assets
function enqueue_weather_block_editor_assets() {
    wp_register_script(
        'weather-block-editor',
        plugins_url('build/index.js', __FILE__),
        array('wp-blocks', 'wp-element', 'wp-editor', 'wp-components', 'wp-i18n', 'wp-api-fetch'),
        filemtime(plugin_dir_path(__FILE__) . 'build/index.js')
    );
    
    wp_register_style(
        'weather-block-editor-style',
        plugins_url('build/editor.css', __FILE__),
        array('wp-edit-blocks'),
        filemtime(plugin_dir_path(__FILE__) . 'build/editor.css')
    );
    
    wp_register_style(
        'weather-block-style',
        plugins_url('build/style.css', __FILE__),
        array(),
        filemtime(plugin_dir_path(__FILE__) . 'build/style.css')
    );
}
add_action('enqueue_block_editor_assets', 'enqueue_weather_block_editor_assets');

// Register REST API endpoint for weather data
function register_weather_api_endpoints() {
    register_rest_route('enterprise-weather/v1', '/forecast', array(
        'methods' =&gt; 'GET',
        'callback' =&gt; 'get_weather_forecast',
        'permission_callback' =&gt; function() {
            return current_user_can('edit_posts');
        },
        'args' =&gt; array(
            'location' =&gt; array(
                'required' =&gt; true,
                'sanitize_callback' =&gt; 'sanitize_text_field'
            ),
            'days' =&gt; array(
                'default' =&gt; 3,
                'sanitize_callback' =&gt; 'absint'
            )
        )
    ));
}
add_action('rest_api_init', 'register_weather_api_endpoints');

// Get weather forecast from external API
function get_weather_forecast($request) {
    $location = $request-&gt;get_param('location');
    $days = $request-&gt;get_param('days');
    
    // Generate cache key
    $cache_key = 'weather_' . md5($location . '_' . $days);
    $cached_data = get_transient($cache_key);
    
    if (false !== $cached_data) {
        return $cached_data;
    }
    
    // Get API key from secure storage
    $api_key = defined('WEATHER_API_KEY') ? WEATHER_API_KEY : get_option('enterprise_weather_api_key');
    
    if (empty($api_key)) {
        return new WP_Error('missing_api_key', 'Weather API key is not configured');
    }
    
    // Make API request
    $api_url = add_query_arg(
        array(
            'location' =&gt; urlencode($location),
            'days' =&gt; $days,
            'key' =&gt; $api_key,
        ),
        'https://api.weatherapi.com/v1/forecast.json'
    );
    
    $response = wp_remote_get($api_url);
    
    if (is_wp_error($response)) {
        return $response;
    }
    
    $status_code = wp_remote_retrieve_response_code($response);
    
    if ($status_code !== 200) {
        $error_message = wp_remote_retrieve_body($response);
        error_log('Weather API Error: ' . $error_message);
        return new WP_Error(
            'api_error',
            'Failed to retrieve weather data. Please try again later.',
            array('status' =&gt; $status_code)
        );
    }
    
    $data = json_decode(wp_remote_retrieve_body($response), true);
    
    // Cache for 1 hour
    set_transient($cache_key, $data, HOUR_IN_SECONDS);
    
    return $data;
}

// Render block on frontend
function render_weather_block($attributes) {
    $location = $attributes['location'];
    $days = $attributes['days'];
    
    // Generate cache key
    $cache_key = 'weather_' . md5($location . '_' . $days);
    $cached_data = get_transient($cache_key);
    
    if (false === $cached_data) {
        // If cache is empty, fetch fresh data
        $api_key = defined('WEATHER_API_KEY') ? WEATHER_API_KEY : get_option('enterprise_weather_api_key');
        
        if (empty($api_key)) {
            return '<div>Weather API key is not configured</div>';
        }
        
        $api_url = add_query_arg(
            array(
                'location' =&gt; urlencode($location),
                'days' =&gt; $days,
                'key' =&gt; $api_key,
            ),
            'https://api.weatherapi.com/v1/forecast.json'
        );
        
        $response = wp_remote_get($api_url);
        
        if (is_wp_error($response) || wp_remote_retrieve_response_code($response) !== 200) {
            return '<div>Failed to retrieve weather data</div>';
        }
        
        $cached_data = json_decode(wp_remote_retrieve_body($response), true);
        set_transient($cache_key, $cached_data, HOUR_IN_SECONDS);
    }
    
    // Begin output buffering to capture HTML
    ob_start();
    ?&gt;
    <div>
        <div>
            <h3>, </h3>
        </div>
        
        <div>
            <div>
                <span>°C</span>
                <img>
            </div>
            <div>
                
            </div>
        </div>
        
        <div>
            
                <div>
                    <div></div>
                    <img>
                    <div>
                        <span>°</span>
                        <span>°</span>
                    </div>
                </div>
            
        </div>
        
        <div>
            &lt;small&gt;Last updated: &lt;/small&gt;
        </div>
    </div>
     {
            setIsLoading(true);
            setError(null);
            
            try {
                const data = await apiFetch({
                    path: `/enterprise-weather/v1/forecast?location=${encodeURIComponent(location)}&amp;days=${days}`
                });
                
                setForecast(data);
                setAttributes({ 
                    cachedForecast: data,
                    lastUpdated: Date.now()
                });
                setIsLoading(false);
            } catch (err) {
                console.error('Weather API Error:', err);
                setError(err.message || 'Failed to fetch weather data');
                setIsLoading(false);
            }
        };
        
        // Fetch data when location or days change
        useEffect(() =&gt; {
            fetchWeatherData();
        }, [location, days]);
        
        // Inspector controls for block settings
        const inspectorControls = (
            &lt;InspectorControls&gt;
                &lt;Panel&gt;
                    &lt;PanelBody title="Weather Settings"&gt;
                        &lt;TextControl
                            label="Location"
                            value={location}
                            onChange={(value) =&gt; setAttributes({ location: value })}
                        /&gt;
                        &lt;RangeControl
                            label="Forecast Days"
                            value={days}
                            onChange={(value) =&gt; setAttributes({ days: value })}
                            min={1}
                            max={7}
                        /&gt;
                        <div>
                            &lt;button 
                                onClick={fetchWeatherData}
                                className="components-button is-secondary"
                            &gt;
                                Refresh Weather Data
                            &lt;/button&gt;
                            {lastUpdated &gt; 0 &amp;&amp; (
                                <p>
                                    Last updated: {new Date(lastUpdated).toLocaleString()}
                                </p>
                            )}
                        </div>
                    &lt;/PanelBody&gt;
                &lt;/Panel&gt;
            &lt;/InspectorControls&gt;
        );
        
        // Render loading state
        if (isLoading) {
            return (
                &lt;&gt;
                    {inspectorControls}
                    <div>
                        &lt;Spinner /&gt;
                        <p>Loading weather data...</p>
                    </div>
                
            );
        }
        
        // Render error state
        if (error) {
            return (
                &lt;&gt;
                    {inspectorControls}
                    <div>
                        <p>Error: {error}</p>
                        &lt;button 
                            onClick={fetchWeatherData}
                            className="components-button is-secondary"
                        &gt;
                            Try Again
                        &lt;/button&gt;
                    </div>
                
            );
        }
        
        // Render weather forecast
        if (!forecast.location || !forecast.current || !forecast.forecast) {
            return (
                &lt;&gt;
                    {inspectorControls}
                    <div>
                        <p>No weather data available</p>
                    </div>
                
            );
        }
        
        return (
            &lt;&gt;
                {inspectorControls}
                <div>
                    <div>
                        <h3>{forecast.location.name}, {forecast.location.country}</h3>
                    </div>
                    
                    <div>
                        <div>
                            <span>{forecast.current.temp_c}°C</span>
                            <img>
                        </div>
                        <div>
                            {forecast.current.condition.text}
                        </div>
                    </div>
                    
                    <div>
                        {forecast.forecast.forecastday.map((day) =&gt; (
                            <div>
                                <div>
                                    {new Date(day.date).toLocaleDateString('en-US', { weekday: 'short' })}
                                </div>
                                <img>
                                <div>
                                    <span>{day.day.maxtemp_c}°</span>
                                    <span>{day.day.mintemp_c}°</span>
                                </div>
                            </div>
                        ))}
                    </div>
                    
                    <div>
                        &lt;small&gt;Last updated: {forecast.current.last_updated}&lt;/small&gt;
                    </div>
                </div>
            
        );
    },
    
    // Use PHP render_callback for front-end rendering
    save: () =&gt; null
});
```


## Conclusion

Integrating third-party APIs into WordPress blocks is a powerful capability for enterprise applications, enabling seamless connections with external services and data sources. We've explored approaches for fetching and displaying external data in blocks, from client-side JavaScript to server-side PHP rendering, and covered robust authentication strategies essential for secure enterprise implementations.

When implementing API integrations in enterprise WordPress applications, remember these key principles:

1. Choose the right approach for data fetching based on your security and user experience requirements
2. Implement caching to improve performance and reduce API calls
3. Secure your authentication credentials using WordPress best practices
4. Provide seamless editing experiences in the Block Editor
5. Implement robust error handling and fallback mechanisms

By following these guidelines, you can create powerful, secure, and performant third-party API integrations that meet enterprise requirements while leveraging the flexibility and extensibility of the WordPress Block Editor.

<div>⁂</div>

[^1]: http://raw.githubusercontent.com/jonathanbossenger/enterp

[^2]: https://kyraweb.ca/integrating-third-party-apis-into-wordpress-a-comprehensive-guide/

[^3]: https://www.reddit.com/r/Wordpress/comments/1doqxvh/reading_external_api_in_gutenberg/

[^4]: https://wordpress.com/plugins/wp-rest-api-authentication

[^5]: https://vipestudio.com/en/web-development-services/custom-gutenberg-blocks-development/

[^6]: https://rtcamp.com/handbook/developing-for-block-editor-and-site-editor/understanding-wordpress-block-api/

[^7]: https://css-tricks.com/rendering-external-api-data-in-wordpress-blocks-on-the-back-end/

[^8]: https://wordpress.org/plugins/wp-rest-api-authentication/

[^9]: https://www.bluent.net/blog/advanced-gutenberg-block-development-tutorial

[^10]: https://wpvip.com/blog/wordpress-block-data/

[^11]: https://wordpress.org/support/topic/api-call-in-custom-block/

[^12]: https://positiwise.com/blog/how-to-integrate-a-third-party-api-into-a-wordpress-website

[^13]: https://kinsta.com/blog/gutenberg-blocks/

[^14]: https://wpvip.com/reasons-love-wordpress-gutenberg-editor/

[^15]: https://gotchamobi.com/stream-articles/category/web-design/6918929/06/26/2023/rendering-external-api-data-in-wordpress-blocks-on-the-back-end-css-tricks/

[^16]: https://kinsta.com/blog/third-party-apis/

[^17]: https://developer.wordpress.org/block-editor/reference-guides/block-api/

[^18]: https://www.easywp.com/blog/wordpress-api-integration/

[^19]: https://stackoverflow.com/questions/39858009/how-to-fetch-and-display-data-from-external-api-in-wordpress

[^20]: https://melapress.com/wordpress-rest-api-security/

[^21]: https://www.codeable.io/developers/gutenberg/

