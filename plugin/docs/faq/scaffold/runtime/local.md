# Local Runtime

## Stoobly Scaffold Local Runtime - Questions & Answers

This document covers local runtime-specific options for Stoobly scaffold. For general runtime topics, see [README.md](README.md). For Docker runtime, see [docker.md](docker.md).

***

### Local Runtime Overview

#### Q: What are the benefits of using local runtime?

**A:** Local runtime offers faster startup times, simpler debugging, direct access to your filesystem, and no Docker dependency.

**Example:**

```bash
stoobly-agent scaffold app create my-app --runtime local

# Faster startup, no container overhead
stoobly-agent scaffold workflow up test
```

#### Q: What are the requirements for local runtime?

**A:** You need Python 3.12, 3.13, or 3.14 installed, and stoobly-agent installed via pipx.

**Example:**

```bash
# Verify Python version
python3 --version  # Should be 3.12, 3.13, or 3.14

# Install stoobly-agent
pipx install stoobly-agent

# Create local runtime scaffold
stoobly-agent scaffold app create my-app --runtime local
```

#### Q: How do local workflows work?

**A:** Local workflows run stoobly-agent directly on your machine, proxying requests to your services without containerization.

**Example:**

```bash
# Create local runtime app
stoobly-agent scaffold app create my-app --runtime local

# Create service
stoobly-agent scaffold service create my-service

# Start workflow (runs locally)
stoobly-agent scaffold workflow up test
```

#### Q: Can I use local runtime if my services are in Docker?

**A:** Yes, local runtime only affects how stoobly-agent runs. Your services can still run in Docker containers.

**Example:**

```bash
# Stoobly runs locally, services in Docker
stoobly-agent scaffold app create my-app --runtime local

# Add service that runs in Docker
stoobly-agent scaffold service create api \
  --hostname api.local \
  --upstream-hostname localhost \
  --upstream-port 8080

# Start your service in Docker
docker run -p 8080:8080 my-api-image

# Start stoobly workflow locally
stoobly-agent scaffold workflow up test
```

#### Q: How do I troubleshoot local runtime issues?

**A:** Check Python version, verify stoobly-agent installation, and review logs.

**Example:**

```bash
# Check Python version
python3 --version

# Verify stoobly-agent
stoobly-agent --version

# Check logs with verbose output
stoobly-agent scaffold workflow up test \
  --log-level debug
```

***

### Local Workflows

#### Q: How do I explicitly use local runtime for a workflow?

**A:** Use the `stoobly-agent scaffold workflow` commands directly.

**Example:**

```bash
# Local runtime via CLI
stoobly-agent scaffold workflow up record
stoobly-agent scaffold workflow up mock
stoobly-agent scaffold workflow up test
```

#### Q: How do I use local runtime?

**A:** Create the app with `--runtime local` (or use default) and use CLI commands for workflows.

**Example:**

```bash
# App created with local runtime (default)
stoobly-agent scaffold app create my-app
stoobly-agent scaffold service create my-service
```

***

### Local Migration

#### Q: How do I migrate from Docker to local runtime?

**A:** Recreate the app with local runtime and use CLI commands instead of Makefile.

**Example:**

```bash
# Currently using Docker
# Recreate with local runtime
stoobly-agent scaffold app create my-app --runtime local

# Now use local runtime commands
stoobly-agent scaffold service create my-service
```

***

### Getting Started with Local Runtime Commands

#### Q: How do I run workflow commands with local runtime?

**A:** Use `stoobly-agent scaffold workflow` commands directly. No Makefile is needed for local runtime.

**Example:**

```bash
# Start a workflow
stoobly-agent scaffold workflow up test

# Stop a workflow
stoobly-agent scaffold workflow down test
```

***

### Recording Workflow

#### Q: How do I start a recording workflow with local runtime?

**A:** Use `stoobly-agent scaffold workflow up record` to start the recording workflow, which sets up the proxy to capture HTTP requests. Use `stoobly-agent intercept enable` to enable intercept to start recording.

**Example:**

```bash
stoobly-agent scaffold workflow up record
stoobly-agent intercept enable
```

#### Q: How do I stop a recording workflow with local runtime?

**A:** Use `stoobly-agent scaffold workflow down record` to stop and clean up the recording workflow.

**Example:**

```bash
stoobly-agent scaffold workflow down record
```

#### Q: How do I view logs from the recording workflow with local runtime?

**A:** Use the scaffold log commands to see what requests were intercepted and the raw workflow process output.

**Example:**

```bash
# Show intercepted request logs (mock/record status, response codes)
stoobly-agent scaffold request logs list record

# Show raw workflow process output (startup errors, config issues)
stoobly-agent scaffold workflow logs record

# List all recorded requests
stoobly-agent request list
```

#### Q: How do I list services in the recording workflow with local runtime?

**A:** Use `stoobly-agent scaffold workflow services` to display all configured services.

**Example:**

```bash
stoobly-agent scaffold workflow services record
```

#### Q: How do I view the recorded requests report with local runtime?

**A:** Use `stoobly-agent request list` to display all intercepted and recorded requests.

**Example:**

```bash
stoobly-agent request list
```

***

### Mock Workflow

#### Q: How do I start a mock workflow with local runtime?

**A:** Use `stoobly-agent scaffold workflow up mock` to start the mock workflow, which serves mocked responses based on recorded data. Use `stoobly-agent intercept enable` to enable intercept to start mocking.

**Example:**

```bash
stoobly-agent scaffold workflow up mock
stoobly-agent intercept enable
```

#### Q: How do I stop a mock workflow with local runtime?

**A:** Use `stoobly-agent scaffold workflow down mock` to stop and clean up the mock workflow.

**Example:**

```bash
stoobly-agent scaffold workflow down mock
```

#### Q: How do I view logs from the mock workflow with local runtime?

**A:** Use the scaffold log commands to verify requests are being served from recordings and diagnose any mock failures.

**Example:**

```bash
# Show intercepted request logs (confirms whether responses are mocked or passed through)
stoobly-agent scaffold request logs list mock

# Show raw workflow process output (startup errors, config issues)
stoobly-agent scaffold workflow logs mock

# List all recorded requests
stoobly-agent request list
```

#### Q: How do I list services in the mock workflow with local runtime?

**A:** Use `stoobly-agent scaffold workflow services` to display all configured services.

**Example:**

```bash
stoobly-agent scaffold workflow services mock
```

#### Q: How do I view the mock requests report with local runtime?

**A:** Use `stoobly-agent request list` to display a list of all mocked requests.

**Example:**

```bash
stoobly-agent request list
```

***

### Test Workflow

#### Q: How do I start a test workflow with local runtime?

**A:** Use `stoobly-agent scaffold workflow up test` to start the test workflow, which runs automated tests against your services. Use `stoobly-agent intercept enable` to enable intercept to start testing.

**Example:**

```bash
stoobly-agent scaffold workflow up test
stoobly-agent intercept enable
```

#### Q: How do I stop a test workflow with local runtime?

**A:** Use `stoobly-agent scaffold workflow down test` to stop and clean up the test workflow.

**Example:**

```bash
stoobly-agent scaffold workflow down test
```

#### Q: How do I view logs from the test workflow with local runtime?

**A:** Use the scaffold log commands to see what requests were tested and their results.

**Example:**

```bash
# Show intercepted request logs (test results, response codes)
stoobly-agent scaffold request logs list test

# Show raw workflow process output (startup errors, config issues)
stoobly-agent scaffold workflow logs test

# List all test requests
stoobly-agent request list
```

#### Q: How do I list services in the test workflow with local runtime?

**A:** Use `stoobly-agent scaffold workflow services` to display all configured services.

**Example:**

```bash
stoobly-agent scaffold workflow services test
```

#### Q: How do I view the test requests report with local runtime?

**A:** Use `stoobly-agent request list` to display a list of all test requests and results.

**Example:**

```bash
stoobly-agent request list
```

***

### Scenario Management

#### Q: How do I create a scenario with local runtime?

**A:** Use `stoobly-agent scenario create` to create a new scenario.

**Example:**

```bash
stoobly-agent scenario create "User Login Flow"
```

#### Q: How do I list all scenarios with local runtime?

**A:** Use `stoobly-agent scenario list` to display all available scenarios.

**Example:**

```bash
stoobly-agent scenario list
```

#### Q: How do I delete a scenario with local runtime?

**A:** Use `stoobly-agent scenario delete` to delete a specific scenario.

**Example:**

```bash
stoobly-agent scenario delete <SCENARIO-KEY>
```

#### Q: How do I snapshot a scenario with local runtime?

**A:** Use `stoobly-agent scenario snapshot` to create committable files for a scenario.

**Example:**

```bash
stoobly-agent scenario snapshot <SCENARIO-KEY>
```

#### Q: How do I reset a scenario to its snapshot state with local runtime?

**A:** Use `stoobly-agent scenario reset` to restore a scenario from its snapshot.

**Example:**

```bash
stoobly-agent scenario reset <SCENARIO-KEY>
```

#### Q: How do I overwrite a scenario with local runtime?

**A:** Use `stoobly-agent scenario overwrite` to overwrite an existing scenario.

**Example:**

```bash
stoobly-agent scenario overwrite <SCENARIO-KEY>
```

***

### Certificate Management

#### Q: How do I install the CA certificate with local runtime?

**A:** Use `stoobly-agent ca-cert install` to install the CA certificate.

**Example:**

```bash
stoobly-agent ca-cert install
```

#### Q: How do I skip the CA certificate installation prompt with local runtime?

**A:** Set the `STOOBLY_CA_CERTS_INSTALL_CONFIRM` environment variable to `y` before running the workflow.

**Example:**

```bash
export STOOBLY_CA_CERTS_INSTALL_CONFIRM=y
stoobly-agent scaffold workflow up test
```

***

### Hostname Management

#### Q: How do I skip the hostname installation prompt with local runtime?

**A:** Set the `STOOBLY_HOSTNAME_INSTALL_CONFIRM` environment variable to `y` to automatically confirm hostname installation.

**Example:**

```bash
export STOOBLY_HOSTNAME_INSTALL_CONFIRM=y
stoobly-agent scaffold workflow up test
```

#### Q: How do I prevent hostname installation with local runtime?

**A:** Set the `STOOBLY_HOSTNAME_INSTALL_CONFIRM` environment variable to `n` to skip hostname installation.

**Example:**

```bash
export STOOBLY_HOSTNAME_INSTALL_CONFIRM=n
stoobly-agent scaffold workflow up mock
```

***

### Intercept Management

#### Q: How do I enable intercept mode with local runtime?

**A:** Use `stoobly-agent intercept enable` with an optional scenario key to enable request interception.

**Example:**

```bash
stoobly-agent intercept enable --scenario-key "<SCENARIO-KEY>"
```

#### Q: How do I disable intercept mode with local runtime?

**A:** Use `stoobly-agent intercept disable` to turn off request interception.

**Example:**

```bash
stoobly-agent intercept disable
```

***

### Environment Variables & Configuration

#### Q: How do I specify a custom application directory with local runtime?

**A:** Use the `--app-dir-path` option when running workflow commands.

**Example:**

```bash
stoobly-agent scaffold workflow up test --app-dir-path /path/to/my-app
```

#### Q: How do I use a custom .env file with workflows in local runtime?

**A:** Set the `STOOBLY_DOTENV_FILE` environment variable to specify your .env file path.

**Example:**

```bash
export STOOBLY_DOTENV_FILE=/path/to/custom.env
stoobly-agent scaffold workflow up mock
```

This file takes priority over variables set with `scaffold app create --env`. See [Where do scaffold environment variables come from?](../README.md#environment-variables) for the full order.

#### Q: How do I specify custom CA certificate directory with local runtime?

**A:** Use the `--ca-certs-dir-path` option when running workflow commands.

**Example:**

```bash
stoobly-agent scaffold workflow up test --ca-certs-dir-path /path/to/certs
```

#### Q: How do I pass additional service options to workflows with local runtime?

**A:** Use the `--service` option multiple times or pass options directly to the workflow command.

**Example:**

```bash
stoobly-agent scaffold workflow up test --service api --service database
```

#### Q: How do I specify a custom context directory with local runtime?

**A:** Use the `--context-dir-path` option when running workflow commands.

**Example:**

```bash
stoobly-agent scaffold workflow up test --context-dir-path /path/to/context
```

***

### Advanced Workflow Options

#### Q: How do I pass additional options to workflow up commands with local runtime?

**A:** Add options directly to the `stoobly-agent scaffold workflow up` command.

**Example:**

```bash
stoobly-agent scaffold workflow up test --detach
```

#### Q: How do I run a workflow with a custom namespace with local runtime?

**A:** Use the `--namespace` option to specify a custom workflow namespace.

**Note:** Namespaces are only supported by test workflows and custom workflows based on the test workflow template. Record and mock workflows do not support namespaces.

**Example:**

```bash
stoobly-agent scaffold workflow up test --namespace my-custom-namespace
```

#### Q: How do I filter services when starting a workflow with local runtime?

**A:** Use the `--service` option to specify which services to include.

**Example:**

```bash
stoobly-agent scaffold workflow up mock --service api --service frontend
```

***

### Combining Multiple Options

#### Q: How do I run a fully automated workflow without any prompts with local runtime?

**A:** Set both hostname and CA certificate confirmation environment variables before running the workflow.

**Example:**

```bash
export STOOBLY_HOSTNAME_INSTALL_CONFIRM=y
export STOOBLY_CA_CERTS_INSTALL_CONFIRM=y
stoobly-agent scaffold workflow up test
```

#### Q: How do I run a workflow with custom directories and environment with local runtime?

**A:** Use command-line options and environment variables together to customize all aspects of the workflow.

**Example:**

```bash
export STOOBLY_DOTENV_FILE=/path/to/.env.production
stoobly-agent scaffold workflow up mock \
  --app-dir-path /path/to/app \
  --service api
```

***

### Local Runtime Troubleshooting

#### Q: How do I check if stoobly-agent is installed?

**A:** Run `stoobly-agent --version` to verify installation.

**Example:**

```bash
stoobly-agent --version
```

#### Q: How do I verify Python version compatibility?

**A:** Run `python3 --version` to check if you have Python 3.12, 3.13, or 3.14.

**Example:**

```bash
python3 --version  # Should be 3.12, 3.13, or 3.14
```

#### Q: What do I do if pipx is not installed?

**A:** Install pipx using your system package manager or Python pip.

**Example:**

```bash
# On macOS
brew install pipx

# On Linux
python3 -m pip install --user pipx
python3 -m pipx ensurepath
```

#### Q: How do I view what services are configured with local runtime?

**A:** Use the workflow-specific services command to list all configured services.

**Example:**

```bash
stoobly-agent scaffold workflow services record
stoobly-agent scaffold workflow services mock
stoobly-agent scaffold workflow services test
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
stoobly-agent scaffold workflow logs record

# Verify services
stoobly-agent scaffold service list
```

***

### Quick Reference

#### Q: What are the most common local runtime commands I'll use?

**A:** Here's a quick reference of the most frequently used commands:

**Example:**

```bash
# Start workflows (then enable intercept)
stoobly-agent scaffold workflow up record    # Start recording
stoobly-agent intercept enable

stoobly-agent scaffold workflow up mock      # Start mocking
stoobly-agent intercept enable

stoobly-agent scaffold workflow up test      # Start testing
stoobly-agent intercept enable

# Disable intercept
stoobly-agent intercept disable

# Stop workflows
stoobly-agent scaffold workflow down record
stoobly-agent scaffold workflow down mock
stoobly-agent scaffold workflow down test

# Manage scenarios
stoobly-agent scenario list
stoobly-agent scenario create "My Scenario"
stoobly-agent scenario snapshot <SCENARIO-KEY>

# View requests
stoobly-agent request list

# View logs (crucial for debugging and verifying mock/record behavior)
stoobly-agent scaffold request logs list record   # what was intercepted during record
stoobly-agent scaffold request logs list mock     # confirms responses served from recordings
stoobly-agent scaffold request logs list test     # test results per request
stoobly-agent scaffold workflow logs mock         # raw workflow process output
```

#### Q: How do I create an alias for easier command usage with local runtime?

**A:** Add an alias to your shell configuration file for convenience.

**Example:**

```bash
# Add to ~/.bashrc or ~/.zshrc
alias swf='stoobly-agent scaffold workflow'

# Then use it like:
swf up record
swf down mock
swf services test
```
