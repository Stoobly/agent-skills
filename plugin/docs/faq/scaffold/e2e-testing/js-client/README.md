# JS Client

The Stoobly JavaScript library (`stoobly`) provides convenient access to `stoobly-agent` from end-to-end testing frameworks such Playwright and Cypress. It enables recording, mocking, and testing HTTP requests directly from your test framework.

## Sub-Pages

* [Setup](setup.md) — Installation, requirements, and importing the library
* [npm Scripts](npm-scripts.md) — Wrapping `stoobly-agent` workflow commands in `package.json`
* [Playwright Integration](playwright.md) — Using `playwrightInterceptor()`, `withPage()`, `withContext()`, and recording
* [Cypress Integration](cypress.md) — Using `cypressInterceptor()`, `enable()`, and recording
* [Configuration](configuration.md) — URL patterns, scenarios, sessions, recording options, and interception control
* [Troubleshooting & Examples](troubleshooting.md) — Debugging, complete examples, and quick reference

## Related Documentation

* For a conceptual intro (intercept modes, scenarios, sessions, context), see [Using the JavaScript Client](../../../../getting-started/integrating-the-javascript-client.md)
* For scaffold integration, see [../README.md](../)
* For Docker runtime, see [../docker.md](../docker.md)
* For local runtime, see [../local.md](../local.md)
