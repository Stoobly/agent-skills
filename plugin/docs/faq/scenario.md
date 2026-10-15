# Scenario

## Stoobly Scenario CLI - Questions & Answers

The scenario CLI enables you to manage collections of related HTTP requests as scenarios. Scenarios help organize test flows, user journeys, and API workflows that involve multiple requests executed in sequence.

***

### Understanding Scenarios

#### Q: What is a scenario in Stoobly?

**A:** A scenario is a collection of related HTTP requests that represent a user flow, test case, or API workflow. Scenarios execute requests in sequence and can validate responses.

**Example:**

```bash
# Create a scenario for user login flow
stoobly-agent scenario create "User Login Flow"

# The scenario will contain multiple requests:
# 1. GET /api/csrf-token
# 2. POST /api/login
# 3. GET /api/user/profile
```

#### Q: Why should I use scenarios?

**A:** Scenarios help organize related requests, enable end-to-end testing, support workflow validation, and make it easy to share test cases with your team via version control.

**Example:**

```bash
# Instead of managing 10 individual requests
stoobly-agent request replay "<REQUEST-KEY-1>"
stoobly-agent request replay "<REQUEST-KEY-2>"
# ... (8 more)

# Use a scenario
stoobly-agent scenario replay user-checkout-flow
```

***

### Creating Scenarios

#### Q: How do I create a new scenario?

**A:** Use `scenario create` with a descriptive name for your scenario.

**Example:**

```bash
stoobly-agent scenario create "User Login Flow"
```

#### Q: How do I create a scenario with a description?

**A:** Use the `--description` option to add context to your scenario.

**Example:**

```bash
stoobly-agent scenario create "API Integration Test" --description "Tests all critical API endpoints for the mobile app"
```

#### Q: How do I create a scenario in a specific project?

**A:** Use the `--project-key` option to specify the project (remote features).

**Example:**

```bash
stoobly-agent scenario create "Payment Flow" --project-key "<PROJECT-KEY>"
```

#### Q: How do I format the output when creating a scenario?

**A:** Use the `--format` option to control how the created scenario is displayed.

**Example:**

```bash
# JSON format:
stoobly-agent scenario create "Checkout Flow" --format json

# Table format
stoobly-agent scenario create "User Registration" --format table

# CSV format
stoobly-agent scenario create "Admin Panel" --format csv
```

#### Q: How do I add requests to a scenario?

**A:** Record or replay requests with the `--scenario-key` option to add them to a scenario.

**Example:**

```bash
# First, create the scenario and note the key
stoobly-agent scenario create "User Login Flow"
# Output: Created scenario with key: <SCENARIO-KEY>

# Then record requests to that scenario
stoobly-agent record -X POST -d '{"username":"user","password":"pass"}' https://api.example.com/login --scenario-key "<SCENARIO-KEY>"

# Or replay and record to the scenario
stoobly-agent request replay "<REQUEST-KEY>" --record --scenario-key "<SCENARIO-KEY>"
```

***

### Listing Scenarios

#### Q: How do I view all scenarios?

**A:** Use `scenario list` to display all scenarios with pagination.

**Example:**

```bash
stoobly-agent scenario list
```

#### Q: How do I paginate through scenarios?

**A:** Use the `--page` and `--size` options to control pagination.

**Example:**

```bash
# Show first 10 scenarios (default)
stoobly-agent scenario list --page 0 --size 10

# Show next 20 scenarios
stoobly-agent scenario list --page 1 --size 20

# Show 50 scenarios per page
stoobly-agent scenario list --page 0 --size 50
```

#### Q: How do I search for specific scenarios?

**A:** Use the `--search` option to filter scenarios by name or description.

**Example:**

```bash
# Search by name
stoobly-agent scenario list --search "login"

# Search by description
stoobly-agent scenario list --search "payment"
```

#### Q: How do I sort scenarios?

**A:** Use `--sort-by` and `--sort-order` options to control sorting.

**Example:**

```bash
# Sort by creation date (newest first, default)
stoobly-agent scenario list --sort-by created_at --sort-order desc

# Sort by creation date (oldest first)
stoobly-agent scenario list --sort-by created_at --sort-order asc

# Sort by name alphabetically
stoobly-agent scenario list --sort-by name --sort-order asc
```

#### Q: How do I format the scenario list output?

**A:** Use the `--format` option to change output format.

**Example:**

```bash
# Table format (default)
stoobly-agent scenario list --format table

# JSON format
stoobly-agent scenario list --format json

# CSV format
stoobly-agent scenario list --format csv
```

#### Q: How do I select specific columns to display?

**A:** Use the `--select` option to choose which columns to show.

**Example:**

```bash
# Show only specific columns
stoobly-agent scenario list --select id --select name --select created_at

# Show key and name only
stoobly-agent scenario list --select key --select name --without-headers
```

***

### Viewing Scenario Details

#### Q: How do I view details about a specific scenario?

**A:** Use `scenario show` with the scenario key.

**Example:**

```bash
stoobly-agent scenario show "<SCENARIO-KEY>"
```

#### Q: How do I format scenario details output?

**A:** Use the `--format` option to control the display format.

**Example:**

```bash
# JSON format for scripting
stoobly-agent scenario show "<SCENARIO-KEY>" --format json

# Table format for readability
stoobly-agent scenario show "<SCENARIO-KEY>" --format table
```

#### Q: How do I get the request count for a scenario?

**A:** View scenario details or use the show command with specific column selection.

**Example:**

```bash
# Show all details including request count
stoobly-agent scenario show "<SCENARIO-KEY>"

# Select specific fields
stoobly-agent scenario show "<SCENARIO-KEY>" --select name --select request_count
```

***

### Replaying Scenarios

#### Q: How do I replay all requests in a scenario?

**A:** Use `scenario replay` with the scenario key to execute all requests in sequence.

**Example:**

```bash
stoobly-agent scenario replay "<SCENARIO-KEY>"
```

#### Q: How do I replay a scenario to a different host?

**A:** Use the `--host` option to override the request host for all requests.

**Example:**

```bash
# Replay to localhost
stoobly-agent scenario replay "<SCENARIO-KEY>" --host localhost:8080

# Replay to staging environment
stoobly-agent scenario replay "<SCENARIO-KEY>" --host staging.example.com
```

#### Q: How do I replay a scenario with a different scheme?

**A:** Use the `--scheme` option to change the protocol for all requests.

**Example:**

```bash
# Force HTTP
stoobly-agent scenario replay "<SCENARIO-KEY>" --scheme http

# Force HTTPS
stoobly-agent scenario replay "<SCENARIO-KEY>" --scheme https
```

#### Q: How do I update a scenario and by adding new responses?

**A:** Use the `--record` flag to capture the replayed responses.

**Example:**

```bash
stoobly-agent scenario replay "<SCENARIO-KEY>" --record
```

#### Q: How do I update a scenario and overwrite existing responses?

**A:** Use the `--overwrite` flag to replace stored responses (local mode only).

**Example:**

```bash
stoobly-agent scenario replay "<SCENARIO-KEY>" --overwrite
```

#### Q: How do I replay a scenario and save to history?

**A:** Use the `--save` flag to persist the replay session (local mode).

**Example:**

```bash
stoobly-agent scenario replay "<SCENARIO-KEY>" --save
```

#### Q: How do I replay a scenario with custom lifecycle hooks?

**A:** Use the `--lifecycle-hooks-path` option to apply custom processing.

**Example:**

```bash
stoobly-agent scenario replay "<SCENARIO-KEY>" --lifecycle-hooks-path ./hooks.py
```

#### Q: How do I increase logging verbosity when replaying?

**A:** Use the `--log-level` option to see more details.

**Example:**

```bash
# Debug level (most verbose)
stoobly-agent scenario replay "<SCENARIO-KEY>" --log-level debug

# Info level
stoobly-agent scenario replay "<SCENARIO-KEY>" --log-level info
```

#### Q: How do I format the replay response output?

**A:** Use the `--format` option to control response display.

**Example:**

```bash
# JSON format with full details
stoobly-agent scenario replay "<SCENARIO-KEY>" --format json
```

***

### Working with Aliases

#### Q: How do I replay a scenario with assigned alias values?

**A:** Use the `--assign` option to set alias values before replay.

**Example:**

```bash
# Assign single alias
stoobly-agent scenario replay "<SCENARIO-KEY>" --assign userId=12345

# Assign multiple aliases
stoobly-agent scenario replay "<SCENARIO-KEY>" --assign userId=12345 --assign token=abcde12345
```

#### Q: How do I validate alias values during scenario execution?

**A:** Use the `--validate` option to specify validation rules for aliases.

**Example:**

```bash
# Validate userId is an integer
stoobly-agent scenario replay "<SCENARIO-KEY>" --validate "userId=?int"

# Validate multiple aliases
stoobly-agent scenario replay "<SCENARIO-KEY>" --validate "userId=?int" --validate "email=?string"
```

#### Q: How do I control alias resolution strategy?

**A:** Use the `--alias-resolve-strategy` option to specify how aliases are resolved.

**Example:**

```bash
# No alias resolution (default)
stoobly-agent scenario replay "<SCENARIO-KEY>" --alias-resolve-strategy none

# First-in-first-out resolution
stoobly-agent scenario replay "<SCENARIO-KEY>" --alias-resolve-strategy fifo

# Last-in-first-out resolution
stoobly-agent scenario replay "<SCENARIO-KEY>" --alias-resolve-strategy lifo
```

#### Q: How do I repeat scenario execution for each alias value?

**A:** Use the `--group-by` option to iterate over alias values.

**Example:**

```bash
# Execute scenario once for each userId value
stoobly-agent scenario replay "<SCENARIO-KEY>" --group-by userId
```

#### Q: How do I use an existing trace for scenario execution?

**A:** Use the `--trace-id` option to leverage a previous trace.

**Example:**

```bash
stoobly-agent scenario replay "<SCENARIO-KEY>" --trace-id trace-<TRACE-ID>
```

***

### Snapshots and Version Control

#### Q: How do I create a snapshot of a scenario?

**A:** Use `scenario snapshot create` to create committable files for the scenario (local mode only).

**Example:**

```bash
stoobly-agent scenario snapshot create "<SCENARIO-KEY>"
```

#### Q: How do I delete a scenario snapshot?

**A:** Use the `--action delete` option with snapshot create command.

**Example:**

```bash
stoobly-agent scenario snapshot create "<SCENARIO-KEY>" --action delete
```

#### Q: How do I snapshot a scenario with decoded response bodies?

**A:** Use the `--decode` flag to decode response bodies in the snapshot.

**Example:**

```bash
stoobly-agent scenario snapshot create "<SCENARIO-KEY>" --decode
```

#### Q: How do I reset a scenario to its snapshot state?

**A:** Use `scenario snapshot reset` to restore a scenario from its snapshot. This reverts the scenario to the exact state from when the snapshot was created, discarding any changes.

{% hint style="warning" %}
This command **reverts** the scenario to the last committed snapshot. It does not merge new requests. If you need to incorporate new traffic, record or replay with `--scenario-key` instead of resetting.
{% endhint %}

**Example:**

```bash
stoobly-agent scenario snapshot reset "<SCENARIO-KEY>"
```

#### Q: What happens when I reset a scenario?

**A:** Reset will:
1. Restore the scenario metadata (name, description) from the snapshot
2. Restore all requests in the scenario to their snapshot states
3. Discard any modifications made since the snapshot was created
4. Return the scenario to an exact known state

**Example:**

```bash
# Before: Scenario has been modified with new requests
# After: Scenario matches the snapshot exactly

stoobly-agent scenario snapshot reset "<SCENARIO-KEY>"
```

#### Q: How do I check what will change before resetting?

**A:** Use `scenario snapshot diff` to preview all changes before applying reset.

**Example:**

```bash
# See what would change
stoobly-agent scenario snapshot diff "<SCENARIO-KEY>"

# See full raw HTTP diffs for each request
stoobly-agent scenario snapshot diff "<SCENARIO-KEY>" --full

# Review the output carefully

# If you want to proceed:
stoobly-agent scenario snapshot reset "<SCENARIO-KEY>"
```

#### Q: How do I force delete when resetting a scenario?

**A:** Use the `--force` flag to hard delete the scenario before resetting.

**Example:**

```bash
stoobly-agent scenario snapshot reset "<SCENARIO-KEY>" --force
```

#### Q: When should I reset a scenario vs. add new requests?

**A:**

- **Reset**: Use when you made unwanted changes and want to discard them
- **Add requests**: Use when you want to keep the snapshot and add new requests

**Example:**

```bash
# Scenario is in snapshot state
# Option 1: Add a new request (keeps snapshot requests)
stoobly-agent record https://api.example.com/new --scenario-key "<SCENARIO-KEY>"

# Option 2: Reset if you change your mind
stoobly-agent scenario snapshot reset "<SCENARIO-KEY>"
```

#### Q: Can I undo a reset?

**A:** No, reset is permanent. The changes are lost. To prevent accidental data loss:
1. Always check diff before resetting
2. Commit snapshots to Git before making changes
3. Use version control to restore previous snapshot states

**Example:**

```bash
# Best practice: Always check before reset
stoobly-agent scenario snapshot diff "<SCENARIO-KEY>"

# Commit snapshots to Git
git add .stoobly/snapshots/
git commit -m "Checkpoint before major changes"

# Now safe to experiment
# If you reset by mistake, can restore from Git
git restore .stoobly/snapshots/
stoobly-agent snapshot apply
```

#### Q: How do I share scenarios with my team via git?

**A:** Create snapshots and commit them to version control.

**Example:**

```bash
# Create snapshot
stoobly-agent scenario snapshot create "<SCENARIO-KEY>" --decode

# Commit to git
git add .stoobly/snapshots/scenarios/
git commit -m "Add user login scenario snapshot"
git push

# Team member pulls and uses
git pull
stoobly-agent snapshot apply # Apply snapshots to local database
```

#### Q: How do I see what changed in a scenario?

**A:** Use `scenario snapshot diff` to view differences between your current scenario and its last snapshot.

**Example:**

```bash
# Show what changed in the scenario
stoobly-agent scenario snapshot diff "<SCENARIO-KEY>"
```

#### Q: What does the scenario diff show?

**A:** The scenario diff displays:
- Changes to scenario metadata (name, description)
- Changes to requests in the scenario (added, removed, or modified)
- Response body changes for each request
- Headers and request properties that differ

**Example:**

```bash
# View detailed scenario changes
stoobly-agent scenario snapshot diff "<SCENARIO-KEY>"

# Show full raw HTTP request diffs for all scenario requests
stoobly-agent scenario snapshot diff "<SCENARIO-KEY>" --full
```

#### Q: How do I see diffs for all scenarios that have changed?

**A:** Run `scenario snapshot diff` without a scenario key to see all scenarios with changes.

**Example:**

```bash
# Show diffs for all scenarios
stoobly-agent scenario snapshot diff

# Show full raw diffs for all scenarios
stoobly-agent scenario snapshot diff --full
```

#### Q: When should I use diff?

**A:** Use diff to review changes before deciding whether to reset a scenario or commit changes to git. It helps prevent accidental loss of important modifications.

**Example:**

```bash
# Check changes before resetting
stoobly-agent scenario snapshot diff "<SCENARIO-KEY>"

# If happy with changes, commit:
stoobly-agent snapshot apply

# If want to discard changes:
stoobly-agent scenario snapshot reset "<SCENARIO-KEY>"
```

***

### Deleting Scenarios

#### Q: How do I delete a scenario?

**A:** Use `scenario delete` with the scenario key.

**Example:**

```bash
stoobly-agent scenario delete "<SCENARIO-KEY>"
```

#### Q: What happens to requests when I delete a scenario?

**A:** The scenario is deleted but the individual requests remain in storage unless explicitly deleted.

**Example:**

```bash
# Delete scenario (requests remain)
stoobly-agent scenario delete "<SCENARIO-KEY>"

# To also delete requests, delete them individually
stoobly-agent request delete "<REQUEST-KEY-1>"
stoobly-agent request delete "<REQUEST-KEY-2>"
```

***

### Scenario Workflows

#### Q: How do I create a complete user flow scenario?

**A:** Create the scenario, then record or add requests in the correct sequence.

**Example:**

```bash
# 1. Create scenario
stoobly-agent scenario create "Complete Checkout Flow"
# Output: Created scenario with key: <SCENARIO-KEY>

# 2. Record requests in sequence
stoobly-agent record https://api.example.com/cart --scenario-key "<SCENARIO-KEY>"
stoobly-agent record -X POST -d '{"items":[]}' https://api.example.com/cart/add --scenario-key "<SCENARIO-KEY>"
stoobly-agent record -X POST https://api.example.com/checkout --scenario-key "<SCENARIO-KEY>"
stoobly-agent record -X POST https://api.example.com/payment --scenario-key "<SCENARIO-KEY>"

# 3. Replay the complete flow
stoobly-agent scenario replay "<SCENARIO-KEY>"
```

#### Q: How do I update a scenario with new requests (add to the scenario)?

**A:** To update a scenario by **adding** requests (without overwriting existing ones), record or replay new requests while specifying the scenario key with the `--scenario-key` option. This appends new requests to the scenario.

**Option 1: Record a new request and add it to the scenario**

```bash
stoobly-agent record https://api.example.com/new-endpoint --scenario-key "<SCENARIO-KEY>"
```

**Option 2: Replay an existing request and add it to the scenario**

```bash
stoobly-agent request replay "<REQUEST-KEY>" --record --scenario-key "<SCENARIO-KEY>"
```

**Option 3: Add multiple new requests to a scenario**

```bash
# Record multiple endpoints and add them all to the same scenario
stoobly-agent record https://api.example.com/endpoint1 --scenario-key "<SCENARIO-KEY>"
stoobly-agent record https://api.example.com/endpoint2 --scenario-key "<SCENARIO-KEY>"
stoobly-agent record https://api.example.com/endpoint3 --scenario-key "<SCENARIO-KEY>"
```

#### Q: How do I update all requests in a scenario by overwriting responses when test cases exist?

**A:** When you have E2E test cases (e.g., Playwright or Cypress), use the Stoobly JavaScript client library to **re-record and overwrite** the scenario's stored responses by running your tests with recording enabled.

**Steps:**

1. Install the Stoobly JavaScript client library:

```bash
npm install stoobly --save-dev
```

2. Set up the interceptor in your tests following the [JavaScript Client Library](https://docs.stoobly.com/faq/scaffold/e2e-testing/js-client) setup instructions.

3. Run your tests with recording enabled by setting the `STOOBLY_RECORD` environment variable:

```bash
STOOBLY_RECORD=true npm test
```

This will re-record all API requests and responses captured during the test run, updating the scenario with the latest data.

More details: [JavaScript Client Library](https://docs.stoobly.com/faq/scaffold/e2e-testing/js-client), [How to Update Scenarios](https://docs.stoobly.com/guides/how-to-update-requests/how-to-update-scenarios/)

#### Q: How do I update all requests in a scenario by overwriting responses when test cases do not exist?

**A:** When you don't have test cases, use lifecycle hooks to add valid credentials and replay the scenario with the `--overwrite` flag. This **overwrites all stored responses** in the scenario with new ones from the live API.

**Steps:**

1. Create a `lifecycle_hooks.py` script to add valid credentials (e.g., an `Authorization` header):

```python
from stoobly_agent.app.proxy.replay.context import ReplayContext

def handle_before_replay(context: ReplayContext):
    flow = context.flow
    request = flow.request
    request.headers['Authorization'] = 'Bearer <TOKEN>'
```

2. Replay the scenario with overwrite enabled:

```bash
stoobly-agent scenario replay --overwrite "<SCENARIO-KEY>" --lifecycle-hooks-path ./lifecycle_hooks.py
```

This replays all requests in the scenario against the live API and overwrites the stored responses with the new ones.

More details: [Updating with Replay](https://docs.stoobly.com/guides/how-to-update-requests/how-to-update-scenarios/updating-with-replay), [Lifecycle Hooks](https://docs.stoobly.com/core-concepts/agent/lifecycle-hooks)

#### Q: How do I test multiple scenarios in sequence?

**A:** Use a script to test scenarios one after another.

**Example:**

```bash
#!/bin/bash
# Replay multiple scenarios

scenarios=("login-flow" "checkout-flow" "admin-panel")

for scenario in "${scenarios[@]}"; do
  echo "Replaying scenario: $scenario"
  if ! stoobly-agent scenario replay $scenario; then
    echo "Failed: $scenario"
    exit 1
  fi
done

echo "All scenarios completed!"
```

***

### Advanced Scenario Operations

#### Q: How do I chain multiple scenarios?

**A:** Use a script to execute scenarios in sequence with dependency handling.

**Example:**

```bash
#!/bin/bash
# Chain scenarios with dependencies

# Setup scenario
stoobly-agent scenario replay setup-data --save
if [ $? -ne 0 ]; then
  echo "Setup failed"
  exit 1
fi

# Main scenario
stoobly-agent scenario replay main-flow --save
if [ $? -ne 0 ]; then
  echo "Main flow failed"
  exit 1
fi

# Cleanup scenario
stoobly-agent scenario replay cleanup --save

echo "All scenarios completed"
```

#### Q: How do I extract data from one scenario to use in another?

**A:** Use aliases and traces to pass data between scenarios.

**Example:**

```bash
# First scenario creates data and assigns aliases
stoobly-agent scenario replay create-user \
  --save \
  --trace-id shared-trace \
  --assign email=user@example.com

# Second scenario uses the trace and aliases
stoobly-agent scenario replay login-user \
  --trace-id shared-trace \
  --assign email=user@example.com
```

***

### Monitoring and Debugging

#### Q: How do I debug a failing scenario?

**A:** Increase log level and use verbose output.

**Example:**

```bash
# Debug with verbose logging
stoobly-agent scenario replay "<SCENARIO-KEY>" --log-level debug

# Replay with logging to see each request
stoobly-agent scenario replay "<SCENARIO-KEY>" --log-level info
```

#### Q: How do I identify which request in a scenario is failing?

**A:** Use detailed output and logging to track request execution.

**Example:**

```bash
# Replay with detailed logging
stoobly-agent scenario replay "<SCENARIO-KEY>" --log-level info

# This shows each request as it executes
```

***

### Quick Reference

#### Q: What are the most common scenario commands?

**A:** Here's a quick reference of frequently used commands:

**Example:**

```bash
# Create scenarios
stoobly-agent scenario create "My Scenario"
stoobly-agent scenario create "API Flow" --description "Tests API endpoints"

# List scenarios
stoobly-agent scenario list
stoobly-agent scenario list --search "login"
stoobly-agent scenario list --format json

# View scenario details
stoobly-agent scenario show "<SCENARIO-KEY>"

# Replay scenarios
stoobly-agent scenario replay "<SCENARIO-KEY>"
stoobly-agent scenario replay "<SCENARIO-KEY>" --host staging.example.com
stoobly-agent scenario replay "<SCENARIO-KEY>" --record

# Snapshots
stoobly-agent scenario snapshot create "<SCENARIO-KEY>"
stoobly-agent scenario snapshot create "<SCENARIO-KEY>" --decode
stoobly-agent scenario snapshot reset "<SCENARIO-KEY>"

# Delete scenarios
stoobly-agent scenario delete "<SCENARIO-KEY>"
```

***

### Best Practices

#### Q: How should I organize my scenarios?

**A:** Group related requests by user flow, feature area, or test type.

**Example:**

```bash
# By user flow
stoobly-agent scenario create "User Registration Flow"
stoobly-agent scenario create "User Login Flow"
stoobly-agent scenario create "User Profile Update Flow"

# By feature area
stoobly-agent scenario create "Shopping Cart - Add Items"
stoobly-agent scenario create "Shopping Cart - Checkout"
stoobly-agent scenario create "Shopping Cart - Order History"

# By test type
stoobly-agent scenario create "Smoke Tests"
stoobly-agent scenario create "Integration Tests"
stoobly-agent scenario create "E2E Tests"
```

#### Q: How often should I snapshot scenarios?

**A:** Snapshot after significant changes or before releases.

**Example:**

```bash
# After adding new requests
stoobly-agent scenario snapshot create "<SCENARIO-KEY>" --decode
git add .stoobly/snapshots/
git commit -m "Update scenario with new endpoints"

# Before release
./snapshot-all-scenarios.sh
git tag -a v1.2.0 -m "Release 1.2.0 with scenario snapshots"
git push --tags
```

#### Q: What's the recommended workflow for scenario management?

**A:** Create → Record → Snapshot → Replay → Update cycle.

**Example:**

```bash
# 1. Create scenario
stoobly-agent scenario create "Payment Flow"

# 2. Record requests
stoobly-agent record "<URL-1>" --scenario-key "<SCENARIO-KEY>"

# 3. Snapshot for version control
stoobly-agent scenario snapshot create "<SCENARIO-KEY>" --decode
git add .stoobly/snapshots/ && git commit -m "Add payment flow"

# 4. Replay regularly
stoobly-agent scenario replay "<SCENARIO-KEY>"

# 5. Update when needed
stoobly-agent record "<URL-2>" --scenario-key "<SCENARIO-KEY>"
stoobly-agent scenario snapshot create "<SCENARIO-KEY>" --decode
git commit -am "Update payment flow with new endpoints"
```
