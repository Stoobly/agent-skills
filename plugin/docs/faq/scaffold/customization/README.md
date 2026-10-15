# Customization

## Stoobly Scaffold Structure - Questions & Answers

After scaffolding an app and services, Stoobly creates a comprehensive directory structure in `.stoobly/services/` with workflow-specific configurations, Docker compose files, and helper scripts.

**📚 Related Documentation:**
- For Docker-specific customization, see [docker.md](docker.md)
- For local runtime customization, see [local.md](local.md)

***

### Understanding Scaffold Structure

#### Q: Where does the scaffold get created?

**A:** The scaffold is created in the `.stoobly/services/` directory within your application directory.

**Example:**

```
# Create app
stoobly-agent scaffold app create my-app --app-dir-path ./my-project

# Scaffold structure created at:
# ./my-project/.stoobly/services/
```

#### Q: What is the overall structure of a scaffolded app?

**A:** The scaffold contains core services (build, entrypoint, gateway, stoobly-ui), user-defined services, a Makefile, and workflow-specific configurations.

**Example:**

```bash
.stoobly/services/
├── Makefile                    # Main Makefile for workflow commands
├── build/                      # Build service (Docker only)
│   ├── mock/
│   ├── record/
│   └── test/
├── entrypoint/                 # Entrypoint service (Docker only)
│   ├── mock/
│   ├── record/
│   └── test/
├── gateway/                    # Gateway service (Docker only)
│   ├── mock/
│   ├── record/
│   └── test/
├── stoobly-ui/                 # Stoobly UI service
│   ├── exec/
│   ├── mock/
│   └── record/
└── your-service/               # Your custom services
    ├── mock/
    ├── record/
    └── test/
```

***

### Docker vs Local Runtime Differences

#### Q: What's the difference between --runtime docker and --runtime local?

**A:** Docker runtime creates additional core services (build, entrypoint, gateway) for containerized execution, while local runtime creates a simpler structure for native execution.

**Example:**

```bash
# Docker runtime (default) - Full container orchestration
stoobly-agent scaffold app create my-app --runtime docker

# Creates:
# - build/ (Docker image building)
# - entrypoint/ (Container entry point)
# - gateway/ (Traefik reverse proxy)
# - stoobly-ui/ (UI service)
# - your-services/

# Local runtime - Simplified structure
stoobly-agent scaffold app create my-app --runtime local

# Creates:
# - stoobly-ui/ (UI service)
# - your-services/ (only user services)
# No build/, entrypoint/, or gateway/ directories
```

**For more details:**
- Docker-specific topics: See [docker.md](docker.md)
- Local-specific topics: See [local.md](local.md)

***

### Understanding Core Services

#### Q: What is the stoobly-ui service?

**A:** The stoobly-ui service provides the web interface for managing requests, scenarios, and configuration. It's available in both Docker and local runtimes.

**Example:**

```bash
.stoobly/services/stoobly-ui/
├── exec/                       # For CLI execution
├── mock/                       # UI for mock workflow
└── record/                     # UI for record workflow

# Access UI at: http://localhost:4200
```

**Note:** For Docker-specific services (build, entrypoint, gateway), see [docker.md](docker.md).

***

### Service-Specific Directories

#### Q: What files are created for each user-defined service?

**A:** Each service gets workflow directories (mock, record, test) with an init script, lifecycle hooks, fixtures, and public directory. Docker runtime also includes docker-compose.yml files.

**Example:**

```bash
.stoobly/services/my-service/
├── mock/
│   ├── docker-compose.yml      # Service definition (Docker only)
│   ├── fixtures.yml            # Response fixtures
│   ├── init                    # Init script
│   ├── lifecycle_hooks.py      # Custom hooks
│   └── public/                 # Static files
├── record/
│   ├── docker-compose.yml      # (Docker only)
│   ├── init
│   └── lifecycle_hooks.py
└── test/
    ├── docker-compose.yml      # (Docker only)
    ├── fixtures.yml
    ├── init
    ├── lifecycle_hooks.py
    └── public/
```

#### Q: Where do I add Stoobly configuration (rewrite, match, filter rules)?

**A:** Add Stoobly configuration in the workflow `init` script so rules are applied before services start.

**Example:**

```bash
# .stoobly/services/my-service/mock/init
#!/bin/bash

# Add rewrite rules
stoobly-agent setting rewrite set \
  --pattern "https://api.production.com/.*" \
  --method GET --method POST \
  --mode mock \
  --hostname localhost:8080

# Add filter rules
stoobly-agent setting filter set \
  --pattern "https://analytics.com/.*" \
  --method GET \
  --mode mock \
  --action exclude
```

#### Q: What is the purpose of the init script?

**A:** The init script runs custom initialization logic during service startup, such as database setup, file preparation, or environment checks.

**Example:**

```bash
# .stoobly/services/my-service/mock/init
#!/bin/bash

# Wait for dependencies
sleep 2

# Setup test data
echo "Initializing test data..."
curl -X POST http://my-service/setup

# Verify service health
curl http://my-service/health
```

**Note:** For Docker-specific docker-compose.yml customization, see [docker.md](docker.md).

#### Q: What is the fixtures.yml file used for?

**A:** The fixtures.yml file contains mock response data for the service, used during mock and test workflows.

**Example:**

```yaml
# .stoobly/services/my-service/mock/fixtures.yml
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
```

#### Q: What is the lifecycle\_hooks.py file used for?

**A:** The lifecycle\_hooks.py file contains Python functions that modify requests/responses during interception.

**Example:**

```python
# .stoobly/services/my-service/mock/lifecycle_hooks.py

def before_request(context):
    # Modify request before sending
    context.request.headers['X-Custom-Header'] = 'test-value'
    return context

def after_response(context):
    # Modify response before returning
    if context.response.status_code == 404:
        context.response.status_code = 200
    return context
```

#### Q: What is the public/ directory used for?

**A:** The public/ directory serves static files for mocking, useful for serving images, CSS, JavaScript, or other assets.

**Example:**

```bash
# .stoobly/services/my-service/mock/public/
mkdir -p .stoobly/services/my-service/mock/public
echo '{"data": "test"}' > .stoobly/services/my-service/mock/public/test.json

# Access via: http://my-service.local/test.json
```

***

### Workflow-Specific Configurations

#### Q: How are workflows organized?

**A:** Each service has separate directories for mock, record, and test workflows, allowing different configurations per workflow.

**Example:**

```bash
my-service/
├── mock/                       # Mock workflow config
│   ├── fixtures.yml           # Mock responses
│   └── docker-compose.yml     # Mock-specific settings
├── record/                     # Record workflow config
│   └── docker-compose.yml     # Record-specific settings
└── test/                       # Test workflow config
    ├── fixtures.yml           # Test fixtures
    └── docker-compose.yml     # Test-specific settings
```

#### Q: Can I have different configurations for different workflows?

**A:** Yes, each workflow directory has its own `init` script, `docker-compose.yml` (Docker only), and fixtures, allowing complete customization per workflow.

**Example:**

```bash
# Mock workflow - Use local fixtures
# .stoobly/services/my-service/mock/init
stoobly-agent setting rewrite set --mode mock --hostname localhost

# Record workflow - Use production
# .stoobly/services/my-service/record/init
stoobly-agent setting rewrite set --mode record --hostname api.production.com

# Test workflow - Use staging
# .stoobly/services/my-service/test/init
stoobly-agent setting rewrite set --mode test --hostname api.staging.com
```

***

### Custom Workflows

#### Q: How do I create a custom workflow?

**A:** Use `scaffold workflow create` to create a new workflow based on a template (mock, record, or test).

**Example:**

```bash
# Create custom 'ci' workflow for a service
stoobly-agent scaffold workflow create ci \
  --template mock \
  --service my-service \
  --app-dir-path ./my-app

# Creates:
# .stoobly/services/my-service/ci/
```

#### Q: What gets created for a custom workflow?

**A:** A custom workflow gets the same structure as standard workflows: docker-compose.yml, init, fixtures.yml, lifecycle\_hooks.py, and public/.

**Example:**

```bash
.stoobly/services/my-service/ci/
├── docker-compose.yml
├── fixtures.yml
├── init
├── lifecycle_hooks.py
└── public/
```

***

### Temporary Runtime Files

#### Q: What is the .stoobly/tmp/ directory?

**A:** The tmp/ directory contains runtime files generated during workflow execution, including logs, run scripts, and Traefik configuration.

**Example:**

```bash
.stoobly/tmp/
├── mock/
│   ├── logs/
│   │   └── requests.json       # Request logs
│   ├── run.sh                  # Generated run script
│   └── traefik.yml             # Traefik config
├── record/
│   └── ...
└── test/
    └── ...
```

**Note:** For Docker-specific run.sh script details, see [docker.md](docker.md).

***

### Troubleshooting

#### Q: How do I debug scaffold issues?

**A:** Check the generated files in .stoobly/services/ and .stoobly/tmp/, and use --dry-run to see what commands would execute.

**Example:**

```bash
# Dry run to see commands
stoobly-agent scaffold workflow up mock --dry-run --app-dir-path ./my-app

# Check generated run script
cat .stoobly/tmp/mock/run.sh

# Check logs
cat .stoobly/tmp/mock/logs/requests.json
```

#### Q: Where can I find workflow logs?

**A:** Workflow logs are stored in `.stoobly/tmp/<workflow>/logs/`.

**Example:**

```bash
# View request logs
cat .stoobly/tmp/mock/logs/requests.json
```

**Note:** For Docker-specific log viewing, see [docker.md](docker.md).

#### Q: How do I verify my service configuration?

**A:** Run the init script manually (if applicable) and verify Stoobly configuration.

**Example:**

```bash
# Run init script
bash .stoobly/services/my-service/mock/init

# Verify Stoobly config
stoobly-agent setting dump
```

**Note:** For Docker-specific configuration verification, see [docker.md](docker.md).

***

### Quick Reference

#### Q: What are the key directories in a scaffold?

**A:** Here's a quick reference of important directories:

**Example:**

```bash
.stoobly/services/
├── Makefile                    # Main workflow commands
├── build/                      # Docker: Image building (Docker only)
├── entrypoint/                 # Docker: Your app runs here (Docker only)
├── gateway/                    # Docker: Traefik proxy (Docker only)
├── stoobly-ui/                 # Stoobly web UI (Both)
├── your-service/               # Your services (Both)
│   ├── mock/                   # Mock workflow
│   ├── record/                 # Record workflow
│   └── test/                   # Test workflow
└── .stoobly/tmp/               # Runtime files (Both)
    ├── mock/
    ├── record/
    └── test/
```

#### Q: What files can I customize?

**A:** You can customize init scripts, fixtures.yml, lifecycle\_hooks.py, and add files to public/. For Docker runtime, you can also customize docker-compose.yml files.

**Example:**

```bash
# Customizable files per service/workflow:
my-service/mock/
├── docker-compose.yml      # ✓ Add your containers (Docker only)
├── fixtures.yml            # ✓ Add mock responses
├── init                    # ✓ Add initialization logic
├── lifecycle_hooks.py      # ✓ Add request/response hooks
└── public/                 # ✓ Add static files
```

**For more details:**
- Docker-specific customization: See [docker.md](docker.md)
- Local runtime customization: See [local.md](local.md)
