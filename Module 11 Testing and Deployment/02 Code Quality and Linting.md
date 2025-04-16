# Code Quality and Linting

## Introduction

Ensuring consistent and maintainable code is critical in large-scale enterprise WordPress projects, particularly when working with the Block Editor (Gutenberg). In this lesson, we will explore two core pillars of code quality: linting and source control practices. Specifically, we will delve into how ESLint and Prettier can be integrated into WordPress block development, and how advanced Git workflows can be employed to streamline team collaboration, code review, and feature integration.

## Implementing ESLint and Prettier for Maintaining Code Standards

In enterprise WordPress projects, especially those involving custom block development, adhering to consistent coding standards is essential to maintain readability, reduce bugs, and simplify onboarding for new team members. ESLint and Prettier are two of the most popular tools in the JavaScript ecosystem that help enforce code quality:

- **ESLint** is a linting utility that analyzes JavaScript/JSX code to detect problematic patterns, coding style issues, and potential errors based on configured rules.
- **Prettier** is an opinionated code formatter that ensures stylistic consistency by automatically formatting your code according to a predefined set of rules.

### Using `@wordpress/scripts` for Linting

WordPress provides a convenient utility package, `@wordpress/scripts`, which bundles ESLint, Prettier, and other build tools with sensible defaults for block development. By leveraging `@wordpress/scripts`, teams can avoid manual setup of individual tools and adhere to the official coding standards more easily.

To begin, install the package in your block plugin project:

```bash
npm install --save-dev @wordpress/scripts
```

Then, update your `package.json` to include linting and formatting commands using the built-in scripts:

```json
"scripts": {
  "lint:js": "wp-scripts lint-js",
  "format:js": "wp-scripts format-js",
}
```

These scripts use the configuration defined by `@wordpress/eslint-plugin` and `@wordpress/prettier-config` by default, providing an out-of-the-box solution that aligns with WordPress development standards.

To lint your JavaScript code:

```bash
npm run lint
```

To automatically fix linting issues:

```bash
wp-scripts lint-js . --fix
```

To format JavaScript files using Prettier:

```bash
npm run format
```

This method reduces configuration overhead and helps standardize tooling across multiple teams and projects.

### Custom Configuration (Optional)

By default, files in directories like `node_modules`, `build`, and `vendor` are ignored. You can [customize ESLint rules](https://eslint.org/docs/latest/use/configure/) by creating an `.eslintrc` file in your project root.

**Example `.eslintrc`:**

```javascript
{
  "extends": ["plugin:@wordpress/eslint-plugin/recommended"],
  "rules": {
    "no-console": "warn",
    "prefer-template": "error",
    "jsdoc/no-undefined-types": "off"
  },
  "ignorePatterns": ["_bin/**/*", "_app/**/*"]
}
```

This configuration enforces best practices like avoiding console logs and using template literals.

You can customize Prettier options in a `.prettierrc` file

**Example `.prettierrc`:**

```json
// Import the default config file and expose it in the project root.
// Configure arrays to display each element on a new line
const config = {
	...require( '@wordpress/prettier-config' ),
	overrides: [
		{
			files: [ '*.json' ],
			options: {
				parser: 'json',
				printWidth: 1,
			},
		},
		// Configure SCSS files to use double quotes
		{
			files: [ '*.scss' ],
			options: {
				singleQuote: false,
			},
		},
	],
};
// Useful for editor integrations.
module.exports = config;
```

Editor integration is also supported—extensions for ESLint and Prettier in editors like Visual Studio Code will pick up these configuration files and provide real-time linting and formatting feedback.

### Other lintings

Your `package.json` can also include scripts to lint styles, JSON and markdown using built-in `wp-scripts` commands:

```json
"scripts": {
    ...
    "lint:css": "wp-scripts lint-style",
    "lint:pkg-json": "wp-scripts lint-pkg-json",
     "lint:md:docs": "wp-scripts lint-md-docs"
}
```

- `lint:css` scripts lints all CSS files in the project using [Stylelint](https://stylelint.io/)
- `lint:pkg-json` script checks for common issues like missing fields or incorrect formatting in your `package.json`.
- `lint:md:docs` lints markdown files in the entire project's directories.

You can [customize Stylelint rules](https://stylelint.io/user-guide/configure) by creating a `.stylelintrc` file.

**Example `.stylelintrc.js`:**

```JSON
{
  "extends": ["@wordpress/stylelint-config/scss"],
  "rules": {
    "max-nesting-depth": 3,
    "selector-class-pattern": "^[a-z]+(-[a-z]+)*$"
  }
}
```

This configuration limits nesting depth to three levels and enforces kebab-case class names.

> [!NOTE]
> Take a look at [this CI workflow configuration](https://github.com/Automattic/wpvip-learn-enterprise-block-editor/blob/trunk/.github/workflows/ci.yml) and [its latest run results](https://github.com/Automattic/wpvip-learn-enterprise-block-editor/actions/workflows/ci.yml).

## Advanced Git Workflows for Managing Block Development in Large Teams

### Challenges in Enterprise Git Workflows

As teams grow, so does the complexity of managing parallel development, long-lived features, and maintaining a stable codebase. Common challenges include:

- Avoiding merge conflicts during simultaneous feature development
- Enforcing review processes and quality gates
- Ensuring CI/CD pipelines validate code before merging

### Git Flow and Trunk-Based Development

Two widely used Git workflows in enterprise teams are **Git Flow** and **Trunk-Based Development**.

#### Git Flow

This workflow defines strict branching models:

- `main` (production)
- `develop` (integration)
- `feature/*`, `release/*`, `hotfix/*`

Feature branches are merged into `develop`, which periodically releases to `main`. This model suits teams with longer release cycles and strict QA processes.

#### Trunk-Based Development

This workflow encourages short-lived branches that are frequently merged into a shared `main` (or `trunk`) branch. All commits are expected to pass automated tests and code reviews before merging. Feature flags or toggles are used to avoid releasing incomplete features.

This approach is increasingly favored in block editor development because of its focus on rapid integration, continuous delivery, and reduced merge debt.

### Feature Branching and Pull Requests

In both models, feature branches and pull requests (PRs) are essential tools:

- **Feature Branches**: Isolated branches for working on individual blocks or components.
- **Pull Requests**: Facilitate peer review, automated linting, and test execution.

Use the following practices in enterprise PR workflows:

- Require approvals from at least two reviewers.
- Integrate ESLint/Prettier checks in CI (e.g., GitHub Actions).
- Use status checks to block merging unless tests and linters pass.

_GitHub Action example_

```
name: Static Code Analysis

on:
  pull_request:
    branches: [main, develop]
  push:
    branches: [main, develop]

jobs:
  lint:
    name: Lint Code
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: "18"
          cache: "npm"

      - name: Install dependencies
        run: npm ci

      - name: Run ESLint
        run: npm run lint:js
        env:
          CI: true

      - name: Run Stylelint
        run: npm run lint:style
        env:
          CI: true
```

The GitHub Action workflow above automates static code analysis for WordPress block editor projects. It runs on pull requests and pushes to main/develop branches, executing ESLint and Stylelint checks to maintain code quality and enforce WordPress coding standards.

### Conventional Commits for Block Development

The [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/) specification provides a standardized format for commit messages that helps teams maintain a clear and consistent commit history. This is particularly valuable in block development where changes often affect multiple components and need to be tracked for versioning and changelog generation.

#### Commit Message Structure

A conventional commit message follows this format:

```
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

Common types for block development include:

- `feat`: New block or feature
- `fix`: Bug fixes
- `docs`: Documentation changes
- `style`: Code style changes
- `refactor`: Code refactoring
- `test`: Adding or modifying tests
- `chore`: Maintenance tasks

#### Benefits for Block Development

1. **Automated Versioning**

   - `feat` commits trigger minor version bumps
   - `fix` commits trigger patch version bumps
   - Breaking changes trigger major version bumps

2. **Changelog Generation**

   - Automated tools can parse commit messages
   - Generate detailed changelogs for releases
   - Track breaking changes and new features

3. **Team Communication**
   - Clear documentation of changes
   - Consistent commit history
   - Better code review process

#### Implementation Example

```bash
# Feature addition
feat(blocks): add new testimonial block component

# Bug fix
fix(blocks): resolve alignment issues in column block

# Breaking change
feat(blocks)!: remove deprecated block attributes

BREAKING CHANGE: The following attributes are no longer supported:
- oldAttribute
- deprecatedSetting
```

Tools like [standard-version](https://github.com/conventional-changelog/standard-version), [semantic-release](https://github.com/semantic-release/semantic-release), and [release-it](https://github.com/release-it/release-it) can automate version management and changelog generation based on your commit history:

- **standard-version**: A simple tool that automatically bumps version numbers and generates changelogs based on commit messages.
- **semantic-release**: A fully automated version management and package publishing tool that determines the next version number, generates changelogs, and publishes the package.
- **release-it**: A flexible tool that can handle version management, changelog generation, and publishing, with support for conventional commits.

These tools integrate well with CI/CD pipelines and can be configured to work with WordPress block development workflows, ensuring consistent versioning and documentation across your project.

## Conclusion

By the end of this lesson, you should now understand how to set up ESLint and Prettier using `@wordpress/scripts` to enforce coding standards within a WordPress block development environment. You should also be familiar with advanced Git workflows that facilitate clean, scalable collaboration in large teams. These practices form the foundation for reliable and maintainable code in modern WordPress enterprise development.

In the next lesson, we will explore how to integrate these tools into a continuous integration and deployment (CI/CD) pipeline for fully automated testing and deployment processes.
