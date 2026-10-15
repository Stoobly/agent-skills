# Config

## Stoobly Config CLI - Questions & Answers

The config CLI manages Stoobly configuration including proxy settings, scenarios, rewrite rules, match rules, filter rules, and project settings. It allows runtime configuration of the agent without editing files directly.

***

### Understanding Configuration

#### Q: What is Stoobly configuration?

**A:** Configuration controls how Stoobly intercepts, modifies, and routes HTTP requests. It includes rewrite rules for transforming requests, match rules for identifying requests, and filter rules for filtering traffic.

**Example:**

```bash
# View current configuration
stoobly-agent setting dump

# View configuration directory
stoobly-agent setting dump --dir
```

#### Q: Where is configuration stored?

**A:** Configuration is stored in the `.stoobly` directory, typically in `~/.stoobly/settings.yml` or your project's `.stoobly/settings.yml`.

**Example:**

```bash
# View config directory location
stoobly-agent setting dump --dir
# Output: /home/user/.stoobly

# View full configuration
stoobly-agent setting dump
```

***

### Viewing and Managing Configuration

#### Q: How do I view the current configuration?

**A:** Use `setting dump` to display all configuration settings.

**Example:**

```bash
# Display configuration as JSON
stoobly-agent setting dump
```

#### Q: How do I save configuration to a file?

**A:** Use the `--save-to-file` flag to export configuration.

**Example:**

```bash
# Save to timestamped file
stoobly-agent setting dump --save-to-file
# Output: Config successfully dumped to stoobly_agent_config_dump_1234567890.json

# View the file
cat stoobly_agent_config_dump_*.json
```

#### Q: How do I reset configuration to defaults?

**A:** Use `config reset` to restore default settings.

**Example:**

```bash
stoobly-agent setting reset
# Output: Reset /home/user/.stoobly/settings.yml to defaults.
```

#### Q: How do I validate my configuration?

**A:** Use `config validate` to check for configuration errors.

**Example:**

```bash
stoobly-agent setting validate
```

***

### Managing Active Scenario

#### Q: How do I set the active scenario?

**A:** Use `config scenario set` with the scenario key to make it active for intercept operations.

**Example:**

```bash
stoobly-agent setting scenario set "<SCENARIO-KEY>"
# Output: Scenario updated!
```

#### Q: How do I view the currently active scenario?

**A:** Use `config scenario show` to display active scenario details.

**Example:**

```bash
stoobly-agent setting scenario show
```

#### Q: How do I clear the active scenario?

**A:** Use `config scenario clear` to remove the active scenario setting.

**Example:**

```bash
stoobly-agent setting scenario clear
# Output: Scenario cleared!
```

#### Q: Why would I set an active scenario?

**A:** Setting an active scenario directs all intercepted requests to that scenario automatically, useful for organizing recorded requests during development.

**Example:**

```bash
# Set active scenario
stoobly-agent setting scenario set user-login-flow

# Start intercepting
stoobly-agent run --intercept --intercept-mode record

# All recorded requests go to user-login-flow scenario
```

***

### Rewrite Rules

#### Q: What are rewrite rules?

**A:** Rewrite rules transform HTTP requests in flight, allowing you to modify URLs, headers, query parameters, or body parameters before they're sent or matched.

**Example:**

```bash
# Rewrite host from old to new
stoobly-agent setting rewrite set \
  --pattern "https://old-api.example.com/.*" \
  --method GET \
  --method POST \
  --mode mock \
  --mode record \
  --hostname new-api.example.com
```

#### Q: How do I create a rewrite rule to change the hostname?

**A:** Use `config rewrite set` with `--hostname` option.

**Example:**

```bash
stoobly-agent setting rewrite set \
  --pattern "https://api.example.com/.*" \
  --method GET \
  --method POST \
  --mode mock \
  --hostname api-staging.example.com
```

#### Q: How do I create a rewrite rule to change the URL path?

**A:** Use the `--path` option to specify the new path.

**Example:**

```bash
# Rewrite /v1/users to /v2/users
stoobly-agent setting rewrite set \
  --pattern "https://api.example.com/v1/users" \
  --method GET \
  --mode mock \
  --mode record \
  --path "/v2/users"
```

#### Q: How do I create a rewrite rule to change the scheme?

**A:** Use the `--scheme` option to switch between http and https.

**Example:**

```bash
# Change https to http
stoobly-agent setting rewrite set \
  --pattern "https://api.example.com/.*" \
  --method GET \
  --mode mock \
  --scheme http
```

#### Q: How do I create a rewrite rule to change the port?

**A:** Use the `--port` option to specify a different port.

**Example:**

```bash
# Redirect to different port
stoobly-agent setting rewrite set \
  --pattern "https://api.example.com/.*" \
  --method GET \
  --mode mock \
  --port 8443
```

#### Q: How do I rewrite request headers?

**A:** Use `--type header` with `--name` and `--value` options.

**Example:**

```bash
# Add/modify Authorization header
stoobly-agent setting rewrite set \
  --pattern "https://api.example.com/.*" \
  --method GET \
  --method POST \
  --mode mock \
  --mode record \
  --type header \
  --name "Authorization" \
  --value "Bearer new-token"
```

#### Q: How do I rewrite query parameters?

**A:** Use `--type query_param` with parameter name and value.

**Example:**

```bash
# Rewrite api_key query parameter
stoobly-agent setting rewrite set \
  --pattern "https://api.example.com/.*" \
  --method GET \
  --mode mock \
  --type query_param \
  --name "api_key" \
  --value "test-key-123"
```

#### Q: How do I rewrite body parameters?

**A:** Use `--type body_param` for POST/PUT request body parameters.

**Example:**

```bash
# Rewrite userId in request body
stoobly-agent setting rewrite set \
  --pattern "https://api.example.com/users" \
  --method POST \
  --mode record \
  --type body_param \
  --name "userId" \
  --value "test-user-123"
```

#### Q: How do I rewrite response headers?

**A:** Use `--type response_header` to modify response headers.

**Example:**

```bash
# Add custom response header
stoobly-agent setting rewrite set \
  --pattern "https://api.example.com/.*" \
  --method GET \
  --mode mock \
  --type response_header \
  --name "X-Custom-Header" \
  --value "custom-value"
```

#### Q: How do I rewrite response parameters?

**A:** Use `--type response_param` to modify response body parameters.

**Example:**

```bash
# Rewrite response data
stoobly-agent setting rewrite set \
  --pattern "https://api.example.com/.*" \
  --method GET \
  --mode mock \
  --type response_param \
  --name "status" \
  --value "success"
```

#### Q: How do I apply rewrite rules to specific modes?

**A:** Use multiple `--mode` options to specify which intercept modes the rule applies to.

**Example:**

```bash
# Apply only to mock and test modes
stoobly-agent setting rewrite set \
  --pattern "https://api.example.com/.*" \
  --method GET \
  --mode mock \
  --mode test \
  --hostname localhost:8080
```

#### Q: How do I create rewrite rules for specific HTTP methods?

**A:** Use multiple `--method` options to target specific methods.

**Example:**

```bash
# Apply to GET and POST only
stoobly-agent setting rewrite set \
  --pattern "https://api.example.com/.*" \
  --method GET \
  --method POST \
  --mode mock \
  --hostname api-mock.local
```

***

### Match Rules

#### Q: What are match rules?

**A:** Match rules determine which request components (headers, query params, body params) are used to match requests to recorded responses during mocking or testing.

**Example:**

```bash
# Match by query parameters and headers
stoobly-agent setting match set \
  --pattern "https://api.example.com/users" \
  --method GET \
  --mode mock \
  --component query_param \
  --component header
```

#### Q: How do I create a match rule to ignore certain components?

**A:** Specify only the components you want to match; unspecified components are ignored.

**Example:**

```bash
# Match only by path, ignore query params
stoobly-agent setting match set \
  --pattern "https://api.example.com/search" \
  --method GET \
  --mode mock
  # No --component means match by path/method only
```

#### Q: How do I match by query parameters?

**A:** Use `--component query_param` to include query parameters in matching.

**Example:**

```bash
stoobly-agent setting match set \
  --pattern "https://api.example.com/search" \
  --method GET \
  --mode mock \
  --component query_param
```

#### Q: How do I match by headers?

**A:** Use `--component header` to include headers in matching.

**Example:**

```bash
# Match by Authorization header
stoobly-agent setting match set \
  --pattern "https://api.example.com/.*" \
  --method GET \
  --method POST \
  --mode mock \
  --component header
```

#### Q: How do I match by body parameters?

**A:** Use `--component body_param` for POST/PUT request body matching.

**Example:**

```bash
stoobly-agent setting match set \
  --pattern "https://api.example.com/users" \
  --method POST \
  --mode mock \
  --component body_param
```

#### Q: How do I match by multiple components?

**A:** Use multiple `--component` options to combine matching criteria.

**Example:**

```bash
# Match by query params AND headers
stoobly-agent setting match set \
  --pattern "https://api.example.com/search" \
  --method GET \
  --mode mock \
  --component query_param \
  --component header
```

***

### Filter Rules

#### Q: What are filter rules?

**A:** Filter rules control which requests are intercepted, allowing you to include or exclude specific URLs, methods, or patterns.

**Example:**

```bash
# Exclude analytics requests from recording
stoobly-agent setting filter set \
  --pattern "https://analytics.example.com/.*" \
  --method GET \
  --method POST \
  --mode record \
  --action exclude
```

#### Q: How do I exclude requests from interception?

**A:** Use `--action exclude` with a pattern to filter out requests.

**Example:**

```bash
# Exclude third-party tracking
stoobly-agent setting filter set \
  --pattern "https://.*google-analytics.com/.*" \
  --method GET \
  --method POST \
  --mode record \
  --mode mock \
  --action exclude
```

#### Q: How do I include only specific requests?

**A:** Use `--action include` with a pattern to whitelist requests.

**Example:**

```bash
# Only intercept API requests
stoobly-agent setting filter set \
  --pattern "https://api.example.com/.*" \
  --method GET \
  --method POST \
  --mode record \
  --action include
```

#### Q: How do I filter by HTTP method?

**A:** Use multiple `--method` options to target specific methods.

**Example:**

```bash
# Exclude OPTIONS requests
stoobly-agent setting filter set \
  --pattern ".*" \
  --method OPTIONS \
  --mode record \
  --mode mock \
  --action exclude
```

#### Q: How do I apply filter rules to specific modes?

**A:** Use multiple `--mode` options to control which intercept modes are affected.

**Example:**

```bash
# Exclude from recording only, still mock
stoobly-agent setting filter set \
  --pattern "https://cdn.example.com/.*" \
  --method GET \
  --mode record \
  --action exclude
```

***

### Project Management (Remote Features)

#### Q: How do I set the active project?

**A:** Use `config project set` with the project key (requires remote features).

**Example:**

```bash
stoobly-agent setting project set "<PROJECT-KEY>"
# Output: Project updated!
```

#### Q: How do I view the current project?

**A:** Use `config project show` to display active project details.

**Example:**

```bash
stoobly-agent setting project show
```

#### Q: How do I switch to local project mode?

**A:** Use `config project local` to use local storage instead of remote.

**Example:**

```bash
stoobly-agent setting project local
# Output: Using local project!
```

#### Q: How do I set my API key for remote features?

**A:** Use `config api-key set` with your API key.

**Example:**

```bash
stoobly-agent setting api-key set your-api-key-here
# Output: API Key updated!
```

***

### Configuration Workflows

#### Q: How do I set up environment-specific rewrite rules?

**A:** Create rewrite rules that redirect production URLs to local or staging environments.

**Example:**

```bash
# Development: Point to localhost
stoobly-agent setting rewrite set \
  --pattern "https://api.production.com/.*" \
  --method GET \
  --method POST \
  --mode mock \
  --mode test \
  --hostname localhost:8080 \
  --scheme http

# Staging: Point to staging server
stoobly-agent setting rewrite set \
  --pattern "https://api.production.com/.*" \
  --method GET \
  --method POST \
  --mode record \
  --hostname api.staging.com
```

#### Q: How do I configure request filtering for testing?

**A:** Combine filter and match rules to control which requests are tested.

**Example:**

```bash
# Exclude external services from tests
stoobly-agent setting filter set \
  --pattern "https://.*amazonaws.com/.*" \
  --method GET \
  --method POST \
  --mode test \
  --action exclude

# Match API requests precisely
stoobly-agent setting match set \
  --pattern "https://api.myapp.com/.*" \
  --method GET \
  --method POST \
  --mode test \
  --component query_param \
  --component header
```

#### Q: How do I mock with modified authentication?

**A:** Use rewrite rules to replace authentication tokens in mock mode.

**Example:**

```bash
# Replace auth tokens for mocking
stoobly-agent setting rewrite set \
  --pattern "https://api.example.com/.*" \
  --method GET \
  --method POST \
  --mode mock \
  --type header \
  --name "Authorization" \
  --value "Bearer test-token"

# Start mocking
stoobly-agent run --intercept --intercept-mode mock
```

***

### Advanced Configuration

#### Q: How do I create rules for specific projects?

**A:** Use the `--project-key` option to scope rules to a specific project.

**Example:**

```bash
# Add rewrite rule to specific project
stoobly-agent setting rewrite set \
  --pattern "https://api.example.com/.*" \
  --method GET \
  --mode mock \
  --hostname localhost \
  --project-key "<PROJECT-KEY>"
```

#### Q: How do I configure rules for microservices?

**A:** Create separate rules for each service endpoint.

**Example:**

```bash
# Auth service
stoobly-agent setting rewrite set \
  --pattern "https://auth.example.com/.*" \
  --method GET \
  --method POST \
  --mode mock \
  --hostname localhost:8001

# User service
stoobly-agent setting rewrite set \
  --pattern "https://users.example.com/.*" \
  --method GET \
  --method POST \
  --mode mock \
  --hostname localhost:8002

# Order service
stoobly-agent setting rewrite set \
  --pattern "https://orders.example.com/.*" \
  --method GET \
  --method POST \
  --mode mock \
  --hostname localhost:8003
```

***

### Troubleshooting

#### Q: How do I debug configuration issues?

**A:** Dump configuration and validate it.

**Example:**

```bash
# View configuration
stoobly-agent setting dump

# Validate configuration
stoobly-agent setting validate

# Check for specific rules
stoobly-agent setting dump | jq '.proxy.rewrite'
stoobly-agent setting dump | jq '.proxy.filter'
stoobly-agent setting dump | jq '.proxy.match'
```

#### Q: How do I reset a specific configuration section?

**A:** Currently, you can only reset all configuration. To reset specific sections, edit the config file manually or use reset and reconfigure.

**Example:**

```bash
# Backup current config
stoobly-agent setting dump --save-to-file

# Reset to defaults
stoobly-agent setting reset

# Reconfigure specific sections
stoobly-agent setting rewrite set ...
```

#### Q: How do I check if my rewrite rules are working?

**A:** Use increased logging and flow detail to see rule application.

**Example:**

```bash
# Start with debug logging
stoobly-agent run --log-level debug --flow-detail 4 --intercept --intercept-mode mock

# Make request and check logs for rewrite application
```

***

### Quick Reference

#### Q: What are the most common config commands?

**A:** Here's a quick reference of frequently used commands:

**Example:**

```bash
# View configuration
stoobly-agent setting dump
stoobly-agent setting dump --dir
stoobly-agent setting dump --save-to-file

# Reset configuration
stoobly-agent setting reset
stoobly-agent setting validate

# Manage scenarios
stoobly-agent setting scenario set "<SCENARIO-KEY>"
stoobly-agent setting scenario show
stoobly-agent setting scenario clear

# Rewrite rules
stoobly-agent setting rewrite set \
  --pattern "URL_PATTERN" \
  --method GET --method POST \
  --mode mock --mode record \
  --hostname new-host.com

# Match rules
stoobly-agent setting match set \
  --pattern "URL_PATTERN" \
  --method GET \
  --mode mock \
  --component query_param --component header

# Filter rules
stoobly-agent setting filter set \
  --pattern "URL_PATTERN" \
  --method GET \
  --mode record \
  --action exclude

# Project management (remote features)
stoobly-agent setting api-key set "<API-KEY>"
stoobly-agent setting project set "<PROJECT-KEY>"
stoobly-agent setting project show
stoobly-agent setting project local
```

***

### Best Practices

#### Q: How should I organize my configuration?

**A:** Use specific patterns for different services and environments.

**Example:**

```bash
# Service-specific patterns
stoobly-agent setting rewrite set --pattern "https://api.myapp.com/.*" ...
stoobly-agent setting rewrite set --pattern "https://auth.myapp.com/.*" ...

# Environment-specific rules
stoobly-agent setting rewrite set --mode mock --hostname localhost ...
stoobly-agent setting rewrite set --mode record --hostname staging.com ...
```

#### Q: When should I use filter rules vs match rules?

**A:** Use filter rules to control what gets intercepted, match rules to control how requests are matched to responses.

**Example:**

```bash
# Filter: Exclude from interception
stoobly-agent setting filter set --pattern "https://cdn..*" --action exclude

# Match: Control matching precision
stoobly-agent setting match set --pattern "https://api..*" --component query_param
```
