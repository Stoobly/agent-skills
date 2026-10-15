# Run

## Stoobly Main CLI Commands - Questions & Answers

The main CLI provides core commands for running the Stoobly agent, initializing contexts, and performing quick record/mock operations without starting the full proxy server.

***

### Getting Started

#### Q: How do I check the Stoobly agent version?

**A:** Use the `--version` flag to display the installed version.

**Example:**

```bash
stoobly-agent --version
```

#### Q: How do I get help for any command?

**A:** Use `--help` or `-h` after any command to see detailed usage information.

**Example:**

```bash
# Main help
stoobly-agent --help

# Command-specific help
stoobly-agent run --help
stoobly-agent record --help
stoobly-agent mock --help
```

***

### Initializing Stoobly

#### Q: How do I initialize a new Stoobly context?

**A:** Use the `init` command to create a new `.stoobly` directory with necessary configuration files.

**Example:**

```bash
# Initialize in current directory
stoobly-agent init

# Creates: .stoobly/ directory with database and configuration
```

#### Q: What does init create?

**A:** The init command creates the `.stoobly` directory structure including database, configuration files, and necessary subdirectories.

**Example:**

```bash
stoobly-agent init

# Created structure:
# .stoobly/
# ├── db/              # Database files
# ├── ca_certs/        # CA certificates
# ├── certs/           # SSL certificates
# ├── logs/            # Log files
# └── config.yml       # Configuration
```

#### Q: When should I run init?

**A:** Run init when setting up Stoobly in a new project directory or when you want to create a fresh Stoobly context.

**Example:**

```bash
# New project setup
mkdir my-project
cd my-project
stoobly-agent init
```

***

### Running the Agent

#### Q: How do I start Stoobly with both proxy and UI?

**A:** Use the `run` command without any flags to start both the proxy server and web UI.

**Example:**

```bash
stoobly-agent run

# Proxy: http://localhost:8080
# UI: http://localhost:4200
```

#### Q: How do I run Stoobly in headless mode (proxy only)?

**A:** Use the `--headless` flag to disable the UI and run only the proxy.

**Example:**

```bash
stoobly-agent run --headless

# Only proxy runs on http://localhost:8080
```

#### Q: How do I run Stoobly without the proxy (UI only)?

**A:** Use the `--proxyless` flag to run only the web UI.

**Example:**

```bash
stoobly-agent run --proxyless

# Only UI runs on http://localhost:4200
```

#### Q: How do I run Stoobly on custom ports?

**A:** Use `--proxy-port` and `--ui-port` to specify custom ports.

**Example:**

```bash
# Custom ports
stoobly-agent run --proxy-port 9090 --ui-port 5000

# Proxy: http://localhost:9090
# UI: http://localhost:5000
```

#### Q: How do I run Stoobly on a specific host/interface?

**A:** Use `--proxy-host` and `--ui-host` to bind to specific network interfaces.

**Example:**

```bash
# Bind to all interfaces (default)
stoobly-agent run --proxy-host 0.0.0.0 --ui-host 0.0.0.0

# Bind to localhost only
stoobly-agent run --proxy-host 127.0.0.1 --ui-host 127.0.0.1

# Bind to specific IP
stoobly-agent run --proxy-host 192.168.1.100
```

#### Q: How do I run Stoobly in detached/background mode?

**A:** Use the `--detached` flag with a file path to redirect output and run in the background.

**Example:**

```bash
# Run in background
stoobly-agent run --detached /tmp/stoobly.log

# Returns PID
# 12345

# View logs
tail -f /tmp/stoobly.log

# Stop later
kill 12345
```

#### Q: How do I enable intercept mode on startup?

**A:** Use the `--intercept` flag to activate interception immediately.

**Example:**

```bash
# Enable intercept on startup
stoobly-agent run --intercept

# With specific mode
stoobly-agent run --intercept --intercept-mode mock
stoobly-agent run --intercept --intercept-mode record
```

#### Q: How do I set the intercept mode on startup?

**A:** Use the `--intercept-mode` option with one of: mock, record, replay, test.

**Example:**

```bash
# Start with mock mode
stoobly-agent run --intercept --intercept-mode mock

# Start with record mode
stoobly-agent run --intercept --intercept-mode record

# Start with test mode
stoobly-agent run --intercept --intercept-mode test
```

***

### Logging and Debugging

#### Q: How do I increase logging verbosity?

**A:** Use the `--log-level` option to control log output detail.

**Example:**

```bash
# Debug level (most verbose)
stoobly-agent run --log-level debug

# Info level (default)
stoobly-agent run --log-level info

# Warning level
stoobly-agent run --log-level warning

# Error level (least verbose)
stoobly-agent run --log-level error
```

#### Q: How do I control proxy flow detail output?

**A:** Use the `--flow-detail` option with levels 0-4.

**Example:**

```bash
# Level 0: No output
stoobly-agent run --flow-detail 0

# Level 1: Shortened URL + status code (default)
stoobly-agent run --flow-detail 1

# Level 2: Full URL + status + headers
stoobly-agent run --flow-detail 2

# Level 3: Level 2 + truncated response content
stoobly-agent run --flow-detail 3

# Level 4: Everything (no truncation)
stoobly-agent run --flow-detail 4
```

#### Q: How do I enable request logging?

**A:** Use the `--request-log-enable` flag to log all intercepted requests.

**Example:**

```bash
# Enable request logging
stoobly-agent run --request-log-enable

# With specific log level
stoobly-agent run --request-log-enable --request-log-level debug

# Without truncating log on startup
stoobly-agent run --request-log-enable --request-log-truncate=false
```

#### Q: How do I view the intercepted requests log?

**A:** Use the request logs list command while the agent is running or after. Use `-f` to stream logs in real time, or filter by method, URL, status code, and more.

**Examples:**

```bash
# Start with logging
stoobly-agent run --request-log-enable

# In another terminal, view logs
stoobly-agent request logs list

# Follow logs in real time (-f / --follow, Ctrl-C to stop)
stoobly-agent request logs list --follow

# Filter by log level
stoobly-agent request logs list --level error

# Filter by URL and method
stoobly-agent request logs list --url /api --method post
```

***

### SSL/TLS Configuration

#### Q: How do I specify a custom CA certificates directory?

**A:** Use the `--ca-certs-dir-path` option to set the CA certificate location.

**Example:**

```bash
stoobly-agent run --ca-certs-dir-path ~/.stoobly/ca_certs
```

#### Q: How do I use custom SSL certificates?

**A:** Use the `--certs` option to provide custom certificate files.

**Example:**

```bash
# Single domain certificate
stoobly-agent run --certs "example.com=/path/to/cert.pem"

# Wildcard certificate
stoobly-agent run --certs "*.example.com=/path/to/wildcard.pem"

# Multiple certificates
stoobly-agent run \
  --certs "api.example.com=/path/to/api-cert.pem" \
  --certs "app.example.com=/path/to/app-cert.pem"
```

#### Q: How do I provide a passphrase for encrypted certificates?

**A:** Use the `--cert-passphrase` option (note: visible in process list, prefer config file).

**Example:**

```bash
stoobly-agent run \
  --certs "example.com=/path/to/encrypted-cert.pem" \
  --cert-passphrase "my-secret-passphrase"
```

#### Q: How do I disable SSL certificate verification?

**A:** Use the `--ssl-insecure` flag to skip upstream SSL verification (use with caution).

**Example:**

```bash
# Disable SSL verification
stoobly-agent run --ssl-insecure

# Useful for self-signed certificates in development
```

***

### Proxy Modes

#### Q: What proxy modes are available?

**A:** Stoobly supports regular, transparent, socks5, reverse, and upstream proxy modes.

**Example:**

```bash
# Regular proxy (default)
stoobly-agent run --proxy-mode regular

# Transparent proxy
stoobly-agent run --proxy-mode transparent

# SOCKS5 proxy
stoobly-agent run --proxy-mode socks5

# Reverse proxy
stoobly-agent run --proxy-mode "reverse:https://api.example.com"

# Upstream proxy
stoobly-agent run --proxy-mode "upstream:http://corporate-proxy:3128"
```

#### Q: How do I set up a reverse proxy?

**A:** Use the reverse proxy mode with the target host specification.

**Example:**

```bash
# Reverse proxy to backend
stoobly-agent run --proxy-mode "reverse:https://api.backend.com"

# All requests to proxy will be forwarded to api.backend.com
curl http://localhost:8080/users
# Actually requests: https://api.backend.com/users
```

#### Q: How do I use an upstream proxy?

**A:** Use the upstream proxy mode to chain through another proxy.

**Example:**

```bash
# Chain through corporate proxy
stoobly-agent run --proxy-mode "upstream:http://proxy.corporate.com:8080"
```

***

### Lifecycle Hooks

#### Q: How do I use lifecycle hooks with the agent?

**A:** Use the `--lifecycle-hooks-path` option to specify a Python script with custom hooks.

**Example:**

```bash
# Create hooks script
cat > hooks.py << 'EOF'
def before_request(context):
    # Modify request before sending
    context.request.headers['X-Custom-Header'] = 'value'
    return context

def after_response(context):
    # Process response
    print(f"Response status: {context.response.status_code}")
    return context
EOF

# Run with hooks
stoobly-agent run --lifecycle-hooks-path ./hooks.py
```

***

### Mocking and Response Fixtures

#### Q: How do I use response fixtures for mocking?

**A:** Use the `--response-fixtures-path` option to provide YAML files with mock responses.

**Example:**

```bash
# Create fixtures file
cat > fixtures.yml << 'EOF'
- GET:
    /users/d+?:
      headers: {}
      path: <RELATIVE-PATH-TO-TO-RESPONSE-FILE>
      status_code: 200
- POST:
    /users:
      headers: {}
      path: <RELATIVE-PATH-TO-TO-RESPONSE-FILE>
      status_code: 200
EOF

# Run with fixtures
stoobly-agent run --response-fixtures-path ./fixtures.yml
```

#### Q: How do I serve static files for mocking?

**A:** Use the `--public-dir-path` option to serve files from a directory.

**Example:**

```bash
# Create public directory
mkdir public
echo '{"data": "test"}' > public/test.json

# Run with public directory
stoobly-agent run --public-dir-path ./public

# Access: http://localhost:8080/test.json
```

#### Q: How do I specify origin for fixtures and public directories?

**A:** Use the format `<PATH>:<ORIGIN>` to match specific origins.

**Example:**

```bash
stoobly-agent run \
  --response-fixtures-path "./fixtures.yml:https://api.example.com" \
  --public-dir-path "./public:https://cdn.example.com"
```

***

### Header Modification

#### Q: How do I modify request headers?

**A:** Use the `--modify-headers` option with header modification patterns.

**Example:**

```bash
# Add header to all requests
stoobly-agent run --modify-headers "/Authorization/Bearer token123"

# Remove header
stoobly-agent run --modify-headers "/X-Debug-Token/"

# Modify specific path
stoobly-agent run --modify-headers "/~u /api/.*//X-Custom/value"

# Multiple modifications
stoobly-agent run \
  --modify-headers "/Authorization/Bearer token" \
  --modify-headers "/X-API-Key/abc123"
```

***

### Quick Record Command

#### Q: How do I quickly record a single request without starting the agent?

**A:** Use the `record` command to make a request and store it.

**Example:**

```bash
# Record GET request
stoobly-agent record https://api.example.com/users

# Record POST request
stoobly-agent record -X POST -d '{"name":"John"}' https://api.example.com/users

# Record with custom headers
stoobly-agent record -H "Authorization: Bearer token" https://api.example.com/profile
```

#### Q: How do I record a request to a scenario?

**A:** Use the `--scenario-key` option to add the recorded request to a scenario.

**Example:**

```bash
# Record to scenario
stoobly-agent record https://api.example.com/users --scenario-key "<SCENARIO-KEY>"
```

#### Q: How do I save recorded response to a file?

**A:** Use the `-o` or `--output` option to write the response.

**Example:**

```bash
# Save response to file
stoobly-agent record https://api.example.com/users -o response.json

# View the file
cat response.json
```

#### Q: How do I format the recorded response output?

**A:** Use the `--format` option to control output format.

**Example:**

```bash
# Raw HTTP format
stoobly-agent record https://api.example.com/users --format raw
```

#### Q: How do I record requests with lifecycle hooks?

**A:** Use the `--lifecycle-hooks-path` option with the record command.

**Example:**

```bash
stoobly-agent record https://api.example.com/users --lifecycle-hooks-path ./hooks.py
```

***

### Quick Mock Command

#### Q: How do I quickly mock a request without starting the agent?

**A:** Use the `mock` command to replay a previously recorded request.

**Example:**

```bash
# Mock a URL (must have been recorded before)
stoobly-agent mock https://api.example.com/users
```

#### Q: How do I mock with custom headers?

**A:** Use the `-H` flag to add custom headers to the mock request.

**Example:**

```bash
stoobly-agent mock -H "Authorization: Bearer token" https://api.example.com/profile
```

#### Q: How do I mock a POST request?

**A:** Use `-X POST` and `-d` for the request body.

**Example:**

```bash
stoobly-agent mock -X POST -d '{"id":123}' https://api.example.com/users/search
```

#### Q: How do I mock with response fixtures?

**A:** Use the `--response-fixtures-path` option to provide mock data.

**Example:**

```bash
stoobly-agent mock https://api.example.com/users --response-fixtures-path ./fixtures.yml
```

#### Q: How do I mock with public directory files?

**A:** Use the `--public-dir-path` option to serve static files.

**Example:**

```bash
stoobly-agent mock https://cdn.example.com/data.json --public-dir-path ./public
```

#### Q: How do I save mocked response to a file?

**A:** Use the `-o` option to write the response to a file.

**Example:**

```bash
stoobly-agent mock https://api.example.com/users -o mock-response.json
```

***

### Connection Strategy

#### Q: What connection strategies are available?

**A:** Stoobly supports `eager` and `lazy` connection strategies for handling upstream connections.

**Example:**

```bash
# Eager: Connect immediately
stoobly-agent run --connection-strategy eager

# Lazy: Connect only when needed (default)
stoobly-agent run --connection-strategy lazy
```

***

### Complete Examples

#### Q: How do I set up a complete development environment?

**A:** Initialize, install CA cert, optionally create a scenario, then start agent with intercept mode.

**Example:**

```bash
# Step 1: Initialize
stoobly-agent init

# Step 2: Install CA certificate
stoobly-agent ca-cert install

# Step 3 (Optional): Create a scenario to organize requests
stoobly-agent scenario create "My User Flow"
# Output: Created scenario with key: <SCENARIO-KEY>

# Step 4: Start agent with intercept
stoobly-agent run --intercept --intercept-mode record

# Step 5: Configure your app to use proxy
export HTTP_PROXY=http://localhost:8080
export HTTPS_PROXY=http://localhost:8080

# Step 6: Make requests (they get recorded)
curl https://api.example.com/users
```

#### Q: How do I run Stoobly for API mocking in tests?

**A:** Start in headless mode with mock mode and fixtures.

**Example:**

```bash
# Start agent
stoobly-agent run \
  --headless \
  --intercept \
  --intercept-mode mock \
  --response-fixtures-path ./test-fixtures.yml \
  --public-dir-path ./test-data

# Run tests
npm test

# Tests use mocked responses
```

#### Q: How do I run Stoobly in CI/CD?

**A:** Use headless, detached mode with specific configuration.

**Example:**

```bash
#!/bin/bash
# CI/CD script

# Start agent in background
stoobly-agent run \
  --headless \
  --intercept \
  --intercept-mode test \
  --detached /tmp/stoobly.log \
  --log-level info

# Get PID
AGENT_PID=$(cat /tmp/stoobly.log | head -1)

# Wait for agent to start
sleep 2

# Run tests
npm test

# Stop agent
kill $AGENT_PID
```

***

### Environment Variables

#### Q: How do I enable latency simulation for mocked responses?

**A:** Set the `AGENT_SIMULATE_LATENCY` environment variable to enable latency simulation. When enabled, Stoobly will simulate the expected response latency based on the `X-Stoobly-Response-Latency` header value in milliseconds.

**Example:**

```bash
# Enable latency simulation
export AGENT_SIMULATE_LATENCY=1

# Start the agent
stoobly-agent run --intercept --intercept-mode mock

# Mocked responses will now simulate latency based on the X-Stoobly-Response-Latency header
```

**How it works:**

- The latency simulation calculates wait time as: `expected_latency - estimated_rtt_network_latency - api_latency`
- `expected_latency` comes from the `X-Stoobly-Response-Latency` response header (in milliseconds)
- `estimated_rtt_network_latency` is fixed at 15ms
- `api_latency` is the time elapsed since the request started
- If the calculated wait time is positive, Stoobly will sleep for that duration before returning the response

**Use cases:**

- Testing how your application handles slow API responses
- Simulating real-world network conditions during development
- Validating timeout and retry logic

**Note:** This feature only works in mock mode and requires the `X-Stoobly-Response-Latency` header to be present in the mocked response.

***

### Troubleshooting

#### Q: How do I debug connection issues?

**A:** Increase log level and flow detail for verbose output.

**Example:**

```bash
stoobly-agent run --log-level debug --flow-detail 4
```

#### Q: How do I verify the agent is running?

**A:** Check the proxy and UI ports.

**Example:**

```bash
# Check proxy
curl -x http://localhost:8080 http://example.com

# Check UI
curl http://localhost:4200
```

#### Q: What do I do if ports are already in use?

**A:** Use custom ports to avoid conflicts.

**Example:**

```bash
# Use different ports
stoobly-agent run --proxy-port 9090 --ui-port 5000
```

***

### Quick Reference

#### Q: What are the essential commands?

**A:** Here's a quick reference of the most common commands:

**Example:**

```bash
# Check version
stoobly-agent --version

# Initialize
stoobly-agent init

# Run agent (full)
stoobly-agent run

# Run headless
stoobly-agent run --headless

# Run with intercept
stoobly-agent run --intercept --intercept-mode record

# Run detached
stoobly-agent run --detached /tmp/stoobly.log

# Quick record
stoobly-agent record https://api.example.com/users

# Quick mock
stoobly-agent mock https://api.example.com/users

# Custom ports
stoobly-agent run --proxy-port 9090 --ui-port 5000

# Debug mode
stoobly-agent run --log-level debug --flow-detail 4

# With fixtures
stoobly-agent run --response-fixtures-path ./fixtures.yml

# With lifecycle hooks
stoobly-agent run --lifecycle-hooks-path ./hooks.py
```
