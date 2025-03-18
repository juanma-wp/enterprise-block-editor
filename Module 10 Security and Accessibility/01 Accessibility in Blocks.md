# Accessibility in Blocks

https://github.com/WordPress/Learn/issues/2188

Accessibility is a fundamental aspect of WordPress development, especially when creating custom blocks for the Block Editor. As WordPress developers, we have a responsibility to ensure that our blocks are usable by everyone, including people with disabilities. 

In this lesson, we'll explore how to develop blocks that meet Web Content Accessibility Guidelines (WCAG) 2.1 standards and how to test them using screen readers and other accessibility testing tools.

## Understanding WCAG 2.1 Standards

WordPress is committed to conforming to all [WCAG 2.2 Level A and Level AA guidelines](https://developer.wordpress.org/coding-standards/wordpress-coding-standards/accessibility/). As a developer creating custom blocks, you need to understand these standards and implement them in our block development process.

WCAG conformance levels are divided into three categories:

**Level A** represents the minimum level of accessibility, addressing barriers that prevent many people from accessing web content.

**Level AA** addresses more complex accessibility concerns that might impact smaller groups of people but are still common needs with broad reach.

**Level AAA** targets very specific needs and may be quite difficult to implement effectively.

## Developing Accessible Blocks

Let's explore the key principles and techniques for developing blocks that meet WCAG 2.2 accessibility standards.

## Use Existing Tools

First and foremost, wherever possible, make use of existing components from the `@wordpress/components` package. This package provides accessible components that meet the WCAG standards.

You can learn more about the available components in the [WordPress Block Editor Handbook](https://developer.wordpress.org/block-editor/reference-guides/packages/packages-components/).

If you do need to create custom block structures, here are some common accessibility practices to follow:

### Use Semantic HTML

Semantic HTML provides meaning to the structure of your content, making it easier for screen readers and other assistive technologies to interpret and navigate your blocks.

If you do need to create a custom block layout instead of using generic `<div>` elements for everything, use appropriate HTML elements:

- `<h1>` through `<h2>` for headings in the correct hierarchical order
- `<p>` for paragraphs of text
- `<button>` for interactive elements that perform an action
- `<a>` for links to other pages or resources

### Implement ARIA Landmarks and Roles

Accessible Rich Internet Applications (ARIA) attributes help provide additional context to assistive technologies. When developing blocks, assign appropriate ARIA landmark roles to different areas of your block[3].

Common ARIA landmarks include:

```jsx
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

Only use ARIA when necessary and when native HTML semantics don't provide enough information. Remember the first rule of ARIA: don't use ARIA if you can use a native HTML element or attribute with the semantics and behavior you require.

### Ensure Keyboard Accessibility

All interactive elements in your blocks must be accessible via keyboard navigation. Users should be able to navigate through your blocks using the Tab key and interact with elements using Enter or Space keys[5][9].

```jsx
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

When implementing image blocks or components that include images, always provide a way for users to add alternative text[5]. This is crucial for screen reader users to understand the content and purpose of images.

```jsx
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

Ensure that text and interactive elements have sufficient color contrast against their backgrounds. The WCAG 2.1 AA standard requires a contrast ratio of at least 4.5:1 for normal text and 3:1 for large text[5].

When developing blocks, avoid hardcoding colors that might not meet contrast requirements. Instead, use WordPress theme colors or provide options for users to customize colors while warning them about potential contrast issues.

```jsx
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

When blocks include dynamic content that updates without a page reload, ensure that screen readers are informed of these changes. Use ARIA live regions for content that updates automatically[7].

```jsx
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

WordPress provides a `.screen-reader-text` CSS class that visually hides content while keeping it accessible to screen readers. This is useful for providing additional context to screen reader users without affecting the visual design[3].

```jsx
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

Developing accessible blocks is only half the battle. We also need to test our blocks to ensure they meet accessibility standards. Let's explore how to test blocks using screen readers and other accessibility testing tools.

### Testing with Screen Readers

Screen readers are assistive technologies that convert text into synthesized speech, allowing users with visual impairments to access digital content. Testing your blocks with screen readers is essential to ensure they're accessible to all users[7][9].

#### Popular Screen Readers

There are several screen readers available for testing:

1. **NVDA (NonVisual Desktop Access)** - Free, open-source screen reader for Windows
2. **VoiceOver** - Built into macOS and iOS
3. **JAWS (Job Access With Speech)** - Commercial screen reader for Windows
4. **TalkBack** - Built into Android devices

For WordPress development, testing with NVDA on Windows and VoiceOver on macOS will cover most use cases.

#### Screen Reader Testing Process

When testing blocks with screen readers, follow these steps[11]:

1. Browse to the page containing your block
2. Start the screen reader
3. Click in the browser address bar and press Enter to reload the page
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

In addition to manual testing with screen readers, several automated tools can help identify accessibility issues in your blocks[5][6][9].

#### WordPress Accessibility Testing Plugins

1. **WP Accessibility** - Helps fix common accessibility issues in WordPress themes

2. **Equalize Digital Accessibility Checker** - Scans your WordPress content for accessibility issues whenever you save or publish[10][13]

3. **Accessibility by UserWay** - Provides an accessibility widget that helps users customize their experience

4. **WP Accessibility Helper** - Offers a toolbar with accessibility options for users

#### Browser-Based Testing Tools

1. **WAVE (Web Accessibility Evaluation Tool)** - Browser extension that evaluates web content for accessibility issues

2. **axe DevTools** - Browser extension that identifies accessibility defects

3. **Lighthouse** - Built into Chrome DevTools, includes accessibility audits

4. **BrowserStack Accessibility Testing Tool** - Allows testing with screen readers on real devices[7]

Here's an example of how to integrate accessibility testing into your development workflow:

1. Use automated tools during development to catch common issues
2. Perform manual testing with screen readers before releasing blocks
3. Include accessibility testing in your continuous integration pipeline

```javascript
// Example of running accessibility tests with Jest and axe-core
import { render } from '@testing-library/react';
import { axe, toHaveNoViolations } from 'jest-axe';
import MyBlock from './MyBlock';

expect.extend(toHaveNoViolations);

test('MyBlock should have no accessibility violations', async () => {
    const { container } = render(<MyBlock />);
    const results = await axe(container);
    expect(results).toHaveNoViolations();
});

```

## Best Practices for Block Accessibility

To summarize, here are the key best practices for developing accessible blocks:

1. Use semantic HTML elements that accurately describe the content's purpose
2. Implement ARIA landmarks and roles when necessary
3. Ensure all interactive elements are keyboard accessible
4. Provide descriptive alternative text for images
5. Maintain sufficient color contrast
6. Handle dynamic content appropriately with ARIA live regions
7. Use the screen-reader-text class for content only meant for screen readers
8. Test blocks with both automated tools and screen readers
9. Follow the WordPress Accessibility Coding Standards

By following these practices, you'll create blocks that are accessible to all users, including those with disabilities, and meet the WCAG 2.1 accessibility standards required for WordPress development.

## Conclusion

Accessibility is not an afterthought but an essential aspect of block development. By designing and developing blocks with accessibility in mind from the start, we create a more inclusive web experience for all users. Remember that accessibility benefits everyone, not just users with disabilities. Features like keyboard navigation and clear content structure improve usability for all users.

As enterprise WordPress developers, we have a responsibility to ensure our blocks meet WCAG 2.1 accessibility standards. By understanding these standards, implementing accessible coding practices, and thoroughly testing our blocks, we can create a more inclusive web experience for everyone.

In the next lesson, we'll explore more advanced topics in block development, building on the accessibility principles we've covered here.

Citations:
[1] https://github.co
[2] https://developer.wordpress.org/coding-standards/wordpress-coding-standards/accessibility/
[3] https://webdevstudios.com/2020/06/04/custom-block-accessible/
[4] https://make.wordpress.org/accessibility/handbook/which-questions-should-you-ask/
[5] https://creativethemes.com/blocksy/blog/wordpress-accessibility-guide/
[6] https://equalizedigital.com/wordpress-page-builder-accessibility/
[7] https://www.browserstack.com/guide/test-websites-with-screen-readers
[8] https://smultron.software/blog/the-implementation-of-wcag-2-1-using-the-example-of-the-uczelniadostepna-pl-project
[9] https://www.wpexplorer.com/accessibility-testing-wordpress/
[10] https://www.hostinger.co.uk/tutorials/best-wordpress-accessibility-plugins
[11] https://doit.illinois.gov/initiatives/accessibility/testing/screen-reader-testing.html
[12] https://www.digitala11y.com/4-wordpress-plugins-to-enhance-the-accessibility-of-your-wordpress-website/
[13] https://blog.hubspot.com/website/wordpress-accessibility-plugin
[14] https://webaim.org/standards/wcag/checklist
[15] https://www.w3.org/WAI/WCAG21/Techniques/
[16] https://www.boia.org/blog/accessibility-tips-using-headings-in-wordpress
[17] https://utiawpguide.tennessee.edu/make-it-accessible/
[18] https://developer.wordpress.org/block-editor/how-to-guides/accessibility/
[19] https://make.wordpress.org/accessibility/2021/05/14/changes-in-wcag-2-1/
[20] https://wordpress.org/support/topic/improving-accessibility-on-wordpress-website-best-practices/
[21] https://accessibe.com/blog/knowledgebase/wordpress-website-accessibility
[22] https://bytes.co/blog/wordpress-accessibility-wcag-content-best-practices/
[23] https://pressable.com/blog/complete-guide-accessibility-wordpress/
[24] https://campuspress.com/wordpress-accessibility-guide/
[25] https://wplift.com/how-to-improve-wordpress-website-accessibility/
[26] https://support.microsoft.com/en-us/office/use-a-screen-reader-with-the-accessibility-checker-4be102bf-8bb1-47f6-a5b8-3e055aa5b342
[27] https://learn.wordpress.org/lesson/testing-your-content-for-accessibility-copy/
[28] https://make.wordpress.org/accessibility/handbook/which-tools-can-i-use/useful-tools/
[29] https://www.siteimprove.com/glossary/screen-reader-testing/
[30] https://equalizedigital.com/how-to-test-your-website-for-accessibility-problems-wordpress-accessibility-meetup/
[31] https://make.wordpress.org/accessibility/handbook/which-tools-can-i-use/other-plugins-to-improve-accessibility/
[32] https://www.w3schools.com/accessibility/accessibility_screen_readers.php
[33] https://www.linkedin.com/pulse/wordpress-accessibility-screen-reader-users-452022-alex-stine
[34] https://filter.agency/resources/accessibility/accessibility-for-wordpress/
[35] https://accessibility-manual.dwp.gov.uk/best-practice/screen-reader-testing
[36] https://make.wordpress.org/accessibility/handbook/test-for-web-accessibility/test-for-web-accessibility-frontend-code/
[37] https://equalizedigital.com/wordpress-accessibility-and-the-gutenberg-editor/
[38] https://www.deque.com/blog/wordpress-accessibility/
[39] https://oit.caes.uga.edu/accessibility-tips-for-wordpress-content-managers/
[40] https://www.w3.org/TR/WCAG21/

---
Answer from Perplexity: pplx.ai/share