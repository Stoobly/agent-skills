# E2E Testing

## Stoobly Scaffold E2E Testing - Questions & Answers

Stoobly scaffold provides first-class support for E2E testing frameworks like Playwright and Cypress, enabling you to record, mock, and validate API interactions during your end-to-end tests.

**📚 Related Documentation:**
- For Docker-specific E2E testing, see [docker.md](docker.md)
- For local runtime E2E testing, see [local.md](local.md)
- For using the JavaScript client library, see [js-client/](js-client/README.md)

**⚠️ Important:** Make commands (e.g., `make -f .stoobly/services/Makefile test`) are **Docker-specific only**. Local runtime uses CLI commands directly (e.g., `stoobly-agent scaffold workflow up test`).

***

### Plugin Support

#### Q: What E2E testing frameworks does Stoobly support?

**A:** Stoobly supports **Playwright** and **Cypress** through the `--plugin` option when creating an app.

**Example:**

```bash
# Create app with Playwright support
stoobly-agent scaffold app create my-app --plugin playwright

# Create app with Cypress support
stoobly-agent scaffold app create my-app --plugin cypress

# Create app with both plugins
stoobly-agent scaffold app create my-app --plugin playwright --plugin cypress
```

**Note:** The files created depend on whether you use `--runtime docker` or `--runtime local`. For Docker-specific file details, see [docker.md](docker.md). For local runtime setup, see [local.md](local.md).

***

### Frontend Service Setup

#### Q: How do I set up a frontend service to serve static assets for E2E tests?

**A:** Frontend services typically only need the `test` workflow since they serve static assets from fixtures rather than recording or mocking requests. Create the service with `--workflow test` and modify the `test/init` script to copy your built application to the public fixtures folder.

**Example:**

```bash
# Create frontend service with only test workflow
stoobly-agent scaffold service create frontend \
  --hostname app.local \
  --port 80 \
  --scheme http \
  --workflow test
```

**Modify the init script to copy built assets:**

```bash
# .stoobly/services/frontend/test/init
#!/bin/bash

# Copy built frontend assets to public fixtures folder
# This makes the application available for E2E tests
cp -r ./dist/. .stoobly/fixtures/public/

# The frontend service will serve these static files during E2E tests
```

**Why only test workflow?**
- Frontend services serve static assets, not dynamic API responses
- No need to record requests (no API logic to capture)
- No need to mock responses (serving files, not making requests)
- Test workflow is needed so E2E tests can request the application under test

***

### Configuration Files

#### Q: What files should I modify to add my Playwright tests?

**A:** The files you modify depend on your runtime:
- **Docker runtime:** Modify entrypoint docker-compose.yml, init script, and Playwright config. See [docker.md](docker.md) for details.
- **Local runtime:** Configure Playwright to use Stoobly's proxy via environment variables or config. See [local.md](local.md) for details.
- **Using JavaScript library:** Use the Stoobly JavaScript client library with Playwright interceptor. See [js-client/](js-client/README.md) for details.

#### Q: What files should I modify to add my Cypress tests?

**A:** The files you modify depend on your runtime:
- **Docker runtime:** Modify entrypoint docker-compose.yml and Cypress config. See [docker.md](docker.md) for details.
- **Local runtime:** Configure Cypress to use Stoobly's proxy via environment variables or config. See [local.md](local.md) for details.
- **Using JavaScript library:** Use the Stoobly JavaScript client library with Cypress interceptor. See [js-client/](js-client/README.md) for details.

#### Q: How do I add custom npm packages for my tests?

**A:** The approach depends on your runtime:
- **Docker runtime:** Use custom Docker images with `PLAYWRIGHT_IMAGE` or `CYPRESS_IMAGE` environment variables. See [docker.md](docker.md) for details.
- **Local runtime:** Install packages directly in your project using npm/yarn. No special configuration needed.

#### Q: How do I pass environment variables to my tests?

**A:** The approach depends on your runtime:
- **Docker runtime:** Add environment variables in docker-compose.yml. See [docker.md](docker.md) for details.
- **Local runtime:** Set environment variables in your shell or use a `.env` file. See [local.md](local.md) for details.

***

### Running E2E Tests

#### Q: How do I record E2E test traffic?

**A:** Use the record workflow to capture all HTTP requests made during your E2E tests.

- **Docker runtime:** See [docker.md](docker.md) for Makefile commands.
- **Local runtime:** See [local.md](local.md) for CLI commands.

#### Q: How do I run E2E tests with mocked responses?

**A:** Use the mock workflow to run tests against recorded responses without hitting real APIs.

- **Docker runtime:** See [docker.md](docker.md) for Makefile commands.
- **Local runtime:** See [local.md](local.md) for CLI commands.

#### Q: How do I run E2E tests with response validation?

**A:** Use the test workflow to validate that responses match expected results.

- **Docker runtime:** See [docker.md](docker.md) for Makefile commands — the entrypoint container runs your test command automatically.
- **Local runtime:** See [local.md](local.md) for CLI commands. Local runtime does **not** run your tests for you — `workflow up test` only starts the proxy; you invoke Playwright/Cypress yourself afterward. To avoid the manual multi-step dance (and to get correct pass/fail exit codes in CI), wrap `workflow up`/`down` and your test command into `package.json` scripts — see [JS Client — npm Scripts](js-client/npm-scripts.md).

#### Q: How do I run specific test files or suites?

**A:** The approach depends on your runtime:
- **Docker runtime:** Modify the command in docker-compose.yml. See [docker.md](docker.md) for details.
- **Local runtime:** Use your test framework's command-line options. See [local.md](local.md) for details.

***

### Debugging

#### Q: How do I view test output and logs?

**A:** The approach depends on your runtime:
- **Docker runtime:** Use Makefile logs commands. See [docker.md](docker.md) for details.
- **Local runtime:** View logs from the workflow directory or Stoobly agent. See [local.md](local.md) for details.

#### Q: How do I debug failing tests?

**A:** The approach depends on your runtime:
- **Docker runtime:** Enable headed mode and increase logging verbosity in docker-compose.yml. See [docker.md](docker.md) for details.
- **Local runtime:** Enable debug mode in your test framework. See [local.md](local.md) for details.

#### Q: How do I save test artifacts (screenshots, videos)?

**A:** The approach depends on your runtime:
- **Docker runtime:** Mount an artifacts directory and configure your test framework. See [docker.md](docker.md) for details.
- **Local runtime:** Configure your test framework to save artifacts to a local directory. See [local.md](local.md) for details.

***

### Advanced Configuration

#### Q: How do I mount test fixtures and data?

**A:** The approach depends on your runtime:
- **Docker runtime:** Add volume mounts to share fixtures between host and container. See [docker.md](docker.md) for details.
- **Local runtime:** Reference fixtures and test data using relative paths from your project root. See [local.md](local.md) for details.

#### Q: How do I handle dynamic data in E2E tests?

**A:** Use Stoobly's rewrite rules to normalize dynamic values in the init script. This applies to both Docker and local runtimes.

**Example:**

```bash
# .stoobly/services/entrypoint/test/init (Docker)
# or .stoobly/services/<service>/test/init (Local)
#!/bin/bash

# Replace dynamic user IDs with fixed test ID
stoobly-agent setting rewrite set \
  --pattern "https://api.local/users/.*" \
  --method GET \
  --mode test \
  --type response_param \
  --name "userId" \
  --value "test-user-123"

# Replace timestamps
stoobly-agent setting rewrite set \
  --pattern "https://api.local/.*" \
  --method GET --method POST \
  --mode test \
  --type response_param \
  --name "timestamp" \
  --value "2024-01-01T00:00:00Z"
```

***

### Best Practices

#### Q: Should I record once and mock for all subsequent runs?

**A:** Yes, this is the recommended approach for fast, reliable E2E tests that don't depend on external APIs.

- **Docker runtime:** See [docker.md](docker.md) for the complete workflow example.
- **Local runtime:** See [local.md](local.md) for the complete workflow example.

#### Q: How do I organize E2E tests by feature?

**A:** Create separate services or custom workflows for different test suites.

- **Docker runtime:** See [docker.md](docker.md) for examples.
- **Local runtime:** See [local.md](local.md) for examples.

***

### CI/CD Integration

#### Q: Which runtime should I use in CI/CD for E2E testing?

**A:** Docker is recommended for CI/CD as it provides consistent, isolated environments. Local runtime can also be used if your CI environment has Python and stoobly-agent installed.

- **Docker runtime:** See [docker.md](docker.md) for CI/CD examples.
- **Local runtime:** See [local.md](local.md) for CI/CD examples.

#### Q: How do I test with different runtimes in CI/CD?

**A:** Create separate CI jobs, each using the desired runtime.

**Example:**

```yaml
# .github/workflows/test.yml
name: Test Different Runtimes

jobs:
  test-docker:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Test with Docker
        run: |
          make -f .stoobly/services/.Makefile test
          make -f .stoobly/services/.Makefile test/down

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

### Complete Examples

#### Q: What's a complete example of setting up Playwright E2E tests?

**A:** Complete examples are available for both runtimes:
- **Docker runtime:** See [docker.md](docker.md) for a complete Docker-based example.
- **Local runtime:** See [local.md](local.md) for local runtime setup guidance.
- **JavaScript library:** See [js-client/](js-client/README.md) for complete Playwright integration examples.

#### Q: How do I use the Stoobly JavaScript client library in my tests?

**A:** Install the `stoobly` package and use the Playwright or Cypress interceptor to record, mock, and test API requests directly from your test framework. See [js-client/](js-client/README.md) for installation, integration, and complete examples.

***

### Quick Reference

**Key files to modify for E2E testing:**

- **Docker runtime:** See [docker.md](docker.md) for file structure and customization details.
- **Local runtime:** See [local.md](local.md) for file structure and customization details.

**Common commands:**

- **Docker runtime (Makefile commands):** See [docker.md](docker.md) for complete command reference.
- **Local runtime (CLI commands):** See [local.md](local.md) for complete command reference.

**For more details:**
- Docker-specific customization: See [docker.md](docker.md)
- Local runtime customization: See [local.md](local.md)
- JavaScript client library: See [js-client/](js-client/README.md)
