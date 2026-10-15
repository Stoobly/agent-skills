# JavaScript Client Library - Troubleshooting & Examples

### Troubleshooting

#### Q: Why are my Playwright requests missing the scenario key?

**A:** Ensure you call `withPage(page)` (or `withContext(context)`) before `enable()`. The `withPage()` call does not reset headers; you can set `withScenarioKey()` or `withScenarioName()` either before or after `withPage()`, as long as it is in effect when `enable()` runs.

**Example:**

```javascript
// Recommended order in beforeEach (Playwright)
test.beforeEach(async ({ page }, testInfo) => {
  interceptor.withScenarioKey('<SCENARIO-KEY>');      // can be before or after withPage
  interceptor.withTestTitle(testInfo.title);
  await interceptor.withPage(page).enable();           // enable after withPage
});
```

***

#### Q: Why aren't my requests being intercepted?

**A:** Verify the URL patterns match your requests, the interceptor is applied in `beforeEach`, and stoobly-agent is running.

**Example:**

```javascript
// Check URL pattern
const interceptor = stoobly.playwrightInterceptor({
  urls: [new RegExp('https://api.example.com/.*')], // Make sure pattern matches
  scenarioKey: '<SCENARIO-KEY>',
});

test.beforeEach(async ({ page }, testInfo) => {
  // Ensure interceptor is applied
  await interceptor.withPage(page).enable();
  interceptor.withTestTitle(testInfo.title);
});

// Verify stoobly-agent is running
// stoobly-agent run --headless
```

#### Q: How do I debug interceptor issues?

**A:** Check the browser's network tab for Stoobly headers and verify the agent is receiving requests.

**Example:**

```javascript
// Add logging to verify interceptor is working
test('debug interceptor', async ({ page }) => {
  const sessionId = await interceptor.withPage(page).enable();
  console.log('Session ID:', sessionId);

  // Check request headers in browser DevTools
  await page.goto('https://example.com');
});
```

**HTTP Headers Injected**

The library injects these custom headers into matching HTTP requests — check the Network tab for them when debugging:

| Header                              | Description                                                                  |
| ------------------------------------ | ----------------------------------------------------------------------------- |
| `X-Stoobly-Proxy-Mode`               | Active intercept mode (mock/record/replay/test)                              |
| `X-Stoobly-Record-Order`             | Recording order (append/overwrite)                                           |
| `X-Stoobly-Record-Policy`            | Recording policy (all/found/not\_found)                                      |
| `X-Stoobly-Record-Strategy`          | Recording strategy (full/minimal)                                            |
| `X-Stoobly-Scenario-Key`             | Base64-encoded scenario identifier                                           |
| `X-Stoobly-Scenario-Name`            | Human-readable scenario name                                                 |
| `X-Stoobly-Session-Id`               | Current session identifier                                                   |
| `X-Stoobly-Test-Title`               | Current test name (auto-detected or manual)                                  |
| `X-Stoobly-Overwrite-Id`             | Present when using `RecordOrder.Overwrite`, first request per URL pattern only |
| `X-Stoobly-Match-Rules`              | Present when using per-URL `matchRules`                                      |
| `X-Stoobly-Rewrite-Rules`            | Present when using per-URL `rewriteRules`                                    |
| `X-Stoobly-Public-Directory-Path`    | Present when using per-URL `publicDirectoryPath`                             |
| `X-Stoobly-Response-Fixtures-Path`   | Present when using per-URL `responseFixturesPath`                            |

#### Q: Why do I get "page is not defined" errors in Playwright?

**A:** Ensure you call `withPage(page)` in `beforeEach` before `enable()`.

**Example:**

```javascript
// BAD: Missing withPage()
test.beforeEach(async ({ page }, testInfo) => {
  await interceptor.enable(); // ERROR: page not set
});

// GOOD: Call withPage() first
test.beforeEach(async ({ page }, testInfo) => {
  await interceptor.withPage(page).enable(); // Correct
  interceptor.withTestTitle(testInfo.title);
});
```

#### Q: How do I handle TypeScript errors?

**A:** The library includes TypeScript definitions. Ensure your tsconfig.json includes the library.

**Example:**

```json
{
  "compilerOptions": {
    "types": ["@playwright/test", "stoobly"],
    "moduleResolution": "node",
    "esModuleInterop": true
  }
}
```

***

### Complete Examples

#### Q: What's a complete Playwright example with recording and mocking?

**A:** Here's a complete example showing recording and mocking workflows with an environment variable to toggle recording mode.

**Example:**

```javascript
import { test, expect } from '@playwright/test';
import Stoobly from 'stoobly';
import { InterceptMode, RecordPolicy, RecordOrder, RecordStrategy } from 'stoobly/constants';

// Toggle recording via environment variable
// Set STOOBLY_RECORD=true to enable recording, or leave unset/empty for mocking
const isRecording = process.env.STOOBLY_RECORD === 'true';

const stoobly = new Stoobly();
const interceptor = stoobly.playwrightInterceptor({
  urls: [new RegExp('https://api.example.com/.*')],
  record: {
    policy: RecordPolicy.All,
    order: RecordOrder.Overwrite,
    strategy: RecordStrategy.Full,
  }
});

test.describe('User API', () => {
  test.beforeEach(async ({ page }, testInfo) => {
    // Configure scenario per test suite
    interceptor.withScenarioKey('<SCENARIO-KEY>');

    // Set the page and test title
    interceptor.withPage(page).withTestTitle(testInfo.title);

    // Enable recording or mocking mode based on environment variable
    // This applies to all tests in the suite
    if (isRecording) {
      await interceptor.enable({ mode: InterceptMode.record });
    } else {
      await interceptor.enable({ mode: InterceptMode.mock });
    }
  });

  test('can fetch users', async ({ page }) => {
    await page.goto('https://example.com/users');

    // If recording: requests are captured for future use
    // If mocking: requests use previously recorded responses
    const users = await page.locator('.user-list').count();
    expect(users).toBeGreaterThan(0);

    // Verify the user list is displayed
    await expect(page.locator('.user-list')).toBeVisible();
  });

  test('can create user', async ({ page }) => {
    await page.goto('https://example.com/users/new');

    // Fill in the form
    await page.fill('input[name="name"]', 'Test User');
    await page.fill('input[name="email"]', 'test@example.com');
    await page.click('button[type="submit"]');

    // If recording: the POST request is captured
    // If mocking: uses previously recorded response
    await expect(page.locator('.success')).toBeVisible();
    await expect(page.locator('.success')).toContainText('User created successfully');
  });

  test('can update user', async ({ page }) => {
    await page.goto('https://example.com/users/1');

    // Edit user details
    await page.click('button:has-text("Edit")');
    await page.fill('input[name="name"]', 'Updated User');
    await page.click('button:has-text("Save")');

    // Verify update was successful
    await expect(page.locator('.success')).toBeVisible();
    await expect(page.locator('h1')).toContainText('Updated User');
  });
});
```

**Usage:**

```bash
# Run tests in mock mode (default - uses recorded responses)
npm test

# Run tests in record mode (captures new requests)
STOOBLY_RECORD=true npm test

# Or set in your .env file
echo "STOOBLY_RECORD=true" >> .env
```

**Workflow:**

1. **First run (recording):** Set `STOOBLY_RECORD=true` to capture all API requests and responses
2. **Subsequent runs (mocking):** Leave `STOOBLY_RECORD` unset to use recorded responses for fast, reliable tests
3. **Update recordings:** Set `STOOBLY_RECORD=true` again when APIs change to refresh the recorded data

#### Q: What's a complete Cypress example with recording and mocking?

**A:** Here's a complete example showing recording and mocking workflows with an environment variable to toggle recording mode.

**Example:**

```javascript
import Stoobly from 'stoobly';
import { InterceptMode, RecordPolicy, RecordOrder, RecordStrategy } from 'stoobly/constants';

// Toggle recording via environment variable
// Set STOOBLY_RECORD=true to enable recording, or leave unset/empty for mocking
const isRecording = Cypress.env('STOOBLY_RECORD') === true || process.env.STOOBLY_RECORD === 'true';

const stoobly = new Stoobly();
const interceptor = stoobly.cypressInterceptor({
  urls: [new RegExp('https://api.example.com/.*')],
  record: {
    policy: RecordPolicy.All,
    order: RecordOrder.Overwrite,
    strategy: RecordStrategy.Full,
  }
});

describe('User API', () => {
  beforeEach(() => {
    // Configure scenario per test suite
    interceptor.withScenarioKey('<SCENARIO-KEY>');

    // Enable recording or mocking mode based on environment variable
    // This applies to all tests in the suite
    if (isRecording) {
      interceptor.enable({ mode: InterceptMode.record });
    } else {
      interceptor.enable({ mode: InterceptMode.mock });
    }
  });

  it('can fetch users', () => {
    cy.visit('https://example.com/users');

    // If recording: requests are captured for future use
    // If mocking: requests use previously recorded responses
    cy.get('.user-list .user').should('have.length.greaterThan', 0);

    // Verify the user list is displayed
    cy.get('.user-list').should('be.visible');
  });

  it('can create user', () => {
    cy.visit('https://example.com/users/new');

    // Fill in the form
    cy.get('input[name="name"]').type('Test User');
    cy.get('input[name="email"]').type('test@example.com');
    cy.get('button[type="submit"]').click();

    // If recording: the POST request is captured
    // If mocking: uses previously recorded response
    cy.get('.success').should('be.visible');
    cy.get('.success').should('contain', 'User created successfully');
  });

  it('can update user', () => {
    cy.visit('https://example.com/users/1');

    // Edit user details
    cy.get('button:contains("Edit")').click();
    cy.get('input[name="name"]').clear().type('Updated User');
    cy.get('button:contains("Save")').click();

    // Verify update was successful
    cy.get('.success').should('be.visible');
    cy.get('h1').should('contain', 'Updated User');
  });
});
```

**Usage:**

```bash
# Run tests in mock mode (default - uses recorded responses)
npm test

# Run tests in record mode (captures new requests)
STOOBLY_RECORD=true npm test

# Or set in Cypress config (cypress.config.js)
export default defineConfig({
  env: {
    STOOBLY_RECORD: true
  }
});

# Or set in your .env file
echo "STOOBLY_RECORD=true" >> .env
```

**Workflow:**

1. **First run (recording):** Set `STOOBLY_RECORD=true` to capture all API requests and responses
2. **Subsequent runs (mocking):** Leave `STOOBLY_RECORD` unset to use recorded responses for fast, reliable tests
3. **Update recordings:** Set `STOOBLY_RECORD=true` again when APIs change to refresh the recorded data

***

### Quick Reference

#### Q: What are the key methods for the interceptor?

**A:** Here's a quick reference of the most common interceptor methods.

**Example:**

```javascript
import { InterceptMode, RecordPolicy, RecordOrder, RecordStrategy } from 'stoobly/constants';

const interceptor = stoobly.playwrightInterceptor(options);

// Enable interception
await interceptor.withPage(page).enable();
await interceptor.enable({ mode: InterceptMode.record }); // Recording mode

// Configure scenario
interceptor.withScenarioKey('key');
interceptor.withScenarioName('name');
interceptor.withSessionId('session-id');
interceptor.withTestTitle('test title');

// Configure recording
interceptor.withRecordPolicy(RecordPolicy.All);
interceptor.withRecordOrder(RecordOrder.Overwrite);
interceptor.withRecordStrategy(RecordStrategy.Full);

// Chainable intercept mode shortcuts
interceptor.withInterceptMode(InterceptMode.mock);
interceptor.withInterceptModeMock();
interceptor.withInterceptModeRecord();
interceptor.withInterceptModeReplay();

// Control interception
await interceptor.disable(); // Remove all interception
await interceptor.enable({ mode: InterceptMode.mock }); // Stop recording, keep mocking

// Change URLs
await interceptor.enable({ urls: [new RegExp('...')] });

// Deprecated aliases (still supported): apply() → enable(), clear() → disable()
```

#### Q: What constants are available?

**A:** Stoobly provides enums for record policies, orders, and strategies.

**Example:**

```javascript
import {
  RecordPolicy,
  RecordOrder,
  RecordStrategy,
  InterceptMode
} from 'stoobly/constants';

// Record Policies
RecordPolicy.All       // Record all requests
RecordPolicy.Found     // Record only if exists
RecordPolicy.NotFound  // Record only if new

// Record Orders
RecordOrder.Overwrite  // Replace existing
RecordOrder.Append     // Always create new

// Record Strategies
RecordStrategy.Full    // Complete data
RecordStrategy.Minimal // Essential data only

// Intercept Modes
InterceptMode.mock     // Mocking mode
InterceptMode.record   // Recording mode
InterceptMode.replay   // Replay mode
InterceptMode.test     // Testing mode
```
