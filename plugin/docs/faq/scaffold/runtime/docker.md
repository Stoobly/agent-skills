# Docker Runtime

## Stoobly Scaffold Docker Runtime - Questions & Answers

This document covers Docker-specific runtime options for Stoobly scaffold. For general runtime topics, see [README.md](README.md). For local runtime, see [local.md](local.md).

***

### Docker Runtime Overview

#### Q: What are the benefits of using Docker runtime?

**A:** Docker provides environment consistency across team members, dependency isolation, easy cleanup, and works the same on all operating systems.

**Example:**

```bash
stoobly-agent scaffold app create my-app --runtime docker

# All team members get the same environment
make -f .stoobly/services/.Makefile record
```

#### Q: What are the requirements for Docker runtime?

**A:** You need Docker installed and running on your machine. The Docker daemon must be accessible.

**Example:**

```bash
# Verify Docker is installed and running
docker --version
docker ps

# Create Docker-based scaffold
stoobly-agent scaffold app create my-app --runtime docker
```

#### Q: How do I specify a custom Docker socket path?

**A:** Use the `--docker-socket-path` option when creating the app.

**Example:**

```bash
stoobly-agent scaffold app create my-app \
  --runtime docker \
  --docker-socket-path /var/run/docker.sock
```

#### Q: How do Docker-based workflows work?

**A:** Docker workflows use docker-compose to orchestrate containers for stoobly-agent, your services, and any dependencies, providing complete isolation.

**Example:**

```bash
# Create Docker-based app
stoobly-agent scaffold app create my-app --runtime docker

# Start workflow (runs in containers)
make -f .stoobly/services/.Makefile record

# View running containers
docker ps

# Stop workflow (cleans up containers)
make -f .stoobly/services/.Makefile record/down
```

#### Q: What files are created when I scaffold an app with Docker runtime?

**A:** Scaffolding creates a `.stoobly/services/` directory in the current directory with Docker configurations, Makefile, service definitions, and workflow templates.

**Example:**

```bash
stoobly-agent scaffold app create my-app --runtime docker
ls -la .stoobly/services/
# build .docker-compose.base.yml .Dockerfile.context gateway Makefile
```

#### Q: Can I use Docker runtime on Windows, Mac, and Linux?

**A:** Yes, Docker runtime works consistently across all platforms as long as Docker Desktop (Windows/Mac) or Docker Engine (Linux) is installed.

**Example:**

```bash
# Same command works on all platforms
stoobly-agent scaffold app create my-app --runtime docker
make -f .stoobly/services/.Makefile record
```

#### Q: How do I troubleshoot Docker runtime issues?

**A:** Check Docker is running, verify permissions, and review container logs.

**Example:**

```bash
# Check Docker status
docker ps

# View container logs
make -f .stoobly/services/.Makefile mock/logs

# Check intercepted request logs
make -f .stoobly/services/.Makefile mock/request/logs

# Follow logs in real time (-f / --follow, Ctrl-C to stop)
make -f .stoobly/services/.Makefile mock/request/logs options="--follow"

# Filter by log level
make -f .stoobly/services/.Makefile mock/request/logs options="--level error"
```

***

### Docker Workflows

#### Q: How do I explicitly use Docker runtime for a workflow?

**A:** Use the Makefile commands, which are configured for Docker execution.

**Example:**

```bash
# Docker runtime via Makefile
make -f .stoobly/services/.Makefile record
make -f .stoobly/services/.Makefile mock
make -f .stoobly/services/.Makefile test
```

#### Q: How do I use Docker runtime?

**A:** Create the app with `--runtime docker` and use Makefile commands for workflows.

**Example:**

```bash
# App created with Docker runtime
stoobly-agent scaffold app create my-app --runtime docker
make -f .stoobly/services/.Makefile record
```

***

### Docker Performance

#### Q: How do I optimize Docker runtime performance?

**A:** Use local Docker images, enable BuildKit, and allocate sufficient resources to Docker.

**Example:**

```bash
# Use local images (faster)
export STOOBLY_IMAGE_USE_LOCAL=1
make -f .stoobly/services/.Makefile record

# Enable Docker BuildKit
export DOCKER_BUILDKIT=1
make -f .stoobly/services/.Makefile record
```

#### Q: How do I override the Stoobly agent base image?

**A:** Set the `STOOBLY_IMAGE` environment variable before building. It's passed as a build arg to `.Dockerfile.context`, which defaults to `stoobly/agent:2.5`.

**Example:**

```bash
export STOOBLY_IMAGE=stoobly/agent:2.5.1
make -f .stoobly/services/.Makefile record
```

***

### Docker Migration

#### Q: How do I migrate from local to Docker runtime?

**A:** Recreate the app with Docker runtime and start using Makefile commands.

**Example:**

```bash
# Currently using local (default)
# Recreate with Docker runtime
stoobly-agent scaffold app create my-app --runtime docker

# Now use Docker runtime commands
make -f .stoobly/services/.Makefile record
```

***

### Getting Started with Make Commands

#### Q: Where is the Makefile located after scaffolding?

**A:** The Makefile is automatically created at `.stoobly/services/.Makefile` in your application directory after scaffolding with `--runtime docker`.

**Example:**

```bash
cd /path/to/your-app
ls .stoobly/services/.Makefile
```

#### Q: How do I run make commands from my project root?

**A:** Use the `-f` flag to specify the Makefile path, or create a symlink in your project root.

**Example:**

```bash
# Option 1: Specify the Makefile path
make -f .stoobly/services/.Makefile record

# Option 2: Create a symlink (one-time setup)
ln -s .stoobly/services/.Makefile Makefile
make record
```

***

### Recording Workflow

#### Q: How do I start a recording workflow?

**A:** Use `make record` to start the recording workflow, which sets up the proxy to capture HTTP requests. Use `make intercept/enable` to enable intercept to start recording.

**Example:**

```bash
make -f .stoobly/services/.Makefile record
make -f .stoobly/services/.Makefile intercept/enable
```

#### Q: How do I stop a recording workflow?

**A:** Use `make record/down` to stop and clean up the recording workflow.

**Example:**

```bash
make -f .stoobly/services/.Makefile record/down
```

#### Q: How do I view logs from the recording workflow?

**A:** Use `make record/logs` to display logs from all services in the recording workflow.

**Example:**

```bash
make -f .stoobly/services/.Makefile record/logs
```

#### Q: How do I view intercepted request logs from the recording workflow?

**A:** Use `make record/request/logs` to list intercepted request log entries. Pass filter or follow flags via the `options` variable.

**Examples:**

```bash
# List intercepted request log entries
make -f .stoobly/services/.Makefile record/request/logs

# Follow logs in real time (-f / --follow, Ctrl-C to stop)
make -f .stoobly/services/.Makefile record/request/logs options="--follow"

# Filter by HTTP method
make -f .stoobly/services/.Makefile record/request/logs options="--method post"

# Filter by URL substring
make -f .stoobly/services/.Makefile record/request/logs options="--url /api/users"

# Filter by HTTP status code
make -f .stoobly/services/.Makefile record/request/logs options="--status-code 500"

# Filter by log level
make -f .stoobly/services/.Makefile record/request/logs options="--level error"

# Output as JSON
make -f .stoobly/services/.Makefile record/request/logs options="--format json"

# Combine filters with follow
make -f .stoobly/services/.Makefile record/request/logs options="--follow --level error --method post"

# Print the log file path
make -f .stoobly/services/.Makefile record/request/logs/path

# Clear the log
make -f .stoobly/services/.Makefile record/request/logs/delete
```

#### Q: How do I list services in the recording workflow?

**A:** Use `make record/services` to display all configured services for the recording workflow.

**Example:**

```bash
make -f .stoobly/services/.Makefile record/services
```

#### Q: How do I view the recorded requests report?

**A:** Use `make record/report` to display a list of all intercepted and recorded requests.

**Example:**

```bash
make -f .stoobly/services/.Makefile record/report
```

***

### Mock Workflow

#### Q: How do I start a mock workflow?

**A:** Use `make mock` to start the mock workflow, which serves mocked responses based on recorded data. Use `make intercept/enable` to enable intercept to start mocking.

**Example:**

```bash
make -f .stoobly/services/.Makefile mock
make -f .stoobly/services/.Makefile intercept/enable
```

#### Q: How do I stop a mock workflow?

**A:** Use `make mock/down` to stop and clean up the mock workflow.

**Example:**

```bash
make -f .stoobly/services/.Makefile mock/down
```

#### Q: How do I view logs from the mock workflow?

**A:** Use `make mock/logs` to display logs from all services in the mock workflow.

**Example:**

```bash
make -f .stoobly/services/.Makefile mock/logs
```

#### Q: How do I view intercepted request logs from the mock workflow?

**A:** Use `make mock/request/logs` to list intercepted request log entries. Pass filter or follow flags via the `options` variable.

**Examples:**

```bash
# List intercepted request log entries
make -f .stoobly/services/.Makefile mock/request/logs

# Follow logs in real time (-f / --follow, Ctrl-C to stop)
make -f .stoobly/services/.Makefile mock/request/logs options="--follow"

# Filter by HTTP method
make -f .stoobly/services/.Makefile mock/request/logs options="--method post"

# Filter by URL substring
make -f .stoobly/services/.Makefile mock/request/logs options="--url /api/users"

# Filter by HTTP status code
make -f .stoobly/services/.Makefile mock/request/logs options="--status-code 500"

# Filter by log level
make -f .stoobly/services/.Makefile mock/request/logs options="--level error"

# Filter by log message
make -f .stoobly/services/.Makefile mock/request/logs options="--message \"Mock failure\""

# Output as JSON
make -f .stoobly/services/.Makefile mock/request/logs options="--format json"

# Combine filters with follow
make -f .stoobly/services/.Makefile mock/request/logs options="--follow --level error --method post"

# Print the log file path
make -f .stoobly/services/.Makefile mock/request/logs/path

# Clear the log
make -f .stoobly/services/.Makefile mock/request/logs/delete
```

#### Q: How do I list services in the mock workflow?

**A:** Use `make mock/services` to display all configured services for the mock workflow.

**Example:**

```bash
make -f .stoobly/services/.Makefile mock/services
```

#### Q: How do I view the mock requests report?

**A:** Use `make mock/report` to display a list of all mocked requests.

**Example:**

```bash
make -f .stoobly/services/.Makefile mock/report
```

***

### Test Workflow

#### Q: How do I start a test workflow?

**A:** Use `make test` to start the test workflow, which runs automated tests against your services. Use `make intercept/enable` to enable intercept to start testing.

**Example:**

```bash
make -f .stoobly/services/.Makefile test
make -f .stoobly/services/.Makefile intercept/enable
```

#### Q: How do I stop a test workflow?

**A:** Use `make test/down` to stop and clean up the test workflow.

**Example:**

```bash
make -f .stoobly/services/.Makefile test/down
```

#### Q: How do I view logs from the test workflow?

**A:** Use `make test/logs` to display logs from all services in the test workflow.

**Example:**

```bash
make -f .stoobly/services/.Makefile test/logs
```

#### Q: How do I view intercepted request logs from the test workflow?

**A:** Use `make test/request/logs` to list intercepted request log entries. Pass filter or follow flags via the `options` variable.

**Examples:**

```bash
# List intercepted request log entries
make -f .stoobly/services/.Makefile test/request/logs

# Follow logs in real time (-f / --follow, Ctrl-C to stop)
make -f .stoobly/services/.Makefile test/request/logs options="--follow"

# Filter by HTTP method
make -f .stoobly/services/.Makefile test/request/logs options="--method post"

# Filter by URL substring
make -f .stoobly/services/.Makefile test/request/logs options="--url /api/users"

# Filter by HTTP status code
make -f .stoobly/services/.Makefile test/request/logs options="--status-code 500"

# Filter by log level
make -f .stoobly/services/.Makefile test/request/logs options="--level error"

# Filter by test title
make -f .stoobly/services/.Makefile test/request/logs options="--test-title \"my test\""

# Output as JSON
make -f .stoobly/services/.Makefile test/request/logs options="--format json"

# Combine filters with follow
make -f .stoobly/services/.Makefile test/request/logs options="--follow --level error --method post"

# Print the log file path
make -f .stoobly/services/.Makefile test/request/logs/path

# Clear the log
make -f .stoobly/services/.Makefile test/request/logs/delete
```

#### Q: How do I list services in the test workflow?

**A:** Use `make test/services` to display all configured services for the test workflow.

**Example:**

```bash
make -f .stoobly/services/.Makefile test/services
```

#### Q: How do I view the test requests report?

**A:** Use `make test/report` to display a list of all test requests and results.

**Example:**

```bash
make -f .stoobly/services/.Makefile test/report
```

***

### Scenario Management

#### Q: How do I create a scenario using make?

**A:** Use `make scenario/create` with the `name` variable to create a new scenario.

**Example:**

```bash
make -f .stoobly/services/.Makefile scenario/create name="User Login Flow"
```

#### Q: How do I list all scenarios using make?

**A:** Use `make scenario/list` to display all available scenarios.

**Example:**

```bash
make -f .stoobly/services/.Makefile scenario/list
```

#### Q: How do I delete a scenario using make?

**A:** Use `make scenario/delete` with the `key` variable to delete a specific scenario.

**Example:**

```bash
make -f .stoobly/services/.Makefile scenario/delete key=<SCENARIO-KEY>
```

#### Q: How do I snapshot a scenario using make?

**A:** Use `make scenario/snapshot` with the `key` variable to create committable files for a scenario.

**Example:**

```bash
make -f .stoobly/services/.Makefile scenario/snapshot key=<SCENARIO-KEY>
```

#### Q: How do I reset a scenario to its snapshot state?

**A:** Use `make scenario/reset` with the `key` variable to restore a scenario from its snapshot.

**Example:**

```bash
make -f .stoobly/services/.Makefile scenario/reset key=<SCENARIO-KEY>
```

#### Q: How do I overwrite a scenario using make?

**A:** Use `make scenario/overwrite` with the `key` variable to overwrite an existing scenario.

**Example:**

```bash
make -f .stoobly/services/.Makefile scenario/overwrite key=<SCENARIO-KEY>
```

***

### Certificate Management

#### Q: How do I install the CA certificate using make?

**A:** The CA certificate is automatically installed when you start a workflow, but you can manually trigger it with `make ca-cert/install`.

**Example:**

```bash
make -f .stoobly/services/.Makefile ca-cert/install
```

#### Q: How do I skip the CA certificate installation prompt?

**A:** Set the `STOOBLY_CA_CERTS_INSTALL_CONFIRM` environment variable to `y` before running the workflow.

**Example:**

```bash
export STOOBLY_CA_CERTS_INSTALL_CONFIRM=y
make -f .stoobly/services/.Makefile record
```

***

### Hostname Management

#### Q: How do I skip the hostname installation prompt?

**A:** Set the `STOOBLY_HOSTNAME_INSTALL_CONFIRM` environment variable to `y` to automatically confirm hostname installation.

**Example:**

```bash
export STOOBLY_HOSTNAME_INSTALL_CONFIRM=y
make -f .stoobly/services/.Makefile record
```

#### Q: How do I prevent hostname installation?

**A:** Set the `STOOBLY_HOSTNAME_INSTALL_CONFIRM` environment variable to `n` to skip hostname installation.

**Example:**

```bash
export STOOBLY_HOSTNAME_INSTALL_CONFIRM=n
make -f .stoobly/services/.Makefile mock
```

***

### Intercept Management

#### Q: How do I enable intercept mode using make?

**A:** Use `make intercept/enable` with an optional `scenario_key` to enable request interception.

**Example:**

```bash
make -f .stoobly/services/.Makefile intercept/enable scenario_key=<SCENARIO-KEY>
```

#### Q: How do I disable intercept mode using make?

**A:** Use `make intercept/disable` to turn off request interception.

**Example:**

```bash
make -f .stoobly/services/.Makefile intercept/disable
```

***

### Environment Variables & Configuration

#### Q: How do I specify a custom application directory?

**A:** Set the `STOOBLY_APP_DIR` environment variable to point to your application directory.

**Example:**

```bash
export STOOBLY_APP_DIR=/path/to/my-app
make -f .stoobly/services/.Makefile record
```

#### Q: How do I use a custom .env file with workflows?

**A:** Set the `STOOBLY_DOTENV_FILE` environment variable to specify your .env file path.

**Example:**

```bash
export STOOBLY_DOTENV_FILE=/path/to/custom.env
make -f .stoobly/services/.Makefile mock
```

This file takes priority over variables set with `scaffold app create --env`. See [Where do scaffold environment variables come from?](../README.md#environment-variables) for the full order.

#### Q: How do I specify custom CA certificate directory?

**A:** Set the `STOOBLY_CA_CERTS_DIR` environment variable to your custom certificate directory.

**Example:**

```bash
export STOOBLY_CA_CERTS_DIR=/path/to/certs
make -f .stoobly/services/.Makefile record
```

#### Q: How do I pass additional service options to workflows?

**A:** Set the `STOOBLY_WORKFLOW_SERVICE_OPTIONS` environment variable with your custom options.

**Example:**

```bash
export STOOBLY_WORKFLOW_SERVICE_OPTIONS="--service api --service database"
make -f .stoobly/services/.Makefile record
```

#### Q: How do I specify a custom context directory?

**A:** Set the `STOOBLY_CONTEXT_DIR` environment variable to your desired context directory.

**Example:**

```bash
export STOOBLY_CONTEXT_DIR=/path/to/context
make -f .stoobly/services/.Makefile test
```

***

### Advanced Workflow Options

#### Q: How do I pass additional options to workflow up commands?

**A:** Use the `options` variable to pass extra options to the workflow.

**Example:**

```bash
make -f .stoobly/services/.Makefile record options="--detach"
```

#### Q: How do I run a workflow with a custom namespace?

**A:** Use the `namespace` variable to specify a custom workflow namespace.

**Note:** Namespaces are only supported by test workflows and custom workflows based on the test workflow template. Record and mock workflows do not support namespaces.

**Example:**

```bash
make -f .stoobly/services/.Makefile test namespace=my-custom-namespace
```

#### Q: How do I filter services when starting a workflow?

**A:** Set the `STOOBLY_WORKFLOW_SERVICE_OPTIONS` environment variable to specify which services to include.

**Example:**

```bash
export STOOBLY_WORKFLOW_SERVICE_OPTIONS="--service api --service frontend"
make -f .stoobly/services/.Makefile mock
```

***

### Combining Multiple Environment Variables

#### Q: How do I run a fully automated workflow without any prompts?

**A:** Set both hostname and CA certificate confirmation environment variables before running the workflow.

**Example:**

```bash
export STOOBLY_HOSTNAME_INSTALL_CONFIRM=y
export STOOBLY_CA_CERTS_INSTALL_CONFIRM=y
make -f .stoobly/services/.Makefile record
```

#### Q: How do I run a workflow with custom directories and environment?

**A:** Set multiple environment variables to customize all aspects of the workflow.

**Example:**

```bash
export STOOBLY_APP_DIR=/path/to/app
export STOOBLY_DOTENV_FILE=/path/to/.env.production
export STOOBLY_WORKFLOW_SERVICE_OPTIONS="--service api"
make -f .stoobly/services/.Makefile mock
```

***

### Docker Runtime Troubleshooting

#### Q: How do I check if stoobly-agent is installed?

**A:** The Makefile automatically checks and installs stoobly-agent via pipx if needed. You can manually trigger this with `make stoobly/install`.

**Example:**

```bash
make -f .stoobly/services/.Makefile stoobly/install
```

#### Q: How do I verify Python version compatibility?

**A:** Use `make python/validate` to check if you have a compatible Python version (3.12, 3.13, or 3.14).

**Example:**

```bash
make -f .stoobly/services/.Makefile python/validate
```

#### Q: What do I do if pipx is not installed?

**A:** The Makefile automatically installs pipx if it's missing. You can manually trigger this with `make pipx/install`.

**Example:**

```bash
make -f .stoobly/services/.Makefile pipx/install
```

#### Q: How do I view what services are configured?

**A:** Use the workflow-specific services command to list all configured services.

**Example:**

```bash
make -f .stoobly/services/.Makefile record/services
make -f .stoobly/services/.Makefile mock/services
make -f .stoobly/services/.Makefile test/services
```

#### Q: How do I validate my service configuration?

**A:** Use `scaffold service show` to view the current configuration.

**Example:**

```bash
stoobly-agent scaffold service show api
```

#### Q: What do I do if a workflow fails to start?

**A:** Check the logs for errors and verify service configurations.

**Example:**

```bash
# View logs
make -f .stoobly/services/.Makefile record/logs

# Verify services
make -f .stoobly/services/.Makefile record/services

# Check Docker containers
docker ps
```

***

### Docker Quick Reference

#### Q: What are the most common make commands I'll use?

**A:** Here's a quick reference of the most frequently used commands:

**Example:**

```bash
# Start workflows (then enable intercept)
make -f .stoobly/services/.Makefile record    # Start recording
make -f .stoobly/services/.Makefile intercept/enable

make -f .stoobly/services/.Makefile mock      # Start mocking
make -f .stoobly/services/.Makefile intercept/enable

make -f .stoobly/services/.Makefile test      # Start testing
make -f .stoobly/services/.Makefile intercept/enable

# Stop workflows
make -f .stoobly/services/.Makefile record/down
make -f .stoobly/services/.Makefile mock/down
make -f .stoobly/services/.Makefile test/down

# View logs
make -f .stoobly/services/.Makefile record/logs
make -f .stoobly/services/.Makefile mock/logs
make -f .stoobly/services/.Makefile test/logs

# Manage scenarios
make -f .stoobly/services/.Makefile scenario/list
make -f .stoobly/services/.Makefile scenario/create name="My Scenario"
make -f .stoobly/services/.Makefile scenario/snapshot key=<SCENARIO-KEY>

# View reports
make -f .stoobly/services/.Makefile record/report
make -f .stoobly/services/.Makefile mock/report
```

#### Q: How do I create an alias for easier make command usage?

**A:** Add an alias to your shell configuration file for convenience.

**Example:**

```bash
# Add to ~/.bashrc or ~/.zshrc
alias smake='make -f .stoobly/services/.Makefile'

# Then use it like:
smake record
smake mock/down
smake scenario/list
```
