# Stoobly agent skills

Skills for [Claude Code](https://claude.com/claude-code) that help you set up, use, and troubleshoot [Stoobly](https://stoobly.com) — API traffic recording, mocking, and testing.

## Install

```
/plugin marketplace add Stoobly/agent-skills
/plugin install stoobly@stoobly
```

Then run `/stoobly`. It asks what you want to do and routes you to the right skill, or to the [Stoobly docs](https://docs.stoobly.com) when no skill fits.

## Skills

Skills are namespaced by the plugin, e.g. `/stoobly:triage`. The entrypoint is `/stoobly`.

| Skill | What it's for |
|---|---|
| `stoobly` | Entry point — asks what you want to do and routes to the right skill. |
| `scaffold-create` | Bootstrap a new Stoobly scaffold app and its backend services. |
| `js-client` | Wire the Stoobly JS client (npm `stoobly`) into an E2E test suite — Playwright and Cypress have dedicated interceptors; other frameworks use the generic one. |
| `update-request` | Update a single recorded request and/or its response via the CLI and save the change as a request snapshot. |
| `triage` | Diagnose general setup and runtime problems — certificate errors, port conflicts, traffic not being intercepted, 499s, stale responses. |
| `troubleshoot-e2e-test` | Debug a failing Playwright or Cypress test by cross-referencing test logs with Stoobly's workflow request logs. |

The skills need the `stoobly-agent` CLI. See the [installation guide](https://docs.stoobly.com).

## Docs

The plugin bundles the Stoobly docs the skills read, so they work offline. The full documentation is at [docs.stoobly.com](https://docs.stoobly.com).

## License

Apache-2.0 — see [LICENSE](LICENSE).
