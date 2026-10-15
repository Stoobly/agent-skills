---
description: An introduction to the Stoobly JavaScript client library for E2E testing
---

# Integrating the JavaScript Client

The Stoobly JavaScript library (npm package `stoobly`) lets you record, mock, and test HTTP requests directly from end-to-end testing frameworks like Playwright and Cypress. Use it alongside a `stoobly-agent` scaffold workflow to control interception from your tests.

## What is the Stoobly JavaScript client library?

The `stoobly` npm package (the Stoobly JavaScript client library) integrates with the [stoobly-agent](installing-the-agent/) to enable:

* **Recording** — Capture HTTP requests and responses for later replay
* **Mocking** — Return pre-recorded responses instead of hitting real APIs
* **Replay** — Replay recorded requests
* **Testing** — Validate API responses against recorded data

## Prerequisite

Before using the JavaScript client library, set up Stoobly for E2E testing:

1. [Install the agent](installing-the-agent/) — `stoobly-agent` must be available on your machine or in CI
2. [Integrate E2E testing](../guides/how-to-integrate-e2e-testing/) — create a scaffold app and services with `--plugin playwright` or `--plugin cypress`

The JavaScript client library builds on that scaffold setup. It does not replace creating the app, services, or workflows.

## Installation

```bash
npm install stoobly --save-dev
```

Requires Node.js 18 or higher. See [Setup](../faq/scaffold/e2e-testing/js-client/setup.md) for import patterns and framework-specific install steps.

## Integrate with your test framework

After installing the package, wire Stoobly into your E2E tests:

* [Playwright](../faq/scaffold/e2e-testing/js-client/playwright.md) — `playwrightInterceptor()`, `withPage()`, `withContext()`, and recording
* [Cypress](../faq/scaffold/e2e-testing/js-client/cypress.md) — `cypressInterceptor()`, `enable()`, and recording

See [Setup](../faq/scaffold/e2e-testing/js-client/setup.md) for import patterns and framework-specific install steps.

## Running the Agent from npm scripts

With your scaffold in place, start the workflow that matches your test mode (`mock`, `record`, or `test`) before running tests. Rather than starting and stopping it by hand in a separate terminal, wrap it in `package.json` scripts so it fits your existing `npm test` flow:

```json
{
  "scripts": {
    "stoobly:mock": "stoobly-agent scaffold workflow up mock --detached",
    "stoobly:mock:down": "stoobly-agent scaffold workflow down mock",
    "test:mock": "npm run stoobly:mock && (npx playwright test; code=$?; npm run stoobly:mock:down; exit $code)"
  }
}
```

See [npm scripts](../faq/scaffold/e2e-testing/js-client/npm-scripts.md) for the full set of recommended scripts, including record/test variants, diagnostics, and CI setup.

## Configure interception

[Configuration](../faq/scaffold/e2e-testing/js-client/configuration.md) covers URL patterns, scenarios, sessions, recording options, and how to start and stop interception from test code.

## Reference

* [JavaScript Client Library FAQ](../faq/scaffold/e2e-testing/js-client/) — full how-to index
* [TypeDoc reference](https://stoobly.github.io/stoobly-js/) — class, method, and type documentation
* [Troubleshooting & examples](../faq/scaffold/e2e-testing/js-client/troubleshooting.md) — debugging and complete examples
* [GitHub README](https://github.com/Stoobly/stoobly-js/blob/main/README.md) — quick-start code samples
