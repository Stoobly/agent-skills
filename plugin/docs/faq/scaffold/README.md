# Scaffold

## Stoobly Scaffold Commands - Questions & Answers for Team Use

Scaffold commands enable Stoobly for team collaboration. The workflow is: create an app, add services, and use the automatically generated record, mock, test, and develop workflows for each service. Custom workflows can be created as needed. **Recommended:** describe your app and services in a declarative `.stoobly/scaffold.yml` and apply it with `stoobly-agent scaffold apply` — the individual `scaffold app create` / `scaffold service create` commands are still available and useful for one-off changes.

***

### Getting Started with Scaffold

#### Q: What's the recommended way to set up a scaffold?

**A:** Create a declarative config at `.stoobly/scaffold.yml` with your app and service creation steps, then apply it with `stoobly-agent scaffold apply`. This keeps the whole setup reviewable, version-controlled, and reproducible by teammates with one command. For the full command reference and config schema, see:

{% content-ref url="apply.md" %}
[apply.md](apply.md)
{% endcontent-ref %}

**Example:**

```yaml
# .stoobly/scaffold.yml
version: 1
commands:
  # Creates .stoobly/services/ in the current directory
  # For E2E testing, add plugin: [playwright] or plugin: [cypress]
  - resource: app
    action: create
    options:
      app_name: my-app
      plugin: [playwright]

  # API service runs on localhost:8080, exposed as api.local:80
  # Note: port and upstream_port must differ when using local: true
  # Run your API service on port 8080 (not 80)
  - resource: service
    action: create
    options:
      service_name: api
      hostname: api.local
      port: 80
      scheme: http
      local: true
      upstream_port: 8080

  # Example: Start record workflow
  # - resource: workflow
  #   action: up
  #   options:
  #     workflow_name: record
  #     ca_certs_install_confirm: y
  #
  # stoobly-agent intercept enable

  # Example: Start mock workflow
  # - resource: workflow
  #   action: up
  #   options:
  #     workflow_name: mock
  #
  # stoobly-agent intercept enable

  # Example: Start test workflow
  # - resource: workflow
  #   action: up
  #   options:
  #     workflow_name: test
  #
  # stoobly-agent intercept enable
```

```bash
# Validate, then apply
stoobly-agent scaffold apply .stoobly/scaffold.yml --dry-run
stoobly-agent scaffold apply .stoobly/scaffold.yml
```

#### Q: What is scaffolding in Stoobly?

**A:** Scaffolding creates a structured project setup for team collaboration, with workflows for recording, mocking, and testing HTTP requests across multiple services. The scaffold structure is created under `.stoobly/services/` in the current directory.

**Example:**

```yaml
# .stoobly/scaffold.yml
version: 1
commands:
  - resource: app
    action: create
    options:
      app_name: my-app

  - resource: service
    action: create
    options:
      service_name: api
```

```bash
stoobly-agent scaffold apply .stoobly/scaffold.yml
```

Or with the individual commands directly:

```bash
# Create a scaffold app (creates .stoobly/services/ in current directory)
stoobly-agent scaffold app create my-app

# Add a service
stoobly-agent scaffold service create api
```

#### Q: Why should I use scaffold for team development?

**A:** Scaffold provides version-controlled configuration, consistent environments across team members, and automated workflow management for recording, mocking, and testing APIs.

**Example:**

```bash
# Team member 1 creates the scaffold from .stoobly/scaffold.yml
stoobly-agent scaffold apply .stoobly/scaffold.yml

# Team member 2 clones and reproduces the exact same setup
git clone <repo>
stoobly-agent scaffold apply .stoobly/scaffold.yml
stoobly-agent scaffold workflow up test 
```

***

### Creating an Application

#### Q: How do I create a new scaffold application?

**A:** Use `scaffold app create` with your application name to generate the scaffold structure under `.stoobly/services/` in the current directory. **By default, the app is created with local runtime** (workflows run directly on your machine). To use Docker runtime instead, specify `--runtime docker`.

**Example:**

```bash
# Creates app with local runtime (default)
stoobly-agent scaffold app create my-app
# Creates .stoobly/services/ structure in current directory

# Or explicitly specify local runtime
stoobly-agent scaffold app create my-app --runtime local

# Or use Docker runtime
stoobly-agent scaffold app create my-app --runtime docker
```

#### Q: How do I create a scaffold app in a specific directory?

**A:** By default, the scaffold is created in the current directory. Use the `--app-dir-path` option only when you need to specify a different location.

**Example:**

```bash
# Create in current directory (default)
stoobly-agent scaffold app create my-app

# Create in specific directory (optional)
stoobly-agent scaffold app create my-app --app-dir-path /path/to/projects
```

#### Q: How do I create a scaffold app with custom ports?

**A:** Use `--proxy-port` and `--ui-port` options to specify custom ports for the proxy and UI.

**Example:**

```bash
stoobly-agent scaffold app create my-app --proxy-port 9090 --ui-port 5000
```

#### Q: How do I set environment variables for my scaffold app?

**A:** Use `--env NAME=VALUE` (repeatable) on `scaffold app create`. *(New in v2.5.2.)* Pairs are written to `.stoobly/services/.env` and merged into every workflow's environment on `workflow up`.

**Example:**

```bash
stoobly-agent scaffold app create my-app \
  --env STOOBLY_IMAGE=stoobly/agent:2.5.2 \
  --env TRAEFIK_IMAGE=traefik:v3.1
```

`NAME` must match `[A-Za-z_][A-Za-z0-9_]*` and `VALUE` must not contain newlines; anything else exits non-zero with a message naming the offending pair. Only the first `=` splits, so `--env URL=postgres://u:p@h/db?a=b` is one pair.

### Environment Variables

#### Q: Where do scaffold environment variables come from?

**A:** Three `.env` files, merged on `workflow up`:

| File | Path | Set by |
|---|---|---|
| App dotenv | `.stoobly/services/.env` | `scaffold app create --env NAME=VALUE` |
| Project dotenv | `./.env`, or the path in `STOOBLY_DOTENV_FILE` | you — copied into the workflow namespace on each run |
| Workflow dotenv | `.stoobly/services/<service>/<workflow>/.env` | generated on `workflow up`; do not edit by hand |

Inside containers and workflow scripts, highest priority first:

1. Project dotenv (`STOOBLY_DOTENV_FILE`, or cwd `.env`)
2. App dotenv (`app create --env`)
3. Scaffold-generated variables (`APP_DIR`, `CONTEXT_DIR`, `WORKFLOW_NAME`, `SERVICE_DNS`, …)

On the **host**, when the Stoobly base image is built, only two sources participate and the order flips toward the shell: an exported shell variable wins over the app dotenv. That is why `export STOOBLY_IMAGE=…` overrides an `--env STOOBLY_IMAGE=…` baked in at app-create time.

#### Q: How do I choose the proxy mode for my scaffold app?

**A:** Use the `--proxy-mode` option to select how Stoobly proxies requests for your app. The default is `forward`, which exposes a single proxy endpoint that your clients/tests connect to (typical forward-proxy setup). The `reverse` mode is only supported with Docker runtime and is recommended when you want to route traffic via hostnames (e.g. `api.local`) and manage `/etc/hosts` entries.

**Example:**

```bash
# Default forward proxy mode (no option needed)
stoobly-agent scaffold app create my-app

# Explicitly set forward proxy mode
stoobly-agent scaffold app create my-app --proxy-mode forward

# Reverse proxy mode (Docker runtime only)
stoobly-agent scaffold app create my-app \
  --runtime docker \
  --proxy-mode reverse
```

**Important:**

- Using `--proxy-mode reverse` with local runtime is not allowed and will fail with an error.
- When using reverse proxy mode with Docker, Stoobly can manage `/etc/hosts` entries for your service hostnames via `scaffold hostname install` or the hostname prompts during `scaffold workflow up`.

#### Q: What does the copy-on-workflow-up (`--copy-on-workflow-up`) option do?

**A:** The `--copy-on-workflow-up` option enables isolated workflow run behavior. When enabled, Stoobly copies the scaffold files from your app directory (`--app-dir-path`) into the Stoobly context directory on each `scaffold workflow up` run. This is useful when you want workflow runs to operate on an isolated copy of the scaffold (for example in CI or when using workflow namespaces), while keeping your source scaffold under version control in your app directory.

**Example:**

```bash
# Enable copy-on-workflow-up behavior
stoobly-agent scaffold app create my-app --copy-on-workflow-up

# Later, when you start a workflow:
stoobly-agent scaffold workflow up record
# Stoobly copies the scaffold into its context tmp directory before running
```

**Notes:**

- `--proxy-mode-reverse` affects how scaffold files are managed at runtime; it does not change the CLI commands you use.
- This option can reduce coupling between your app directory and the runtime context, especially when multiple workflows or namespaces run concurrently.

#### Q: How do I create a scaffold app with test framework integrations?

**A:** Use the `--plugin` option to include Cypress or Playwright integrations.

**Example:**

```bash
# With Cypress
stoobly-agent scaffold app create my-app --plugin cypress

# With Playwright
stoobly-agent scaffold app create my-app --plugin playwright

# With both
stoobly-agent scaffold app create my-app --plugin cypress --plugin playwright
```

#### Q: What runtime environments are supported for scaffold apps?

**A:** Scaffold supports two runtime environments: **local (default)** and Docker. When you run `scaffold app create` without specifying `--runtime`, it defaults to local runtime, which runs workflows directly on your machine. Use `--runtime docker` to run workflows in Docker containers instead.

**Example:**

```bash
# Local runtime (default - no --runtime option needed)
stoobly-agent scaffold app create my-app

# Or explicitly specify local runtime
stoobly-agent scaffold app create my-app --runtime local

# Docker runtime (must be explicitly specified)
stoobly-agent scaffold app create my-app --runtime docker
```

#### Q: How do I check whether my app is configured with local or Docker runtime?

**A:** Check the `.stoobly/services/.config.yml` file and look for the `APP_RUNTIME` configuration property to see which runtime is configured.

**Example:**

```bash
# View the configuration file
cat .stoobly/services/.config.yml

# Or check for the specific property
grep APP_RUNTIME .stoobly/services/.config.yml
```

The `APP_RUNTIME` property will indicate whether the app is configured to run with `local` or `Docker`.

#### Q: What files are created when I scaffold an app?

**A:** Scaffolding creates a `.stoobly/services/` directory in the current directory with service definitions and workflow templates.

**Example:**

```bash
stoobly-agent scaffold app create my-app
ls -la .stoobly/services/
# build .config.yml entrypoint .gitignore
```

***

### Creating Services

#### Q: How do I add a service to my scaffold app?

**A:** Use `scaffold service create` with the service name to add a new service. By default, commands use the current directory. Use `--app-dir-path` only when you need to specify a different location.

**Example:**

```bash
# From within the app directory (default)
stoobly-agent scaffold service create api

# Or specify the app directory
stoobly-agent scaffold service create api --app-dir-path ./my-app
```

**Note:** If you maintain `.stoobly/scaffold.yml` to recreate your setup from scratch, add a matching `service` step to it too, so `stoobly-agent scaffold apply .stoobly/scaffold.yml` stays reproducible. See [Apply](apply.md) for the config schema.

#### Q: How do I create a service with a custom hostname?

**A:** Use the `--hostname` option to specify a custom hostname for the service.

**Example:**

```bash
# From within the app directory
stoobly-agent scaffold service create api --hostname api.example.com
```

#### Q: How do I create a service that proxies to a service that does not run on localhost?

**A:** Configure the service using `--hostname`, `--port`, and `--scheme` to define how clients connect to Stoobly. Only use `--upstream-hostname`, `--upstream-port`, and `--upstream-scheme` when the upstream server differs from these values.

**Important:**

* The `--upstream-*` options are overrides, not replacements. Always set `--hostname`, `--port`, and `--scheme` first, then add `--upstream-*` options **only when they differ**. If the upstream uses the same port and scheme, do not specify `--upstream-port` or `--upstream-scheme`.

**Example:**

```bash
# Example 1. When upstream differs from service (e.g., proxying production API locally)
# Clients connect to http://api.local:80 → Stoobly forwards to https://api.production.com:443
stoobly-agent scaffold service create api \
  --hostname api.local \
  --port 80 \
  --scheme http \
  --upstream-hostname api.production.com \
  --upstream-port 443 \
  --upstream-scheme https

# Example 2. When upstream matches service hostname and port (no --upstream-* needed)
# Clients connect to http://app.example.com:80 → Stoobly forwards to http://app.example.com:80
stoobly-agent scaffold service create ui \
  --hostname app.example.com \
  --port 80 \
  --scheme http
```

#### Q: How do I create a service for the application under test?

**A:** Create a service for your main application (the application being tested) using only the `test` workflow. Do not use `--local`, `--workflow record`, or `--workflow mock` options since the application under test should not be recorded or mocked.

**Example:**

```
# Application under test service is run locally for development but built when testing,  running on http://localhost:4200
# DO NOT set --local option, this is the service used for development or testing
# DO NOT set --workflow record and --workflow mock options, main application should not have either
# Specify a hostname, e.g. main.local
# Specify a port e.g. 4200
# Run your frontend service on the same port
stoobly-agent scaffold service create main.local \
  --hostname main.local \
  --port 4200 \
  --scheme http \
  --workflow test
```

#### Q: For Docker runtime, how do I create a service that proxies to a service that runs on localhost?

**A:** Use `--hostname`, `--port`, and `--scheme` to define how clients connect to Stoobly, then add the `--local` flag. You must specify `--upstream-port` with a different port than `--port`, and run your local service on that upstream port.

**Important:**

* Docker runtime uses a **forward proxy** or **reverse proxy**
* `--port` - The port Stoobly listens on (what clients/tests connect to)
* `--scheme` - The scheme Stoobly uses (http or https)
* `--local` - Automatically sets `--upstream-hostname` to localhost
* `--upstream-port` - **Required and must be different from `--port`** - This is where your local service runs
* `--upstream-scheme` - **Optional** - Only specify if your local service uses a different scheme than `--scheme`
* Do not specify `--upstream-hostname` when using `--local` (it will be ignored)
* **You must run your local service on `--upstream-port`, not on `--port`**
* Different types of services have different recommended option configurations, match against the following examples.

**Example:**

```bash
# Local API service running on http://localhost:3000
# Specify a hostname, e.g. api.local
# Specify a port e.g. 3000
# Run your API service on a different port from 3000, e.g. 3001
# Clients connect to http://api.local:3000 → Stoobly forwards to http://localhost:3001
stoobly-agent scaffold service create api.local \
  --hostname api.local \
  --port 3000 \
  --scheme http \
  --local \
  --upstream-port 3001
```

#### Q: For local runtime, how do I create a service that proxies to a service that runs on localhost?

**A:** Set the hostname to `localhost` and do not use the `--local` option. Specify `--port` to match the port where your local service runs.

**Important:**

* Local runtime uses a **forward proxy**
* `--hostname` - Must be `localhost` for local runtime
* `--port` - The port where your local service runs (and where Stoobly will proxy)
* `--scheme` - The scheme your service uses (http or https)
* `--local` - **Do not use** this option with local runtime (it's only for Docker runtime)

**Example:**

```bash
# Local API service running on http://localhost:3000
# Hostname must be localhost for local runtime
# Port matches the port your service runs on
# Do NOT use --local option
stoobly-agent scaffold service create api \
  --hostname localhost \
  --port 3000 \
  --scheme http
```

#### Q: How do I set the port for a service?

**A:** Use the `--port` option to specify the port where Stoobly exposes the service (the port your tests/clients will use). For local services (using `--local`), you must specify `--upstream-port` with a different port and run your service on that upstream port.

**Key difference:**

* `--port` - The port Stoobly listens on (what you connect to in tests) - **always required**
* `--upstream-port` - The port of the actual service Stoobly proxies to - **required when using `--local` and must be different from `--port`**

**Example:**

```bash
# Remote service where Stoobly port matches upstream port (no --upstream-port needed)
# Specify a hostname, e.g. api.local
# Specify a port, e.g. 8080
# Clients connect to http://api.local:8080 → Stoobly forwards to http://api.local:8080
stoobly-agent scaffold service create api \
  --hostname api.local \
  --port 8080 \
  --scheme http

# Docker runtime local service (not applicable to local runtime) - ports must differ
# Local API service normally runs on http://localhost:3000
# Specify a hostname, e.g. api.local
# Specify a port e.g. 3000
# Run your API service on a different port from the specified one from 3000, e.g. 3001
# Clients connect to http://api.local:3000 → Stoobly forwards to http://localhost:3001
stoobly-agent scaffold service create api \
  --hostname api.local \
  --port 3000 \
  --scheme http \
  --local \
  --upstream-port 3001
```

#### Q: How do I control the startup order of services?

**A:** Use the `--priority` option (1.0-9.0) where lower values start first.

**Example:**

```bash
# Database starts first (priority 1)
stoobly-agent scaffold service create database --priority 1.0

# API starts second (priority 2)
stoobly-agent scaffold service create api --priority 2.0

# Frontend starts last (priority 3)
stoobly-agent scaffold service create frontend --priority 3.0
```

#### Q: How do I pass environment variables through to a service?

**A:** Use the `--env-name` option (repeatable) to list variable **names** the service should receive from the workflow environment. It takes names, not `NAME=VALUE` pairs — the values come from the workflow `.env`.

**Example:**

```bash
stoobly-agent scaffold service create api \
  --env-name DATABASE_URL \
  --env-name API_KEY
```

To set the values themselves, see [Environment Variables](#environment-variables) above.

{% hint style="warning" %}
**Changed in v2.5.2:** this option was named `--env` before v2.5.2. If you have a `.stoobly/scaffold.yml` with `env:` on a `service create` step, rename the key to `env_name:` — `scaffold apply` rejects unknown option keys during validation, so the config fails before any step runs.
{% endhint %}

#### Q: How do I create a service with specific workflows?

**A:** By default, all four workflows (record, mock, test, develop) are created. Use the `--workflow` option only when you want to create a subset of workflows.

**Common pattern:** Frontend services that serve static assets typically only need the `test` workflow, as they don't require recording or mocking. The test workflow serves the built application from fixtures for E2E testing.

**Example:**

```bash
# All workflows (default - no --workflow needed)
stoobly-agent scaffold service create api

# Frontend service - only test workflow (serves static assets from fixtures)
stoobly-agent scaffold service create frontend \
  --hostname app.local \
  --workflow test

# Only mock and test workflows
stoobly-agent scaffold service create api --workflow mock --workflow test

# Only record workflow
stoobly-agent scaffold service create api --workflow record

# Only develop workflow
stoobly-agent scaffold service create api --workflow develop
```

#### Q: What are the default workflows created for each service?

**A:** By default, four workflows are created: `record` (capture requests), `mock` (serve mocked responses), `test` (validate responses), and `develop` (redirect requests to a local development server via synced rewrite rules). Use `--workflow` if you want to create only a subset.

**Example:**

```bash
stoobly-agent scaffold service create api
# Creates: record, mock, test, and develop workflows automatically

stoobly-agent scaffold service create api --workflow mock --workflow test
# Creates only mock and test, skipping record and develop
```

#### Q: How do I set up a frontend service to serve static assets for E2E testing?

**A:** Frontend services typically only need the `test` workflow since they serve static assets from fixtures rather than recording or mocking requests. Modify the service's `test/init` script (located at `.stoobly/services/<service-name>/test/init`) to copy your built application to the public fixture folder.

**Example:**

```bash
# Create frontend service with only test workflow
stoobly-agent scaffold service create frontend \
  --hostname app.local \
  --workflow test

# Edit .stoobly/services/frontend/test/init to copy built assets
# Add this to the init script:
# cp -r $APP_DIR_PATH/dist/. .stoobly/fixtures/public/
```

**Example init script:**

```bash
#!/bin/bash
# .stoobly/services/frontend/test/init

# Copy built frontend assets to public fixtures folder
cp -r $APP_DIR_PATH/dist/dist/. .stoobly/fixtures/public/

# The frontend service will serve these static files during E2E tests
```

This setup allows your E2E tests to request the application under test from the frontend service, which serves the static assets from the fixtures folder.

***

### Managing Services

#### Q: How do I list all services in my scaffold app?

**A:** Use `scaffold service list` to display all configured services. Run from your app directory, or use `--app-dir-path` to specify the location.

**Example:**

```bash
# From within the app directory (default)
stoobly-agent scaffold service list

# Or specify the app directory
stoobly-agent scaffold service list --app-dir-path ./my-app
```

#### Q: How do I view details about a specific service?

**A:** Use `scaffold service show` with the service name.

**Example:**

```bash
stoobly-agent scaffold service show api
```

#### Q: How do I list services in a specific format?

**A:** Use the `--format` option to specify output format (table, json, csv).

**Example:**

```bash
# JSON format
stoobly-agent scaffold service list --format json

# CSV format
stoobly-agent scaffold service list --format csv
```

#### Q: How do I filter services by workflow?

**A:** Use the `--workflow` option to filter services by their workflows.

**Example:**

```bash
# Show only services with record workflow
stoobly-agent scaffold service list --workflow record

# Show services with mock or test workflows
stoobly-agent scaffold service list --workflow mock --workflow test
```

#### Q: How do I update a service configuration?

**A:** Use `scaffold service update` with the options you want to change.

**Example:**

```bash
stoobly-agent scaffold service update api \
  --hostname api.newdomain.com \
  --port 9090
```

**Note:** If you maintain `.stoobly/scaffold.yml`, update the matching `service` step's options too, so `stoobly-agent scaffold apply .stoobly/scaffold.yml` stays reproducible. See [Apply](apply.md) for the config schema.

#### Q: How do I rename a service?

**A:** Use `scaffold service update` with the `--name` option.

**Example:**

```bash
stoobly-agent scaffold service update old-api --name new-api
```

**Note:** If you maintain `.stoobly/scaffold.yml`, update the matching `service` step to use the new `service_name`, so `stoobly-agent scaffold apply .stoobly/scaffold.yml` stays reproducible. See [Apply](apply.md) for the config schema.

#### Q: How do I delete a service?

**A:** Use `scaffold service delete` with the service name.

**Example:**

```bash
stoobly-agent scaffold service delete api
```

**Note:** If you maintain `.stoobly/scaffold.yml`, remove the matching `service` step from it too, so `stoobly-agent scaffold apply .stoobly/scaffold.yml` stays reproducible. See [Apply](apply.md) for the config schema.

***

### Working with Workflows

#### Q: What workflows are available by default?

**A:** Four workflows are created by default for each service: `record` (capture requests), `mock` (serve mocked responses), `test` (validate responses), and `develop` (redirect requests to a local development server). See "What is the develop workflow and how do I use it?" below for details on develop.

**Example:**

```bash
# Start record workflow
stoobly-agent scaffold workflow up record
stoobly-agent intercept enable

# Start mock workflow
stoobly-agent scaffold workflow up mock
stoobly-agent intercept enable

# Start test workflow
stoobly-agent scaffold workflow up test
stoobly-agent intercept enable

# Start develop workflow
stoobly-agent scaffold workflow up develop
stoobly-agent intercept enable
```

#### Q: How do I start a workflow?

**A:** Use `scaffold workflow up` with the workflow name to start all services in that workflow. Run from your app directory, or use `--app-dir-path` to specify the location.

**Example:**

```bash
# From within the app directory (default)
stoobly-agent scaffold workflow up test

# Or specify the app directory
stoobly-agent scaffold workflow up test
```

#### Q: How do I stop a workflow?

**A:** Use `scaffold workflow down` with the workflow name to stop all services.

**⚠️ IMPORTANT:** Always use `stoobly-agent scaffold workflow down` to stop workflows. **DO NOT** use Unix commands like `pkill`, `kill`, or `docker stop` - these can leave the system in an inconsistent state.

**Example:**

```bash
stoobly-agent scaffold workflow down record
```

#### Q: How do I check if a workflow is running?

**A:** Use `scaffold workflow show` to determine if a scaffold workflow is currently running. If a workflow is running, it shows the namespace, runtime (Docker or local), and when it was started.

**Example:**

```bash
stoobly-agent scaffold workflow show
```

```
═════════════════════════════════════════════
  Workflow: mock
═════════════════════════════════════════════
  Namespace   mock
  Status      running
  Runtime     docker
  Started     2025-11-14 10:32:17
```

To also see the scaffold services running under the workflow, use the `--verbose` flag:

```bash
stoobly-agent scaffold workflow show --verbose
```

#### Q: How do I view logs from a workflow?

**A:** There are two log commands and both are important. Use `scaffold request logs list` to see what requests were intercepted and whether they were mocked or passed through — this is the first thing to check when verifying mock behavior or debugging. Use `scaffold workflow logs` for the raw workflow process output such as startup errors and configuration issues.

**Example:**

```bash
# Show intercepted request logs — confirms mock success/failure per request
stoobly-agent scaffold request logs list mock

# Stream request logs in real time
stoobly-agent scaffold request logs list mock --follow

# Show raw workflow process output (startup errors, config issues)
stoobly-agent scaffold workflow logs mock
```

Replace `mock` with `record` or `test` depending on which workflow is running.

#### Q: What is the develop workflow and how do I use it?

**A:** The `develop` workflow redirects requests for a service's real hostname to a local development server, instead of recording, mocking, or testing fixed responses. It's useful when you want to work against a service your app already calls (e.g. a staging or production URL) while transparently routing that traffic to code you're actively developing locally.

The redirection is driven by [rewrite rules](https://docs.stoobly.com/core-concepts/agent/proxy-settings/rewrite-rules) generated from the service's configured upstream hostname, port, and scheme. It's created automatically along with record, mock, and test — no extra flag needed unless you're limiting a service to a subset of workflows (see "How do I create a service with specific workflows?" above).

**Example:**

```bash
# Creates record, mock, test, AND develop workflows for the service
stoobly-agent scaffold service create api

# Sync develop rewrite rules from the service's upstream hostname, port, and scheme
# Re-run this any time the service's upstream configuration changes
stoobly-agent scaffold workflow rewrite develop

# Start the develop workflow
stoobly-agent scaffold workflow up develop
stoobly-agent intercept enable
```

#### Q: How do I start a workflow with specific services only?

**A:** Use the `--service` option to select which services to run.

**Example:**

```bash
# Run only api and database services
stoobly-agent scaffold workflow up test --service api --service database
```

#### Q: How do I start a workflow in detached mode?

**A:** Use the `--detached` flag to run the workflow in the background.

**Example:**

```bash
stoobly-agent scaffold workflow up test --detached
```

#### Q: How do I skip hostname installation prompts?

**A:** Use the `--hostname-install-confirm` option to automatically confirm or deny hostname installation.

**Example:**

```bash
# Automatically confirm
stoobly-agent scaffold workflow up record --hostname-install-confirm y

# Automatically deny
stoobly-agent scaffold workflow up record --hostname-install-confirm n
```

#### Q: How do I skip CA certificate installation prompts?

**A:** Use the `--ca-certs-install-confirm` option to automatically confirm or deny CA certificate installation.

**Example:**

```bash
stoobly-agent scaffold workflow up record --ca-certs-install-confirm y
```

#### Q: Which workflows support custom namespaces?

**A:** Namespaces are **only supported by test workflows** and custom workflows based on the test workflow template. Record and mock workflows do **not** support namespaces.

**Supported:**
- `test` workflow
- Custom workflows created with `--template test`

**Not Supported:**
- `record` workflow
- `mock` workflow
- Custom workflows created with `--template record` or `--template mock`

**Example:**

```bash
# Supported - test workflow with namespace
stoobly-agent scaffold workflow up test --namespace my-namespace

# Supported - custom workflow based on test workflow template
stoobly-agent scaffold workflow create my-test --template test --service api
stoobly-agent scaffold workflow up my-test --namespace my-namespace

# NOT Supported - record workflow (will cause errors)
stoobly-agent scaffold workflow up record --namespace my-namespace  # Does NOT work

# NOT Supported - mock workflow (will cause errors)
stoobly-agent scaffold workflow up mock --namespace my-namespace  # Does NOT work
```

***

### Creating Custom Workflows

#### Q: How do I create a custom workflow?

**A:** Use `scaffold workflow create` with a workflow name and template to create a custom workflow.

**Example:**

```bash
# Create a custom workflow based on the record template
stoobly-agent scaffold workflow create staging \
  --template record \
  --service api
```

#### Q: What templates can I use for custom workflows?

**A:** You can use `mock`, `record`, or `test` as templates for custom workflows.

**Example:**

```bash
# Custom workflow based on mock template
stoobly-agent scaffold workflow create dev-mock --template mock --service api

# Custom workflow based on test workflow template
stoobly-agent scaffold workflow create integration-test --template test --service api
```

#### Q: How do I create a custom workflow for multiple services?

**A:** Use multiple `--service` options to include multiple services in the workflow.

**Example:**

```bash
stoobly-agent scaffold workflow create full-stack \
  --template record \
  --service api \
  --service frontend \
  --service database
```

#### Q: How do I copy an existing workflow?

**A:** Use `scaffold workflow copy` to duplicate a workflow with a new name.

**Example:**

```bash
# Copy record workflow to staging workflow
stoobly-agent scaffold workflow copy record staging \
  --service api
```

***

### SSL/TLS Certificate Management

#### Q: How do I generate SSL certificates for HTTPS services?

**A:** Use `scaffold app mkcert`.

**Example:**

```bash
# Generate SSL certs for all services
stoobly-agent scaffold app mkcert

# Generate SSL certs for all services in a workflow
stoobly-agent scaffold app mkcert --workflow <WORKFLOW-NAME>
```

#### Q: How do I generate certificates for specific services only?

**A:** Use the `--service` option with the mkcert command.

**Example:**

```bash
stoobly-agent scaffold app mkcert --service api --service frontend
```

***

### Hostname Management

#### Q: How do I install hostnames for services?

**A:** Use `scaffold hostname install` to add service hostnames to `/etc/hosts`.

**Example:**

```bash
stoobly-agent scaffold hostname install --workflow record
```

#### Q: How do I uninstall hostnames for services?

**A:** Use `scaffold hostname uninstall` to remove service hostnames from `/etc/hosts`.

**Example:**

```bash
stoobly-agent scaffold hostname uninstall --workflow record
```

#### Q: How do I manage hostnames for specific services?

**A:** Use the `--service` option to target specific services.

**Example:**

```bash
stoobly-agent scaffold hostname install --workflow record --service api
```

***

### Team Collaboration Workflows

#### Q: How do I set up a scaffold project for my team?

**A:** Create the scaffold app, add services, and commit the configuration to version control so team members can use it.

**Example:**

```bash
stoobly-agent scaffold app create team-project

stoobly-agent scaffold service create local-api \
  --hostname api.local \
  --local \
  --port 3000 \
  --scheme http \
  --upstream-port 3001

stoobly-agent scaffold service create staging-api \
  --hostname api.staging \
  --port 443 \
  --scheme https

stoobly-agent scaffold service create frontend \
  --hostname frontend.local \
  --port 4200 \
  --scheme http \
  --workflow test

# Commit to git
git add .stoobly/
git commit -m "Add Stoobly scaffold configuration"
git push

# Team members use it
git pull
stoobly-agent scaffold workflow up mock
stoobly-agent intercept enable
```

#### Q: How do I share recorded requests with my team?

**A:** Use scenario snapshots to create committable files that can be shared via version control.

**Example:**

```bash
# Record requests
stoobly-agent scaffold workflow up record
stoobly-agent intercept enable

# Create snapshot
stoobly-agent scenario snapshot <SCENARIO-KEY>

# Commit and push
git add .stoobly/snapshots/
git commit -m "Add API request snapshots"
git push
```

#### Q: How do I run the same workflow across different environments?

**A:** Create custom workflows for each environment using templates.

**Example:**

```bash
# Create staging workflow
stoobly-agent scaffold workflow create staging --template record --service api

# Create production workflow
stoobly-agent scaffold workflow create production --template record --service api

# Use them
stoobly-agent scaffold workflow up staging
stoobly-agent scaffold workflow up production
```

***

### Quick Reference

#### Q: What's the complete end-to-end workflow for setting up and using a team project?

**A:** This is a comprehensive step-by-step guide covering the full workflow from initial setup through running tests and sharing results with your team.

**Example:**

```bash
# 1. Create app
stoobly-agent scaffold app create team-project

# 2. Add services
stoobly-agent scaffold service create local-api \
  --hostname api.local \
  --local \
  --port 3000 \
  --scheme http \
  --upstream-port 3001

stoobly-agent scaffold service create staging-api \
  --hostname api.staging \
  --port 443 \
  --scheme https

stoobly-agent scaffold service create frontend \
  --hostname frontend.local \
  --port 4200 \
  --scheme http \
  --workflow test

# 3. Start record workflow
stoobly-agent scaffold workflow up record
stoobly-agent intercept enable

# 4. Make requests (they get recorded)
curl http://api.local/users

# 5. Stop workflow
stoobly-agent scaffold workflow down record

# 6. Start mock workflow
stoobly-agent scaffold workflow up mock
stoobly-agent intercept enable

# 7. Run tests against mocks
npm test

# 8. Create snapshots and commit
stoobly-agent scenario snapshot <SCENARIO-KEY> 
git add .stoobly/
git commit -m "Add scaffold and snapshots"
git push

# 9. Setup test workflow
# Update .stoobly/services/entrypoint/test/run to run test command e.g. `npm test`

# 10. Start test workflow
stoobly-agent scaffold workflow up test
stoobly-agent intercept enable
```
