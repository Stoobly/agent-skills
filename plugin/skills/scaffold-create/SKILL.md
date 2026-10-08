---
name: scaffold-create
description: Bootstraps a new Stoobly scaffold app and services. Use when the user
  wants to get started with Stoobly, create a scaffold app, add services, or run
  their first workflow.
---

# Scaffold Create

You are helping the user bootstrap a Stoobly scaffold application.

## Step 1: Detect existing state

Check if `.stoobly/services/.config.yml` exists in the current directory.

- If it exists, read it and report the user's current config (app name, runtime, proxy mode). Then ask: "You already have a scaffold app. Do you want to add more services, or start fresh?"
  - If they want to add more services, tell them to add a `service create` block to their existing `.stoobly/scaffold.yml` (same schema as Step 4 below) and re-run `stoobly-agent scaffold apply` — see https://docs.stoobly.com/faq/scaffold for the full command reference. Stop here.
  - If they want to start fresh, continue to Step 2.
- If it does not exist, proceed to Step 2.

## Step 2: Read the docs

Read `../../docs/faq/scaffold/README.md` and `../../docs/faq/scaffold/apply.md` to get current command syntax and the `scaffold.yml` schema.

## Step 3: Ask the user these questions (ask all at once, in a single message)

1. **App name**: What do you want to call this scaffold app? (e.g., `my-app`)
2. **Runtime**: Local (faster, no Docker required) or Docker (recommended for teams and CI/CD)?
3. **Proxy mode**: Forward proxy (your app/tests explicitly use `http://localhost:8080` as a proxy) or Reverse proxy (traffic routed transparently by hostname — better for browser-based apps)?
4. **E2E testing**: Are you setting up E2E tests? If so, which framework — Playwright, Cypress, both, or neither?
5. **Services**: List the backend service(s) you want to mock. For each, provide:
   - A short name (e.g., `api`, `payments-service`)
   - The hostname of the real service (e.g., `api.example.com` or `localhost`)
   - The port (e.g., `3000`, `443`)
   - The scheme (`http` or `https`)

## Step 4: Write scaffold.yml and apply

`scaffold apply` requires `stoobly-agent` v2.5.0+. Run `stoobly-agent --version` and confirm it meets that minimum. If it's older, tell the user to upgrade first (`pipx upgrade stoobly-agent` or `docker pull stoobly/agent`) and stop here.

Ask the user where to put the config (default: `.stoobly/scaffold.yml` in the project root).

Build a `version: 1` config with a `commands` list from the Step 3 answers, translating flags to snake_case per the docs:
- One `app` / `create` step with `app_name`, `runtime`, `proxy_mode`, and `plugin` (from the E2E framework answer, e.g. `[playwright]`, `[cypress]`, both, or omitted)
- One `service` / `create` step per service, with `service_name`, `hostname`, `port`, and `scheme`

Follow the schema from `apply.md` exactly. Then show the resulting YAML to the user and ask: "Ready to apply this? (yes / let me review first)"

Once confirmed:
1. Write the config file to disk
2. Validate: `stoobly-agent scaffold apply <path> --dry-run`
3. If validation passes, apply for real: `stoobly-agent scaffold apply <path>`

After execution, report whether it succeeded or failed. If it fails, stop and show the full error output before asking how to proceed. Steps that already ran are not rolled back on failure.

## Step 5: Surface next steps

After all commands succeed, tell the user:

- How to route traffic through the proxy (proxy env vars for forward proxy, or hostname setup for reverse proxy)
- If recording HTTPS, remind them the CA cert was installed (or how to install it manually if skipped)
- Offer to immediately start the record workflow

End with:

> **Next steps:**
> - Your scaffold config is saved at `<path>` — commit it to version control so teammates can reproduce this setup with `stoobly-agent scaffold apply <path>`.
> - Now that your Stoobly scaffold is created, you can now run workflows! See [How to Run a Workflow](https://docs.stoobly.com/guides/how-to-integrate-e2e-testing/how-to-run-a-workflow)
> - Or to further customize a workflow, see [Customizing a Workflow](https://docs.stoobly.com/guides/how-to-integrate-e2e-testing/how-to-scaffold-an-app/customizing-a-workflow)
>
> Docs: https://docs.stoobly.com/faq/scaffold
> 
> Runtime comparison: https://docs.stoobly.com/faq/scaffold/runtime
>
> Scaffold Apply reference: https://docs.stoobly.com/faq/scaffold/apply
