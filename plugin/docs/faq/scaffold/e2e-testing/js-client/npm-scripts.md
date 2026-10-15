# JavaScript Client Library - npm Scripts

### Basic Workflow Scripts

#### Q: How do I run stoobly-agent workflows from npm scripts?

**A:** Add `stoobly:<workflow>` and `stoobly:<workflow>:down` scripts to your `package.json` that call the `stoobly-agent` CLI directly. The `stoobly` npm package is a client library, not a CLI wrapper, so scripts call `stoobly-agent` (installed separately — see [Setup](setup.md)), not `npx stoobly`.

**Example:**

```json
{
  "scripts": {
    "stoobly:mock": "stoobly-agent scaffold workflow up mock --detached",
    "stoobly:mock:down": "stoobly-agent scaffold workflow down mock",
    "stoobly:record": "stoobly-agent scaffold workflow up record --detached",
    "stoobly:record:down": "stoobly-agent scaffold workflow down record",
    "stoobly:test": "stoobly-agent scaffold workflow up test --detached",
    "stoobly:test:down": "stoobly-agent scaffold workflow down test",
    "stoobly:develop": "stoobly-agent scaffold workflow up develop --detached",
    "stoobly:develop:down": "stoobly-agent scaffold workflow down develop"
  }
}
```

Run with `npm run stoobly:mock`, then `npm run stoobly:mock:down` when finished.

`mock`, `record`, `test`, and `develop` aren't the only valid workflow names — any workflow created with `scaffold workflow create` (see [Creating Custom Workflows](../../README.md)) works the same way in these scripts.

#### Q: What if my `package.json` isn't at the scaffold app root?

**A:** Scaffold commands default to the current directory for both the app scaffold (`.stoobly/`) and the Stoobly context data. If your npm scripts run from elsewhere — for example a `package.json` nested in a monorepo package — pass `--app-dir-path` pointing at the scaffold app root, and `--context-dir-path` if you want context data (recordings, scenarios) to live somewhere other than that root's default. Both accept relative paths, resolved from the directory `npm run` executes in.

**Example:**

```json
{
  "scripts": {
    "stoobly:mock": "stoobly-agent scaffold workflow up mock --detached --app-dir-path ../.. --context-dir-path .",
    "stoobly:mock:down": "stoobly-agent scaffold workflow down mock --app-dir-path ../.. --context-dir-path ."
  }
}
```

`down` needs the same `--app-dir-path`/`--context-dir-path` values as `up` — they identify which running workflow to stop. See the [Scaffold FAQ](../README.md) for more on `--app-dir-path`.

#### Q: Why do my scripts need `--detached`?

**A:** On Docker runtime, `scaffold workflow up` attaches to the entrypoint service's logs by default and doesn't return until you kill it — so a script like `"stoobly-agent scaffold workflow up mock && npx playwright test"` hangs at `up` and never reaches your tests. Pass `--detached` so `up` starts the workflow in the background and returns immediately.

**Example:**

```bash
# Hangs — up never returns, playwright test never runs
stoobly-agent scaffold workflow up mock && npx playwright test

# Returns immediately, tests run next
stoobly-agent scaffold workflow up mock --detached && npx playwright test
```

On local runtime, `up` already starts the agent in the background and returns on its own, so `--detached` is a no-op there. Passing it on every script is safe either way and keeps one script working across both runtimes — see [Docker](../docker.md) and [Local](../local.md) for the runtime differences.

***

### Running Tests

#### Q: How do I bring the workflow up, run tests, and tear it down in one command?

**A:** Run `up`, then your test command, capturing its exit code before tearing down — otherwise a failing test run either leaves the workflow running (if you stop at the first failure) or gets its failure masked by `down`'s own exit code (if you naively chain everything with `;`). `scaffold workflow down` is safe to call even when nothing is running (it no-ops), so it's fine to run unconditionally after `up` regardless of whether `up` itself succeeded.

**Example:**

```json
{
  "scripts": {
    "stoobly:mock": "stoobly-agent scaffold workflow up mock --detached",
    "stoobly:mock:down": "stoobly-agent scaffold workflow down mock",
    "test:mock": "npm run stoobly:mock && npx playwright test; RESULT=$?; npm run stoobly:mock:down; exit $RESULT"
  }
}
```

`RESULT=$?` right after `up && test` captures whichever of the two failed (or `0` if both succeeded), `down` always runs next, and `exit $RESULT` makes sure a test failure still fails the npm script — important for CI to detect it.

`scaffold workflow up` returns control to the shell right away (with `--detached`, always, on both runtimes) rather than staying attached, so teardown needs this explicit `down` step regardless of how your test command itself is structured. That's true even if your test command is a wrapper that also starts and stops your app's dev server — common approaches include `start-server-and-test`, Playwright's built-in [`webServer`](https://playwright.dev/docs/test-webserver) config, or just running the dev server in a separate terminal. Whichever you use, it only manages your app's dev server, not the Stoobly workflow — `stoobly:mock`/`stoobly:mock:down` still wrap it as shown above.

If that dev server itself makes HTTPS calls through the Stoobly proxy, it may need to be told to trust Stoobly's certificate separately from your browser — see the [CA Cert FAQ](../../../ca-cert.md).

#### Q: How do I pick the intercept mode from an npm script?

**A:** Set `STOOBLY_INTERCEPT_MODE` inline before your test command. See [Configuration](configuration.md) for how the JS client reads this variable.

**Example:**

```json
{
  "scripts": {
    "stoobly:record": "stoobly-agent scaffold workflow up record --detached",
    "stoobly:record:down": "stoobly-agent scaffold workflow down record",
    "test:record": "npm run stoobly:record && cross-env STOOBLY_INTERCEPT_MODE=record npx playwright test; RESULT=$?; npm run stoobly:record:down; exit $RESULT"
  }
}
```

`cross-env` sets the variable in a way that works on both POSIX shells (macOS/Linux) and Windows; without it, `STOOBLY_INTERCEPT_MODE=record npx playwright test` only works on POSIX shells.

***

### Diagnostics

#### Q: How do I check what's running or view logs from npm?

**A:** Add scripts for `scaffold workflow show`, `scaffold workflow logs`, and `scaffold request logs list`. There are two distinct log commands and both are worth having: `workflow logs` shows raw workflow process output (startup errors, config issues), while `request logs list` shows what requests were intercepted and whether they were mocked or passed through — the first thing to check when debugging mock behavior. `show` works with no arguments (lists all running workflows); the two log commands require a workflow name, so pass it through with `npm run <script> --`.

**Example:**

```json
{
  "scripts": {
    "stoobly:show": "stoobly-agent scaffold workflow show",
    "stoobly:workflow-logs": "stoobly-agent scaffold workflow logs",
    "stoobly:request-logs": "stoobly-agent scaffold request logs list"
  }
}
```

```bash
npm run stoobly:show
npm run stoobly:workflow-logs -- mock --follow
npm run stoobly:request-logs -- mock --follow
```

`--follow` streams either log in real time and works the same way on both.

***

### CI/CD

#### Q: How do I make these scripts work in CI?

**A:** `scaffold workflow up` can prompt to install the CA certificate (first run of the record workflow) and, on Docker runtime, to install hostnames — both hang a non-interactive CI job. Pass `--ca-certs-install-confirm y` and `--hostname-install-confirm y` to answer them automatically, and install the CA cert as a separate setup step.

**Example:**

```json
{
  "scripts": {
    "stoobly:test": "stoobly-agent scaffold workflow up test --detached --ca-certs-install-confirm y --hostname-install-confirm y",
    "stoobly:test:down": "stoobly-agent scaffold workflow down test"
  }
}
```

```yaml
# .github/workflows/test.yml
name: E2E Tests

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v4
      - name: Install stoobly-agent
        run: pipx install stoobly-agent
      - name: Install Stoobly CA cert
        run: sudo stoobly-agent ca-cert install
      - name: Install dependencies
        run: npm ci
      - name: Run E2E tests
        run: npm run test:mock
```

See [Local](../local.md) for more on running the local runtime in CI.
