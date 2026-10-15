# Local Runtime Customization

## Stoobly Scaffold Local Runtime - Questions & Answers

This document covers local runtime-specific customization options for Stoobly scaffold. For general customization topics, see [README.md](README.md). For Docker runtime customization, see [docker.md](docker.md).

***

### Local Runtime Structure

#### Q: Which files are created for both Docker and local runtime?

**A:** Both runtimes create the Makefile, stoobly-ui service, and user-defined service directories.

**Example:**

```bash
# Common to both:
.stoobly/services/
├── Makefile                    # Both
├── stoobly-ui/                 # Both
└── your-service/               # Both
```

#### Q: What's the difference between --runtime docker and --runtime local?

**A:** Docker runtime creates additional core services (build, entrypoint, gateway) for containerized execution, while local runtime creates a simpler structure for native execution.

**Example:**

```bash
# Local runtime - Simplified structure
stoobly-agent scaffold app create my-app --runtime local

# Creates:
# - stoobly-ui/ (UI service)
# - your-services/ (only user services)
# No build/, entrypoint/, or gateway/ directories
```

**Note:** With local runtime, your services run natively on your machine rather than in containers. You'll need to configure your applications to use Stoobly's proxy directly.

***

### Local Runtime Customization

#### Q: How do I configure my application to use Stoobly with local runtime?

**A:** Configure your application to use Stoobly's proxy by setting HTTP_PROXY and HTTPS_PROXY environment variables, or by configuring your application's HTTP client to use the proxy.

**Example:**

```bash
# Set proxy environment variables
export HTTP_PROXY=http://localhost:8080
export HTTPS_PROXY=http://localhost:8080

# Run your application
npm start
```

#### Q: How do I add environment variables for local runtime?

**A:** For local runtime, set environment variables in your shell or use a `.env` file that your application reads. Unlike Docker runtime, there's no docker-compose.yml to configure.

**Example:**

```bash
# Set environment variables before running
export DATABASE_URL=postgres://localhost/testdb
export API_KEY=test-key-123
export NODE_ENV=test

# Or use a .env file
cat > .env << EOF
DATABASE_URL=postgres://localhost/testdb
API_KEY=test-key-123
NODE_ENV=test
EOF
```

#### Q: How do I run tests with local runtime?

**A:** Run your tests directly with proxy environment variables set, or configure your test framework to use Stoobly's proxy.

**Example:**

```bash
# Set proxy and run tests
export HTTP_PROXY=http://localhost:8080
export HTTPS_PROXY=http://localhost:8080
npx playwright test

# Or configure in your test config
# playwright.config.ts
export default defineConfig({
  use: {
    proxy: {
      server: 'http://localhost:8080',
    },
  },
});
```

#### Q: How do I use a custom context directory?

**A:** Use the `--context-dir-path` option when running workflow commands to specify a custom Stoobly data directory.

**Example:**

```bash
stoobly-agent scaffold workflow up test --context-dir-path /path/to/custom/.stoobly
```

#### Q: How do I run workflows with custom namespaces?

**A:** Use the `--namespace` option to specify a custom workflow namespace when running workflow commands.

**Note:** Namespaces are only supported by test workflows and custom workflows based on the test workflow template. Record and mock workflows do not support namespaces.

**Example:**

```bash
stoobly-agent scaffold workflow up test --namespace my-custom-namespace
```

#### Q: How do I run workflows in dry-run mode?

**A:** Use the `--dry-run` flag to see what commands would be executed without running them.

**Example:**

```bash
stoobly-agent scaffold workflow up test --dry-run
```

#### Q: What's the difference between init and run scripts?

**A:** The `init` script runs before services start to set up Stoobly configuration, while `run` script runs after init completes.

**Example:**

```bash
# init script - Runs first
# .stoobly/services/entrypoint/mock/init
#!/bin/bash
# Add custom Stoobly configuration here
stoobly-agent setting rewrite set --pattern "..." --hostname localhost

# run script - Runs after container init
# Custom entrypoint logic
```

***

### Advanced Configuration

#### Q: How do I increase logging verbosity?

**A:** Use the `--log-level` option when starting a workflow to set the logging level.

**Example:**

```bash
stoobly-agent scaffold workflow up test --log-level debug
```

#### Q: How do I follow workflow logs in real-time?

**A:** Use the `-f` / `--follow` flag with the scaffold workflow logs command to stream logs continuously.

**Example:**

```bash
stoobly-agent scaffold workflow logs record --follow
```

***

### Local Runtime Troubleshooting

#### Q: How do I verify my local service configuration?

**A:** Check the service's init script and run it manually if needed, then verify Stoobly configuration.

**Example:**

```bash
# View service init script
cat .stoobly/services/my-service/mock/init

# Run init script
bash .stoobly/services/my-service/mock/init

# Verify Stoobly config
stoobly-agent setting dump
```

#### Q: How do I view intercepted request logs for a workflow?

**A:** Use the `stoobly-agent scaffold request logs list` command to inspect intercepted request logs for a workflow.

**Examples:**

```bash
# List intercepted request log entries for a workflow
stoobly-agent scaffold request logs list record
stoobly-agent scaffold request logs list mock
stoobly-agent scaffold request logs list test

# Follow logs in real time (-f / --follow, Ctrl-C to stop)
stoobly-agent scaffold request logs list test --follow

# Filter by service name
stoobly-agent scaffold request logs list mock --service-name my-service

# Filter by namespace
stoobly-agent scaffold request logs list mock --namespace my-namespace

# Filter by HTTP method
stoobly-agent scaffold request logs list test --method post

# Filter by URL substring
stoobly-agent scaffold request logs list mock --url /api/users

# Filter by HTTP status code
stoobly-agent scaffold request logs list test --status-code 500

# Filter by log level
stoobly-agent scaffold request logs list test --level error

# Filter by log message
stoobly-agent scaffold request logs list mock --message "Mock failure"

# Filter by scenario name
stoobly-agent scaffold request logs list mock --scenario-name my-scenario

# Output as JSON
stoobly-agent scaffold request logs list mock --format json

# Select specific columns
stoobly-agent scaffold request logs list test --select method,url,status-code

# Combine filters with follow
stoobly-agent scaffold request logs list test --follow --level error --method post

# Use a custom context directory
stoobly-agent scaffold request logs list mock \
  --context-dir-path /path/to/custom/.stoobly
```

#### Q: How do I find the log file path for a workflow?

**A:** Use the `stoobly-agent scaffold request logs path` command to print the underlying intercepted request log file path for a workflow.

**Example:**

```bash
stoobly-agent scaffold request logs path mock
stoobly-agent scaffold request logs path test
```

#### Q: How do I delete intercepted request logs for a workflow?

**A:** Use the `stoobly-agent scaffold request logs delete` command to truncate (delete) intercepted request logs for a workflow.

**Example:**

```bash
stoobly-agent scaffold request logs delete mock
stoobly-agent scaffold request logs delete test
```