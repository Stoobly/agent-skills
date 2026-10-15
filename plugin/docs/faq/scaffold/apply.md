# Apply

## Stoobly Scaffold Apply - Questions & Answers

{% hint style="info" %}
`stoobly-agent scaffold apply` requires **stoobly-agent v2.5.0 or later**.
{% endhint %}

`scaffold apply` runs an ordered list of scaffold CLI commands from a config file (by convention, `.stoobly/scaffold.yml`). Each step in the config maps 1:1 to `stoobly-agent scaffold <resource> <action> …`, so an entire app-and-services setup can be written once, reviewed in a pull request, checked into version control, and reproduced by anyone on the team with a single command — instead of typing out a sequence of `scaffold app create` / `scaffold service create` invocations by hand.

***

### Getting Started

#### Q: How do I apply a scaffold config?

**A:** Point `scaffold apply` at the config file. Validate first with `--dry-run`, then apply for real.

**Example:**

```bash
stoobly-agent scaffold apply .stoobly/scaffold.yml --dry-run
stoobly-agent scaffold apply .stoobly/scaffold.yml
```

#### Q: What does the CLI command look like?

**A:**

```
Usage: stoobly-agent scaffold apply [OPTIONS] [PATH]

  Apply scaffold commands from a config file

Options:
  --dry-run             If set, runs validation and logs only.
  --format [yaml|json]  Config file format.  [default: yaml]
  -h, --help            Show this message and exit.
```

| Argument / option | Description |
|---|---|
| `PATH` | Path to the config file. **Optional since v2.5.1** — defaults to `.stoobly/scaffold.yml` in the current context. Must be an existing regular file (not a directory); a missing file exits non-zero. |
| `--dry-run` | Validates the config and logs what each step would do, without invoking any commands. |
| `--format` | `yaml` (default) or `json`. Does not sniff the file extension — a `.json` file applied without `--format json` is parsed as YAML. |

#### Q: Can I omit the path?

**A:** Yes, since v2.5.1. `stoobly-agent scaffold apply` with no path applies `.stoobly/scaffold.yml` from the current context.

The context isn't simply the current directory — `stoobly-agent` walks **up** from the working directory looking for an existing `.stoobly/` owned by the current user, so the bare command works from any subdirectory of a scaffolded repo. `STOOBLY_AGENT_CONTEXT_DIR` / `STOOBLY_CONTEXT_DIR` override that search when set.

{% hint style="warning" %}
If no `.stoobly/` is found anywhere up the tree, resolution falls back to `~/.stoobly` — so running the bare command in a repo that was never scaffolded looks in your home directory, not the repo. Pass `PATH` explicitly the first time, or run from a directory that already has `.stoobly/`.
{% endhint %}

***

### The Config File

#### Q: What's the top-level structure of `scaffold.yml`?

**A:** Two required keys: `version` (must be the integer `1`) and `commands` (a non-empty ordered list of steps).

**Example:**

```yaml
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
      hostname: api.example.com
      scheme: https
      port: 443
```

Steps run **in order** and stop on the first failure.

#### Q: What does a step look like?

**A:** Each entry in `commands` has a `resource`, an `action`, and an optional `options` mapping.

| Key | Required | Description |
|---|---|---|
| `resource` | yes | The scaffold command group, e.g. `app`, `service`. |
| `action` | yes | The subcommand under that group, e.g. `create`. |
| `options` | no | Mapping of options and positional arguments for that command. Defaults to `{}`. |

#### Q: Which resources and actions can I use in a config file?

**A:** `scaffold apply` only supports two levels — `resource` then a leaf `action` — so any scaffold subcommand nested deeper than that (like `scaffold request logs list`) can't be expressed in a config file. The complete, usable set is:

| `resource` | valid `action` values |
|---|---|
| `app` | `create` |
| `service` | `create`, `list`, `show`, `delete`, `update` |
| `workflow` | `create`, `copy`, `show`, `up`, `down`, `logs`, `mkcert`, `rewrite`, `filter`, `validate` |
| `hostname` | `install`, `uninstall` |

`apply` and `describe` aren't usable as a `resource` (they're standalone commands, not groups), and `request` is a group whose only child (`logs`) is itself a group, so it has no usable leaf action either.

#### Q: How do I translate a CLI flag into an `options` key?

**A:** Use the **snake_case** version of the flag name — `--app-dir-path` becomes `app_dir_path`, `--copy-on-workflow-up` becomes `copy_on_workflow_up`. Positional arguments (like the app or service name) use their argument name as the key.

| Kind | YAML shape |
|---|---|
| Flag | boolean: `quiet: true` |
| Single value | scalar: `hostname: api.example.com` |
| Multi-value option (`multiple=True`) | list: `plugin: [playwright]` (a bare scalar is also accepted as a single value) |
| Positional argument | key matching the argument name: `app_name: my-app`, `service_name: api` |

An option key that doesn't exist on the target command is a validation error before anything runs.

Some frequently used options have a fixed set of accepted values:

- `app create`: `plugin: [cypress, playwright]`, `proxy_mode: [forward, reverse]`, `runtime: [docker, local]`
- `service create`: `scheme` / `upstream_scheme`: `[http, https]`, `workflow: [develop, mock, record, test]`

**Changed in v2.5.2:** `service create`'s `--env` is now `--env-name`, so the config key is `env_name`. A config still using `env` fails validation with `Unknown options for command: env` before any step runs.

#### Q: Which paths get resolved automatically, and against what?

**A:** These six option keys are path-expanded: `app_dir_path`, `context_dir_path`, `script_path`, `ca_certs_dir_path`, `certs_dir_path`, `docker_socket_path`. For each, `~` is expanded first, then — if the path is still relative — it's resolved **against the config file's directory**, not your current working directory. So a relative path in `.stoobly/scaffold.yml` resolves relative to `.stoobly/`, no matter where you run `scaffold apply` from. Lists of paths (e.g. multiple `context_dir_path` entries) are expanded element-wise. All other options are passed through unchanged.

{% hint style="warning" %}
`scaffold apply` never creates directories. If an option is required to already exist (for example `context_dir_path`), the underlying command's validation will fail if it doesn't.
{% endhint %}

#### Q: Can I use JSON instead of YAML?

**A:** Yes — pass `--format json`. The schema is identical, just expressed as JSON.

**Example:**

```json
{
  "version": 1,
  "commands": [
    {
      "resource": "app",
      "action": "create",
      "options": {
        "app_name": "my-app",
        "plugin": ["playwright"]
      }
    },
    {
      "resource": "service",
      "action": "create",
      "options": {
        "service_name": "api",
        "hostname": "api.example.com",
        "scheme": "https",
        "port": 443
      }
    }
  ]
}
```

```bash
stoobly-agent scaffold apply .stoobly/scaffold.json --format json --dry-run
stoobly-agent scaffold apply .stoobly/scaffold.json --format json
```

***

### Full Example

#### Q: What does a larger, multi-service config look like?

**A:** A config can chain any number of `app`, `service`, and `workflow` steps. This example creates an app across two directories in a monorepo, adds three services (one local, one external with an OpenAPI spec, one with a custom `test` workflow), a custom `ci` workflow, and starts the `mock` workflow.

```yaml
version: 1
commands:
  - resource: app
    action: create
    options:
      app_name: monorepo
      app_dir_path: ~/monorepo
      context_dir_path:
        - ~/monorepo/apps/app-1
        - ~/monorepo/apps/app-2
      copy_on_workflow_up: true
      ui_port: 4201
      plugin: [playwright]

  - resource: service
    action: create
    options:
      service_name: dashboard
      app_dir_path: ~/monorepo
      context_dir_path:
        - ~/monorepo/apps/app-1
      hostname: local.stoobly.com
      scheme: http
      port: 80
      local: true

  - resource: service
    action: create
    options:
      service_name: google
      app_dir_path: ~/monorepo
      context_dir_path:
        - ~/monorepo/apps/app-1
        - ~/monorepo/apps/app-2
      env_name: [TEST]
      hostname: www.google.com
      scheme: https
      openapi_specification: true
      port: 443

  - resource: workflow
    action: create
    options:
      workflow_name: ci
      app_dir_path: ~/monorepo
      service: [google]
      template: mock

  - resource: service
    action: create
    options:
      service_name: assets
      app_dir_path: ~/monorepo
      hostname: http.badssl.com
      scheme: http
      port: 80
      detached: true
      workflow: [test]

  - resource: workflow
    action: up
    options:
      workflow_name: mock
      app_dir_path: ~/monorepo
      context_dir_path: ~/monorepo/apps/app-1
      log_level: warning
```

***

### Validation and Failure Behavior

#### Q: When does validation happen?

**A:** The **entire config is validated up front**, before any step runs — every step's `resource`, `action`, and `options` are checked. A typo in the last step means the first step never runs either; you find out immediately rather than partway through applying.

#### Q: What happens if a step fails partway through?

**A:** `scaffold apply` stops at the first failing step and exits with that step's own exit code. Steps that already ran are **not** rolled back — their side effects (files created, services registered, etc.) remain in place. Later steps in the config are not run.

#### Q: Does `--dry-run` guarantee the real apply will succeed?

**A:** Not entirely. `--dry-run` runs full config validation (schema, unknown resource/action/option, accepted values, missing required options) and logs one line per step, but it never actually invokes a command. That means checks that only happen when a command runs — like a required directory existing on disk — are **not** exercised by `--dry-run`. Treat a clean dry run as "the config itself is well-formed," not as "applying it will definitely succeed."

**Example:**

```bash
$ stoobly-agent scaffold apply .stoobly/scaffold.yml --dry-run
[INFO] Scaffold would apply app create my-app
[INFO] Scaffold would apply service create api
```

Without `--dry-run`, the same steps log as `applying …` before each one actually runs:

```bash
$ stoobly-agent scaffold apply .stoobly/scaffold.yml
[INFO] Scaffold applying app create my-app
[INFO] Scaffold applying service create api
```

{% hint style="info" %}
Only **positional** argument values (like the app or service name) are logged — option flags and their values are deliberately omitted from the log line, so nothing sensitive in `options` gets echoed. Logs go to stderr; setting `STOOBLY_AGENT_LOG_LEVEL=warning` suppresses the `applying …` / `would apply …` lines.
{% endhint %}

#### Q: What are the common validation errors, and what do they mean?

**A:** All of the following are checked before any step is invoked and exit with code `1`:

| Condition | Message |
|---|---|
| Missing `version` | `Missing required property: version` |
| Unsupported `version` | `Unsupported version: … Supported versions: 1` |
| Missing or empty `commands` | `commands must be a non-empty list` |
| Config file is empty, or its root isn't a mapping | `Config file is empty` / `Config root must be a mapping` |
| Invalid YAML/JSON | `Failed to parse config file: …` |
| Step missing `resource` or `action` | `commands[i] missing required property: resource` |
| Unknown `resource` | `Unknown resource: …` |
| `resource` isn't a command group | `Resource '…' is not a command group` |
| Unknown `action` | `Unknown action '…' for resource '…'` |
| `action` is a group, not a leaf command | `Action '…' for resource '…' is a group, not a command` |
| Unknown option key | `commands[i]: Unknown options for command: …` |
| Flag given a non-boolean value | `commands[i].options.… must be a boolean flag` |
| Value not in the accepted list | `commands[i].options.… has invalid value …. Accepted values: …` |
| Missing a required option/argument | `commands[i] missing required option(s): …` |

A `PATH` that doesn't exist or is a directory fails Click's own argument check and exits with code `2` instead.

***

### Related

{% content-ref url="../../guides/how-to-integrate-e2e-testing/how-to-scaffold-an-app/applying-a-scaffold-config.md" %}
[applying-a-scaffold-config.md](../../guides/how-to-integrate-e2e-testing/how-to-scaffold-an-app/applying-a-scaffold-config.md)
{% endcontent-ref %}

{% content-ref url="../../guides/how-to-integrate-e2e-testing/how-to-update-a-scaffold/" %}
[how-to-update-a-scaffold](../../guides/how-to-integrate-e2e-testing/how-to-update-a-scaffold/)
{% endcontent-ref %}
