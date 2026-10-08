---
name: js-client
description: Wires the Stoobly JavaScript client (npm `stoobly`) into an E2E test
  suite — interceptor setup, scenarios, npm scripts, and a verification run. Has
  dedicated support for Playwright and Cypress today, plus a generic interceptor
  for other frameworks (Puppeteer, WebdriverIO, TestCafe, vanilla JS, etc.). Use
  when the user wants to integrate Stoobly with their E2E tests, add Stoobly to
  Playwright/Cypress/another test framework, set up the JS client, use the
  `stoobly` npm package, or record/mock API traffic from their test code.
---

# JS Client Integration

You are helping the user wire the Stoobly JavaScript client library into their E2E test suite. This skill **writes code** into the user's repo (interceptor wiring, npm scripts) after showing exactly what will change and getting confirmation — it does not just explain the docs.

The library has dedicated, framework-tailored interceptors for **Playwright** and **Cypress** today — these are the popular examples covered in detail below, not an exhaustive list. Any other framework (Puppeteer, WebdriverIO, TestCafe, Nightwatch, Selenium, a custom runner, or plain Node/browser code) works through the generic `interceptor()` method; more framework-specific interceptors may be added over time, so re-check `stoobly/package.json`'s exports if a framework the user names isn't listed here.

If the test suite is already wired and a test is failing, use `/troubleshoot-e2e-test` instead — this skill is for first-time authoring, not debugging an existing integration.

## Step 1: Detect repo state (read-only)

Read `package.json` to determine:

- **E2E framework**: check `dependencies`/`devDependencies` for `@playwright/test`, `cypress`, and other common runners (`puppeteer`, `webdriverio`, `testcafe`, `nightwatch`, `selenium-webdriver`, etc.). Playwright and Cypress get dedicated wiring (Step 5); anything else uses the generic interceptor.
- **Existing `stoobly` dependency**: already installed?
- **Package manager**: infer from the lockfile present (`package-lock.json` → npm, `yarn.lock` → yarn, `pnpm-lock.yaml` → pnpm).
- **Module system**: `"type": "module"` (ESM) vs CommonJS.

Also check for a framework config file (`playwright.config.*`, `cypress.config.*`, or equivalent), a `tsconfig.json` (determines whether wiring should be TypeScript or JavaScript), and existing test/support directory layout so new files match it.

Run `node --version` and confirm it's 18 or higher — the library requires it.

If no framework is detected, ask the user which one they're using rather than guessing. If more than one is present, ask which they want wired.

## Step 2: Check prerequisites and hand off if needed

1. Run `stoobly-agent --version`. If not found, stop here — the `stoobly` npm package is a **client library**, not a CLI; it controls interception but a separate `stoobly-agent` install runs the workflows alongside it. Point the user at installation: `https://docs.stoobly.com/faq/installation`.
2. Check whether `.stoobly/services/.config.yml` exists in the current directory (existence check only — do not read its contents; see the hard constraint on scaffold state). The JS client needs a running `stoobly-agent` scaffold workflow to talk to. If it doesn't exist, tell the user and offer `/scaffold-create` — do not duplicate that flow here.
3. If it exists, run `stoobly-agent scaffold describe` to get `app_dir_path` and `context_dir_path`. Note whether `package.json` lives at `app_dir_path` or elsewhere (monorepo) — this determines whether npm scripts need `--app-dir-path`/`--context-dir-path` in Step 6.

## Step 3: Read the docs

Read files under `../../docs/faq/scaffold/e2e-testing/js-client/`:

- `setup.md` — installation, imports, `Stoobly` constructor
- `playwright.md` or `cypress.md` — if one of these applies; for any other framework, rely on `configuration.md` and the generic `interceptor()` reference in `setup.md`/`troubleshooting.md` instead, since there's no dedicated per-framework doc yet
- `configuration.md` — URL patterns, scenarios, sessions, record settings, intercept mode control (applies to every framework, dedicated or generic)
- `npm-scripts.md` — wrapping workflow commands in `package.json`

## Step 4: Ask configuration questions (single message)

1. **URLs to intercept** — which API base URL(s) should Stoobly intercept? (string or regex; multiple allowed)
2. **Scope** (Playwright only) — `withContext()` (recommended: covers pages created mid-test, extensions, service workers) or `withPage()` (single-page tests only)? Not applicable to other frameworks.
3. **Scenario naming** — confirm deriving scenario names from a hierarchical test path where the framework exposes one (`testInfo.titlePath.join(' > ')` in Playwright, `this.test.titlePath().join(' > ')` in Cypress) rather than managing a `scenarioKey` by hand. For frameworks without an equivalent, ask what should identify a scenario (suite/test name, a fixed string, etc.). This is the recommended default; only ask further if they want something else.
4. **`context.request` usage** (Playwright only) — do any tests call `context.request` / use Playwright's `APIRequestContext` directly? These aren't covered by `context.route()` and need extra header wiring.
5. **Mode control** — confirm using the `STOOBLY_INTERCEPT_MODE` environment variable (defaults to `mock`, set to `record` to capture) instead of hardcoding the mode.

## Step 5: Install and write the wiring

Install as a dev dependency with the detected package manager:

```bash
npm install stoobly --save-dev
# or
yarn add -D stoobly
```

Then show the **full proposed file contents** (new files) or diffs (edits to existing files) for the following, and ask "Ready to write these? (yes / let me review first)" before writing anything.

Use the dedicated Playwright or Cypress wiring below when one of those applies; otherwise use the generic interceptor further down.

### Playwright

Write a fixture module (path matching the repo's existing test layout, e.g. `tests/fixtures/stoobly.ts` or `.js`) that exports an extended `test`:

- `base.extend({ stooblyInterceptor: [async ({ context, page }, use, testInfo) => { ... }, { auto: true }] })`. `auto: true` is required so interception starts before any other fixture makes a network call.
- Inside the fixture: `const stoobly = new Stoobly()` (defaults to `http://localhost:4200`), then `stoobly.playwrightInterceptor({ urls, scenarioName, mode })`.
- `await interceptor.withContext(context).enable()` (or `.withPage(page)` per the Step 4 answer), then `interceptor.withTestTitle(testInfo.title)`.
- Mode: read `process.env.STOOBLY_INTERCEPT_MODE`, default `'mock'`, map to `InterceptMode` from `stoobly/constants`.
- If the user uses `context.request`: after `enable()`, add `await context.setExtraHTTPHeaders((interceptor as any).headers)` with a comment that `context.route()` does not intercept `APIRequestContext` calls, so headers must be copied onto the context explicitly.

Tell the user to import `test` from this fixture module instead of `@playwright/test` in their spec files. Offer to update one spec file as a worked example rather than rewriting the whole suite unasked.

### Cypress

Wire into `cypress/support/e2e.js` or `.ts` (create if it doesn't exist, otherwise append):

- Module-level: `const interceptor = stoobly.cypressInterceptor({ urls, mode })`.
- `beforeEach(function () { ... })` — must use the `function()` form, not an arrow function, so `this.test.titlePath()` is available.
- Inside: derive `scenarioName` from `this.test.titlePath().join(' > ')`, call `interceptor.withScenarioName(scenarioName)`, then `interceptor.enable()`.
- Comment explaining why `enable()` must run in `beforeEach`: Cypress clears all `cy.intercept`s between tests.
- If the repo has any synchronous `XMLHttpRequest` calls, flag that they'll hang Cypress with the interceptor active and should be converted to async/`cy.request`.

### Other frameworks (generic interceptor)

For any framework without a dedicated method — Puppeteer, WebdriverIO, TestCafe, Nightwatch, Selenium, a custom runner, or plain Node/browser code — use `stoobly.interceptor(settings)`. It patches `fetch`/`XMLHttpRequest` directly rather than hooking into a specific framework's routing layer, so it works anywhere JS requests originate, but you're responsible for calling `enable()`/`disable()` at the right point in that framework's lifecycle (equivalent of Playwright's fixture or Cypress's `beforeEach`) since there's no framework-specific auto-wiring to lean on.

- `const stoobly = new Stoobly()`, then `const interceptor = stoobly.interceptor({ scenarioName, urls, mode })`.
- Call `interceptor.enable()` wherever the framework runs setup before each test (its equivalent of `beforeEach`/a fixture/a hook), and `interceptor.disable()` in the matching teardown if the framework reuses the process across tests and you need a clean slate between them.
- Set `withTestTitle()` from whatever the framework exposes as the current test name, if anything.
- Because there's no built-in per-page/per-context scoping, verify network calls actually get intercepted in Step 7 before assuming this works — frameworks that run tests in separate processes/workers (rather than a shared Node process) may need the interceptor set up per worker.

### TypeScript

If `tsconfig.json` exists, mention that types are available from `stoobly/types`, and check `compilerOptions.types` includes `"stoobly"`, plus `moduleResolution` and `esModuleInterop` are set (see Troubleshooting doc) — fix if missing.

## Step 6: Add npm scripts

Per `npm-scripts.md`, add `stoobly:<workflow>` / `stoobly:<workflow>:down` script pairs for the workflow(s) the user runs (typically `mock`, and `record` if they'll capture traffic from tests), plus one combined script that brings the workflow up, runs tests, captures the exit code, and tears down regardless of pass/fail:

```json
{
  "scripts": {
    "stoobly:mock": "stoobly-agent scaffold workflow up mock --detached",
    "stoobly:mock:down": "stoobly-agent scaffold workflow down mock",
    "test:mock": "npm run stoobly:mock && npx playwright test; RESULT=$?; npm run stoobly:mock:down; exit $RESULT"
  }
}
```

(Swap `npx playwright test` for the repo's actual test command — `npx cypress run`, `npx testcafe`, or whatever the framework detected in Step 1 uses.)

Apply these footguns from the docs:

- **Always pass `--detached`** on `workflow up`. On Docker runtime, `up` attaches to logs and never returns without it, hanging the script before tests ever run; on local runtime it's a no-op. Passing it always keeps one script correct on both runtimes.
- **Capture the exit code before teardown.** Chaining with `&&` alone stops at the first failure and never tears down; chaining with `;` alone lets `down`'s exit code mask a test failure. Use the `RESULT=$?` pattern above so a failing test still fails the npm script (important for CI).
- **Use `cross-env`** for `STOOBLY_INTERCEPT_MODE=record` if the project needs to run on Windows — a bare env-var prefix only works on POSIX shells.
- **Pass `--app-dir-path`/`--context-dir-path`** on both `up` and `down` if Step 2 found `package.json` isn't at the scaffold app root (monorepo), using the same values on both so `down` can find what `up` started.

## Step 7: Verify

Don't assume the wiring works — run it:

1. Bring up the record workflow and run the suite once with `STOOBLY_INTERCEPT_MODE=record` to capture traffic.
2. Bring up the mock workflow and run the suite again with the default (`mock`) mode.
3. From `app_dir_path`, run `stoobly-agent scaffold request logs list <workflow>` and confirm the expected requests appear.

If nothing appears in the logs, interception isn't happening — check the URL patterns match, that `enable()` actually runs at the right point (fixture `auto: true` / Cypress `beforeEach` / the equivalent hook for another framework), and that `stoobly-agent` is up. If requests appear with **status 499**, they were intercepted but had no matching recording — hand off to `/troubleshoot-e2e-test` for that diagnosis.

Report the verification result plainly, including full output if either run failed — don't report success unless the logs actually confirm interception.

## Step 8: Surface next steps

End with:

> **Next steps:**
> - See [How to Record Requests](https://docs.stoobly.com/guides/how-to-record-requests/) to capture more traffic, and [How to Mock APIs](https://docs.stoobly.com/guides/how-to-mock-apis/) to serve it back without hitting real APIs.
> - See [Snapshot](https://docs.stoobly.com/faq/snapshot) to commit recordings to Git so teammates get the same mocks.
> - Run `/troubleshoot-e2e-test` if a test starts failing against mocks later.
> - For CI, install `stoobly-agent` via `pipx`, install the CA cert as a separate setup step, and pass `--ca-certs-install-confirm y --hostname-install-confirm y` to `workflow up` so it doesn't hang on interactive prompts — see the CI/CD section of the npm scripts doc.
>
> Docs: https://docs.stoobly.com/faq/scaffold/e2e-testing/js-client

## Reference documentation

| Topic | Doc |
| ----- | --- |
| Installation & imports | [Setup](../../docs/faq/scaffold/e2e-testing/js-client/setup.md) |
| Playwright integration | [Playwright](../../docs/faq/scaffold/e2e-testing/js-client/playwright.md) |
| Cypress integration | [Cypress](../../docs/faq/scaffold/e2e-testing/js-client/cypress.md) |
| URL/scenario/session/record configuration | [Configuration](../../docs/faq/scaffold/e2e-testing/js-client/configuration.md) |
| npm scripts & CI | [npm Scripts](../../docs/faq/scaffold/e2e-testing/js-client/npm-scripts.md) |
| Debugging & complete examples | [Troubleshooting](../../docs/faq/scaffold/e2e-testing/js-client/troubleshooting.md) |
| Concepts (intercept modes, scenarios, sessions, context) | [Integrating the JavaScript Client](../../docs/getting-started/integrating-the-javascript-client.md) |
| Full API reference | [TypeDoc reference](https://stoobly.github.io/stoobly-js/) |
| Scaffold setup | [Scaffold Create](scaffold-create.md) |
| Debug a failing E2E test | [Troubleshoot E2E Test](troubleshoot-e2e-test.md) |
| Share recordings with teammates | [Snapshot FAQ](https://docs.stoobly.com/faq/snapshot) |
