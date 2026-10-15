# Docker Runtime Customization

## Stoobly Scaffold Docker Runtime - Questions & Answers

This document covers Docker-specific customization options for Stoobly scaffold. For general customization topics, see [README.md](README.md). For local runtime customization, see [local.md](local.md).

***

### Docker Runtime Structure

#### Q: Which files are created for Docker runtime only?

**A:** Docker runtime creates build/, entrypoint/, and gateway/ service directories with Docker Compose configurations.

**Example:**

```bash
# Docker-only services:
.stoobly/services/
├── build/                      # Docker only
├── entrypoint/                 # Docker only
├── gateway/                    # Docker only
└── stoobly-ui/                 # Both Docker and local
```

***

### Understanding the Entrypoint Service

#### Q: What is the entrypoint service?

**A:** The entrypoint service is the main container that runs your application or test code within the Docker network. It's where your actual service or test scripts execute.

**Example:**

```bash
# Entrypoint service structure
.stoobly/services/entrypoint/
├── mock/
│   ├── docker-compose.yml      # Service definition
│   ├── init                    # Initialization script
│   └── run                     # Entrypoint script
├── record/
│   ├── docker-compose.yml
│   ├── init
│   └── run
└── test/
    ├── docker-compose.yml
    ├── init
    └── run
```

#### Q: What is the purpose of the entrypoint service?

**A:** The entrypoint service provides a containerized environment where your application runs and makes HTTP requests that get intercepted by Stoobly's proxy through the gateway.

**Example:**

```bash
# Workflow flow:
# 1. Entrypoint container starts
# 2. Your app/tests run inside entrypoint
# 3. HTTP requests go through gateway (Traefik)
# 4. Gateway routes to Stoobly proxy
# 5. Stoobly records/mocks/tests the requests
```

#### Q: When is the entrypoint service used?

**A:** The entrypoint service is used in Docker runtime workflows to run your application code, test suites, or any process that makes HTTP requests you want to intercept.

**Example:**

```bash
# Start mock workflow
make -f .stoobly/services/Makefile mock

# Entrypoint container runs your app
# App makes requests → Gateway → Stoobly proxy → Mocked responses
```

#### Q: How do I customize the entrypoint service?

**A:** Edit the `docker-compose.yml` file in the entrypoint workflow directory to add your application container, environment variables, or volumes.

**Example:**

```yaml
# .stoobly/services/entrypoint/mock/docker-compose.yml
services:
  entrypoint.my-app:
    image: my-app:latest
    depends_on:
      entrypoint.init:
        condition: service_completed_successfully
    networks:
      app.ingress: {}
    environment:
      - HTTP_PROXY=http://gateway:80
      - HTTPS_PROXY=http://gateway:80
    profiles:
      - ${WORKFLOW_NAME}
    command: npm test
```

***

### Understanding Docker Core Services

#### Q: What is the build service?

**A:** The build service creates the Docker image for Stoobly agent with your application's context, ensuring consistent environments across workflows.

**Example:**

```bash
.stoobly/services/build/
├── mock/
│   ├── init
│   ├── docker-compose.yml      # Builds stoobly image
│   └── init
└── ...
```

#### Q: What is the gateway service?

**A:** The gateway service runs Traefik reverse proxy that routes HTTP traffic from your services through Stoobly's proxy for interception.

**Example:**

```bash
.stoobly/services/gateway/
├── mock/
│   └── docker-compose.yml      # Traefik configuration
└── ...

# Gateway routes:
# your-service:80 → gateway:80 → stoobly-proxy:8080 → upstream
```

***

### Docker-Specific Customization

#### Q: How do I add my application to the entrypoint service?

**A:** Edit the entrypoint docker-compose.yml to include your application container.

**Example:**

```yaml
# .stoobly/services/entrypoint/test/docker-compose.yml
services:
  entrypoint.my-app:
    image: node:18
    working_dir: /app
    volumes:
      - ../../..:/app
    depends_on:
      entrypoint.init:
        condition: service_completed_successfully
    networks:
      app.ingress: {}
    environment:
      - HTTP_PROXY=http://gateway:80
      - HTTPS_PROXY=http://gateway:80
      - NO_PROXY=localhost,127.0.0.1
    profiles:
      - ${WORKFLOW_NAME}
    command: npm test
```

#### Q: How do I run my test suite in the entrypoint?

**A:** Configure the entrypoint service with your test command and ensure it uses the proxy.

**Example:**

```yaml
# .stoobly/services/entrypoint/test/docker-compose.yml
services:
  entrypoint.playwright-tests:
    image: mcr.microsoft.com/playwright:latest
    working_dir: /tests
    volumes:
      - ../../..:/tests
    depends_on:
      entrypoint.init:
        condition: service_completed_successfully
    networks:
      app.ingress: {}
    environment:
      - HTTP_PROXY=http://gateway:80
      - HTTPS_PROXY=http://gateway:80
    profiles:
      - ${WORKFLOW_NAME}
    command: npx playwright test
```

#### Q: How do I add environment variables to my Docker service?

**A:** Add environment variables in the docker-compose.yml file for your service.

**Example:**

```yaml
# .stoobly/services/my-service/mock/docker-compose.yml
services:
  my-service.app:
    environment:
      - DATABASE_URL=postgres://localhost/testdb
      - API_KEY=test-key-123
      - NODE_ENV=test
      - DEBUG=true
```

#### Q: How do I mount volumes for my Docker service?

**A:** Add volume mounts in the docker-compose.yml to share files between host and container.

**Example:**

```yaml
# .stoobly/services/my-service/mock/docker-compose.yml
services:
  my-service.app:
    volumes:
      - ../../..:/app                    # Mount project root
      - ./fixtures.yml:/app/fixtures.yml # Mount fixtures
      - ./public:/app/public             # Mount public files
```

#### Q: What is the docker-compose.yml file used for?

**A:** The docker-compose.yml defines the service container, its dependencies, networks, environment variables, and how it connects to other services.

**Example:**

```yaml
# .stoobly/services/my-service/mock/docker-compose.yml
services:
  my-service.proxy:
    depends_on:
      my-service.init:
        condition: service_completed_successfully
    environment:
      - STOOBLY_HOSTNAME=my-service.local
      - STOOBLY_UPSTREAM_HOSTNAME=api.production.com
    networks:
      app.ingress: {}
    profiles:
      - ${WORKFLOW_NAME}
```

#### Q: How do I use a custom context directory?

**A:** Set the `STOOBLY_CONTEXT_DIR` environment variable to your desired context directory before running Makefile commands.

**Example:**

```bash
export STOOBLY_CONTEXT_DIR=/path/to/context
make -f .stoobly/services/Makefile test
```

#### Q: How do I run workflows with custom namespaces?

**A:** Use the `namespace` variable to specify a custom workflow namespace when running Makefile commands.

**Note:** Namespaces are only supported by test workflows and custom workflows based on the test workflow template. Record and mock workflows do not support namespaces.

**Example:**

```bash
make -f .stoobly/services/Makefile test namespace=my-custom-namespace
```

#### Q: How do I run workflows in dry-run mode?

**A:** Use the `--dry-run` flag with the CLI command to see what commands would be executed without running them.

**Example:**

```bash
stoobly-agent scaffold workflow up test --dry-run
```

***

### Advanced Configuration

#### Q: How do I increase logging verbosity?

**A:** Use the make logs command with options to control logging verbosity. You can also set the `workflow_log_extra_options` variable to pass additional options.

**Example:**

```bash
# View logs for a workflow
make -f .stoobly/services/.Makefile record/logs

# View logs with options (check available options with -h)
make -f .stoobly/services/.Makefile record/logs options="-h"

# View logs for specific containers
make -f .stoobly/services/.Makefile record/logs options="--container init"

# Set extra log options globally
make -f .stoobly/services/.Makefile record/logs workflow_log_extra_options="--tail=100"
```

#### Q: How do I follow workflow logs in real-time?

**A:** Use the make logs command with the `-f` / `--follow` option to stream logs continuously.

**Example:**

```bash
# Follow logs for a workflow
make -f .stoobly/services/.Makefile record/logs options="--follow"

# Follow logs for specific containers
make -f .stoobly/services/.Makefile record/logs options="--follow --container service --service gateway"

# Follow logs for specific service
make -f .stoobly/services/.Makefile record/logs options="--follow --service api"
```

***

### Docker Troubleshooting

#### Q: How do I view Docker logs for my services?

**A:** Use docker-compose logs to view container output.

**Example:**

```bash
# View Docker logs
docker-compose -f .stoobly/services/my-service/mock/docker-compose.yml logs

# View logs for specific service
docker-compose -f .stoobly/services/my-service/mock/docker-compose.yml logs my-service.proxy

# Follow logs in real-time
docker-compose -f .stoobly/services/my-service/mock/docker-compose.yml logs -f
```

#### Q: What is the run.sh script?

**A:** The run.sh script is auto-generated by the scaffold workflow up command and contains the Docker Compose commands to start the workflow.

**Example:**

```bash
# Generated run.sh
#!/bin/bash
docker-compose -f .stoobly/services/build/mock/docker-compose.yml up
docker-compose -f .stoobly/services/gateway/mock/docker-compose.yml up -d
docker-compose -f .stoobly/services/my-service/mock/docker-compose.yml up -d
# ... more services
```
