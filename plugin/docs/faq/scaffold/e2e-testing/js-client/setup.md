# JavaScript Client Library - Installation and Setup

### Installation and Setup

#### Q: How do I install the Stoobly JavaScript library?

**A:** Install the library as a dev dependency using npm or yarn.

**Example:**

```bash
# Install with npm
npm install stoobly --save-dev

# Install with yarn
yarn add -D stoobly
```

#### Q: What are the requirements for using the Stoobly JavaScript library?

**A:** The library requires Node.js 18 or higher and works with Playwright and Cypress test frameworks.

**Example:**

```bash
# Check Node version
node --version
# Should output v18.0.0 or higher

# Install with Playwright
npm install stoobly @playwright/test --save-dev

# Install with Cypress
npm install stoobly cypress --save-dev
```

#### Q: How do I import the Stoobly library?

**A:** Import the library using ES modules or CommonJS syntax.

**Example:**

```javascript
// ES modules (recommended)
import Stoobly from 'stoobly';
import { RecordPolicy, RecordOrder, RecordStrategy } from 'stoobly/constants';

// CommonJS
const Stoobly = require('stoobly');
const { RecordPolicy, RecordOrder, RecordStrategy } = require('stoobly/constants');
```

#### Q: How do I create a `Stoobly` instance, and what if my agent isn't on the default port?

**A:** Pass the agent's URL to the `Stoobly` constructor. It defaults to `http://localhost:4200` if omitted.

**Example:**

```javascript
const stoobly = new Stoobly();  // Uses default localhost:4200
// OR
const stoobly = new Stoobly('http://custom-agent:4200');
```

#### Q: Are TypeScript types available?

**A:** Yes, import types from `stoobly/types`:

```typescript
import {
  InterceptorSettings,
  RecordSettings,
  InterceptorUrl,
  Page,      // Playwright page type
  Route,     // Playwright route type
  Request    // Playwright request type
} from 'stoobly/types';
```
