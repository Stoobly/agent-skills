# Intercept

## Stoobly Intercept CLI - Questions & Answers

The intercept CLI allows you to dynamically configure the stoobly-agent **after** it's already running with `stoobly-agent run`. This enables you to change modes, policies, and strategies without restarting the agent.

***

### Getting Started with Intercept

#### Q: What is the intercept feature?

**A:** Intercept is a feature that allows the stoobly-agent proxy to capture, modify, and control HTTP requests in real-time. You can enable/disable it and configure its behavior while the agent is running.

**Example:**

```bash
# Start the agent
stoobly-agent run

# In another terminal, enable intercept
stoobly-agent intercept enable
```

#### Q: How do I check the current intercept configuration?

**A:** Use the `intercept show` command to display the current mode, policy, strategy, and status.

**Example:**

```bash
stoobly-agent intercept show
# Output: Mock with policy: 'found', strategy: 'None', enabled
```

***

### Enabling and Disabling Intercept

#### Q: How do I enable intercept mode?

**A:** Use `stoobly-agent intercept enable` to activate request interception on a running agent.

**Example:**

```bash
# Start agent first
stoobly-agent run

# In another terminal, enable intercept
stoobly-agent intercept enable
# Output: Intercept enabled!
```

#### Q: How do I disable intercept mode?

**A:** Use `stoobly-agent intercept disable` to deactivate request interception without stopping the agent.

**Example:**

```bash
stoobly-agent intercept disable
# Output: Intercept disabled!
```

#### Q: Can I enable intercept when starting the agent?

**A:** Yes, use the `--intercept` flag when running the agent to enable it at startup.

**Example:**

```bash
stoobly-agent run --intercept --intercept-mode mock
```

***

### Configuring Intercept Modes

#### Q: What intercept modes are available?

**A:** There are four intercept modes: `mock`, `record`, `replay`, and `test` (test requires remote features enabled).

**Example:**

```bash
# View available modes
stoobly-agent intercept set --help

# Set to mock mode
stoobly-agent intercept set --mode mock

# Set to record mode
stoobly-agent intercept set --mode record

# Set to replay mode
stoobly-agent intercept set --mode replay

# Set to test mode (if remote features enabled)
stoobly-agent intercept set --mode test
```

#### Q: How do I switch from record mode to mock mode?

**A:** Use the `intercept configure --mode` command to change modes while the agent is running.

**Example:**

```bash
# Currently in record mode, switch to mock
stoobly-agent intercept set --mode mock
# Output: Updating intercept mode to mock
```

#### Q: What happens when I change the intercept mode?

**A:** When you change modes, intercept is automatically disabled, the mode is updated, and you need to re-enable intercept for the new mode to take effect.

**Example:**

```bash
# Change mode (intercept gets disabled)
stoobly-agent intercept set --mode record

# Re-enable intercept
stoobly-agent intercept enable
```

***

### Mock Mode Configuration

#### Q: What is mock mode?

**A:** Mock mode serves previously recorded responses instead of making real HTTP requests, useful for testing without external dependencies.

**Example:**

```bash
stoobly-agent intercept set --mode mock
stoobly-agent intercept enable
```

#### Q: What policies are available for mock mode?

**A:** Mock mode supports two policies: `all` (mock all requests) and `found` (only mock requests with recorded responses).

**Example:**

```bash
# Mock all requests
stoobly-agent intercept set --mode mock --policy all

# Only mock requests that have recorded responses
stoobly-agent intercept set --mode mock --policy found
```

#### Q: How do I configure mock mode to only serve recorded responses?

**A:** Set the mock policy to `found` to only mock requests that have matching recorded responses.

**Example:**

```bash
stoobly-agent intercept set --mode mock --policy found
stoobly-agent intercept enable
```

***

### Record Mode Configuration

#### Q: What is record mode?

**A:** Record mode captures HTTP requests and responses as they pass through the proxy, storing them for later replay or mocking.

**Example:**

```bash
stoobly-agent intercept set --mode record
stoobly-agent intercept enable
```

#### Q: What policies are available for record mode?

**A:** Record mode supports four policies: `all`, `api`, `found`, and `not_found`.

**Example:**

```bash
# Record all requests
stoobly-agent intercept set --mode record --policy all

# Record only API requests
stoobly-agent intercept set --mode record --policy api

# Record only requests with existing records
stoobly-agent intercept set --mode record --policy found

# Record only new requests (not previously recorded)
stoobly-agent intercept set --mode record --policy not_found
```

#### Q: What recording strategies are available?

**A:** Record mode supports two strategies: `full` (capture complete request/response) and `minimal` (capture only essential data).

**Example:**

```bash
# Full recording (complete data)
stoobly-agent intercept set --mode record --strategy full

# Minimal recording (essential data only)
stoobly-agent intercept set --mode record --strategy minimal
```

#### Q: What recording orders are available?

**A:** Record mode supports two orders: `append` (add new records) and `overwrite` (replace existing records).

**Example:**

```bash
# Append new recordings
stoobly-agent intercept set --mode record --order append

# Overwrite existing recordings
stoobly-agent intercept set --mode record --order overwrite
```

#### Q: How do I configure record mode to only capture new requests?

**A:** Set the record policy to `not_found` to only record requests that haven't been captured before.

**Example:**

```bash
stoobly-agent intercept set --mode record --policy not_found
stoobly-agent intercept enable
```

#### Q: How do I configure record mode to overwrite existing recordings?

**A:** Set the record order to `overwrite` to replace existing recordings with new captures.

**Example:**

```bash
stoobly-agent intercept set --mode record --order overwrite
stoobly-agent intercept enable
```

***

### Replay Mode Configuration

#### Q: What is replay mode?

**A:** Replay mode re-executes previously recorded requests, useful for testing and debugging.

**Example:**

```bash
stoobly-agent intercept set --mode replay
stoobly-agent intercept enable
```

#### Q: What policies are available for replay mode?

**A:** Replay mode currently supports the `all` policy to replay all intercepted requests.

**Example:**

```bash
stoobly-agent intercept set --mode replay --policy all
stoobly-agent intercept enable
```

***

### Test Mode Configuration

#### Q: What is test mode?

**A:** Test mode validates responses against expected results, useful for automated testing and contract validation.

**Example:**

```bash
stoobly-agent intercept set --mode test
stoobly-agent intercept enable
```

#### Q: What policies are available for test mode?

**A:** Test mode supports the `found` policy to only test requests with recorded responses.

**Example:**

```bash
stoobly-agent intercept set --mode test --policy found
stoobly-agent intercept enable
```

#### Q: What test strategies are available?

**A:** Test mode supports four strategies: `contract`, `custom`, `diff`, and `fuzzy`.

**Example:**

```bash
# Contract testing (validate against schema)
stoobly-agent intercept set --mode test --strategy contract

# Custom testing (use custom validation logic)
stoobly-agent intercept set --mode test --strategy custom

# Diff testing (compare exact responses)
stoobly-agent intercept set --mode test --strategy diff

# Fuzzy testing (allow minor variations)
stoobly-agent intercept set --mode test --strategy fuzzy
```

#### Q: How do I set up contract testing?

**A:** Configure test mode with the contract strategy to validate responses against defined schemas.

**Example:**

```bash
stoobly-agent intercept set --mode test --strategy contract --policy found
stoobly-agent intercept enable
```

***

### Advanced Configuration

#### Q: Can I configure multiple options at once?

**A:** Yes, you can combine mode, policy, strategy, and order options in a single command.

**Example:**

```bash
# Configure record mode with all options
stoobly-agent intercept set --mode record --policy not_found --strategy full --order append

# Configure test mode with strategy and policy
stoobly-agent intercept set --mode test --strategy diff --policy found
```

#### Q: How do I configure intercept for a specific scenario?

**A:** First configure the mode and settings, then enable intercept. Use lifecycle hooks for scenario-specific logic.

**Example:**

```bash
# Configure for recording a specific API
stoobly-agent intercept set --mode record --policy api --strategy full
stoobly-agent intercept enable

# Run your tests/requests
# ...

# Switch to mock mode for testing
stoobly-agent intercept set --mode mock --policy found
stoobly-agent intercept enable
```

***

### Workflow Examples

#### Q: How do I set up a record-then-mock workflow?

**A:** Start by recording requests, then switch to mock mode to replay them. Optionally create a scenario before recording to organize your requests.

**Example:**

```bash
# Step 1: Start the agent
stoobly-agent run

# Step 2 (Optional): Create a scenario to organize requests
stoobly-agent scenario create "My User Flow"
# Output: Created scenario with key: <SCENARIO-KEY>

# Step 3: Configure and enable recording
stoobly-agent intercept set --mode record --policy all --strategy full
stoobly-agent intercept enable

# Step 4: Make requests (they get recorded)
# ... perform your API calls ...

# Step 5: Switch to mock mode
stoobly-agent intercept set --mode mock --policy found
stoobly-agent intercept enable

# Step 6: Run tests against mocked responses
# ... run your test suite ...
```

#### Q: How do I set up continuous recording with overwrite?

**A:** Configure record mode with overwrite order to always capture the latest responses.

**Example:**

```bash
stoobly-agent run --intercept --intercept-mode record

# In another terminal
stoobly-agent intercept set --mode record --order overwrite --policy all
```

#### Q: How do I set up automated testing with validation?

**A:** Configure test mode with your preferred validation strategy.

**Example:**

```bash
# Start agent
stoobly-agent run

# Configure test mode with diff strategy
stoobly-agent intercept set --mode test --strategy diff --policy found
stoobly-agent intercept enable

# Run your test suite - responses will be validated automatically
```

***

### Troubleshooting

#### Q: What happens if I try to set an invalid policy for a mode?

**A:** The command will fail with an error message showing valid policies for that mode.

**Example:**

```bash
# This will fail if policy is invalid for the mode
stoobly-agent intercept set --mode mock --policy not_found
# Output: Error: Valid policies for mock are ['all', 'found']
```

#### Q: Why does intercept get disabled when I change modes?

**A:** Changing modes requires reconfiguring the proxy behavior, so intercept is automatically disabled. You need to re-enable it after changing modes.

**Example:**

```bash
# Change mode (intercept disabled automatically)
stoobly-agent intercept set --mode record

# Check status
stoobly-agent intercept show
# Output: Record with policy: 'all', strategy: 'full', disabled

# Re-enable intercept
stoobly-agent intercept enable
```

#### Q: How do I verify my intercept configuration is working?

**A:** Use `intercept show` to check the current configuration and status.

**Example:**

```bash
stoobly-agent intercept show
# Output: Mock with policy: 'found', strategy: 'None', enabled
```

**Technique 2 — Inspect response headers with curl:**

Stoobly injects `x-stoobly-*` headers into responses it serves. Re-run your request with `-v` (verbose) to see them:

```bash
curl -v --proxy http://localhost:8080 https://your-api.example.com/path
```

Look for a header like `x-stoobly-mock: 1` in the response. If it's present, the response came from Stoobly — not the real service. This is the quickest way to confirm end-to-end that the proxy is intercepting and serving your request.

**Technique 3 — Edit a recorded response (most definitive):**

Open the Stoobly UI at `http://localhost:4200`, find the recorded request, edit its response body to something obviously fake, then re-run your curl. If you receive the fake value, Stoobly owns the response.

#### Q: Can I configure intercept before enabling it?

**A:** Yes, you can configure all settings first, then enable intercept to apply them.

**Example:**

```bash
# Configure first
stoobly-agent intercept set --mode record --policy all --strategy full --order append

# Then enable
stoobly-agent intercept enable
```

***

### Quick Reference

#### Q: What's a quick reference for common intercept commands?

**A:** Here are the most frequently used intercept commands:

**Example:**

```bash
# View current configuration
stoobly-agent intercept show

# Enable/disable intercept
stoobly-agent intercept enable
stoobly-agent intercept disable

# Switch modes
stoobly-agent intercept set --mode mock
stoobly-agent intercept set --mode record
stoobly-agent intercept set --mode replay
stoobly-agent intercept set --mode test

# Configure record mode
stoobly-agent intercept set --mode record --policy all --strategy full --order append

# Configure mock mode
stoobly-agent intercept set --mode mock --policy found

# Configure test mode
stoobly-agent intercept set --mode test --strategy diff --policy found

# Complete workflow
stoobly-agent intercept set --mode record --policy all
stoobly-agent intercept enable
# ... make requests ...
stoobly-agent intercept set --mode mock --policy found
stoobly-agent intercept enable
# ... run tests ...
```

***

### Integration with Other Features

#### Q: How does intercept work with scenarios?

**A:** Intercept can be configured to work with specific scenarios for organized request management.

**Example:**

```bash
# Record requests to a scenario
stoobly-agent run --intercept --intercept-mode record --scenario-key "<SCENARIO-KEY>"

# Configure intercept for the scenario
stoobly-agent intercept set --mode record --policy all
```

#### Q: Can I use intercept with lifecycle hooks?

**A:** Yes, intercept works seamlessly with lifecycle hooks for custom request/response processing.

**Example:**

```bash
# Start with lifecycle hooks
stoobly-agent run --lifecycle-hooks-path ./hooks.py

# Configure intercept
stoobly-agent intercept set --mode mock --policy found
stoobly-agent intercept enable
```

```

This comprehensive Q&A guide covers all aspects of the intercept CLI, providing users with clear examples for configuring and managing request interception dynamically while the agent is running.
```
