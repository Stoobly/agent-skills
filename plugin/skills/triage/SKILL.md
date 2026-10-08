---
name: triage
description: Diagnoses and fixes general Stoobly setup and runtime problems. Use when the
  user has HTTPS/certificate errors, traffic that isn't being intercepted, port
  conflicts, Stoobly failing to start, standalone (non-scaffold) issues, or wants a
  broad "something's broken" triage — including 499s and wrong responses outside of a
  test run. If the problem is a failing scaffolded E2E test (Playwright/Cypress), use
  troubleshoot-e2e-test instead.
---

# Triage

You are helping the user diagnose and fix a problem with their Stoobly setup or usage.

## Step 1: Detect existing state

Check if `.stoobly/services/.config.yml` exists in the current directory.

- If it exists, read it and note the runtime (`APP_RUNTIME`: docker or local) and proxy mode (`APP_PROXY_MODE`: forward or reverse).
- If it does not exist, the user is running standalone (no scaffold app).

## Step 2: Ask the user these questions (ask all at once, in a single message)

1. **Symptom**: What are you seeing? Choose the closest match:
   - 499 errors on some or all requests
   - HTTPS/certificate errors (SSL handshake failures, untrusted cert warnings)
   - Traffic not being intercepted (requests going directly to the real service)
   - Wrong or stale responses (getting old recorded data or unexpected responses)
   - Port conflict (Stoobly won't start, address already in use)
   - Something else — describe it

2. **Setup**: Are you using a scaffold app or running Stoobly standalone?

3. **Context**: Did this show up while running an E2E test (Playwright, Cypress, etc.), or during general usage?

**Routing check — do this before Step 3.** If the answer to question 3 is that the problem surfaced during an E2E test run, hand off now regardless of which symptom was chosen in question 1. `troubleshoot-e2e-test` covers test-framework log cross-referencing, trace inspection, and the recording-fix workflow that this skill does not.

To hand off:

1. Invoke the `troubleshoot-e2e-test` skill using your assistant's skill invocation (e.g. `/troubleshoot-e2e-test`).
2. Carry over what Step 1 already established so the other skill does not rediscover it — whether `.stoobly/services/.config.yml` exists, and the `APP_RUNTIME` / `APP_PROXY_MODE` values.

Do not continue to Step 3 below — `troubleshoot-e2e-test` owns the rest of the diagnosis. Return here only if it concludes the cause is environmental (certificate, port conflict, or intercept never enabled).

## Step 3: Read the relevant docs

Based on the symptom, read the corresponding doc.

| Symptom | Doc |
|---|---|
| 499 errors | `../../docs/guides/how-to-mock-apis/troubleshooting.md` |
| HTTPS / cert errors | `../../docs/faq/ca-cert.md` |
| Traffic not intercepted | `../../docs/faq/intercept.md` and `../../docs/faq/run.md` |
| Wrong / stale responses | `../../docs/guides/how-to-mock-apis/troubleshooting.md` |
| Port conflict | `../../docs/faq/run.md` |

If the user has a scaffold app, also read the runtime-specific doc:
- Docker: `../../docs/faq/scaffold/runtime/docker.md`
- Local: `../../docs/faq/scaffold/runtime/local.md`

## Step 4: Run diagnostics and fixes

Using the docs you just read, determine the exact diagnostic and fix commands for their specific setup (scaffold+docker, scaffold+local, or standalone).

**Always start by checking the logs** — run these immediately before anything else:

For scaffold apps:
```bash
# Show what requests the workflow saw and their mock/record status
stoobly-agent scaffold request logs list <workflow>   # e.g. mock, record, test

# Show the raw workflow process output (startup errors, config issues)
stoobly-agent scaffold workflow logs <workflow>
```

For standalone (no scaffold):
```bash
# Show intercepted request logs (supports --follow, --level, --status-code filters)
stoobly-agent request logs list
```

Run diagnostic commands immediately using Bash (no confirmation needed — these are read-only checks). Show the output and interpret it for the user.

For fix commands, show what you plan to run and ask: "Ready to apply this fix? (yes / let me review first)" Once confirmed, execute each fix command using Bash. After each command, report whether it succeeded or failed. If a command fails, stop and show the full error output before asking how to proceed.

## Step 5: Surface next steps

End with context-appropriate next steps. Examples:

> - If you resolved a 499 issue: run the scaffold mock workflow again to confirm responses are now served correctly.
> - If you updated recordings: run scaffold test workflow to verify the new recordings match the live API.
> - If you're still stuck: check logs with `stoobly-agent scaffold request logs list <workflow>` and `stoobly-agent scaffold workflow logs <workflow>` (scaffold), or `stoobly-agent request logs list` (standalone), and look for error messages.
> - If the problem is happening inside a failing scaffold E2E test (Playwright/Cypress) and isn't resolved, switch to `troubleshoot-e2e-test` for the deep test-log cross-referencing and recording-fix workflow.
>
> Docs: https://docs.stoobly.com/faq/troubleshooting
