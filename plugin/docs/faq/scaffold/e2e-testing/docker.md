# Docker Runtime E2E Testing

## Stoobly Scaffold Docker Runtime E2E Testing - Questions & Answers

This document covers Docker-specific E2E testing customization options for Stoobly scaffold. For general E2E testing topics, see [README.md](README.md). For local runtime E2E testing, see [local.md](local.md).

***

### Docker Plugin Support

#### Q: What files are created when I use the --plugin option with Docker runtime?

**A:** When you specify a plugin with Docker runtime, Stoobly creates a custom Dockerfile and entrypoint script in the `entrypoint/test/` directory specifically configured for that E2E framework.

**Files created for Playwright:**

* `.stoobly/services/entrypoint/test/.Dockerfile.playwright` - Stoobly-maintained base image that pulls in the stock Playwright image (override via `PLAYWRIGHT_IMAGE`)
* `.stoobly/services/entrypoint/test/.entrypoint.sh` - Entrypoint script with CA certificate setup
* `.stoobly/services/entrypoint/test/docker-compose.yml` - Docker compose configuration
* `.stoobly/services/entrypoint/test/init` - Initialization script

**Files created for Cypress:**

* `.stoobly/services/entrypoint/test/.Dockerfile.cypress` - Stoobly-maintained base image that pulls in the stock Cypress image (override via `CYPRESS_IMAGE`)
* `.stoobly/services/entrypoint/test/docker-compose.yml` - Docker compose configuration
* `.stoobly/services/entrypoint/test/init` - Initialization script

#### Q: Which base images do the Stoobly-maintained Dockerfiles use, and how do I override them?

**A:** Each is a Docker build arg with a default baked into the Dockerfile (`${VAR:-default}`). The generated `docker-compose.yml` passes the arg through from your environment, so leaving it unset uses the default, and exporting it before `make` / `workflow up` overrides it — without touching the managed `.Dockerfile.*` files.

| Build arg | Default | Used by |
|---|---|---|
| `PLAYWRIGHT_IMAGE` | `mcr.microsoft.com/playwright:v1.63.0-resolute` | `.Dockerfile.playwright` |
| `CYPRESS_IMAGE` | `cypress/browsers:24.21.0` | `.Dockerfile.cypress` |
| `GOLANG_IMAGE` | `golang:1.25.8-alpine` | both plugin Dockerfiles (builds `gosu`) |
| `GOSU_VERSION` | `1.19` | both plugin Dockerfiles |

The reverse-proxy gateway image overrides the same way — export before `make` / `workflow up` — but `TRAEFIK_IMAGE` is a Compose interpolation variable (`image: ${TRAEFIK_IMAGE:-traefik:v3}` in the generated `docker-compose.yml`) rather than a Dockerfile build arg, since there's no Stoobly-maintained Dockerfile for it.

***

### Docker Initial Setup

#### Q: How do I create a scaffold config for Docker E2E testing?

**A:** Create a declarative config at `.stoobly/scaffold.yml` with your app and service creation steps. Apply it with `stoobly-agent scaffold apply`. Include workflow steps as commented examples for reference. `scaffold apply` itself is runtime-agnostic — for the full command reference and config schema, see:

{% content-ref url="../apply.md" %}
[apply.md](../apply.md)
{% endcontent-ref %}

**Example:**

```yaml
# .stoobly/scaffold.yml
version: 1
commands:
  - resource: app
    action: create
    options:
      app_name: my-e2e-tests
      runtime: docker
      plugin: [playwright]

  # API runs on localhost:4000, exposed as api.local:80
  # Note: upstream_port is required when using local: true and must differ from port
  # Run your API service on port 4000 (not 80)
  - resource: service
    action: create
    options:
      service_name: api
      hostname: api.local
      port: 80
      scheme: http
      local: true
      upstream_port: 4000

  # UI service (remote, served on app.local:80 for testing)
  - resource: service
    action: create
    options:
      service_name: ui
      hostname: app.local
      port: 80
      scheme: http
      workflow: [test]

  # Example: Start record workflow
  # make -f .stoobly/services/Makefile record
  # make -f .stoobly/services/Makefile intercept/enable

  # Example: Start mock workflow
  # make -f .stoobly/services/Makefile mock
  # make -f .stoobly/services/Makefile intercept/enable

  # Example: Start test workflow
  # make -f .stoobly/services/Makefile test
  # make -f .stoobly/services/Makefile intercept/enable
```

```bash
stoobly-agent scaffold apply .stoobly/scaffold.yml --dry-run
stoobly-agent scaffold apply .stoobly/scaffold.yml
```

#### Q: How do I scaffold an app for Playwright E2E testing with Docker?

**A:** Create an app with the `--plugin playwright` option and `--runtime docker` to set up Playwright-specific configurations.

**Example:**

```bash
# Create app with Playwright
stoobly-agent scaffold app create my-e2e-tests \
  --plugin playwright \
  --runtime docker
```

#### Q: How do I scaffold an app for Cypress E2E testing with Docker?

**A:** Create an app with the `--plugin cypress` option and `--runtime docker` to set up Cypress-specific configurations.

**Example:**

```bash
# Create app with Cypress
stoobly-agent scaffold app create my-cypress-tests \
  --plugin cypress \
  --runtime docker
```

***

### Docker Configuration Files

#### Q: What files should I modify to add my Playwright tests with Docker?

**A:** After scaffolding, you need to modify the entrypoint service docker-compose.yml to run your tests and your Playwright configuration to use the proxy. To set up Stoobly rules, add them to the workflow `init` script. The example below shows how you would override the Stoobly-managed `entrypoint.playwright` service definition—only do this when you need additional customizations.

**Example - Modify docker-compose.yml:**

```yaml
# .stoobly/services/entrypoint/test/docker-compose.yml
services:
  entrypoint.playwright:
    build:
      context: .
      dockerfile: .Dockerfile.playwright
      args:
        PLAYWRIGHT_IMAGE: mcr.microsoft.com/playwright:v1.63.0-resolute
        USER_ID: ${USER_ID}
    working_dir: /home/stoobly/tests
    volumes:
      - ../../..:/home/stoobly/tests    # Mount your test directory
    depends_on:
      entrypoint.init:
        condition: service_completed_successfully
    networks:
      app.ingress: {}
    environment:
      - HTTP_PROXY=http://gateway:80
      - HTTPS_PROXY=http://gateway:80
      - NO_PROXY=localhost,127.0.0.1
    profiles:
      - ${WORKFLOW_NAME}
    command: npx playwright test          # Your test command
```

**Example - Add Stoobly rules in init script:**

```bash
# .stoobly/services/entrypoint/test/init
#!/bin/bash

# Rewrite production URLs to local services
stoobly-agent setting rewrite set \
  --pattern "https://api.production.com/.*" \
  --method GET --method POST \
  --mode test \
  --hostname api.local

# Exclude third-party tracking
stoobly-agent setting filter set \
  --pattern "https://.*analytics.com/.*" \
  --method GET \
  --mode test \
  --action exclude
```

**Example - Create Playwright config:**

```javascript
// playwright.config.ts (in your project root)
import { defineConfig } from '@playwright/test';

export default defineConfig({
  use: {
    proxy: {
      server: 'http://gateway:80',
    },
    ignoreHTTPSErrors: true,
  },
});
```

#### Q: What files should I modify to add my Cypress tests with Docker?

**A:** For Cypress, modify the entrypoint service docker-compose.yml and Cypress configuration to work with Stoobly's proxy. As with Playwright, the example below overrides the managed `entrypoint.cypress` service definition—only override it when you need custom behavior.

**Example - Modify docker-compose.yml:**

```yaml
# .stoobly/services/entrypoint/test/docker-compose.yml
services:
  entrypoint.cypress:
    build:
      context: .
      dockerfile: .Dockerfile.cypress
      args:
        CYPRESS_IMAGE: cypress/browsers:24.21.0
        USER_ID: ${USER_ID}
    working_dir: /home/stoobly/e2e
    volumes:
      - ../../..:/home/stoobly/e2e        # Mount your test directory
    depends_on:
      entrypoint.init:
        condition: service_completed_successfully
    networks:
      app.ingress: {}
    environment:
      - HTTP_PROXY=http://gateway:80
      - HTTPS_PROXY=http://gateway:80
    profiles:
      - ${WORKFLOW_NAME}
    command: cypress run                   # Your test command
```

**Example - Create Cypress config:**

```javascript
// cypress.config.js (in your project root)
const { defineConfig } = require('cypress');

module.exports = defineConfig({
  e2e: {
    env: {
      HTTP_PROXY: 'http://gateway:80',
      HTTPS_PROXY: 'http://gateway:80',
    },
  },
});
```

#### Q: How do I add custom npm packages for my Docker tests?

**A:** Don't edit the Stoobly-maintained Dockerfiles (files beginning with a dot). Instead, point `PLAYWRIGHT_IMAGE` or `CYPRESS_IMAGE` at a custom image that already includes your dependencies.

```bash
# Example: override Playwright image when running a workflow
PLAYWRIGHT_IMAGE=my-org/playwright-with-helpers:latest \
  make -f .stoobly/services/Makefile test

# For Cypress
CYPRESS_IMAGE=my-org/cypress-with-plugins:latest \
  make -f .stoobly/services/Makefile test
```

Build those images however you like (e.g., start from `mcr.microsoft.com/playwright` or `cypress/browsers` and add packages), then reference them with the environment variable. Stoobly will use your custom image without touching the managed `.Dockerfile.*` files.

{% hint style="warning" %}
For Cypress, build from `cypress/browsers`, not `cypress/included`. The managed Dockerfile installs the Cypress binary itself (`npx cypress install`), which expects a `browsers`-style base image without Cypress preinstalled.
{% endhint %}

#### Q: How do I pass environment variables to my Docker tests?

**A:** Add environment variables in the docker-compose.yml file under the `environment` section.

**Example:**

```yaml
# .stoobly/services/entrypoint/test/docker-compose.yml
services:
  entrypoint.playwright:
    environment:
      - HTTP_PROXY=http://gateway:80
      - HTTPS_PROXY=http://gateway:80
      - BASE_URL=http://app.local
      - API_URL=http://api.local
      - TEST_USER_EMAIL=test@example.com
      - TEST_USER_PASSWORD=testpass123
      - NODE_ENV=test
```

***

### Running Docker E2E Tests

#### Q: How do I record E2E test traffic with Docker?

**A:** Use the record workflow to capture all HTTP requests made during your E2E tests.

**Example:**

```bash
# Start record workflow
make -f .stoobly/services/Makefile record
make -f .stoobly/services/Makefile intercept/enable

# Tests run automatically in entrypoint container
# All HTTP requests are recorded

# View recorded requests
stoobly-agent request list

# Create scenario from recorded requests
stoobly-agent scenario create "E2E User Flow"

# Stop workflow
make -f .stoobly/services/Makefile record/down
```

#### Q: How do I run E2E tests with mocked responses using Docker?

**A:** Use the mock workflow to run tests against recorded responses without hitting real APIs.

**Example:**

```bash
# Ensure you have recorded responses
stoobly-agent scenario list

# Start mock workflow
make -f .stoobly/services/Makefile mock
make -f .stoobly/services/Makefile intercept/enable

# Tests run automatically in entrypoint container
# Tests run against mocked responses
# No real API calls are made

# Stop workflow
make -f .stoobly/services/Makefile mock/down
```

#### Q: How do I run E2E tests with response validation using Docker?

**A:** Use the test workflow to validate that responses match expected results.

**Example:**

```bash
# Start test workflow
make -f .stoobly/services/Makefile test
make -f .stoobly/services/Makefile intercept/enable

# Tests run automatically in entrypoint container
# Tests run with response validation
# Stoobly compares actual vs expected responses

# View test results
make -f .stoobly/services/Makefile test/logs

# Stop workflow
make -f .stoobly/services/Makefile test/down
```

#### Q: How do I run specific test files or suites with Docker?

**A:** Modify the command in docker-compose.yml to target specific tests.

**Example for Playwright:**

```yaml
# Run specific test file
command: npx playwright test tests/login.spec.ts

# Run tests with specific tag
command: npx playwright test --grep @smoke

# Run tests in headed mode for debugging
command: npx playwright test --headed
```

**Example for Cypress:**

```yaml
# Run specific spec
command: cypress run --spec "cypress/e2e/login.cy.js"

# Run with specific browser
command: cypress run --browser chrome

# Run with video recording
command: cypress run --video
```

***

### Docker Debugging

#### Q: How do I view test output and logs with Docker?

**A:** Use the logs command to see test execution output.

**Example:**

```bash
# View logs while tests are running
make -f .stoobly/services/Makefile test/logs

# View logs for specific service
make -f .stoobly/services/Makefile test/logs options="--service <SERVICE-NAME>"

# View logs for specific namespace
make -f .stoobly/services/Makefile test/logs options="--namespace <NAMESPACE>"
```

#### Q: How do I debug failing Docker tests?

**A:** Enable headed mode and increase logging verbosity in the docker-compose.yml.

**Example:**

```yaml
# .stoobly/services/entrypoint/test/docker-compose.yml
services:
  entrypoint.playwright:
    environment:
      - DEBUG=pw:api                    # Playwright debug logs
      - PWDEBUG=1                       # Playwright debug mode
    command: npx playwright test --headed --debug
```

#### Q: How do I save test artifacts (screenshots, videos) with Docker?

**A:** Mount an artifacts directory to persist test outputs and configure your test framework to save artifacts.

**Example:**

```yaml
# .stoobly/services/entrypoint/test/docker-compose.yml
services:
  entrypoint.playwright:
    volumes:
      - ../../..:/home/stoobly/tests
      - ../../../test-results:/home/stoobly/tests/test-results  # Artifacts
```

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

***

### Docker Best Practices

#### Q: Should I record once and mock for all subsequent runs with Docker?

**A:** Yes, this is the recommended approach for fast, reliable E2E tests that don't depend on external APIs.

**Example:**

```bash
# Step 1: Record once (against real APIs)
make -f .stoobly/services/Makefile record
make -f .stoobly/services/Makefile intercept/enable

# Step 2: Create scenario from recordings
stoobly-agent scenario create "User Journey - Login to Checkout"

# Step 3: Snapshot for version control
stoobly-agent scenario snapshot user-journey --decode
git add .stoobly/snapshots/ && git commit -m "Add E2E test snapshots"

# Step 4: All future runs use mocks (fast, no external dependencies)
make -f .stoobly/services/Makefile mock
make -f .stoobly/services/Makefile intercept/enable
```

#### Q: How do I organize E2E tests by feature with Docker?

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
make -f .stoobly/services/Makefile test STOOBLY_WORKFLOW_SERVICE_OPTIONS="--service auth-tests"
```

***

### Docker CI/CD Integration

#### Q: Which runtime should I use in CI/CD for E2E testing?

**A:** Docker is recommended for CI/CD as it provides consistent, isolated environments for E2E tests. Set the Makefile option `workflow_up_extra_options="--no-publish"` to prevent Docker port conflicts in shared CI hosts.

**Example:**

```bash
#!/bin/bash

# Update .stoobly/services/entrypoint/test/run to run test command e.g. `npm test`

# CI/CD script with Docker runtime
export STOOBLY_HOSTNAME_INSTALL_CONFIRM=y
export STOOBLY_CA_CERTS_INSTALL_CONFIRM=y

# Use Docker runtime
make -f .stoobly/services/.Makefile test workflow_up_extra_options="--no-publish"

# Cleanup
make -f .stoobly/services/.Makefile test/down
```

#### Q: How do I test with Docker runtime in CI/CD?

**A:** Use Makefile commands in your CI/CD pipeline. Include the Makefile option `workflow_up_extra_options="--no-publish"` on test workflow runs to avoid host port binding collisions.

**Example:**

```yaml
# .github/workflows/test.yml
name: Test with Docker

jobs:
  test-docker:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Test with Docker
        run: |
          make -f .stoobly/services/.Makefile test workflow_up_extra_options="--no-publish"
          make -f .stoobly/services/.Makefile test/down
```

#### Q: How do I use the Makefile in CI/CD pipelines?

**A:** Set all confirmation environment variables to avoid prompts and run the desired workflow commands. For CI, set the Makefile option `workflow_up_extra_options="--no-publish"` on test workflow runs.

**Example:**

```bash
#!/bin/bash
# CI/CD script example
export STOOBLY_HOSTNAME_INSTALL_CONFIRM=y
export STOOBLY_CA_CERTS_INSTALL_CONFIRM=y
export STOOBLY_APP_DIR=$PWD

# Start test workflow
make -f .stoobly/services/.Makefile test workflow_up_extra_options="--no-publish"

# Run your tests here
# ...

# Clean up
make -f .stoobly/services/.Makefile test/down
```

#### Q: How do I run workflows in Docker containers for CI/CD?

**A:** The Makefile automatically handles Docker-based workflows. Ensure Docker is running and accessible.

**Example:**

```bash
# Verify Docker is running
docker ps

# Start workflow (automatically uses Docker)
make -f .stoobly/services/.Makefile record
```

***

### Docker Advanced Configuration

#### Q: How do I mount test fixtures and data with Docker?

**A:** Add volume mounts to share fixtures between host and container.

**Example:**

```yaml
# .stoobly/services/entrypoint/test/docker-compose.yml
services:
  entrypoint.playwright:
    volumes:
      - ../../..:/home/stoobly/tests           # Project root
      - ../../../test-data:/home/stoobly/data  # Test data
      - ../../../fixtures:/home/stoobly/fixtures # Fixtures
```

***

### Docker Complete Example

#### Q: What's a complete example of setting up Playwright E2E tests with Docker?

**A:** Here's a full example from scaffold to running tests.

**Step 1: Scaffold the app**

```bash
# Create app with Playwright
stoobly-agent scaffold app create my-e2e-app \
  --plugin playwright \
  --proxy-port 8080 \
  --ui-port 4200

# Add frontend service (test workflow only - serves static assets from fixtures)
stoobly-agent scaffold service create frontend \
  --hostname app.local \
  --port 80 \
  --scheme http \
  --workflow test

# Add API service (all workflows created by default)
# Remote upstream where hostname/port/scheme differ
stoobly-agent scaffold service create api \
  --hostname api.local \
  --port 443 \
  --scheme https \
  --upstream-hostname api.production.com
```

**Step 2: Configure entrypoint docker-compose.yml**

```yaml
# .stoobly/services/entrypoint/test/docker-compose.yml
services:
  entrypoint.playwright:
    build:
      context: .
      dockerfile: .Dockerfile.playwright
      args:
        PLAYWRIGHT_IMAGE: mcr.microsoft.com/playwright:v1.63.0-resolute
        USER_ID: ${USER_ID}
    working_dir: /home/stoobly/app
    volumes:
      - ../../..:/home/stoobly/app
    depends_on:
      entrypoint.init:
        condition: service_completed_successfully
    networks:
      app.ingress: {}
    environment:
      - HTTP_PROXY=http://gateway:80
      - HTTPS_PROXY=http://gateway:80
      - BASE_URL=http://app.local
      - API_URL=http://api.local
    profiles:
      - ${WORKFLOW_NAME}
    command: npx playwright test
```

**Step 3: Configure Stoobly rules**

```bash
# .stoobly/services/entrypoint/test/init
#!/bin/bash

# Rewrite API calls
stoobly-agent setting rewrite set \
  --pattern "https://api.production.com/.*" \
  --method GET --method POST --method PUT --method DELETE \
  --mode test \
  --hostname api.local

# Exclude analytics
stoobly-agent setting filter set \
  --pattern "https://.*analytics.com/.*" \
  --method GET --method POST \
  --mode test \
  --action exclude
```

**Step 4: Create Playwright config**

```javascript
// playwright.config.ts
import { defineConfig } from '@playwright/test';

export default defineConfig({
  testDir: './tests',
  use: {
    baseURL: process.env.BASE_URL || 'http://app.local',
    proxy: {
      server: 'http://gateway:80',
    },
    ignoreHTTPSErrors: true,
    screenshot: 'only-on-failure',
    video: 'retain-on-failure',
  },
  outputDir: 'test-results',
});
```

**Step 5: Create test**

```javascript
// tests/login.spec.ts
import { test, expect } from '@playwright/test';

test('user can login', async ({ page }) => {
  await page.goto('/login');
  
  await page.fill('[name="email"]', 'test@example.com');
  await page.fill('[name="password"]', 'password123');
  await page.click('button[type="submit"]');
  
  await expect(page).toHaveURL('/dashboard');
  await expect(page.locator('h1')).toContainText('Welcome');
});
```

**Step 6: Run tests**

```bash
# Run with recording
make -f .stoobly/services/Makefile record
make -f .stoobly/services/Makefile intercept/enable

# Run with mocking
make -f .stoobly/services/Makefile mock
make -f .stoobly/services/Makefile intercept/enable

# Run with validation
make -f .stoobly/services/Makefile test
make -f .stoobly/services/Makefile intercept/enable
```
