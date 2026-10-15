# Runtime

## Stoobly Scaffold Runtime Options - Questions & Answers

The `--runtime` option when creating a scaffold app determines whether workflows run locally on your machine or in Docker containers. **By default, `scaffold app create` uses local runtime** (no `--runtime` option needed). You can only specify one runtime option at a time: either `local` or `docker`, but not both.

**📚 Related Documentation:**
- For Docker-specific runtime details, see [docker.md](docker.md)
- For local runtime-specific details, see [local.md](local.md)

***

### Understanding Runtime Options

#### Q: What runtime options are available for scaffold apps?

**A:** Scaffold supports two runtime options: `local` (native execution on your machine) and `docker` (containerized execution). **`scaffold app create` defaults to local runtime** - you don't need to specify `--runtime local` unless you want to be explicit.

**Example:**

```bash
# Local runtime (default - no --runtime option needed)
stoobly-agent scaffold app create my-app

# Or explicitly specify local runtime (optional, same as default)
stoobly-agent scaffold app create my-app --runtime local

# Docker runtime (must be explicitly specified)
stoobly-agent scaffold app create my-app --runtime docker
```

#### Q: What's the difference between local and Docker runtime?

**A:** Local runtime executes workflows directly on your machine using your installed Python and dependencies, while Docker runtime runs workflows in isolated containers with consistent environments.

**Example:**

```bash
# Docker: Isolated, consistent, requires Docker installed
stoobly-agent scaffold app create my-app --runtime docker

# Local: Uses your machine's Python, faster startup, no Docker needed
stoobly-agent scaffold app create my-app --runtime local
```

#### Q: Which runtime option should I choose?

**A:** **Local runtime is the default** and is recommended for faster iteration and simpler setup. Use Docker runtime for team consistency and isolation. You can only choose one runtime option per app.

**Example:**

```bash
# Local runtime (default - no --runtime option needed)
stoobly-agent scaffold app create my-app

# Or explicitly specify local runtime
stoobly-agent scaffold app create my-app --runtime local

# Docker runtime (must be explicitly specified)
stoobly-agent scaffold app create my-app --runtime docker

# Note: You cannot use both --runtime local and --runtime docker together
```

#### Q: What's the usage difference between local and Docker runtime?

**A:** The main difference is in the commands you use to run workflows. Local runtime uses direct CLI commands, while Docker runtime uses Makefile commands.

**Example:**

```bash
# Local runtime (default)
stoobly-agent scaffold app create my-app

# Start workflow with local runtime
stoobly-agent scaffold workflow up record
stoobly-agent scaffold workflow up mock
stoobly-agent scaffold workflow up test

# Stop workflow
stoobly-agent scaffold workflow down record

# Docker runtime
stoobly-agent scaffold app create my-app --runtime docker

# Start workflow with Docker runtime
make -f .stoobly/services/.Makefile record
make -f .stoobly/services/.Makefile mock
make -f .stoobly/services/.Makefile test

# Stop workflow
make -f .stoobly/services/.Makefile record/down
make -f .stoobly/services/.Makefile mock/down
make -f .stoobly/services/.Makefile test/down
```

**For more details:**
- Docker runtime: See [docker.md](docker.md)
- Local runtime: See [local.md](local.md)

***

### Switching From One Runtime To Another

#### Q: Do both runtimes use the same configuration?

**A:** Yes, both runtimes share the same service configurations, workflows, and recorded data. However, you can only choose one runtime option when creating an app.

**Example:**

```bash
# Create app with local runtime (default)
stoobly-agent scaffold app create my-app

# Or create app with Docker runtime
stoobly-agent scaffold app create my-app --runtime docker

# Add service (works with either runtime)
stoobly-agent scaffold service create api --hostname api.local

# Use Docker runtime (if app was created with --runtime docker)
make -f .stoobly/services/.Makefile test

# Or use local runtime (if app was created with --runtime local or default)
stoobly-agent scaffold workflow up test
```

#### Q: Can I switch between local and Docker runtime after creating the app?

**A:** Yes, you can recreate the app with a different runtime option. Note that you can only use one runtime option at a time.

**Example:**

```bash
# Initially created with Docker
stoobly-agent scaffold app create my-app --runtime docker

# Later, recreate to switch to local runtime
stoobly-agent scaffold app create my-app --runtime local
# (This updates the configuration without deleting data)

# Note: You cannot use both --runtime local and --runtime docker together
```

***

### Runtime-Specific Workflows

#### Q: How do I use a specific runtime?

**A:** The runtime is determined when you create the app. Use CLI commands for local runtime, or Makefile commands for Docker runtime.

- **Docker runtime:** See [docker.md](docker.md) for details.
- **Local runtime:** See [local.md](local.md) for details.

***

### Performance Considerations

#### Q: Which runtime is faster for development?

**A:** Local runtime typically has faster startup times, while Docker provides better isolation and consistency.

**Example:**

```bash
# Local: Faster startup
time stoobly-agent scaffold workflow up test
# ~2-5 seconds

# Docker: Slower startup but isolated
time make -f .stoobly/services/.Makefile test
# ~10-30 seconds (first time, includes image pull)
```

#### Q: Does Docker runtime use more resources?

**A:** Yes, Docker adds overhead for containerization, but provides better isolation and cleanup.

**Example:**

```bash
# Docker: More memory/CPU but isolated
make -f .stoobly/services/.Makefile test
docker stats  # View resource usage

# Local: Less overhead but shares system resources
stoobly-agent scaffold workflow up test
```

**For optimization tips:**
- Docker runtime: See [docker.md](docker.md) for performance optimization.
- Local runtime: Generally requires no special optimization.

***

### Team Collaboration with Different Runtimes

#### Q: How do I set up a project that works for team members with different preferences?

**A:** Choose one runtime for the project. Team members can recreate the app with their preferred runtime if needed, as configurations are shared.

**Example:**

```bash
# Project setup (team lead) - choose one runtime
stoobly-agent scaffold app create team-project --runtime docker
stoobly-agent scaffold service create api --hostname api.local

# Document in README:
# Docker users: make -f .stoobly/services/.Makefile test
# Local users: Recreate with --runtime local, then use: stoobly-agent scaffold workflow up test
```

#### Q: How do I ensure consistency when team members use different runtimes?

**A:** Use the same service configurations and share recorded data via version control.

**Example:**

```bash
# Both runtimes share the same data
git add .stoobly/
git commit -m "Add scaffold configuration and snapshots"

# Team member A (Docker)
git pull
make -f .stoobly/services/.Makefile mock

# Team member B (Local)
git pull
stoobly-agent scaffold workflow up mock
```

#### Q: Can local and Docker users work on the same project simultaneously?

**A:** If the app was created with one runtime, team members using a different runtime will need to recreate the app with their preferred runtime. They should use different workflow namespaces to avoid conflicts.

**Note:** Namespaces are only supported by test workflows and custom workflows based on the test workflow template.

**Example:**

```bash
# Docker user (app created with --runtime docker)
make -f .stoobly/services/.Makefile test namespace=docker-dev

# Local user (recreates app with --runtime local, different machine or namespace)
stoobly-agent scaffold app create my-app --runtime local
stoobly-agent scaffold workflow up test --namespace local-dev
```

***

### Migration Between Runtimes

#### Q: How do I migrate between runtimes?

**A:** Recreate the app with the desired runtime and use the appropriate commands.

- **From Docker to local:** See [local.md](local.md) for migration steps.
- **From local to Docker:** See [docker.md](docker.md) for migration steps.

#### Q: Will my recorded data work with both runtimes?

**A:** Yes, recorded data is stored in `.stoobly/` and works with both runtimes.

**Example:**

```bash
# Record with Docker
make -f .stoobly/services/.Makefile record
make -f .stoobly/services/.Makefile intercept/enable
# ... make requests ...
make -f .stoobly/services/.Makefile record/down

# Mock with local (uses same data)
stoobly-agent scaffold workflow up mock
```

***

### Quick Reference

#### Q: What's a quick comparison of local vs Docker runtime?

**A:** Here's a side-by-side comparison:

**Local Runtime:**

* ✅ Faster startup
* ✅ Simpler debugging
* ✅ Less resource usage
* ✅ No Docker required
* ❌ Requires Python 3.12+
* ❌ Less isolation
* ❌ Environment differences possible

**Docker Runtime:**

* ✅ Consistent across all environments
* ✅ Complete isolation
* ✅ Easy cleanup
* ✅ Works on Windows, Mac, Linux
* ❌ Slower startup
* ❌ Requires Docker installed
* ❌ More resource usage

**Example:**

```bash
# Choose based on your needs:

# Speed & simplicity → Local
stoobly-agent scaffold app create my-app --runtime local

# Team consistency & isolation → Docker
stoobly-agent scaffold app create my-app --runtime docker
```

#### Q: What's the recommended setup for a new team project?

**A:** Choose one runtime based on your team's needs. Local (default) offers faster iteration, while Docker provides consistency.

**Example:**

```bash
# Recommended team setup - choose one runtime
stoobly-agent scaffold app create team-project \
  --runtime docker \
  --plugin cypress \
  --proxy-port 8080 \
  --ui-port 4200

# Or use local runtime (default)
stoobly-agent scaffold app create team-project \
  --plugin cypress \
  --proxy-port 8080 \
  --ui-port 4200

# Add services
stoobly-agent scaffold service create api --hostname api.local
stoobly-agent scaffold service create frontend --hostname frontend.local

# Document the chosen runtime in README
# Docker: make -f .stoobly/services/.Makefile test
# Local: stoobly-agent scaffold workflow up test
```
