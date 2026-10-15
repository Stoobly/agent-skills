# JavaScript Client Library - Configuration

### Configuring URL Patterns

#### Q: How do I specify which URLs to intercept?

**A:** Use the `urls` array with strings or regular expressions to filter which requests Stoobly intercepts.

**Example:**

```javascript
const stoobly = new Stoobly();

// Exact URL match
const interceptor1 = stoobly.playwrightInterceptor({
  urls: ['https://api.example.com/users'],
});

// Regex pattern (match all API endpoints)
const interceptor2 = stoobly.playwrightInterceptor({
  urls: [new RegExp('https://api.example.com/.*')],
});

// Multiple URLs
const interceptor3 = stoobly.playwrightInterceptor({
  urls: [
    'https://api.example.com/users',
    'https://api.example.com/products',
    new RegExp('https://cdn.example.com/.*'),
  ],
});
```

#### Q: How do I configure per-URL options like match rules or fixture paths?

**A:** Pass `InterceptorUrl` objects in the `urls` array to attach per-URL configuration such as match rules, rewrite rules, a public directory path, or a response fixtures path.

**InterceptorUrl shape:**

```typescript
interface InterceptorUrl {
  pattern: RegExp | string;       // required: the URL to match
  matchRules?: MatchRule[];        // optional: which request components to match on
  rewriteRules?: RewriteRule[];    // optional: rules to rewrite request parameters or URL parts
  publicDirectoryPath?: string;    // optional: path to a public directory for static responses
  responseFixturesPath?: string;   // optional: path to response fixture files
}
```

**Example:**

```javascript
import Stoobly from 'stoobly';
import { InterceptMode, RequestParameter } from 'stoobly/constants';

const stoobly = new Stoobly();
const interceptor = stoobly.playwrightInterceptor({
  urls: [
    // Simple string — no per-URL config needed
    'https://api.example.com/users',

    // Simple RegExp — matches all API endpoints
    new RegExp('https://api.example.com/products/.*'),

    // InterceptorUrl object — match only on headers during replay
    {
      pattern: new RegExp('https://api.example.com/orders/.*'),
      matchRules: [
        { modes: [InterceptMode.replay], components: RequestParameter.Header },
      ],
    },

    // InterceptorUrl object — serve responses from local fixtures
    {
      pattern: 'https://api.example.com/catalog',
      responseFixturesPath: './fixtures/catalog',
    },
  ],
});
```

**Use `InterceptorUrl` when:**

1. You need different match rules for specific endpoints
2. You want to serve static fixture files for certain URLs
3. You need rewrite rules applied to a subset of intercepted URLs

---

#### Q: How do I change the intercepted URLs dynamically?

**A:** Pass a new `urls` array to `enable()` to update which URLs are intercepted.

**Example:**

```javascript
const interceptor = stoobly.playwrightInterceptor({
  urls: [new RegExp('https://api.example.com/.*')],
});

test.beforeEach(async ({ page }) => {
  await interceptor.withPage(page).enable();
});

test('changes URLs dynamically', async ({ page }) => {
  // Initially intercepts api.example.com
  await page.goto('https://example.com');

  // Change to intercept different URLs
  await interceptor.enable({
    urls: [new RegExp('https://cdn.example.com/.*')]
  });

  // Now intercepts cdn.example.com instead
  await page.goto('https://example.com');
});
```

***

### Request Matching

#### Q: What determines whether a request matches a recording during mocking?

**A:** Stoobly matches requests against recordings using these components, in order (case-sensitive):

1. **HTTP Method** (required) — GET, POST, PUT, DELETE, etc.
2. **Path** (required) — e.g., `/users`
3. **Query Parameters** — Sorted alphabetically before comparison
4. **Headers** — Sorted alphabetically before comparison
5. **Body** — Strict matching if provided
6. **Body Parameters** — Parsed from JSON or form-urlencoded, sorted alphabetically

If a `scenarioKey` or `scenarioName` is set, only requests within that scenario are considered; otherwise any matching recorded request is used. Multiple matches within a scenario return responses in recording order. Use per-URL `matchRules` (see above) to relax which components are compared.

***

### Scenarios and Sessions

#### Q: How do I specify a scenario for my tests?

**A:** Prefer `scenarioName` in the interceptor options to associate requests with a scenario. This is more intuitive and avoids managing keys. You can also derive it from your test framework (e.g., Playwright `testInfo.titlePath.join(' > ')`).

**Example:**

```javascript
// Using scenario name (recommended)
const interceptor1 = stoobly.playwrightInterceptor({
  scenarioName: 'my-test-scenario',
  urls: [new RegExp('https://api.example.com/.*')],
});

// Deriving scenario name from environment or test metadata
const scenarioFromEnv = process.env.STOOBLY_SCENARIO_NAME || 'default-scenario';
const interceptor2 = stoobly.playwrightInterceptor({
  scenarioName: scenarioFromEnv,
  urls: [new RegExp('https://api.example.com/.*')],
});
```

#### Q: How do I change the scenario dynamically?

**A:** Use `withScenarioName()` to update the scenario.

**Example:**

```javascript
const interceptor = stoobly.playwrightInterceptor({
  urls: [new RegExp('https://api.example.com/.*')],
});

test('switches scenarios', async ({ page }) => {
  // Use scenario A
  interceptor.withScenarioName('scenario-a');
  await interceptor.withPage(page).enable();
  await page.goto('https://example.com');

  // Switch to scenario B
  interceptor.withScenarioName('scenario-b');
  await page.goto('https://example.com/other');
});
```

#### Q: What is a session ID and how do I use it?

**A:** A session ID groups requests together within a scenario. It defaults to the current timestamp but can be customized for test reproducibility.

**Example:**

```javascript
// Default session ID (auto-generated timestamp)
const interceptor1 = stoobly.playwrightInterceptor({
  urls: [new RegExp('https://api.example.com/.*')],
});

// Custom session ID
const interceptor2 = stoobly.playwrightInterceptor({
  sessionId: 'test-session-123',
  urls: [new RegExp('https://api.example.com/.*')],
});

// Change session ID dynamically
test('changes session', async ({ page }) => {
  interceptor.withSessionId('session-1');
  await interceptor.withPage(page).enable();
  await page.goto('https://example.com');

  interceptor.withSessionId('session-2');
  await page.goto('https://example.com/other');
});
```

***

### Recording Configuration

#### Q: What record policies are available?

**A:** Stoobly supports three record policies: `All` (record everything), `Found` (record only if request exists), and `NotFound` (record only new requests).

**Example:**

```javascript
import { RecordPolicy } from 'stoobly/constants';

// Record all requests (default)
const interceptor1 = stoobly.playwrightInterceptor({
  urls: [new RegExp('https://api.example.com/.*')],
  record: { policy: RecordPolicy.All },
});

// Record only if request already exists in scenario
const interceptor2 = stoobly.playwrightInterceptor({
  urls: [new RegExp('https://api.example.com/.*')],
  record: { policy: RecordPolicy.Found },
});

// Record only new requests (not already in scenario)
const interceptor3 = stoobly.playwrightInterceptor({
  urls: [new RegExp('https://api.example.com/.*')],
  record: { policy: RecordPolicy.NotFound },
});
```

#### Q: What's the difference between record orders?

**A:** `Overwrite` replaces existing requests with the same signature, while `Append` always creates new request records.

**Example:**

```javascript
import { RecordOrder } from 'stoobly/constants';

// Overwrite existing requests (useful for updating mocks)
const interceptor1 = stoobly.playwrightInterceptor({
  urls: [new RegExp('https://api.example.com/.*')],
  record: { order: RecordOrder.Overwrite },
});

// Append new requests (useful for collecting multiple responses)
const interceptor2 = stoobly.playwrightInterceptor({
  urls: [new RegExp('https://api.example.com/.*')],
  record: { order: RecordOrder.Append },
});
```

**Note:** `Overwrite` is sent only once **per URL pattern** per session. Subsequent requests to the same URL pattern use `Append` behavior. Each URL pattern is tracked independently, so multiple patterns can each receive one overwrite.

#### Q: What record strategies are available?

**A:** Stoobly supports `Full` (record complete request/response) and `Minimal` (record only essential data).

**Example:**

```javascript
import { RecordStrategy } from 'stoobly/constants';

// Full recording (complete request and response data)
const interceptor1 = stoobly.playwrightInterceptor({
  urls: [new RegExp('https://api.example.com/.*')],
  record: { strategy: RecordStrategy.Full },
});

// Minimal recording (essential data only)
const interceptor2 = stoobly.playwrightInterceptor({
  urls: [new RegExp('https://api.example.com/.*')],
  record: { strategy: RecordStrategy.Minimal },
});
```

#### Q: How do I change record settings dynamically?

**A:** Use `withRecordPolicy()`, `withRecordOrder()`, and `withRecordStrategy()` methods.

**Example:**

```javascript
import { InterceptMode, RecordPolicy, RecordOrder, RecordStrategy } from 'stoobly/constants';

const interceptor = stoobly.playwrightInterceptor({
  urls: [new RegExp('https://api.example.com/.*')],
});

test('changes record settings', async ({ page }) => {
  // Set record options
  interceptor.withRecordPolicy(RecordPolicy.All);
  interceptor.withRecordOrder(RecordOrder.Overwrite);
  interceptor.withRecordStrategy(RecordStrategy.Full);

  await interceptor.withPage(page).enable({ mode: InterceptMode.record });
  await page.goto('https://example.com');

  // Change to minimal strategy
  interceptor.withRecordStrategy(RecordStrategy.Minimal);
  await page.goto('https://example.com/other');
});
```

***

### Controlling Interception

#### Q: How do I manage the intercept mode (mock, record, replay) for tests?

**A:** The simplest, CI-friendly way is to set the `STOOBLY_INTERCEPT_MODE` environment variable before running your tests. This avoids hardcoding modes in code and keeps behavior consistent across local and CI.

Try:

1. Set an environment variable for the whole test run:
   - macOS/Linux:
     ```bash
     export STOOBLY_INTERCEPT_MODE=mock
     npx playwright test
     ```
   - One-off run:
     ```bash
     STOOBLY_INTERCEPT_MODE=record npx playwright test
     ```
2. Set per-interceptor in code if needed for a specific suite (derive from the environment variable):
   ```javascript
   import Stoobly from 'stoobly';
   import { InterceptMode } from 'stoobly/constants';

   // Derive mode from environment with a safe default
   const envMode = (process.env.STOOBLY_INTERCEPT_MODE || 'mock').toLowerCase();
   const mode =
     envMode === 'record' ? InterceptMode.record :
     envMode === 'replay' ? InterceptMode.replay :
     InterceptMode.mock; // default

   const stoobly = new Stoobly();
   const interceptor = stoobly.playwrightInterceptor({
     urls: [new RegExp('https://api.example.com/.*')],
     mode,
   });
   ```

More details: Intercept FAQ (`https://docs.stoobly.com/faq/intercept`)

#### Q: How do I stop recording requests?

**A:** Call `enable({ mode: InterceptMode.mock })` (or another non-record mode) to stop recording while keeping interception active. `apply`/`clear` remain as deprecated aliases for `enable`/`disable`.

**Example:**

```javascript
import { InterceptMode } from 'stoobly/constants';

test('stops recording', async ({ page }) => {
  // Start recording
  await interceptor.withPage(page).enable({ mode: InterceptMode.record });
  await page.goto('https://example.com'); // Recorded

  // Stop recording (still intercepted — typically mock)
  await interceptor.enable({ mode: InterceptMode.mock });
  await page.goto('https://example.com/other'); // Mock / your non-record mode
});
```

#### Q: How do I completely remove the interceptor?

**A:** Use `disable()` to remove all interception and reset session state for the next `enable()`.

**Example:**

```javascript
test('removes interceptor', async ({ page }) => {
  // Enable interceptor
  await interceptor.withPage(page).enable();
  await page.goto('https://example.com'); // Intercepted

  // Remove interceptor
  await interceptor.disable();
  await page.goto('https://example.com/other'); // Not intercepted
});
```

#### Q: What's the difference between `disable()` and switching modes with `enable()`?

**A:** `disable()` tears down interception (routes / patches) and resets the session id that `enable()` will assign next. To **keep** intercepting but stop recording, call `enable({ mode: InterceptMode.mock })` instead of `disable()`.

**Example:**

```javascript
import { InterceptMode } from 'stoobly/constants';

// Switch mode — keeps interception active
test('switch from record to mock', async ({ page }) => {
  await interceptor.withPage(page).enable({ mode: InterceptMode.record });
  await page.goto('https://example.com'); // Recorded

  await interceptor.enable({ mode: InterceptMode.mock });
  await page.goto('https://example.com'); // Mock only
});

// disable() — removes everything
test('disable example', async ({ page }) => {
  await interceptor.withPage(page).enable();
  await page.goto('https://example.com'); // Intercepted

  await interceptor.disable();
  await page.goto('https://example.com'); // No interception
});
```

***

### Advanced Configuration

#### Q: How do I set a custom Stoobly UI URL?

**A:** Pass the UI URL to the Stoobly constructor if your agent is running on a different port or host.

**Example:**

```javascript
// Default (http://localhost:4200)
const stoobly1 = new Stoobly();

// Custom URL
const stoobly2 = new Stoobly('http://localhost:8080');

// Remote agent
const stoobly3 = new Stoobly('https://stoobly-agent.company.com');
```

#### Q: How do I use test titles for request grouping?

**A:** Set test titles using `withTestTitle()` to group requests by test name in the Stoobly UI.

**Example:**

```javascript
// Playwright (manual test title)
test.beforeEach(async ({ page }, testInfo) => {
  await interceptor.withPage(page).enable();
  interceptor.withTestTitle(testInfo.title);
});

// Cypress (auto-detected test title)
beforeEach(() => {
  interceptor.enable();
  // Test title is automatically detected
});

// Clear test title
interceptor.withTestTitle(undefined);
```

#### Q: Can I use the interceptor without a test framework?

**A:** Yes, use the generic `interceptor()` method for vanilla JavaScript applications.

**Example:**

```javascript
import Stoobly from 'stoobly';

const stoobly = new Stoobly();
const interceptor = stoobly.interceptor({
  scenarioName: '<SCENARIO-NAME>',
  urls: [new RegExp('https://api.example.com/.*')],
});

// Enable interception
const sessionId = interceptor.enable();
console.log('Session ID:', sessionId);

// Make requests (fetch/XMLHttpRequest will be intercepted)
fetch('https://api.example.com/users');

// Stop interception
interceptor.disable();
```

#### Q: How do I read or update the Stoobly agent's configuration from my tests?

**A:** Use the `config` property on the `Stoobly` instance to dump the agent's full configuration, get a summary, or set the active scenario.

**Example:**

```javascript
const stoobly = new Stoobly();

// Get full configuration
const config = await stoobly.config.dump();

// Get configuration summary
const summary = await stoobly.config.summary();

// Set active scenario
await stoobly.config.scenario.set('<SCENARIO-KEY>');
```

***

### Constants Reference

#### Q: What other constants does `stoobly/constants` export besides RecordPolicy, RecordOrder, RecordStrategy, and InterceptMode?

**A:** `stoobly/constants` also exports enums for mocking, replaying, testing, and request filtering/matching:

```typescript
enum MockPolicy {
  All = 'all'     // Mock all requests (return 404 if not found)
  Found = 'found' // Mock only requests with recordings (pass-through others)
}

enum ReplayPolicy {
  All = 'all'  // Replay all requests
}

enum TestPolicy {
  All = 'all'     // Test all requests
  Found = 'found' // Test only requests with recordings
}

enum TestStrategy {
  Diff = 'diff'     // Exact difference comparison
  Fuzzy = 'fuzzy'   // Approximate comparison
  Custom = 'custom' // Custom comparison logic
}

enum FilterAction {
  Exclude = 'exclude' // Block matching URLs
  Include = 'include' // Allow matching URLs
}

enum RequestParameter {
  Header = 'Header'          // HTTP headers
  BodyParam = 'Body Param'   // Request body parameters
  QueryParam = 'Query Param' // URL query parameters
}
```

`MockPolicy` is set via `mock: { policy }` in `InterceptorSettings` (or `withMockPolicy()`). `RequestParameter` is used in per-URL `matchRules`/`rewriteRules` (see [Configuring URL Patterns](#configuring-url-patterns) above) to specify which request components to match on or rewrite.
