## Curriculum (revised version)

### Module 1: Introduction and Tooling Setup

* **1.1: Overview of Gutenberg Development**  
  * What is Gutenberg?  
  * The Editor vs the Site Editor  
* **1.2: Setting Up Your Environment**  
  * Installing Node.js and npm  
  * Overview of the @wordpress npm packages  
* **1.3: Project Scaffolding**  
  * Scaffolding a new block using create-block  
  * Directory and file structure best practices  
  * The WordPress Scripts package for efficient development  
  * Setting up a block development scaffold/environment

### Module 2: Building Custom Blocks

* **2.1: Anatomy of a Gutenberg Block**  
  * Block registration using block.json and registerBlockType  
  * Key properties: edit, save, attributes, supports  
* **2.2: Block Development Workflow**  
  * Using wp-scripts for build processes  
  * Hot reloading and debugging  
* **2.3: More Block Features**  
  * Dynamic blocks with PHP integration  
  * Handling complex attributes (e.g., nested blocks, media fields)  
  * Internationalization (i18n) for blocks  
* **2.4: Styling Blocks**  
  * Editor styles vs. frontend styles  
  * Adding CSS Selectors  
  * Managing responsive design in blocks

### Module 3: Using Components from @wordpress/components

* **3.1: Overview of the @wordpress/components Package**  
  * Introduction to common reusable components  
  * Benefits of using prebuilt components  
* **3.2: Key Components and Their Applications**  
  * Button: Creating interactive buttons  
  * BaseControl: Enhancing forms with labeled controls  
  * TextControl: Capturing user input in text fields  
  * SelectControl: Dropdown menus for option selection  
  * ToggleControl: Managing boolean states with toggles  
  * CheckboxControl and RadioControl: Building form inputs  
  * RangeControl: Adjusting numeric values with sliders
  * Popover: Creating contextual menus or overlays
  * Notice: Displaying inline notifications  
  * ToolbarButton: Adding buttons to block toolbars   
  * ColorPicker: Selecting and applying colors  
  * PanelBody and PanelRow: Structuring settings panels
* **3.3: Best Practices for Using Components**  
  * Customizing and styling components  
  * Handling component state and interactivity

### Module 4: Advanced Block Development

* **4.1: Bindings**  
  * Understanding block bindings  
  * Enhancing block interactions with bindings  
* **4.2: Complex InnerBlocks**  
  * How InnerBlocks allow nested content and structured layouts  
  * Using the allowedBlocks and templateLock properties  
  * Adding custom settings to InnerBlocks through block attributes  
* **4.3: Context**  
  * Using context to manage block states  
  * Sharing data between nested blocks  
* **4.4: Deprecation**  
  * Handling block versioning and deprecation  
  * Strategies for maintaining backward compatibility  
* **4.5: Transforms**  
  * Transforming blocks to and from other blocks  
  * Use cases and practical examples  
  * Adding custom transformations  
  * Applying filters for conditional transforms  
* **4.6: Variations**  
  * Creating block variations  
  * Managing reusable configurations

### Module 5: Patterns, Layouts, and Templates

* **5.1: Block Categories**  
  * Building and managing block-specific categories and libraries  
* **5.2: Patterns**  
  * Understanding block patterns  
  * Creating and registering custom patterns  
  * Using patterns for reusable content  
* **5.3: Layouts**  
  * Designing flexible layouts with layout attributes  
  * Managing alignment and responsive layouts  
* **5.4: Templates**  
  * Building block templates for custom post types  
  * Programmatically applying templates  
  * Editing and managing template parts

### Module 6: Customizing the Editor Experience

* **6.1: Slot & Fill System**  
  * Injecting UI elements into the editor  
  * Examples: adding controls to the toolbar, sidebar panels  
* **6.2: Custom Panels and Inspectors**  
  * Creating sidebar panels for custom settings  
  * Enhancing the block inspector UI  
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
  * Introduction and purpose  
  * Examples of dynamic interactivity in blocks  
* **8.2: Implementing the Interactivity API**  
  * Adding event-driven behaviors to blocks  
  * Integrating with existing Gutenberg features  
* **8.3: Best Practices and Performance**  
  * Ensuring compatibility with core and third-party blocks  
  * Optimizing for smooth user interactions

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

### Module 14: Capstone Project

* **14.1: Building a fully custom, enterprise-ready Gutenberg block suite**  
  * A custom block library, implementing:  
    * Data fetching and storing  
    * Interactivity API  
    * Transforms  
    * Variations  
  * Blocks patterns and layouts  
  * Editor SlotFill Sections  
  * Integration testing  
* **14.2: Code review and collaborative improvement**

This curriculum is designed to be comprehensive and challenging, preparing engineers to handle the complexities of Gutenberg in large-scale, enterprise environments. It focuses on the technical depth and scalability necessary for enterprise applications, ensuring that participants gain not only knowledge but also practical skills for advanced development tasks.