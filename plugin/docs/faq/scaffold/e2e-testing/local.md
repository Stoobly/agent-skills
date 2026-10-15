# Local Runtime E2E Testing

## Stoobly Scaffold Local Runtime E2E Testing - Questions & Answers

This document covers local runtime-specific E2E testing customization options for Stoobly scaffold. For general E2E testing topics, see [README.md](README.md). For Docker runtime E2E testing, see [docker.md](docker.md).

***

### Local Initial Setup

#### Q: How do I scaffold an app for Playwright E2E testing with local runtime?

**A:** Create an app with the `--plugin playwright` option and `--runtime local` to set up Playwright-specific configurations.

**Example:**

```bash
# Create app with Playwright
stoobly-agent scaffold app create my-e2e-tests \
  --plugin playwright \
  --runtime local
```

#### Q: How do I scaffold an app for Cypress E2E testing with local runtime?

**A:** Create an app with the `--plugin cypress` option and `--runtime local` to set up Cypress-specific configurations.

**Example:**

```bash
# Create app with Cypress
stoobly-agent scaffold app create my-cypress-tests \
  --plugin cypress \
  --runtime local
```

**Note:** With local runtime, there's no entrypoint service, so you'll run your tests directly on your machine with proxy configuration.

***

### Local Configuration Files

#### Q: What files should I modify to add my Playwright tests with local runtime?

**A:** Configure your Playwright tests to use Stoobly's proxy by setting environment variables or configuring the proxy in your Playwright config.

**Example - Set proxy environment variables:**

```bash
# Set proxy environment variables
export HTTP_PROXY=http://localhost:8080
export HTTPS_PROXY=http://localhost:8080

# Run your tests
npx playwright test
```

**Example - Create Playwright config:**

```javascript
// playwright.config.ts (in your project root)
import { defineConfig } from '@playwright/test';

export default defineConfig({
  use: {
    proxy: {
      server: 'http://localhost:8080',
    },
    ignoreHTTPSErrors: true,
  },
});
```

**Note:** `ignoreHTTPSErrors: true` covers the browser Playwright drives, but if your test process itself (or a Node dev server behind the proxy) makes HTTPS calls, Node needs to separately trust Stoobly's CA certificate — see the [CA Cert FAQ](../../ca-cert.md).

#### Q: What files should I modify to add my Cypress tests with local runtime?

**A:** Configure Cypress to use Stoobly's proxy by setting environment variables or configuring the proxy in your Cypress config.

**Example - Set proxy environment variables:**

```bash
# Set proxy environment variables
export HTTP_PROXY=http://localhost:8080
export HTTPS_PROXY=http://localhost:8080

# Run your tests
npx cypress run
```

**Example - Create Cypress config:**

```javascript
// cypress.config.js (in your project root)
const { defineConfig } = require('cypress');

module.exports = defineConfig({
  e2e: {
    env: {
      HTTP_PROXY: 'http://localhost:8080',
      HTTPS_PROXY: 'http://localhost:8080',
    },
  },
});
```

#### Q: How do I pass environment variables to my local tests?

**A:** Set environment variables in your shell or use a `.env` file that your test framework reads.

**Example:**

```bash
# Set environment variables before running tests
export BASE_URL=http://app.local
export API_URL=http://api.local
export TEST_USER_EMAIL=test@example.com
export TEST_USER_PASSWORD=testpass123
export NODE_ENV=test

# Run tests
npx playwright test
```

Or use a `.env` file:

```bash
# .env
BASE_URL=http://app.local
API_URL=http://api.local
TEST_USER_EMAIL=test@example.com
TEST_USER_PASSWORD=testpass123
NODE_ENV=test
```

***

### Running Local E2E Tests

**💡 Tip:** The examples below run each step manually across terminals for clarity. For day-to-day use, wrap `workflow up`/`down` and your test command into `package.json` scripts so a single `npm run test:mock` (etc.) handles bring-up, test run, and teardown — including correct CI exit-code propagation. See [JS Client — npm Scripts](js-client/npm-scripts.md).

#### Q: How do I record E2E test traffic with local runtime?

**A:** Use the record workflow to capture all HTTP requests made during your E2E tests.

**Example:**

```bash
# Start record workflow and enable recording (local runtime uses CLI commands, not Makefile)
stoobly-agent scaffold workflow up record
stoobly-agent intercept enable

# In another terminal, set proxy and run tests
export HTTP_PROXY=http://localhost:8080
export HTTPS_PROXY=http://localhost:8080
npx playwright test

# View recorded requests
stoobly-agent request list

# Create scenario from recorded requests
stoobly-agent scenario create "E2E User Flow"

# Stop workflow
stoobly-agent scaffold workflow down record
```

**Note:** Make commands (e.g., `make -f .stoobly/services/Makefile record`) are Docker-specific. Local runtime uses `stoobly-agent scaffold workflow up/down` commands directly.

#### Q: How do I run E2E tests with mocked responses using local runtime?

**A:** Use the mock workflow to run tests against recorded responses without hitting real APIs.

**Example:**

```bash
# Ensure you have recorded responses
stoobly-agent scenario list

# Start mock workflow and enable intercept
stoobly-agent scaffold workflow up mock
stoobly-agent intercept enable

# In another terminal, run tests with proxy environment variables
export HTTP_PROXY=http://localhost:8080
export HTTPS_PROXY=http://localhost:8080
npx playwright test

# Tests run against mocked responses
# No real API calls are made

# Stop workflow
stoobly-agent scaffold workflow down mock
```

#### Q: How do I run E2E tests with response validation using local runtime?

**A:** Use the test workflow to validate that responses match expected results.

**Example:**

```bash
# Start test workflow and enable intercept
stoobly-agent scaffold workflow up test
stoobly-agent intercept enable

# In another terminal, run tests with proxy environment variables
export HTTP_PROXY=http://localhost:8080
export HTTPS_PROXY=http://localhost:8080
npx playwright test

# Tests run with response validation
# Stoobly compares actual vs expected responses

# View test results (check logs in .stoobly/tmp/test/logs/)
cat .stoobly/tmp/test/logs/requests.json

# Stop workflow
stoobly-agent scaffold workflow down test
```

#### Q: How do I run E2E tests with local runtime?

**A:** Start the Stoobly workflow using CLI commands (not Makefile), then run your tests directly with proxy environment variables set.

#### Q: How do I run specific test files or suites with local runtime?

**A:** Use your test framework's command-line options to target specific tests.

**Example for Playwright:**

```bash
# Run specific test file
npx playwright test tests/login.spec.ts

# Run tests with specific tag
npx playwright test --grep @smoke

# Run tests in headed mode for debugging
npx playwright test --headed
```

**Example for Cypress:**

```bash
# Run specific spec
npx cypress run --spec "cypress/e2e/login.cy.js"

# Run with specific browser
npx cypress run --browser chrome

# Run with video recording
npx cypress run --video
```

***

### Local Debugging

#### Q: How do I view test output and logs with local runtime?

**A:** View logs from the workflow directory or check Stoobly agent logs directly.

**Example:**

```bash
# View logs from the workflow directory
cat .stoobly/tmp/test/logs/requests.json

# Or check Stoobly agent logs directly
stoobly-agent request list
```

#### Q: How do I debug failing local tests?

**A:** Enable debug mode in your test framework and increase logging verbosity.

**Example for Playwright:**

```bash
# Enable debug mode
export DEBUG=pw:api
export PWDEBUG=1

# Run tests in headed mode
npx playwright test --headed --debug
```

**Example for Cypress:**

```bash
# Run in interactive mode
npx cypress open

# Or with debug logging
DEBUG=cypress:* npx cypress run
```

#### Q: How do I save test artifacts (screenshots, videos) with local runtime?

**A:** Configure your test framework to save artifacts to a local directory.

**Example for Playwright:**

```javascript
// playwright.config.ts
export default defineConfig({
  use: {
    screenshot: 'only-on-failure',
    video: 'retain-on-failure',
  },
  outputDir: 'test-results',
});
```

**Example for Cypress:**

```javascript
// cypress.config.js
module.exports = defineConfig({
  e2e: {
    screenshotOnRunFailure: true,
    video: true,
  },
});
```

***

### Local Best Practices

#### Q: Should I record once and mock for all subsequent runs with local runtime?

**A:** Yes, this is the recommended approach for fast, reliable E2E tests that don't depend on external APIs.

**Example:**

```bash
# Step 1: Create scenario from recordings
stoobly-agent scenario create "User Journey - Login to Checkout"

# Step 2: Record once (against real APIs)
stoobly-agent scaffold workflow up record
stoobly-agent intercept enable

# Run tests in another terminal, then:
stoobly-agent scaffold workflow down record

# Step 3: Snapshot for version control
stoobly-agent scenario snapshot user-journey --decode
git add .stoobly/snapshots/ && git commit -m "Add E2E test snapshots"

# Step 4: All future runs use mocks (fast, no external dependencies)
stoobly-agent scaffold workflow up mock
stoobly-agent intercept enable
# Run tests in another terminal
```
#### Q: How do I organize E2E tests by feature with local runtime?

**A:** Create separate services or custom workflows for different test suites.

**Example:**

```bash
# Create service for auth tests
stoobly-agent scaffold service create auth-tests \
  --workflow test

# Create service for checkout tests
stoobly-agent scaffold service create checkout-tests \
  --workflow test

# Run specific test suite
stoobly-agent scaffold workflow up test --service auth-tests
```

***

### Local CI/CD Integration

#### Q: Can I use local runtime in CI/CD for E2E testing?

**A:** Yes, if your CI environment has Python and stoobly-agent installed, local runtime can be faster for E2E tests.

**Example:**

```bash
#!/bin/bash
# CI/CD script with local runtime

# Install Stoobly
pipx install stoobly-agent

# Install Stoobly ca-certs
sudo stoobly-agent ca-cert install

# Use local runtime
stoobly-agent scaffold workflow up test

# Cleanup
stoobly-agent scaffold workflow down test
```

#### Q: How do I test with local runtime in CI/CD?

**A:** Install stoobly-agent and use CLI commands in your CI/CD pipeline.

**Example:**

```yaml
# .github/workflows/test.yml
name: Test with Local

jobs:
  test-local:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Install stoobly-agent
        run: pipx install stoobly-agent
      - name: Test with Local
        run: |
          stoobly-agent scaffold workflow up test
          stoobly-agent scaffold workflow down test
```

***

### Local Advanced Configuration

#### Q: How do I access test fixtures and data with local runtime?

**A:** Reference fixtures and test data using relative paths from your project root. No volume mounting is needed since everything runs on your local machine.

**Example:**

```javascript
// tests/login.spec.ts
import { test, expect } from '@playwright/test';
import fs from 'fs';
import path from 'path';

test('user can login', async ({ page }) => {
  // Read test data from local file
  const testData = JSON.parse(
    fs.readFileSync(path.join(__dirname, '../test-data/users.json'), 'utf8')
  );
  
  await page.goto('/login');
  await page.fill('[name="email"]', testData.email);
  await page.fill('[name="password"]', testData.password);
  await page.click('button[type="submit"]');
});
```
