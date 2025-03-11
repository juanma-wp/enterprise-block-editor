## Curriculum (3rd revision)

### Module 1: Introduction

* **1.1: Overview of Gutenberg Development**
  * What is Gutenberg?
  * The Editor vs the Site Editor
* **1.2: Overview of Block themes**
  * Templates and template parts
  * Patterns
  * Global Settings and Styles
  * Custom functionality (functions.php) and styling (style.css)

### Module 2: Building with Blocks

* **2.1: Alternatives to Custom blocks**
  * What to consider before reaching for custom blocks
  * Introduce block patterns, block templates, and block variations
  * How to decide which fits the needs of the project
* **2.2: Patterns**
  * Understanding block patterns
  * Creating and registering custom patterns
  * Using patterns for reusable content
* **2.3: Templates**
  * Building block templates for custom post types
  * Programmatically applying templates
* **2.4: Variations**
  * Creating block variations
  * Managing reusable configurations

### Module 3: Building Custom Blocks

* **1.2: Setting Up Your Environment**
  * Installing Node.js and npm
  * Overview of the @wordpress npm packages
* **1.3: Project Scaffolding**
  * Scaffolding a new block using create-block
  * Directory and file structure best practices
  * The WordPress Scripts package for efficient development
  * Setting up a block development scaffold/environment
* **1.4: Anatomy of a Gutenberg Block**
  * Block registration using block.json and registerBlockType
  * Key properties: edit, save, attributes, supports
* **1.3: Block Development Workflow**
  * Using wp-scripts for build processes
  * Hot reloading and debugging

### Module 4: Using Components from @wordpress/components

* **4.1: Overview of the @wordpress/components Package**
  * Introduction to common reusable components
  * Benefits of using prebuilt components
* **4.2: Key Components and Their Applications**
  * Button: Creating interactive buttons
  * BaseControl: Enhancing forms with labeled controls
  * PanelBody and PanelRow: Structuring settings panels
  * TextControl: Capturing user input in text fields
  * SelectControl: Dropdown menus for option selection
  * ToggleControl: Managing boolean states with toggles
  * CheckboxControl and RadioControl: Building form inputs
  * RangeControl: Adjusting numeric values with sliders
  * Notice: Displaying inline notifications
  * ToolbarButton: Adding buttons to block toolbars
  * Popover: Creating contextual menus or overlays
  * ColorPicker: Selecting and applying colors
* **4.3: Best Practices for Using Components**
  * Customizing and styling components
  * Handling component state and interactivity

### Module 5: Advanced Block Development

* **5.1: More Block Features**
  * Dynamic blocks with PHP integration
  * Nested blocks with InnerBlocks
  * Internationalization (i18n) for blocks
* **5.2: Block Categories**
  * Building and managing block-specific categories and libraries
* **5.3: Styling Blocks**
  * Editor styles vs. frontend styles
  * Adding CSS Selectors
  * Managing responsive design in blocks
* **5.4: Bindings**
  * Understanding block bindings
  * Enhancing block interactions with bindings
* **5.5: Complex InnerBlocks**
  * How InnerBlocks allow nested content and structured layouts
  * Using the allowedBlocks and templateLock properties
  * Adding custom settings to InnerBlocks through block attributes
* **5.6: Context**
  * Using context to manage block states
  * Sharing data between nested blocks
* **5.7: Deprecation**
  * Handling block versioning and deprecation
  * Strategies for maintaining backward compatibility
* **5.8: Transforms**
  * Transforming blocks to and from other blocks
  * Use cases and practical examples
  * Adding custom transformations
  * Applying filters for conditional transforms

### 

### Module 6: Customizing the Editor Experience

* **6.1: Custom Block Controls**
  * Custom sidebar panels for a block ([Settings Sidebar](https://developer.wordpress.org/block-editor/getting-started/fundamentals/block-in-the-editor/#settings-sidebar))
  * Custom options in the block’s toolbar ([Block Toolbar](https://developer.wordpress.org/block-editor/getting-started/fundamentals/block-in-the-editor/#block-toolbar))
* **6.2: Slot & Fill System**
  * Injecting UI elements into the editor
  * Examples: adding controls to the toolbar, sidebar panels
* **6.3: Registering Custom Block Styles**
  * Adding style variations to existing blocks
  * Managing style previews
* **6.4: Editor Filters and Actions**
  * Hooking into editor.BlockEdit and editor.BlockListBlock
  * Modifying block settings dynamically
* **6.5 Using the WordPress VIP Block Governance Plugin**
  * Managing editorial permissions and design governance

### Module 7: WordPress Data Layer and wp.data

* **7.1: Introduction to the WordPress Data Layer**
  * Stores and actions overview
  * Accessing core data stores
* **7.2: Fetching Data with wp.data**
  * Querying posts, terms, and users
  * Customizing queries for specific use cases
* **7.3: Using useSelect and useDispatch**
  * Fetching and managing WordPress data
  * Subscribing to store changes
* **7.4: Modifying Data**
  * Updating post settings and options
  * Managing nonces and security considerations
* **7.5: Creating Custom Data Stores**
  * Using useRegistry
  * State management with Redux
  * Registering custom stores
  * Using selectors and resolvers

### Module 8: Interactivity API

* **8.1: Overview of the Interactivity API**
  * Key Features
  * Benefits of using the Interactivity API
  * Example of using Interactivity API in a block
* **8.2: Implementing the Interactivity API**
  * Directives
  * Store
* **8.3: Global State, Local Context, and Derived State**
  * Global State
  * Local Context
  * Derived State

### Module 9: Performance and Optimization

* **9.1: Optimizing JavaScript for Gutenberg**
  * Understanding the block rendering cycle
  * Code splitting and lazy loading
  * Reducing bundle size
  * Caching strategies for block data and REST API responses
* **9.2: Managing Editor Performance**
  * Assuring a good experience for your editorial users
  * Profiling React components
  * Avoiding unnecessary renders
  * Efficient use of WordPress queries within dynamic blocks
* **9.3: Load Testing and Monitoring**
  * Load testing blocks with tools like k6 or Apache JMeter
  * Monitoring and logging block performance in production

### Module 10: Security and Accessibility

* **10.1: Accessibility in Blocks**
  * Developing blocks that meet WCAG 2.1 accessibility standards
  * Testing with screen readers
* **10.2: Security Best Practices for Blocks**
  * Sanitizing and validating block data
  * Escaping dynamic output
  * Validating user roles and capabilities
* **10.3: Data Privacy and GDPR Compliance**
  * Implementing privacy-by-design principles in block development.
  * Ensuring compliance with GDPR and other data protection regulations.

### Module 11: Testing and Deployment

* **11.1: Testing Gutenberg Blocks**
  * Writing unit tests with Jest
  * Integration tests with Playwright
* **11.2: Continuous Integration and Deployment**
  * Setting up CI/CD pipelines for Gutenberg development
  * Automating block deployment and versioning in enterprise environments
* **11.3: Code Quality and Linting**
  * Implementing ESLint and Prettier for maintaining code standards
  * Advanced Git workflows for managing block development in large teams

### Module 12: Integration with Enterprise Systems

* **12.1: Headless WordPress and Gutenberg**
  * Benefits and challenges of a headless setup
  * Decoupling the front-end from WordPress using the REST API or GraphQL
  * Fetching Gutenberg block data via the REST API
  * Handling Block Attributes in Headless
* **12.2: Integrating Third-Party APIs**
  * Fetching and displaying external data in blocks
  * Authentication strategies
* **12.3: Building Full-Site Editing Features**
  * Building and extending block themes with theme.json
  * Creating template parts and custom templates
* **12.4: Multisite and Multilingual Considerations**
  * Global vs. Site-Specific Blocks
  * Customizing block permissions based on user roles across sites
  * Compatibility with plugins like WPML for block translation

### Module 13: Roadmap and Case Studies

* **13.1: Upcoming Features and Roadmap**
  * Multi-user Collaboration
  * Staying informed about Gutenberg's evolution and upcoming features
* **13.2: Community Contribution**
  * How to contribute to Gutenberg's open-source development
  * Engaging with the WordPress community
* **13.3: Real-World Enterprise Use Cases**
  * Analysis of successful Gutenberg implementations in enterprise projects.
  * Lessons learned and best practices from large-scale Gutenberg deployments.