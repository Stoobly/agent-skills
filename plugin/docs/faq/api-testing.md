# API Testing

## Stoobly API Testing CLI - Questions & Answers

The API testing CLI enables you to validate individual requests and entire scenarios by replaying them and comparing responses against expected outcomes. These testing strategies help ensure API reliability and catch regressions.

***

### Testing Requests

#### Q: How do I test a request to validate its response?

**A:** Use `request test` with the request key to replay and validate the response.

**Example:**

```bash
stoobly-agent request test "<REQUEST-KEY>"
```

#### Q: What test strategies are available?

**A:** Four strategies: `diff` (exact match), `contract` (schema validation), `fuzzy` (flexible match), and `custom` (custom logic).

**Example:**

```bash
# Diff testing (exact comparison, default)
stoobly-agent request test "<REQUEST-KEY>" --strategy diff

# Contract testing (schema validation)
stoobly-agent request test "<REQUEST-KEY>" --strategy contract

# Fuzzy testing (allows minor variations)
stoobly-agent request test "<REQUEST-KEY>" --strategy fuzzy

# Custom testing (use lifecycle hooks)
stoobly-agent request test "<REQUEST-KEY>" --strategy custom --lifecycle-hooks-path ./test-hooks.py
```

#### Q: How do I continue testing even if a test fails?

**A:** Use the `--aggregate-failures` flag to continue execution on failure.

**Example:**

```bash
stoobly-agent request test "<REQUEST-KEY>" --aggregate-failures
```

#### Q: How do I control which test results are displayed?

**A:** Use the `--output-level` option to filter results.

**Example:**

```bash
# Show all tests (passed, failed, skipped)
stoobly-agent request test "<REQUEST-KEY>" --output-level passed

# Show only failed tests
stoobly-agent request test "<REQUEST-KEY>" --output-level failed

# Show only skipped tests
stoobly-agent request test "<REQUEST-KEY>" --output-level skipped
```

#### Q: How do I filter which properties are tested?

**A:** Use the `--filter` option to selectively test properties.

**Example:**

```bash
# Test all properties (default)
stoobly-agent request test "<REQUEST-KEY>" --filter all

# Test only alias properties
stoobly-agent request test "<REQUEST-KEY>" --filter alias

# Test only link properties
stoobly-agent request test "<REQUEST-KEY>" --filter link
```

#### Q: How do I test a request with mock dependencies?

**A:** Use `--response-fixtures-path` or `--public-dir-path` to provide mock responses for dependencies.

**Example:**

```bash
# Use response fixtures
stoobly-agent request test "<REQUEST-KEY>" --response-fixtures-path ./fixtures/responses.yml

# Use public directory for static files
stoobly-agent request test "<REQUEST-KEY>" --public-dir-path ./public

# Use both
stoobly-agent request test "<REQUEST-KEY>" \
  --response-fixtures-path ./fixtures/responses.yml \
  --public-dir-path ./public
```

#### Q: How do I test a request and save the results?

**A:** Use the `--save` flag to persist test results (remote features).

**Example:**

```bash
stoobly-agent request test "<REQUEST-KEY>" --save
```

#### Q: How do I save test results to a specific report?

**A:** Use the `--report-key` option to add results to a report (remote features).

**Example:**

```bash
stoobly-agent request test "<REQUEST-KEY>" --report-key "<REPORT-KEY>"
```

#### Q: How do I test with assigned and validated aliases?

**A:** Combine `--assign` and `--validate` options for comprehensive testing.

**Example:**

```bash
stoobly-agent request test "<REQUEST-KEY>" \
  --assign userId=12345 \
  --assign token=abcde12345 \
  --validate "userId=?int" \
  --validate "token=?string"
```

#### Q: How do I batch test multiple requests?

**A:** Use a script to test multiple requests and collect results.

**Example:**

```bash
#!/bin/bash
# Test all requests in a scenario

scenario_key="<SCENARIO-KEY>"
failed=0

for key in $(stoobly-agent request list --scenario-key $scenario_key --format json | jq -r '.[].id'); do
  echo "Testing request: $key"
  if ! stoobly-agent request test $key --strategy diff; then
    ((failed++))
  fi
done

echo "Tests completed. Failed: $failed"
exit $failed
```

#### Q: How do I use request commands in a CI/CD pipeline?

**A:** Test recorded requests as part of your automated test suite.

**Example:**

```bash
#!/bin/bash
# CI/CD test script

# List all test requests
request_keys=$(stoobly-agent request list --scenario-key "<SCENARIO-KEY>" --format json | jq -r '.[].id')

# Test each request
failed=0
for key in $request_keys; do
  if ! stoobly-agent request test $key --strategy diff --output-level failed; then
    ((failed++))
  fi
done

if [ $failed -gt 0 ]; then
  echo "Failed $failed tests"
  exit 1
fi

echo "All tests passed"
```

#### Q: How do I integrate request testing with monitoring?

**A:** Periodically replay and test requests to monitor API health.

**Example:**

```bash
#!/bin/bash
# Monitoring script

# Replay critical requests
for key in user-login api-health payment-flow; do
  if ! stoobly-agent request test $key --strategy fuzzy --log-level error; then
    # Alert team
    echo "ALERT: Request $key failed"
    # Send notification (email, Slack, PagerDuty, etc.)
  fi
done
```

***

### Testing Scenarios

#### Q: How do I test a scenario to validate responses?

**A:** Use `scenario test` with the scenario key to replay and validate all requests.

**Example:**

```bash
stoobly-agent scenario test "<SCENARIO-KEY>"
```

#### Q: What test strategies are available for scenarios?

**A:** Four strategies: `diff` (exact match), `contract` (schema validation), `fuzzy` (flexible match), and `custom` (custom logic).

**Example:**

```bash
# Diff testing (exact comparison, default)
stoobly-agent scenario test "<SCENARIO-KEY>" --strategy diff

# Contract testing (schema validation)
stoobly-agent scenario test "<SCENARIO-KEY>" --strategy contract

# Fuzzy testing (allows minor variations)
stoobly-agent scenario test "<SCENARIO-KEY>" --strategy fuzzy

# Custom testing (use lifecycle hooks)
stoobly-agent scenario test "<SCENARIO-KEY>" --strategy custom --lifecycle-hooks-path ./test-hooks.py
```

#### Q: How do I continue testing even if a request fails?

**A:** Use the `--aggregate-failures` flag to continue execution on failure.

**Example:**

```bash
stoobly-agent scenario test "<SCENARIO-KEY>" --aggregate-failures
```

#### Q: How do I control which test results are displayed?

**A:** Use the `--output-level` option to filter results.

**Example:**

```bash
# Show all tests (passed, failed, skipped, default)
stoobly-agent scenario test "<SCENARIO-KEY>" --output-level passed

# Show only failed tests
stoobly-agent scenario test "<SCENARIO-KEY>" --output-level failed

# Show only skipped tests
stoobly-agent scenario test "<SCENARIO-KEY>" --output-level skipped
```

#### Q: How do I filter which properties are tested?

**A:** Use the `--filter` option to selectively test properties.

**Example:**

```bash
# Test all properties (default)
stoobly-agent scenario test "<SCENARIO-KEY>" --filter all

# Test only alias properties
stoobly-agent scenario test "<SCENARIO-KEY>" --filter alias

# Test only link properties
stoobly-agent scenario test "<SCENARIO-KEY>" --filter link
```

#### Q: How do I test a scenario with mock dependencies?

**A:** Use `--response-fixtures-path` or `--public-dir-path` to provide mock responses.

**Example:**

```bash
# Use response fixtures
stoobly-agent scenario test "<SCENARIO-KEY>" --response-fixtures-path ./fixtures/responses.yml

# Use public directory for static files
stoobly-agent scenario test "<SCENARIO-KEY>" --public-dir-path ./public

# Use both
stoobly-agent scenario test "<SCENARIO-KEY>" \
  --response-fixtures-path ./fixtures/responses.yml \
  --public-dir-path ./public
```

#### Q: How do I save test results to a report?

**A:** Use the `--report-key` option to add results to a report (remote features).

**Example:**

```bash
stoobly-agent scenario test "<SCENARIO-KEY>" --report-key "<REPORT-KEY>"
```

#### Q: How do I save test results?

**A:** Use the `--save` flag to persist test results (remote features).

**Example:**

```bash
stoobly-agent scenario test "<SCENARIO-KEY>" --save
```

***

### Environment-Specific Testing

#### Q: How do I run the same scenario test against different environments?

**A:** Use the `--host` option to target different environments.

**Example:**

```bash
# Test against local
stoobly-agent scenario test "<SCENARIO-KEY>" --host localhost:8080

# Test against staging
stoobly-agent scenario test "<SCENARIO-KEY>" --host staging.example.com

# Test against production
stoobly-agent scenario test "<SCENARIO-KEY>" --host api.example.com
```

#### Q: How do I create environment-specific test scenarios?

**A:** Create separate scenarios for each environment or use the same scenario with different hosts.

**Example:**

```bash
# Option 1: Separate scenarios
stoobly-agent scenario create "Login Flow - Local"
stoobly-agent scenario create "Login Flow - Staging"
stoobly-agent scenario create "Login Flow - Production"

# Option 2: One scenario, different hosts
stoobly-agent scenario create "Login Flow"
# Use with different hosts:
stoobly-agent scenario test login-flow --host localhost:8080
stoobly-agent scenario test login-flow --host staging.example.com
```

***

### CI/CD Integration

#### Q: How do I use scenario tests in CI/CD pipelines?

**A:** Test scenarios as part of your automated test suite.

**Example:**

```bash
#!/bin/bash
# CI/CD test script

# Test critical scenarios
scenarios=(
  "user-registration"
  "user-login"
  "checkout-flow"
  "admin-operations"
)

failed=0
for scenario in "${scenarios[@]}"; do
  echo "Testing: $scenario"
  if ! stoobly-agent scenario test $scenario --strategy diff --output-level failed; then
    ((failed++))
    echo "❌ Failed: $scenario"
  else
    echo "✅ Passed: $scenario"
  fi
done

if [ $failed -gt 0 ]; then
  echo "Failed $failed scenario(s)"
  exit 1
fi

echo "All scenarios passed!"
```

#### Q: How do I generate test reports from scenario tests?

**A:** Use JSON format and process the output.

**Example:**

```bash
#!/bin/bash
# Generate test report

timestamp=$(date +%Y%m%d_%H%M%S)
report_file="test-report-${timestamp}.json"

# Run tests and capture output
stoobly-agent scenario test "<SCENARIO-KEY>" --format json > "$report_file"

# Process results
passed=$(jq '[.results[] | select(.status=="passed")] | length' "$report_file")
failed=$(jq '[.results[] | select(.status=="failed")] | length' "$report_file")

echo "Test Report: $passed passed, $failed failed"
echo "Full report: $report_file"
```

***

### Advanced Testing Operations

#### Q: How do I chain multiple test scenarios?

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

# Main test scenario
stoobly-agent scenario test main-flow --strategy diff
if [ $? -ne 0 ]; then
  echo "Main flow failed"
  exit 1
fi

# Cleanup scenario
stoobly-agent scenario replay cleanup --save

echo "All scenarios completed"
```

#### Q: How do I conditionally execute test scenarios?

**A:** Use scripts with conditional logic based on scenario results.

**Example:**

```bash
#!/bin/bash
# Conditional scenario execution

# Run smoke tests first
if stoobly-agent scenario test smoke-tests --output-level failed; then
  echo "Smoke tests passed, running full suite"
  stoobly-agent scenario test full-test-suite
else
  echo "Smoke tests failed, skipping full suite"
  exit 1
fi
```

***

### Monitoring and Debugging

#### Q: How do I debug a failing scenario test?

**A:** Increase log level and use verbose output.

**Example:**

```bash
# Debug with verbose logging
stoobly-agent scenario test "<SCENARIO-KEY>" --log-level debug --output-level failed

# Replay with logging to see each request
stoobly-agent scenario replay "<SCENARIO-KEY>" --log-level info
```

#### Q: How do I identify which request in a scenario test is failing?

**A:** Use detailed output and logging to track request execution.

**Example:**

```bash
# Test with all output levels
stoobly-agent scenario test "<SCENARIO-KEY>" --output-level passed --log-level info

# This shows each request as it executes and its result
```

#### Q: How do I monitor scenario test health over time?

**A:** Set up periodic scenario testing with result tracking.

**Example:**

```bash
#!/bin/bash
# Monitoring script

while true; do
  timestamp=$(date +%Y-%m-%d_%H:%M:%S)
  
  if stoobly-agent scenario test health-check --strategy fuzzy; then
    echo "$timestamp - HEALTHY"
  else
    echo "$timestamp - UNHEALTHY - ALERT SENT"
    # Send alert (email, Slack, PagerDuty, etc.)
  fi
  
  sleep 300  # Check every 5 minutes
done
```

***

