# JavaScript Client Library - Playwright Integration

### Playwright Integration

#### Q: How do I integrate Stoobly with Playwright tests?

**A:** Prefer initializing Stoobly in a custom Playwright fixture so interception starts before any other fixtures that might trigger network calls. In that fixture, set `withContext()` or `withPage()`, call `enable()` (pass `{ mode: InterceptMode.record }` when recording), and set `withTestTitle()` using `testInfo`.

Instead of managing a `scenarioKey`, derive a scenario name from the hierarchical test title path — this is more intuitive and one less thing to configure. Compute it from `testInfo.titlePath` and pass it in the interceptor options:

`const scenarioName = testInfo.titlePath.join(' > '); // pass as { scenarioName }`

**Example:**

```javascript
import { test as base, expect } from '@playwright/test';
import Stoobly from 'stoobly';

// Create a test fixture that initializes Stoobly early
const test = base.extend({
  stooblyInterceptor: [
    async ({ context, page }, use, testInfo) => {
      const stoobly = new Stoobly();
      const scenarioName = testInfo.titlePath.join(' > ');
      const interceptor = stoobly.playwrightInterceptor({
        urls: [new RegExp('https://api.example.com/.*')],
        scenarioName,
      });

      // Choose one based on scope:
      await interceptor.withContext(context).enable(); // recommended for multi-page tests
      // await interceptor.withPage(page).enable();    // for single-page tests
      interceptor.withTestTitle(testInfo.title);

      await use(undefined);
    },
    { auto: true }, // ensure it runs automatically before the test starts
  ],
});

test.describe('My Tests', () => {
  test('can fetch data', async ({ page }) => {
    await page.goto('https://example.com');
    // Your test code here
  });
});
```

#### Q: Why do I need to call `withPage()` and `withTestTitle()` in Playwright?

**A:** Playwright doesn't provide a global API to auto-detect the current page or test title, so you must explicitly set them in a setup point. Use a custom fixture (recommended) so interception is active before other fixtures run, or set them in `beforeEach` if you cannot use a fixture.

**Example:**

```javascript
// Fixture approach (recommended)
const test = base.extend({
  stooblyInterceptor: [
    async ({ context, page }, use, testInfo) => {
      const stoobly = new Stoobly();
      const interceptor = stoobly.playwrightInterceptor({
        urls: [/https:\/\/api\.example\.com\/.*/],
        scenarioName: testInfo.titlePath.join(' > '),
      });
      await interceptor.withContext(context).enable();
      interceptor.withTestTitle(testInfo.title);
      await use(undefined);
    },
    { auto: true },
  ],
});

// Fallback: beforeEach (if fixtures not possible)
// test.beforeEach(async ({ page }, testInfo) => {
//   await interceptor.withPage(page).enable();
//   interceptor.withTestTitle(testInfo.title);
// });
```

#### Q: When should I use `withContext()` instead of `withPage()`?

**A:** Use `withContext()` when you need to intercept requests from all pages in a browser context, including new pages created during tests. Use `withPage()` when you only want to intercept requests from a specific page.

**Key differences:**

- **`withPage()`** - Intercepts requests only from the specified page. New pages created in the same context will NOT be intercepted.
- **`withContext()`** - Intercepts requests from all pages in the browser context, including pages created with `context.newPage()`, browser extensions, and service workers.
- **Both together** - You can use both `withContext()` and `withPage()` to ensure all pages are intercepted.

**Example:**

```javascript
import { test } from '@playwright/test';
import Stoobly from 'stoobly';

const stoobly = new Stoobly();
const interceptor = stoobly.playwrightInterceptor({
  urls: [new RegExp('https://api.example.com/.*')],
});

// Scenario 1: Single page interception
test.describe('Page-only interception', () => {
  // Prefer setting this in a fixture; shown here only if fixtures aren't used
  test.beforeEach(async ({ page }, testInfo) => {
    await interceptor.withPage(page).enable();
    interceptor.withTestTitle(testInfo.title);
    interceptor.withScenarioName(testInfo.titlePath.join(' > '));
  });

  test('intercepts only the fixture page', async ({ context, page }) => {
    await page.goto('https://example.com'); // ✓ Intercepted

    // Create a new page
    const page2 = await context.newPage();
    await page2.goto('https://example.com'); // ✗ NOT intercepted
    await page2.close();
  });
});

// Scenario 2: Context-wide interception (recommended for multi-page tests)
test.describe('Context-wide interception', () => {
  // Prefer setting this in a fixture; shown here only if fixtures aren't used
  test.beforeEach(async ({ context }, testInfo) => {
    await interceptor.withContext(context).enable();
    interceptor.withTestTitle(testInfo.title);
    interceptor.withScenarioName(testInfo.titlePath.join(' > '));
  });

  test('intercepts all pages in context', async ({ context }) => {
    const page1 = await context.newPage();
    await page1.goto('https://example.com'); // ✓ Intercepted

    const page2 = await context.newPage();
    await page2.goto('https://example.com'); // ✓ Intercepted

    await page1.close();
    await page2.close();
  });
});

// Scenario 3: Both page and context interception
test.describe('Dual interception', () => {
  // Prefer setting this in a fixture; shown here only if fixtures aren't used
  test.beforeEach(async ({ context, page }, testInfo) => {
    await interceptor.withContext(context).withPage(page).enable();
    interceptor.withTestTitle(testInfo.title);
  });

  test('intercepts fixture page and new pages', async ({ context, page }) => {
    await page.goto('https://example.com'); // ✓ Intercepted (via both)

    const page2 = await context.newPage();
    await page2.goto('https://example.com'); // ✓ Intercepted (via context)
    await page2.close();
  });
});
```

**Use `withContext()` when:**

1. Your tests create multiple pages (popups, new tabs)
2. You're testing browser extensions that make background requests
3. You're testing service workers
4. You want consistent interception across all pages without calling `withPage()` for each

**Use `withPage()` when:**

1. You only work with the test fixture page
2. You want fine-grained control over which pages are intercepted
3. You're following the typical Playwright test pattern with a single page

#### Q: How do I intercept `context.request` (API tests) in Playwright?

**A:** `withPage()` and `withContext()` use Playwright's `page.route()` / `context.route()` internally, which only intercepts requests made by **browser pages** — not requests made via `context.request` (Playwright's `APIRequestContext`). For API tests that use `context.request`, you must also call `context.setExtraHTTPHeaders()` with the interceptor's headers after `enable()`.

This works because `context.request` calls still travel through any proxy configured in `playwright.config.ts`. Adding the Stoobly headers via `setExtraHTTPHeaders` ensures those calls carry the correct `X-Stoobly-Proxy-Mode`, `X-Stoobly-Scenario-Key`, and session headers to the proxy.

**Example:**

```javascript
import { test } from '@playwright/test';
import Stoobly from 'stoobly';
import { InterceptMode, RecordPolicy, RecordOrder, RecordStrategy } from 'stoobly/constants';

const isRecording = process.env.STOOBLY_RECORD === 'true';

const stoobly = new Stoobly();
const interceptor = stoobly.playwrightInterceptor({
  urls: [new RegExp('http://localhost:3000/api/.*')],
  record: {
    policy: RecordPolicy.All,
    order: RecordOrder.Overwrite,
    strategy: RecordStrategy.Full,
  },
});

test.describe('API Tests', () => {
  test.beforeEach(async ({ context }, testInfo) => {
    interceptor.withContext(context);
    interceptor.withTestTitle(testInfo.title);
    interceptor.withScenarioKey('<SCENARIO-KEY>');

    if (isRecording) {
      await interceptor.enable({ mode: InterceptMode.record });
    } else {
      await interceptor.enable({ mode: InterceptMode.mock });
    }

    // Required for context.request: context.route() does NOT intercept APIRequestContext.
    // Copy interceptor headers onto the context so context.request calls carry them to the proxy.
    await context.setExtraHTTPHeaders((interceptor as any).headers);
  });

  test('fetches data via context.request', async ({ context }) => {
    const resp = await context.request.get('/api/users');
    // Request reaches proxy with Stoobly headers → recorded or mocked correctly
  });
});
```

**Key points:**
- `withContext(context)` sets up `context.route()` for browser page requests only
- `context.setExtraHTTPHeaders((interceptor as any).headers)` must be called **after** `enable()` so the headers reflect the current session ID, scenario key, and proxy mode
- `headers` is a `protected` field on the interceptor class; `(interceptor as any).headers` accesses it at runtime

***

#### Q: How do I record requests in Playwright tests?

**A:** Set intercept **`mode`** to record—either in the interceptor constructor (`mode: InterceptMode.record`) or when calling `enable({ mode: InterceptMode.record })`.

**Example:**

```javascript
import { test as base } from '@playwright/test';
import Stoobly from 'stoobly';
import { InterceptMode, RecordPolicy, RecordOrder, RecordStrategy } from 'stoobly/constants';

// Fixture-based recording setup
const test = base.extend({
  stooblyInterceptor: [
    async ({ context }, use, testInfo) => {
      const stoobly = new Stoobly();
      const interceptor = stoobly.playwrightInterceptor({
        urls: [new RegExp('https://api.example.com/.*')],
        scenarioName: testInfo.titlePath.join(' > '),
        mode: InterceptMode.record,
        record: {
          policy: RecordPolicy.All,
          order: RecordOrder.Overwrite,
          strategy: RecordStrategy.Full,
        }
      });
      await interceptor.withContext(context).enable();
      interceptor.withTestTitle(testInfo.title);
      await use(undefined);
    },
    { auto: true },
  ],
});

test.describe('Record Requests', () => {
  test('records API calls', async ({ page }) => {
    await page.goto('https://example.com');
    // All API requests matching urls will be recorded
  });
});
```
