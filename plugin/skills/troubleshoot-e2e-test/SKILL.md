---
name: troubleshoot-e2e-test
description: Troubleshoots E2E test failures by correlating Stoobly workflow request logs with Playwright or Cypress output. Use when E2E tests fail in CI or locally, mocks behave incorrectly, requests are not intercepted, or the user asks to debug scaffold test or mock workflows.
---

# Troubleshoot E2E Test

You are helping the user debug a failing E2E test.

## Instructions

Follow these steps in order when a scaffolded E2E test fails.

### 1. Resolve paths and running workflow

**Resolve app and context directories.** 

Run:

```bash
stoobly-agent scaffold describe
```

The command prints JSON with `app_dir_path`, `context_dir_path`, and config. 

Use the paths as follows for the rest of this skill:

- **Scaffold commands** (`stoobly-agent scaffold ...`) — run from `app_dir_path` (or pass `--app-dir-path <app_dir_path>`).
- **All other commands** (`stoobly-agent scenario ...`, `stoobly-agent request ...`, `stoobly-agent describe`, etc.) — run from `context_dir_path`.

```bash
cd <app_dir_path>
```

**Determine the running workflow.** From the app directory, run:

```bash
stoobly-agent scaffold workflow show
```

Note the `Workflow` name from the output (for example `test` or `mock`). Use that name as `<workflow>` in the steps below. If no workflow is running, ask the user which workflow they used for the failing test.

### 2. Get workflow request logs

From the app directory, list Stoobly workflow request logs. These show which requests were intercepted and whether each was mocked, passed through, or failed.

```bash
stoobly-agent scaffold request logs list <workflow>
```

Useful flags:

```bash
stoobly-agent scaffold request logs list <workflow> --level error
stoobly-agent scaffold request logs list <workflow> --message "Mock failure"
```

### 3. Cross-reference request logs with test framework logs

**Determine the E2E test framework.** Identify whether the project uses Playwright, Cypress, or another runner. Check scaffold plugins, `package.json` devDependencies, or config files (`playwright.config.ts`, `cypress.config.ts`).

**Find the framework logs.** Collect the test runner output for the failing test. For Playwright, inspect the **trace** when available — it is the recommended starting point for network activity and step-by-step execution.

**CI (failed run artifacts).** Use the trace, screenshot, and video from the failed CI run. Do not re-run the test with different flags to debug — re-running can change timing, data, or pass flakily.

```bash
npx playwright show-trace <trace.zip>   # from CI artifacts or test-results/
```

**Local (reproduce and capture).** Re-run with tracing only when debugging locally and no trace from the failure exists yet:

```bash
npx playwright test --trace on
npx playwright show-trace <trace.zip>
```

For Cypress, use the Cypress runner output, screenshots, and the Network tab in browser DevTools. In CI, use uploaded screenshots and videos from the failed run rather than re-running with different settings.

**Cross-reference framework logs with Stoobly workflow request logs.** For each failing request:

1. Find the URL, method, and approximate timestamp in the test framework logs (or Playwright trace network panel).
2. Find the same request in `stoobly-agent scaffold request logs list <workflow>`.
3. Compare status codes and response bodies.

If viewing the response body is needed, note the `request_key` and `context_dir_path` from the logs. Change directory to `context_dir_path` and run the following command:

```bash
stoobly-agent request response show <REQUEST-KEY>
```

Do not read snapshot files under `.stoobly/snapshots/` directly — always inspect mocks through `stoobly-agent request response show`.

### 4. Determine the root cause

**Read the test and the code under test (CUT) first.** Before diagnosing Stoobly configuration, open the failing test file and the application or service code it exercises. Understanding what the test expects and what the CUT actually requests clarifies whether a missing mock is a Stoobly misconfiguration or an intentional gap.

#### Status 499 or wrong recorded response in Stoobly request logs

A **499** status code means Stoobly intercepted the request but could not find a matching recorded request in the scenario. Confirm the failing request appears in workflow request logs with status 499 before proceeding.

A 499 can mean one of the following:

1. **No scenario key or name was passed.** The interceptor ran without `withScenarioKey()` / `withScenarioName()` (or equivalent), so Stoobly had no scenario to search. Warn the user and point them to set a scenario before enabling intercept. See [JS client troubleshooting](../../docs/faq/scaffold/e2e-testing/js-client/troubleshooting.md).

2. **The request was never recorded** — often because a filter rule in `.stoobly/settings.yml` excluded it, because the **record policy** is `api`, or because of the record strategy in use. That may be intentional. Use the test and CUT from above to decide whether the CUT actually needs that request:
   - If the CUT does **not** need it, treat the 499 as expected noise, or update the test (for example assertions or steps that still expect that request).
   - If the CUT **does** need it, diagnose why recording was skipped:
     - Run `stoobly-agent intercept show` (from `context_dir_path`) to check the record policy. When policy is `api`, Stoobly records only responses whose `Content-Type` is JSON (`application/json` or `application/*json`), XML (`application/xml` or `application/*xml`), or `text/plain`. HTML pages, images, scripts, and other asset responses are not recorded — compare the failing request's response `Content-Type` in the test framework logs or Playwright trace.
     - Check `.stoobly/settings.yml` for filter rules that exclude the request.
     - If `api` policy skipped a response the CUT needs, change to `all` and re-record: `stoobly-agent intercept set --mode record --policy all`, then run the record workflow again. If a filter rule blocked it, adjust that rule instead.
     - If a response should qualify under `api` but was still skipped (for example missing or incorrect `Content-Type`), fix the upstream response or use `all` policy when re-recording.

#### Request in test framework logs but not in Stoobly request logs

If the failing request appears in the test framework logs but not in Stoobly workflow request logs, it is not a 499 issue — the request was never intercepted. Check whether the test framework (for example Cypress or Playwright) has its proxy setting configured so traffic is routed through Stoobly.

#### Cause is environmental rather than test-specific

If the evidence points at setup rather than the test — HTTPS/certificate failures, a port conflict, Stoobly failing to start, or intercept never enabled — hand back to `triage`, which owns those fixes. Invoke it via your assistant's skill invocation. Pass along the workflow name and anything already learned from `stoobly-agent scaffold describe`.

#### No mock issues found in Stoobly request logs

If workflow request logs and test framework logs show no mock mismatches, missing interception, or other Stoobly issues, the likely cause is that the **test itself needs to be updated** — for example assertions, selectors, or expected UI/API behavior that no longer match the application. Update the test accordingly rather than changing Stoobly configuration or mocks.

#### Git status reveals uncommitted snapshots

If `git status` shows uncommitted files under `.stoobly/snapshots/`, the mocks were likely updated recently and are still correct. **Do not modify them.**

To inspect a mock, use the request response show command (never edit snapshot files by hand):

```bash
stoobly-agent request response show <REQUEST-KEY>
```

Uncommitted snapshots also point away from a Stoobly mock problem and toward a **test-related issue** — for example assertions, selectors, or expected UI/API behavior that no longer match the application. Focus on updating the test rather than changing mocks or Stoobly configuration.

## Reference documentation

| Topic | Doc |
| ----- | --- |
| Scaffold workflow logs | [Scaffold FAQ](../../docs/faq/scaffold/README.md) |
| Playwright integration | [Playwright](../../docs/faq/scaffold/e2e-testing/js-client/playwright.md) |
| Cypress integration | [Cypress](../../docs/faq/scaffold/e2e-testing/js-client/cypress.md) |
| Mock troubleshooting | [Troubleshooting](../../docs/guides/how-to-mock-apis/troubleshooting.md) |
| Configuration | [Config](../../docs/faq/config.md) |
| Request commands | [Request](../../docs/faq/request.md) |
| Scenario commands | [Scenario](../../docs/faq/scenario.md) |
| Snapshots | [Snapshot](../../docs/faq/snapshot.md) |
| Update request or response | [Update Request](update-request.md) |
| Triage general Stoobly problems | [Triage](triage.md) |
| E2E JS client troubleshooting | [JS client troubleshooting](../../docs/faq/scaffold/e2e-testing/js-client/troubleshooting.md) |
