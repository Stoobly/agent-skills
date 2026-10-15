# JavaScript Client Library - Cypress Integration

### Cypress Integration

#### Q: How do I integrate Stoobly with Cypress tests?

**A:** Create a Stoobly instance and Cypress interceptor, then call `enable()` in your `beforeEach` hook.

**Example:**

```javascript
import Stoobly from 'stoobly';
import { RecordPolicy, RecordOrder, RecordStrategy } from 'stoobly/constants';

const stoobly = new Stoobly();
const interceptor = stoobly.cypressInterceptor({
  urls: [new RegExp('https://api.example.com/.*')],
});

describe('My Tests', () => {
  // Use function() form to access Mocha context
  beforeEach(function () {
    // Derive hierarchical scenario name from the test path
    const scenarioName = this.test.titlePath().join(' > ');
    interceptor.withScenarioName(scenarioName);

    // Enable interceptor in beforeEach
    interceptor.enable();
  });

  it('can fetch data', () => {
    cy.visit('https://example.com');
    // Your test code here
  });
});
```

#### Q: Why must I call `enable()` in `beforeEach` for Cypress?

**A:** Cypress automatically clears all intercepts before every test, so you must reapply the Stoobly interceptor in `beforeEach`.

**Example:**

```javascript
describe('API Tests', () => {
  beforeEach(function () {
    // Required: Reapply interceptor for each test
    // Cypress clears cy.intercept between tests
    const scenarioName = this.test.titlePath().join(' > ');
    interceptor.withScenarioName(scenarioName);
    interceptor.enable();
  });

  it('test 1', () => {
    // Interceptor is active
  });

  it('test 2', () => {
    // Interceptor must be reapplied (done in beforeEach)
  });
});
```

#### Q: How do I record requests in Cypress tests?

**A:** Set **`mode: InterceptMode.record`** in the interceptor settings (constructor options) and call `enable()` as usual, **or** call `enable({ mode: InterceptMode.record })` in `beforeEach`.

**Example:**

```javascript
import Stoobly from 'stoobly';
import { InterceptMode, RecordPolicy, RecordOrder, RecordStrategy } from 'stoobly/constants';

const stoobly = new Stoobly();
const interceptor = stoobly.cypressInterceptor({
  urls: ['https://api.example.com/users'],
  mode: InterceptMode.record,
  record: {
    policy: RecordPolicy.All,
    order: RecordOrder.Overwrite,
    strategy: RecordStrategy.Full,
  }
});

describe('Record Requests', () => {
  beforeEach(function () {
    const scenarioName = this.test.titlePath().join(' > ');
    interceptor.withScenarioName(scenarioName);
    interceptor.enable();
  });

  it('records API calls', () => {
    cy.visit('https://example.com');
    // All API requests matching urls will be recorded
  });
});
```

#### Q: What's the warning about synchronous requests in Cypress?

**A:** Synchronous XMLHttpRequest calls will cause Cypress to hang. Avoid using synchronous requests when Stoobly interceptor is active.

**Example:**

```javascript
// BAD: Synchronous request (will hang Cypress)
const xhr = new XMLHttpRequest();
xhr.open('GET', 'https://api.example.com/data', false); // false = synchronous
xhr.send();

// GOOD: Asynchronous request (works correctly)
const xhr = new XMLHttpRequest();
xhr.open('GET', 'https://api.example.com/data', true); // true = asynchronous
xhr.send();

// BETTER: Use fetch or Cypress commands
cy.request('GET', 'https://api.example.com/data');
```
