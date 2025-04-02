# Accessibility in Blocks

As a WordPress developer, you are responsible for ensuring that your blocks are usable by everyone, including people with disabilities.

In this lesson, we'll explore how to develop blocks that meet Web Content Accessibility Guidelines standards and how to test them using screen readers and other accessibility testing tools.

## Understanding WCAG Standards

WordPress is committed to conforming to all [WCAG 2.2 Level A and Level AA guidelines](https://developer.wordpress.org/coding-standards/wordpress-coding-standards/accessibility/). As a developer creating custom blocks, you need to understand these standards and implement them in our block development process.

WCAG conformance levels are divided into three categories:

**Level A** represents the minimum level of accessibility, addressing barriers that prevent many people from accessing web content.

**Level AA** addresses more complex accessibility concerns that might impact smaller groups of people but are still common needs with broad reach.

**Level AAA** targets very specific needs and may be quite difficult to implement effectively.

## Developing Accessible Blocks

Let's explore the key principles and techniques for developing blocks that meet WCAG accessibility standards.

## Use Existing Tools

First and foremost, wherever possible, make use of existing components from the `@wordpress/components` package. This package provides accessible components that meet the WCAG standards.

You can learn more about the available components in the [WordPress Block Editor Handbook](https://developer.wordpress.org/block-editor/reference-guides/packages/packages-components/).

If you do need to create custom block structures, here are some common accessibility practices to follow:

### Use Semantic HTML

Semantic HTML provides meaning to the structure of your content, making it easier for screen readers and other assistive technologies to interpret and navigate your blocks.

Instead of using generic `<div>` elements for everything, use appropriate HTML elements, for example:

- `<h1>` through `<h2>` for headings in the correct hierarchical order
- `<p>` for paragraphs of text
- `<button>` for interactive elements that perform an action
- `<a>` for links to other pages or resources

### Implement ARIA Landmarks and Roles

[Accessible Rich Internet Applications](https://www.w3.org/WAI/standards-guidelines/aria/) (ARIA) attributes help provide additional context to assistive technologies. When developing blocks, assign appropriate ARIA landmark roles to different areas of your blocks.

Common ARIA landmarks include:

```
// Navigation block
<nav aria-label="Main Navigation">
    // Navigation content
</nav>

// Search block
<div role="search" aria-label="Site Search">
    // Search form
</div>

// Main content block
<main role="main">
    // Main content
</main>
```

Only use ARIA when necessary and when native HTML semantics don't provide enough information. The first rule of ARIA is that you don't use ARIA if you can use a native HTML element or attribute with the semantics and behavior you require.

### Ensure Keyboard Accessibility

All interactive elements in your blocks must be accessible via keyboard navigation. Users should be able to navigate through your blocks using the Tab key and interact with elements using **Enter** or **Space** keys.

```
// Example of a custom button component with keyboard support
function CustomButton({ onClick, children }) {
    return (
        <button
            onClick={onClick}
            onKeyDown={(event) => {
                if (event.key === 'Enter' || event.key === ' ') {
                    onClick();
                }
            }}
        >
            {children}
        </button>
    );
}
```

Make sure that focus states are clearly visible. The WordPress editor provides focus styles by default, but if you're creating custom UI components, ensure they have visible focus indicators.

### Provide Alternative Text for Images

When implementing block layouts that include images, always provide a way for users to add alternative text. This is important for screen reader users to understand the content and purpose of images.

```
// Example of an image block with alt text support
import { useBlockProps, RichText } from '@wordpress/block-editor';

export default function Edit({ attributes, setAttributes }) {
    const { imageUrl, altText } = attributes;
    const blockProps = useBlockProps();

    return (
        <div { ...blockProps }>
            {imageUrl ? (
                <figure>
                    <img
                        src={imageUrl}
                        alt={altText || ''}
                    />
                    <RichText
                        tagName="figcaption"
                        value={altText}
                        onChange={(newAltText) => setAttributes({ altText: newAltText })}
                        placeholder="Add alt text for image"
                    />
                </figure>
            ) : (
                <p>Please select an image</p>
            )}
        </div>
    );
}
```

### Maintain Color Contrast

Ensure that text and interactive elements have sufficient color contrast against their backgrounds. The WCAG 2.1 AA standard requires a contrast ratio of at least 4.5:1 for normal text and 3:1 for large text.

When developing blocks, avoid hardcoding colors that might not meet contrast requirements. Instead, use WordPress theme colors or provide options for users to customize colors while warning them about potential contrast issues.

```
// Example of using theme colors in a block
import { useBlockProps, PanelColorSettings } from '@wordpress/block-editor';
import { InspectorControls } from '@wordpress/block-editor';

export default function Edit({ attributes, setAttributes }) {
    const { textColor, backgroundColor } = attributes;
    const blockProps = useBlockProps({
        style: {
            color: textColor,
            backgroundColor: backgroundColor,
        },
    });

    return (
        <>
            <InspectorControls>
                <PanelColorSettings
                    title="Color Settings"
                    colorSettings={[
                        {
                            value: textColor,
                            onChange: (color) => setAttributes({ textColor: color }),
                            label: 'Text Color',
                        },
                        {
                            value: backgroundColor,
                            onChange: (color) => setAttributes({ backgroundColor: color }),
                            label: 'Background Color',
                        },
                    ]}
                />
            </InspectorControls>
            <div { ...blockProps }>
                <p>Block content with customizable colors</p>
            </div>
        </>
    );
}
```

### Handle Dynamic Content Appropriately

When blocks include dynamic content that updates without a page reload, ensure that screen readers are informed of these changes. Use ARIA live regions for content that updates automatically.

```
// Example of using ARIA live regions for dynamic content
import { useState, useEffect } from '@wordpress/element';

function DynamicContentBlock() {
    const [content, setContent] = useState('Initial content');

    useEffect(() => {
        const timer = setTimeout(() => {
            setContent('Updated content');
        }, 5000);

        return () => clearTimeout(timer);
    }, []);

    return (
        <div aria-live="polite">
            {content}
        </div>
    );
}
```

### Use the Screen Reader Text Class

WordPress provides a `.screen-reader-text` CSS class that visually hides content while keeping it accessible to screen readers. This is useful for providing additional context to screen reader users without affecting the visual design.

```
// Example of using screen-reader-text
function SkipLink() {
    return (
        <a className="screen-reader-text" href="#main-content">
            Skip to main content
        </a>
    );
}
```

## Testing Block Accessibility

Developing accessible blocks is only half the battle. You also need to test your blocks to ensure they meet accessibility standards. Let's explore how to test blocks using screen readers and other accessibility testing tools.

### Testing with Screen Readers

Screen readers are assistive technologies that convert text into synthesized speech, allowing users with visual impairments to access digital content. Testing your blocks with screen readers ensures they're accessible to all users.

#### Popular Screen Readers

There are several screen readers available for testing:

1. **NVDA (NonVisual Desktop Access)** \- Free, open-source [screen reader for Windows](https://www.nvaccess.org/download/)
2. **VoiceOver** \- Built into [macOS and iOS](https://support.apple.com/en-za/guide/voiceover/welcome/mac)
3. **JAWS (Job Access With Speech)** \- Commercial [screen reader for Windows](https://www.freedomscientific.com/products/software/jaws/)
4. **TalkBack** \- Built into [Android devices](https://support.google.com/accessibility/android/answer/6283677?hl=en)

For WordPress development, testing with NVDA on Windows and VoiceOver on macOS will cover most use cases.

#### Screen Reader Testing Process

When testing blocks with screen readers, follow these steps:

1. Browse to the page containing your block
2. Start the screen reader
3. Refresh or reload the page
4. Set the mouse aside and rely only on keyboard navigation
5. Press Esc to ensure you're not in Forms/Focus Mode (for NVDA)

Test your blocks in two ways:

1. **Content Reading**: Use the Down Arrow key to read through all content
2. **Interactive Elements**: Use the Tab key to navigate through interactive elements

Here's what to check during screen reader testing:

**Semantic Structure**: Does the screen reader correctly announce headings, lists, and other structural elements?

**Interactive Elements**: Are buttons, links, and form controls properly labeled and announced?

**Image Descriptions**: Are images described appropriately through alt text?

**Dynamic Updates**: Are changes to content properly announced?

**Keyboard Navigation**: Can all interactive elements be accessed and operated using only the keyboard?

### Automated Accessibility Testing Tools

In addition to manual testing with screen readers, several automated tools can help identify accessibility issues in your blocks. These generally fall into two categories: WordPress plugins and browser-based tools.

#### WordPress Accessibility Testing Plugins

1. [**WP Accessibility**](https://wordpress.org/plugins/wp-accessibility/) \- Helps fix common accessibility issues in WordPress themes.

2. [**Equalize Digital Accessibility Checker**](https://wordpress.org/plugins/accessibility-checker/) \- Scans your WordPress content for accessibility issues whenever you save or publish.

3. [**Accessibility by UserWay**](https://wordpress.org/plugins/userway-accessibility-widget/) \- Provides an accessibility widget that helps users customize their experience.

4. [**WP Accessibility Helper**](https://wordpress.org/plugins/wp-accessibility-helper/) \- Offers a toolbar with accessibility options for users.

#### Browser-Based Testing Tools

1. [**WAVE (Web Accessibility Evaluation Tool)**](https://wave.webaim.org/) \- Browser extension that evaluates web content for accessibility issues

2. [**axe DevTools**](https://www.deque.com/axe/devtools/) \- A browser extension that identifies accessibility defects

3. [**Lighthouse**](https://developer.chrome.com/docs/lighthouse/accessibility/scoring) \- Built into Chrome DevTools, includes accessibility audits

4. [**BrowserStack Accessibility Testing Tool**](https://www.browserstack.com/docs/accessibility/overview/introduction) \- Allows testing with screen readers on real devices\[7\]

Here's an example of how to integrate accessibility testing into your development workflow:

1. Use automated tools during development to catch common issues
2. Perform manual testing with screen readers before releasing code
3. Include accessibility testing in your continuous integration pipeline

## Accessibility as a feature

Accessibility should not be an afterthought but an essential part of the block development process.

By designing and developing blocks with accessibility in mind, you create a more inclusive web experience for all users. Remember that accessibility benefits everyone, not just users with disabilities. Features like keyboard navigation and clear content structure improve usability for all users.  