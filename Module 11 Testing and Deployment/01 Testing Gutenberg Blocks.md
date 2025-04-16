# Testing Gutenberg Blocks

## Overview

Testing is a crucial part of the development lifecycle for Gutenberg blocks, especially in enterprise WordPress environments. Properly written tests ensure that blocks behave as expected under a variety of conditions and interactions. This lesson focuses on two primary testing strategies:

- **Unit testing with Jest**, which validates the logic and behavior of individual block components.
- **Integration (end-to-end) testing with Playwright**, which simulates real-world user interactions within the editor.

The examples in this lesson follow modern best practices and leverage `@wordpress/scripts` to run both test suites in a standardized way.

By the end of this lesson, you will be able to:

- Identify the correct type of test for a given scenario.
- Configure and run tests using WordPress-supported tools.
- Write both unit and end-to-end tests for your custom Gutenberg blocks.

---

## Unit Testing With Jest

Unit tests are responsible for validating individual JavaScript functions or React components used in your block. This allows you to verify block behavior in isolation.

### Running Unit Tests with `@wordpress/scripts`

When using `@wordpress/scripts`, Jest is already configured for unit testing. Ensure your `package.json` includes the following script:

```json
"scripts": {
  "test:unit": "wp-scripts test-unit-js"
}
```

When running `npm run test:unit`, Jest will look for [test files with `.test.js` suffix or test files under a `test` folder](https://developer.wordpress.org/block-editor/reference-guides/packages/packages-scripts/#test-unit-js).

> [!NOTE]
> For this section we'll use as a reference unit tests for the `copyright-date-block` block explained in the [Tutorial](https://developer.wordpress.org/block-editor/getting-started/tutorial/) of the Block Editor Handbook. The code of the block can be found [here](https://github.com/Automattic/wpvip-learn-enterprise-block-editor/tree/trunk/examples/copyright-date-block-with-tests) along with the [the unit tests](https://github.com/Automattic/wpvip-learn-enterprise-block-editor/tree/trunk/examples/copyright-date-block-with-tests/test/unit) referenced in this section.

### Testing Block Registration and Attributes

One of the first things we can test for a block is that it's properly registered and configured with the correct attributes and support settings, which are essential for the block to function correctly in the WordPress editor.

To do that we can leverage the following WordPress core methods:

- [`createBlock`](https://developer.wordpress.org/block-editor/reference-guides/packages/packages-blocks/#createblock),
- [`registerBlockType`](https://developer.wordpress.org/block-editor/reference-guides/packages/packages-blocks/#registerblocktype)
- [`unregisterBlockType`](https://developer.wordpress.org/block-editor/reference-guides/packages/packages-blocks/#unregisterblocktype)

> [!NOTE]
> The code of the `block.json` being tested with the code below is available [here](https://github.com/Automattic/wpvip-learn-enterprise-block-editor/blob/trunk/examples/copyright-date-block-with-tests/src/block.json)

**File: `test/unit/block-registration.test.js`**

```js
import {
  createBlock,
  registerBlockType,
  unregisterBlockType,
} from "@wordpress/blocks";

import blockJson from "../../src/block.json";
import "@testing-library/jest-dom";

describe("Block Registration", () => {
  beforeEach(() => {
    registerBlockType(
      "create-block/copyright-date-block-with-tests",
      blockJson
    );
  });

  afterEach(() => {
    unregisterBlockType("create-block/copyright-date-block-with-tests");
  });

  it("should create a block with default attributes", () => {
    const block = createBlock("create-block/copyright-date-block-with-tests");
    expect(block.name).toBe("create-block/copyright-date-block-with-tests");
    expect(block.attributes.fallbackCurrentYear).toBeUndefined();
    expect(block.attributes.showStartingYear).toBeUndefined();
    expect(block.attributes.startingYear).toBeUndefined();
  });

  it("should handle all valid attributes", () => {
    const attributes = {
      fallbackCurrentYear: "2024",
      showStartingYear: true,
      startingYear: "2020",
    };
    const block = createBlock(
      "create-block/copyright-date-block-with-tests",
      attributes
    );
    expect(block.attributes.fallbackCurrentYear).toBe("2024");
    expect(block.attributes.showStartingYear).toBe(true);
    expect(block.attributes.startingYear).toBe("2020");
  });

  it("should have correct block supports configuration", () => {
    const { supports } = blockJson;

    expect(supports.color.background).toBe(false);
    expect(supports.color.text).toBe(true);
    expect(supports.html).toBe(false);
    expect(supports.typography.fontSize).toBe(true);
  });
});
```

The test suite above focuses on teting three main aspects:

1. **Block Registration Setup**

   - Before each test, it registers the block using `registerBlockType`
   - After each test, it cleans up by unregistering the block with `unregisterBlockType`.
   - The block is identified by the name `create-block/copyright-date-block-with-tests`

2. **Block Attributes Testing**

   - Tests that a block can be created with default attributes (all undefined)
   - Tests that a block can be created with all valid attributes:
     - `fallbackCurrentYear` (string)
     - `showStartingYear` (boolean)
     - `startingYear` (string)

3. **Block Supports Configuration**
   - Verifies the block's support configuration:
     - No background color support
     - Text color support enabled
     - HTML support disabled
     - Font size support enabled

### Testing the Edit Component

The `Edit` component, as a React component, can be tested to ensure it provides the expected functionality in the Block Editor.

We can use [React Testing Library](https://testing-library.com/docs/react-testing-library/intro/) and Jest to verify both the visual output and the interactive functionality of the Edit component, ensuring it correctly handles user input and displays the output properly.

> [!NOTE]
> The code of the `edit.js` file being tested with the code below is available [here](https://github.com/Automattic/wpvip-learn-enterprise-block-editor/blob/trunk/examples/copyright-date-block-with-tests/src/edit.js)

**File: `test/unit/edit.test.js`**

```js
import { render, screen, fireEvent } from "@testing-library/react";
import userEvent from "@testing-library/user-event";
import "@testing-library/jest-dom";
import Edit from "../../src/edit";

const mockEditProps = {
  attributes: {
    fallbackCurrentYear: "2024",
    showStartingYear: false,
    startingYear: "",
  },
  setAttributes: jest.fn(),
  clientId: "test-id",
  className: "wp-block-copyright-date",
};

const originalDate = global.Date;
global.Date = class extends Date {
  getFullYear() {
    return 2024;
  }
};

beforeEach(() => {
  mockEditProps.setAttributes.mockClear();
  jest.clearAllMocks();
});

afterAll(() => {
  global.Date = originalDate;
});

describe("Edit Component", () => {
  it("matches snapshot", () => {
    const { container: renderedEditComponent } = render(
      <Edit {...mockEditProps} />
    );
    expect(renderedEditComponent).toMatchSnapshot();
  });

  it("displays current year by default", () => {
    render(<Edit {...mockEditProps} />);
    expect(screen.getByText(/© 2024/i)).toBeInTheDocument();
  });

  describe("Settings Panel", () => {
    it("renders settings panel", () => {
      render(<Edit {...mockEditProps} />);
      expect(screen.getByTestId("panel-body")).toHaveAttribute(
        "data-title",
        "Settings"
      );
    });

    it("toggles show starting year setting", async () => {
      const user = userEvent.setup();
      render(<Edit {...mockEditProps} />);
      const toggle = screen.getByLabelText("Show starting year");
      await user.click(toggle);
      expect(mockEditProps.setAttributes).toHaveBeenCalledWith({
        showStartingYear: true,
      });
    });

    it("shows starting year input when toggle is enabled", () => {
      const propsWithStartingYear = {
        ...mockEditProps,
        attributes: { ...mockEditProps.attributes, showStartingYear: true },
      };
      render(<Edit {...propsWithStartingYear} />);
      expect(screen.getByLabelText("Starting year")).toBeInTheDocument();
    });

    it("updates starting year when input changes", async () => {
      const setAttributes = jest.fn();
      const propsWithStartingYear = {
        ...mockEditProps,
        attributes: { ...mockEditProps.attributes, showStartingYear: true },
        setAttributes,
      };
      render(<Edit {...propsWithStartingYear} />);
      const input = screen.getByLabelText("Starting year");
      fireEvent.change(input, { target: { value: "2020" } });
      expect(setAttributes).toHaveBeenCalledWith({ startingYear: "2020" });
    });
  });

  describe("Display Logic", () => {
    it("displays year range when starting year is set", () => {
      const propsWithRange = {
        ...mockEditProps,
        attributes: {
          ...mockEditProps.attributes,
          showStartingYear: true,
          startingYear: "2020",
        },
      };
      render(<Edit {...propsWithRange} />);
      expect(screen.getByText(/© 2020–2024/i)).toBeInTheDocument();
    });

    it("displays only current year when starting year is not set", () => {
      render(<Edit {...mockEditProps} />);
      expect(screen.getByText(/© 2024/i)).toBeInTheDocument();
    });
  });
});
```

The test suite above focuses on testing four main aspects:

1. **Component Setup**

   - Mocks the Date object to return a fixed year (2024) for consistent testing
   - Sets up mock props including attributes and a mock `setAttributes` function
   - Cleans up mocks between tests

2. **Basic Rendering Tests**

   - Verifies the component matches a snapshot
   - Confirms the default display shows the current year (© 2024)

3. **Settings Panel Tests**

   - Verifies the settings panel renders correctly
   - Tests the "Show starting year" toggle functionality
   - Tests the starting year input field appears when toggle is enabled
   - Verifies that changing the starting year input updates the attributes

4. **Display Logic Tests**
   - Tests the year range display (e.g., "© 2020–2024") when a starting year is set
   - Verifies that only the current year is shown when no starting year is set

#### Snapshots

Snapshots are a feature of Jest that capture the rendered output of a component at a specific point in time. They serve as a "golden copy" of how your component should look.

```javascript
it("matches snapshot", () => {
  const { container: renderedEditComponent } = render(
    <Edit {...mockEditProps} />
  );
  expect(renderedEditComponent).toMatchSnapshot();
});
```

**How Snapshots Work:**

1. First Run

   - When you first run the test, Jest creates a snapshot file
   - This file contains the exact HTML structure of your rendered component
   - The snapshot is stored in a `__snapshots__` directory

2. Subsequent Runs
   - Jest compares the current render with the stored snapshot
   - If they match, the test passes
   - If they don't match, the test fails

**Updating snapshots**

If a snapshot test fails, it means the component's output changed.

- If it's a mistake, the test caught a bug
- If it's expected, update the snapshot by running:

```bash
# --testPathPattern is optional but will be much faster by only running matching tests
npm run test:unit -- --updateSnapshot --testPathPattern path/to/tests
```

#### Mocks

[Mocks](https://jestjs.io/docs/mock-functions) are fake objects or functions that simulate the behavior of real objects/functions in a controlled way. They're used in testing to:

- Isolate the code being tested
- Control the test environment
- Verify interactions
- Avoid side effects from real implementations

The `test/unit/edit.test.js` file includes internal mocks like `mockEditProps` and `global.Date`. Additionally, [the `__mocks__` folder](https://jestjs.io/docs/manual-mocks#mocking-user-modules) can define alternative versions of packages used by the `Edit` component to better support testing.

Below you can find an example of a mock of the `@wordpress/block-editor.js` package used in the `Edit` component of the `copyright-date-block` block

**File: `test/unit/__mocks__/@wordpress/block-editor.js`**

```js
const useBlockProps = jest.fn(() => ({
  className: `wp-block-copyright-date`.trim(),
}));

useBlockProps.save = jest.fn(useBlockProps);

const BlockControlsMock = ({ children }) => (
  <div data-testid="block-controls">{children}</div>
);

const InspectorControlsMock = ({ children }) => (
  <div data-testid="inspector-controls">{children}</div>
);

const RichTextMock = ({ value, tagName: Tag = "div", className, onChange }) => {
  if (onChange) {
    return (
      <Tag
        data-testid="rich-text"
        className={className}
        onChange={(e) => onChange(e.target.innerHTML)}
        dangerouslySetInnerHTML={{ __html: value }}
      />
    );
  }
  return (
    <Tag
      data-testid="rich-text-content"
      className={className}
      dangerouslySetInnerHTML={{ __html: value }}
    />
  );
};

RichTextMock.Content = ({ value, tagName: Tag = "div", className }) => (
  <Tag
    data-testid="rich-text-content"
    className={className}
    dangerouslySetInnerHTML={{ __html: value }}
  />
);

export {
  useBlockProps,
  BlockControlsMock as BlockControls,
  RichTextMock as RichText,
  InspectorControlsMock as InspectorControls,
};
```

The `data-testid` props are used to reliably access elements during testing, while [`jest.fn`](https://jestjs.io/docs/jest-object#jestfnimplementation) allows us to mock functions, enabling us to verify whether they were called and verify the arguments they were called with.

### Testing the Save Component

The `save` function, can be tested to ensure it returns the expected markup that will be stored in the database.

> [!NOTE]
> The code of the `save.js` file being tested with the code below is available [here](https://github.com/Automattic/wpvip-learn-enterprise-block-editor/blob/trunk/examples/copyright-date-block-with-tests/src/save.js)

**File: `test/unit/save.test.js`**

```js
import { render } from "@testing-library/react";
import "@testing-library/jest-dom";
import Save from "../../src/save";

describe.only("Save Component", () => {
  it("returns null when fallbackCurrentYear is not provided", () => {
    const testAttributes = {
      fallbackCurrentYear: null,
      showStartingYear: false,
      startingYear: null,
    };
    const { container } = render(<Save attributes={testAttributes} />);
    expect(container.firstChild).toBeNull();
  });

  it("displays only current year when showStartingYear is false", () => {
    const testAttributes = {
      fallbackCurrentYear: "2024",
      showStartingYear: false,
      startingYear: "2020",
    };
    const { container } = render(<Save attributes={testAttributes} />);
    expect(container.textContent).toBe("© 2024");
  });

  it("displays only current year when startingYear is not provided", () => {
    const testAttributes = {
      fallbackCurrentYear: "2024",
      showStartingYear: true,
      startingYear: null,
    };
    const { container } = render(<Save attributes={testAttributes} />);
    expect(container.textContent).toBe("© 2024");
  });

  it("displays year range when both showStartingYear and startingYear are provided", () => {
    const testAttributes = {
      fallbackCurrentYear: "2024",
      showStartingYear: true,
      startingYear: "2020",
    };
    const { container } = render(<Save attributes={testAttributes} />);
    expect(container.textContent).toBe("© 2020–2024");
  });

  it("applies correct block props", () => {
    const testAttributes = {
      fallbackCurrentYear: "2023",
      showStartingYear: false,
      startingYear: "",
    };
    const { container } = render(<Save attributes={testAttributes} />);
    const paragraph = container.querySelector("p");
    expect(paragraph).toBeTruthy();
    expect(paragraph).toHaveClass("wp-block-copyright-date");
  });
});
```

This test suite above focuses on five main scenarios for how the markup of the `copyright-date-block` should be returned.

1. **Empty State Handling**

   - Tests that the component returns `null` when `fallbackCurrentYear` is not provided
   - This is a safety check to prevent rendering invalid content

2. **Single Year Display**

   - Tests that only the current year is shown (e.g., "© 2024") when:
     - `showStartingYear` is false, even if `startingYear` is provided
     - `startingYear` is not provided, even if `showStartingYear` is true

3. **Year Range Display**

   - Tests that a year range is shown (e.g., "© 2020–2024") when:
     - Both `showStartingYear` and `startingYear` are provided
     - This is the full copyright range display

4. **Block Styling**
   - Tests that the correct CSS classes are applied
   - Verifies the block has the proper WordPress block class (`wp-block-copyright-date`)
   - Ensures the content is wrapped in a paragraph tag

The tests use [React Testing Library](https://testing-library.com/docs/react-testing-library/intro/) to verify both the content and structure of the saved output, ensuring the block will display correctly on the front end of the WordPress site.

### Debugging Unit Tests

For debugging unit tests in your block development, you can set the following script in your `package.json`

```bash
"test:unit:debug": "wp-scripts --inspect-brk test-unit-js --runInBand --no-cache",
```

With that, you can run tests in debug mode:

1. Run `npm run test:unit:debug`
2. Open Chrome and navigate to `chrome://inspect`
3. Click "Open dedicated DevTools for Node"
4. Add `debugger` statements in your test files where you want to pause execution

You can also isolate specific tests during debugging using `.only` or skip tests by adding `.skip` to `describe()` or `test()`:

```javascript
describe.only("my test suite", () => {
  test("specific test", () => {
    // Your test code
  });
});
```

## Integration Testing With Playwright

Integration or end-to-end (E2E) testing simulates how a user interacts with a block within the actual editor. For this, [Playwright](https://playwright.dev/) is a recommended tool, which is integrated into the `@wordpress/scripts` test suite.

### Installing Playwright and Dependencies

Ensure your `package.json` includes `@wordpress/scripts` and `@wordpress/env` as `devDependencies`.

The following scripts are recommended for a comprehensive E2E testing workflow:

```json
"scripts": {
  "wp-env": "wp-env",
  "env:start": "wp-env start",
  "env:stop": "wp-env stop",
  "test:e2e": "wp-scripts test-playwright",
  "test:e2e:help": "wp-scripts test-playwright --help",
  "test:e2e:debug": "wp-scripts test-playwright --debug",
  "test:e2e:ui": "wp-scripts test-playwright --ui",
  "test:e2e:headed": "wp-scripts test-playwright --headed"
}
```

Setting up E2E tests requires two main configuration files that define how tests will be executed and in what environment:

1. `playwright.config.js`: This file configures how Playwright executes your tests

```javascript
/* eslint-disable import/no-extraneous-dependencies */
// @ts-check
const path = require("path");
const { defineConfig } = require("@playwright/test");
const wpScriptsPlaywrightConfig = require("@wordpress/scripts/config/playwright.config.js");

/**
 * @see https://playwright.dev/docs/test-configuration
 */
module.exports = defineConfig({
  ...wpScriptsPlaywrightConfig,
  testDir: "./test/e2e",
  use: {
    ...wpScriptsPlaywrightConfig.use,
    baseURL: process.env.WP_BASE_URL || "http://localhost:8890",
    storageState: path.join(process.cwd(), "artifacts/storage-state.json"),
  },
});
```

2. `.wp-env.json`: This file defines your WordPress testing environment configuration

```json
{
  "core": "WordPress/WordPress",
  "plugins": ["."],
  "env": {
    "development": {
      "port": 8889
    },
    "tests": {
      "port": 8890
    }
  },
  "config": {
    "WP_DEBUG": true,
    "SCRIPT_DEBUG": true,
    "WP_ENVIRONMENT_TYPE": "development",
    "WP_AUTO_LOGIN": true
  }
}
```

### Running E2E Tests

To run E2E tests you can do:

```bash
# Start WordPress environment (required before running tests)
npm run env:start

# Run tests in headless mode (CI-friendly)
npm run test:e2e
# This mode is perfect for CI pipelines and quick test runs
```

This will run JS files at a `spec` folder or files with the `.spec.js` suffix.

Playwright offers several modes for running tests, each suited for different scenarios:

```bash
# Run tests with UI for debugging (recommended during development)
npm run test:e2e:debug
# Opens Playwright's UI mode where you can:
# - See test execution in real-time
# - Debug step by step
# - View test reports
# - Rerun specific tests

# Run tests in headed mode (visible browser)
npm run test:e2e:headed
# Useful for:
# - Watching test execution in real browser
# - Debugging visual issues
# - Understanding test flow
```

Some good practices for working with Playwright E2E tests are:

1. Use UI mode (`test:e2e:debug`) during test development for better debugging
2. Add descriptive test names and comments
3. Use `page.pause()` in UI mode to pause execution at specific points
4. Leverage Playwright's built-in retry mechanisms for flaky tests
5. Group related tests using `describe` blocks
6. Use `beforeEach` and `afterEach` for common setup and cleanup

> [!NOTE]
> For this section we'll use as a reference unit tests for the `copyright-date-block` block explained in the [Tutorial](https://developer.wordpress.org/block-editor/getting-started/tutorial/) of the Block Editor Handbook. The code of the block can be found [here](https://github.com/Automattic/wpvip-learn-enterprise-block-editor/tree/trunk/examples/copyright-date-block-with-tests) along with the [the E2E tests](https://github.com/Automattic/wpvip-learn-enterprise-block-editor/tree/trunk/examples/copyright-date-block-with-tests/test/e2e) referenced in this section.

### Testing the Block's Behavior in the WordPress Editor

With E2E tests we can test the Block's behavior in the environment of the Block Editor.

Let's check the following example:

[**File: `test/e2e/editor.spec.js`**](https://github.com/Automattic/wpvip-learn-enterprise-block-editor/blob/trunk/examples/copyright-date-block-with-tests/test/e2e/editor.spec.js)

```js
const { test, expect } = require("@wordpress/e2e-test-utils-playwright");
const {
  setupPostWithCopyrightBlock,
  setupPostWithStartingYearCopyrightBlock,
} = require("./utils");

test.describe("Copyright Date Block Editor Test Suite", () => {
  test.beforeEach(async ({ admin }) => {
    await admin.visitAdminPage("index.php");
  });

  test("should insert a copyright date block and verify it's visible", async ({
    admin,
    editor,
    page,
  }) => {
    await setupPostWithCopyrightBlock({ admin, editor, page });
    const editorIframe = page.frameLocator('iframe[name="editor-canvas"]');
    await expect(
      editorIframe.locator(
        '[data-type="create-block/copyright-date-block-with-tests"]'
      )
    ).toBeVisible();
  });

  test("should display the current year in the block by default", async ({
    admin,
    editor,
    page,
  }) => {
    await setupPostWithCopyrightBlock({ admin, editor, page });
    const currentYear = new Date().getFullYear().toString();
    const editorIframe = page.frameLocator('iframe[name="editor-canvas"]');
    const blockInIframe = editorIframe.locator(
      '[data-type="create-block/copyright-date-block-with-tests"]'
    );

    await expect(blockInIframe).toBeVisible();
    await expect(blockInIframe).toContainText(`© ${currentYear}`);
    await expect(page.locator(".components-snackbar")).toContainText(
      "Draft saved",
      {
        message: "Post should be saved successfully",
      }
    );

    const postContent = await editor.getEditedPostContent();
    expect(postContent).toContain(
      "<!-- wp:create-block/copyright-date-block-with-tests"
    );
    expect(postContent).toContain(currentYear);
  });

  test("should add starting year to the copyright block and verify it's saved", async ({
    admin,
    editor,
    page,
  }) => {
    const startingYear = "2020";
    await setupPostWithStartingYearCopyrightBlock(
      { admin, editor, page },
      startingYear
    );
    const currentYear = new Date().getFullYear().toString();
    const editorIframe = page.frameLocator('iframe[name="editor-canvas"]');
    const blockInIframe = editorIframe.locator(
      '[data-type="create-block/copyright-date-block-with-tests"]'
    );

    await expect(blockInIframe).toBeVisible();
    await expect(blockInIframe).toContainText(
      `© ${startingYear}–${currentYear}`
    );
    await expect(page.locator(".components-snackbar")).toContainText(
      "Draft saved",
      {
        message: "Post should be saved successfully",
      }
    );

    const postContent = await editor.getEditedPostContent();
    expect(postContent).toContain(
      "<!-- wp:create-block/copyright-date-block-with-tests"
    );
    expect(postContent).toContain(`"startingYear":"${startingYear}"`);
    expect(postContent).toContain('"showStartingYear":true');
  });

  test("should toggle off starting year and verify only current year is displayed", async ({
    admin,
    editor,
    page,
  }) => {
    const startingYear = "2020";
    await setupPostWithStartingYearCopyrightBlock(
      { admin, editor, page },
      startingYear
    );
    const editorIframe = page.frameLocator('iframe[name="editor-canvas"]');
    const copyrightBlock = editorIframe.locator(
      '[data-type="create-block/copyright-date-block-with-tests"]'
    );

    await expect(copyrightBlock).toBeVisible();
    await editor.selectBlocks(copyrightBlock);
    await page.waitForTimeout(300);
    await editor.openDocumentSettingsSidebar();

    const showStartingYearToggle = page.locator(
      '.components-toggle-control label:has-text("Show starting year")'
    );
    await expect(showStartingYearToggle).toBeVisible();
    await showStartingYearToggle.click();

    const currentYear = new Date().getFullYear().toString();
    await page.waitForTimeout(300);
    await expect(copyrightBlock).toContainText(`© ${currentYear}`);
    await expect(copyrightBlock).not.toContainText(startingYear);
  });
});
```

Let's break down the main components and test cases of the test above:

**Test Setup**:

- The test uses `@wordpress/e2e-test-utils-playwright` for testing
- It includes utility functions from `./utils` for setting up test posts with copyright blocks
- Tests run in the WordPress admin environment

**Test Suite Structure**:

- The test suite contains 4 main test cases:

_Basic Block Insertion Test_

```javascript
test("should insert a copyright date block and verify it's visible");
```

- Creates a new post with a copyright block
- Verifies the block is visible in the editor iframe
- Uses the editor canvas iframe to locate and check the block

_Current Year Display Test_

```javascript
test("should display the current year in the block by default");
```

- Creates a post with a copyright block
- Verifies the block shows the current year (e.g., "© 2024")
- Checks both the visual display and the saved post content
- Ensures the post saves successfully

_Starting Year Test_

```javascript
test("should add starting year to the copyright block and verify it's saved");
```

- Creates a post with a copyright block that includes a starting year (2020)
- Verifies the block shows a year range (e.g., "© 2020–2024")
- Checks the saved post content includes the starting year attribute
- Verifies the `showStartingYear` setting is saved correctly

_Starting Year Toggle Test_

```javascript
test(
  "should toggle off starting year and verify only current year is displayed"
);
```

- Creates a post with a starting year enabled
- Selects the block and opens the settings sidebar
- Toggles off the "Show starting year" option
- Verifies only the current year is displayed
- Ensures the starting year is no longer visible

As the Block Editor is displayed as an iFrame in the Post Editor (`page.frameLocator('iframe[name="editor-canvas"]')`) we need to locate the block inside the iFrame

**Key Testing Patterns:**

- Uses iframe handling for the editor canvas
- Implements proper waiting mechanisms for UI updates
- Verifies both visual elements and saved content
- Tests both block insertion and configuration
- Includes save state verification

**Technical Details:**

- Uses Playwright's test framework
- Handles WordPress admin interface
- Manages editor iframe interactions
- Implements proper async/await patterns
- Includes error handling and timeout management

### Testing Block's rendering in the Frontend

With E2E tests we can test the Block's behavior in the frontend.

Let's check the following example:

[**File: `test/e2e/frontend.spec.js`**](https://github.com/Automattic/wpvip-learn-enterprise-block-editor/blob/trunk/examples/copyright-date-block-with-tests/test/e2e/frontend.spec.js)

```js
const { test, expect } = require("@wordpress/e2e-test-utils-playwright");
const {
  setupPostWithCopyrightBlock,
  setupPostWithStartingYearCopyrightBlock,
  navigateToPostFrontend,
} = require("./utils");

test.describe("Copyright Date Block Frontend Test Suite", () => {
  test("should display the copyright block with current year correctly on the frontend", async ({
    admin,
    editor,
    page,
  }) => {
    await setupPostWithCopyrightBlock({ admin, editor, page });
    await editor.publishPost();
    await navigateToPostFrontend({ page, test });

    const copyrightBlock = page.locator(
      ".wp-block-create-block-copyright-date-block-with-tests"
    );
    await expect(copyrightBlock).toBeVisible({ timeout: 30000 });

    const currentYear = new Date().getFullYear().toString();
    await expect(copyrightBlock).toContainText(`© ${currentYear}`, {
      timeout: 10000,
    });
  });

  test("should display copyright block with starting year and current year on the frontend", async ({
    admin,
    editor,
    page,
  }) => {
    const startingYear = "2020";
    await setupPostWithStartingYearCopyrightBlock(
      { admin, editor, page },
      startingYear
    );
    await editor.publishPost();
    await navigateToPostFrontend({ page, test });

    const copyrightBlock = page.locator(
      ".wp-block-create-block-copyright-date-block-with-tests"
    );
    await expect(copyrightBlock).toBeVisible({ timeout: 30000 });

    const currentYear = new Date().getFullYear().toString();
    await expect(copyrightBlock).toContainText(
      `© ${startingYear}–${currentYear}`,
      { timeout: 10000 }
    );
  });

  test("should verify dynamic rendering shows current year even when post was published in a different year", async ({
    admin,
    editor,
    page,
  }) => {
    await setupPostWithCopyrightBlock({ admin, editor, page });

    const editorIframe = page.frameLocator('iframe[name="editor-canvas"]');
    await page.waitForTimeout(1000);
    const copyrightBlockEditor = editorIframe.locator(
      '[data-type="create-block/copyright-date-block-with-tests"]'
    );
    await expect(copyrightBlockEditor).toBeVisible({ timeout: 15000 });
    const initialContent = await copyrightBlockEditor.textContent();

    await editor.publishPost();
    await navigateToPostFrontend({ page, test });

    const copyrightBlock = page.locator(
      ".wp-block-create-block-copyright-date-block-with-tests"
    );
    await expect(copyrightBlock).toBeVisible({ timeout: 30000 });

    const currentYear = new Date().getFullYear().toString();
    await expect(copyrightBlock).toContainText(`© ${currentYear}`, {
      timeout: 10000,
    });
    const content = await copyrightBlock.textContent();
    expect(content).toBe(initialContent);
  });

  test("should respect typography settings on the frontend", async ({
    admin,
    editor,
    page,
  }) => {
    await setupPostWithCopyrightBlock({ admin, editor, page });

    const editorIframe = page.frameLocator('iframe[name="editor-canvas"]');
    const copyrightBlock = editorIframe.locator(
      '[data-type="create-block/copyright-date-block-with-tests"]'
    );
    await expect(copyrightBlock).toBeVisible({ timeout: 15000 });
    await editor.selectBlocks(copyrightBlock);
    await page.waitForTimeout(500);

    await editor.openDocumentSettingsSidebar();
    await page.getByRole("tab", { name: "Styles" }).click();
    await page.getByRole("button", { name: "Text" }).click();
    await page.getByRole("option", { name: "Accent 3" }).click();
    await page.getByRole("radio", { name: "Large", exact: true }).click();

    await editor.publishPost();
    await navigateToPostFrontend({ page, test });

    const frontendCopyrightBlock = page.locator(
      ".wp-block-create-block-copyright-date-block-with-tests"
    );
    await expect(frontendCopyrightBlock).toBeVisible();
    await expect(frontendCopyrightBlock).toHaveClass(/has-accent-3-color/);
    await expect(frontendCopyrightBlock).toHaveClass(/has-large-font-size/, {
      timeout: 10000,
    });
  });
});
```

The test suite above focuses on 4 main test cases:

**Basic Frontend Display Test**:

```javascript
test(
  "should display the copyright block with current year correctly on the frontend"
);
```

- Creates a post with a copyright block
- Publishes the post
- Navigates to the frontend
- Verifies the block is visible
- Checks that it displays the current year correctly (e.g., "© 2024")
- Uses direct page locators (not iframe) since it's testing the frontend

**Starting Year Frontend Test**:

```javascript
test(
  "should display copyright block with starting year and current year on the frontend"
);
```

- Creates a post with a copyright block that includes a starting year (2020)
- Publishes the post
- Navigates to the frontend
- Verifies the block shows both years in the correct format (e.g., "© 2020–2024")
- Uses longer timeouts (30s) to account for frontend loading

**Dynamic Year Update Test**:

```javascript
test(
  "should verify dynamic rendering shows current year even when post was published in a different year"
);
```

- Tests the dynamic nature of the copyright year
- Creates and publishes a post with the block
- Captures the initial content
- Verifies that the frontend always shows the current year
- Ensures the block updates dynamically without needing to republish
- Tests the render.php functionality

**Typography Settings Test**:

```javascript
test("should respect typography settings on the frontend");
```

- Creates a post with the copyright block
- Configures block settings in the editor:
  - Sets text color to "Accent 3"
  - Sets font size to "Large"
- Publishes the post
- Navigates to the frontend
- Verifies the styling is correctly applied:
  - Checks for the correct color class (`has-accent-3-color`)
  - Checks for the correct font size class (`has-large-font-size`)

Key Technical Aspects:

- Uses `@wordpress/e2e-test-utils-playwright` for testing
- Implements proper navigation between admin and frontend
- Handles different contexts (editor vs frontend)
- Uses appropriate timeouts for frontend loading
- Tests both content and styling
- Verifies dynamic content updates
- Includes proper error handling and waiting mechanisms

This test suite ensures that the copyright block:

- Displays correctly on the frontend
- Shows the correct year(s)
- Updates dynamically
- Maintains styling settings
- Works properly after publishing

---

## Summary

Unit and integration tests form a complementary strategy for validating the quality of Gutenberg blocks. Unit tests ensure that individual components work as expected in isolation, while integration tests simulate how a user will interact with your block in the actual WordPress editor.

By leveraging `@wordpress/scripts`, you can streamline your testing workflows using:

```bash
npm run test:unit   # For Jest unit tests
npm run test:e2e # For Playwright end-to-end tests
```

This setup provides full coverage from implementation logic to user experience, ensuring your blocks remain stable and maintainable over time.
