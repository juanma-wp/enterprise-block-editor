## Continuous Integration and Deployment for Gutenberg Block Development

In modern enterprise WordPress development, delivering robust and maintainable block-based features using the Block Editor (Gutenberg) demands automation, consistency, and repeatability. Implementing CI/CD (Continuous Integration and Continuous Deployment) pipelines is a critical practice that ensures code quality, minimizes human error, and enables seamless deployments across development, staging, and production environments.

This lesson explores practical strategies for setting up and managing CI/CD pipelines specifically tailored for Gutenberg block development. It highlights how automation can enhance team collaboration, enforce version control standards, and accelerate the delivery of high-quality blocks in enterprise-grade WordPress environments.

## Setting up CI/CD pipelines for Gutenberg development

Continuous Integration involves automating the process of testing and validating code changes before they are merged into a shared repository. Continuous Deployment automates the process of pushing these validated changes into production or staging environments. Together, CI/CD ensures that each code change is reliable, tested, and ready for deployment.

In enterprise WordPress environments, CI/CD pipelines for Gutenberg development typically manage the following tasks:

- Linting and static code analysis of JavaScript and PHP code
- Running automated tests (unit, integration, or end-to-end)
- Building and bundling JavaScript and CSS assets
- Versioning block assets for cache busting and traceability
- Deploying code and compiled assets to staging or production servers

Implementing CI/CD in Gutenberg development improves collaboration, minimizes deployment risks, and supports Agile delivery practices.

### GitHub Actions

To set up a CI/CD pipeline for Gutenberg blocks, you will typically use tools such as GitHub Actions, GitLab CI, CircleCI, or Bitbucket Pipelines. This section focuses on GitHub Actions, but the concepts are transferable to other platforms.

[GitHub Actions](https://docs.github.com/en/actions) is a powerful continuous integration and continuous delivery (CI/CD) platform that enables you to automate your build, test, and deployment pipeline. It allows you to create [custom workflows](https://docs.github.com/en/actions/using-workflows) directly in your GitHub repository, where each workflow is defined by a [YAML file](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions) that specifies the sequence of jobs and steps to execute. GitHub Actions provides a rich ecosystem of [pre-built actions](https://github.com/marketplace?type=actions) that can be combined to create sophisticated automation workflows, while also supporting [custom actions](https://docs.github.com/en/actions/creating-actions) for specialized tasks. This flexibility makes it an ideal choice for automating Gutenberg block development workflows, from code quality checks to deployment processes.

**Example: GitHub Actions Workflow for Gutenberg Blocks**

To trigger the workflows defined under `jobs` when the specified `on` events occur, place a `.yml` file with the following content in the repository's `.github/workflows` directory.

```yaml
name: CI/CD Pipeline

on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Repository
        uses: actions/checkout@v3

      - name: Set Up Node.js
        uses: actions/setup-node@v3
        with:
          node-version: "18"

      - name: Install Dependencies
        run: npm ci

      - name: Lint JavaScript
        run: npm run lint:js

      - name: Build Block Assets
        run: npm run build

      - name: Upload Artifact
        uses: actions/upload-artifact@v3
        with:
          name: block-assets
          path: build/
```

The GitHub Action (CI/CD pipeline) above runs automatically on pushes to main or pull requests targeting main, and do the following things:

- Verifies the code can build successfully
- Checks code quality with linters
- Creates production-ready assets
- Makes those assets available for deployment

This workflow could be extended by adding PHP code checks (`phpcs`), PHPUnit tests, or integration with deployment tools.

> [!NOTE]
> Take a look at [this CI workflow configuration](https://github.com/Automattic/wpvip-learn-enterprise-block-editor/blob/trunk/.github/workflows/ci.yml) and [its latest run results](https://github.com/Automattic/wpvip-learn-enterprise-block-editor/actions/workflows/ci.yml).

### Git Hooks and Husky Integration

While CI/CD pipelines run on remote servers, Git hooks provide a way to enforce code quality and run automated checks locally before code is pushed to the repository. [Husky](https://typicode.github.io/husky/) is a modern tool that makes it easy to manage Git hooks in your Gutenberg block development workflow.

#### Setting Up Husky for Gutenberg Development

To integrate Husky into your project:

1. Install Husky and its dependencies:

```bash
npm install husky --save-dev
npm pkg set scripts.prepare="husky install"
npm run prepare
```

2. Install and configure commitlint for commit message validation:

```bash
npm install --save-dev @commitlint/cli @commitlint/config-conventional
```

Create a `commitlint.config.js` file in your project root:

```javascript
module.exports = {
  extends: ["@commitlint/config-conventional"],
  rules: {
    "type-enum": [
      2,
      "always",
      ["feat", "fix", "docs", "style", "refactor", "test", "chore"],
    ],
  },
};
```

3. Create a pre-commit hook to run linting and tests:

```bash
npx husky add .husky/pre-commit "npm run lint:js && npm test"
```

4. Create a commit-msg hook to enforce commit message conventions:

```bash
npx husky add .husky/commit-msg 'npx --no -- commitlint --edit "$1"'
```

### Example Husky Configuration

A typical Husky configuration for Gutenberg block development might include:

```json
{
  "husky": {
    "hooks": {
      "pre-commit": "npm run lint:js && npm test",
      "commit-msg": "commitlint -E HUSKY_GIT_PARAMS",
      "pre-push": "npm run build"
    }
  }
}
```

#### Husky Configuration and Best Practices

Husky enhances Gutenberg block development through several key features and practices:

1. **Early Error Detection**

   - Run linting and tests locally before pushing code
   - Catch issues early in the development process
   - Keep hooks lightweight and focused on essential checks

2. **Workflow Consistency**

   - Automate code quality checks across the team
   - Enforce commit message conventions
   - Document hook requirements and provide clear error messages

3. **Tool Integration**

   - Seamlessly works with ESLint, Prettier, and testing frameworks
   - Supports custom scripts and commands
   - Optimize check execution order and use caching where appropriate

4. **Performance Optimization**
   - Run checks only on staged files
   - Minimize impact on development workflow
   - Consider parallel execution for independent tasks

## Automating Block Deployment and Versioning in Enterprise Environments

Enterprise WordPress environments require robust deployment and versioning strategies to maintain consistency, traceability, and reliability across development, staging, and production environments. This section explores practical approaches to automating these processes for Gutenberg blocks.

### Understanding Enterprise Deployment Requirements

Enterprise WordPress deployments typically involve:

1. **Multiple Environments**

   - Development: For active development and testing
   - Staging: For client review and QA testing
   - Production: The live environment
   - Each environment may have different configurations and requirements

2. **Asset Management**

   - Compiled JavaScript and CSS files
   - Block registration files
   - Plugin/theme dependencies
   - Media assets

3. **Version Control**
   - Semantic versioning for releases
   - Changelog maintenance
   - Git tags and branches
   - Backward compatibility considerations

### Implementing Automated Deployment

#### 1. Setting Up Deployment Workflows

Create a GitHub Actions workflow that handles deployment across environments:

```yaml
name: Deploy Block Assets

on:
  push:
    branches: [main]
  workflow_dispatch:
    inputs:
      environment:
        description: "Target environment"
        required: true
        default: "staging"
        type: choice
        options:
          - staging
          - production

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: ${{ github.event.inputs.environment || 'staging' }}

    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: "18"

      - name: Install dependencies
        run: npm ci

      - name: Build assets
        run: npm run build

      - name: Deploy to WordPress
        uses: appleboy/scp-action@master
        with:
          host: ${{ secrets.HOST }}
          username: ${{ secrets.USERNAME }}
          key: ${{ secrets.SSH_KEY }}
          source: "build/*"
          target: "/var/www/html/wp-content/plugins/my-block/"
          strip_components: 1
```

#### 2. Environment-Specific Configurations

Create environment-specific configurations using WordPress hooks:

```php
add_action('init', function() {
    $env = getenv('WP_ENV') ?: 'development';

    // Load environment-specific configurations
    switch ($env) {
        case 'production':
            define('BLOCK_ASSETS_URL', 'https://cdn.example.com/blocks/');
            break;
        case 'staging':
            define('BLOCK_ASSETS_URL', 'https://staging.example.com/blocks/');
            break;
        default:
            define('BLOCK_ASSETS_URL', plugins_url('build/', __FILE__));
    }
});
```

### Versioning Strategies

#### 1. Semantic Versioning Implementation

Implement semantic versioning in your block's `block.json`:

```json
{
  "name": "my-block",
  "version": "1.2.3",
  "title": "My Block",
  "category": "widgets",
  "attributes": {
    "content": {
      "type": "string",
      "source": "html",
      "selector": "p"
    }
  }
}
```

#### 2. Automated Version Bumping

<!-- npm version -->
<!-- utilities like semantic_version -->
<!-- [Conventional Commits specification](https://www.conventionalcommits.org/en/v1.0.0/) can be handy to automate version bumping and Changelogs -->

Create a workflow for automated version management:

```yaml
name: Version Management

on:
  push:
    branches: [main]

jobs:
  version:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Bump version
        run: |
          # Update block.json version
          VERSION=$(npm version patch --no-git-tag-version)
          sed -i "s/\"version\": \".*\"/\"version\": \"${VERSION#v}\"/" block.json

          # Generate changelog
          conventional-changelog -p angular -i CHANGELOG.md -s

          # Commit changes
          git config --global user.name 'GitHub Action'
          git config --global user.email 'action@github.com'
          git add block.json CHANGELOG.md
          git commit -m "chore: bump version to ${VERSION}"
          git push
```

### Best Practices for Enterprise Deployment

1. **Security Considerations**

   - Use environment-specific secrets
   - Implement proper access controls
   - Encrypt sensitive data
   - Regular security audits

2. **Performance Optimization**

   - Asset minification and compression
   - CDN integration
   - Cache management
   - Load testing

3. **Monitoring and Rollback**

   - Implement health checks
   - Set up monitoring alerts
   - Create rollback procedures
   - Maintain deployment logs

4. **Compliance and Documentation**
   - Maintain deployment documentation
   - Track changes in changelog
   - Document environment configurations
   - Follow enterprise security policies

### Example: Complete Deployment Pipeline

```yaml
name: Complete Block Deployment

on:
  push:
    branches: [main]
  workflow_dispatch:
    inputs:
      environment:
        description: "Target environment"
        required: true
        default: "staging"
        type: choice
        options:
          - staging
          - production

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
      - run: npm ci
      - run: npm test

  build:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
      - run: npm ci
      - run: npm run build
      - uses: actions/upload-artifact@v3
        with:
          name: block-assets
          path: build/

  deploy:
    needs: build
    runs-on: ubuntu-latest
    environment: ${{ github.event.inputs.environment || 'staging' }}
    steps:
      - uses: actions/download-artifact@v3
        with:
          name: block-assets
          path: build/

      - name: Deploy to WordPress
        uses: appleboy/scp-action@master
        with:
          host: ${{ secrets.HOST }}
          username: ${{ secrets.USERNAME }}
          key: ${{ secrets.SSH_KEY }}
          source: "build/*"
          target: "/var/www/html/wp-content/plugins/my-block/"
          strip_components: 1

      - name: Clear Cache
        uses: appleboy/ssh-action@master
        with:
          host: ${{ secrets.HOST }}
          username: ${{ secrets.USERNAME }}
          key: ${{ secrets.SSH_KEY }}
          script: |
            wp cache flush
            wp rewrite flush
```

This comprehensive approach to block deployment and versioning ensures that your enterprise WordPress environment maintains high standards of quality, security, and reliability while enabling efficient development workflows.

## Conclusion

By the end of this lesson, you should be able to explain how CI/CD practices enhance Gutenberg block development workflows in enterprise WordPress contexts. You've seen how to construct a CI/CD pipeline to automate testing, asset building, versioning, and deployment. Additionally, you should now be capable of modifying existing pipelines to support automated block deployment across different environments.

As your projects grow in complexity, refining your CI/CD processes becomes even more critical to maintaining quality and delivery velocity.
