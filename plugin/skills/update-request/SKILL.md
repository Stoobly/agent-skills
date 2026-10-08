---
name: update-request
description: Updates a recorded request and/or its response via the Stoobly CLI, then persists the change with a request snapshot. Use when a mock response is wrong or outdated, a recorded request body/URL/method/headers need changing, or the user asks to update a request or response without editing snapshot files by hand.
---

# Update Request

You are helping the user update a recorded request and/or its response.

## Instructions

Follow these steps when a stored request or response needs to change. Do not edit snapshot files directly. Scaffold E2E tests version snapshots under `.stoobly/snapshots/`.

### 1. Resolve the context directory

Run all `request` and `scenario` commands from the context directory (the directory that contains `.stoobly`).

If you do not already know the path, run:

```bash
stoobly-agent scaffold describe
```

Use `context_dir_path` from the JSON output:

```bash
cd <context_dir_path>
```

### 2. Identify the request key

You need `<REQUEST-KEY>` before updating. If the caller already provided it, skip to Step 3.

**From a scenario:**

```bash
stoobly-agent scenario list --search "<scenario or test name>"
stoobly-agent request list --scenario-key <SCENARIO-KEY>
```

**From a URL or path search:**

```bash
stoobly-agent request list --search "/api/users"
```

Note the `key` for the request to update.

### 3. Inspect the current request and response

```bash
stoobly-agent request response show <REQUEST-KEY>
```

Compare the stored request/response with the expected values (body, status, headers, method, path, and so on).

### 4. Update the local database

Apply only the fields that need to change.

**Response** — body, status, headers, or latency:

```bash
stoobly-agent request response update <REQUEST-KEY> --body '{"id":1}'
stoobly-agent request response update <REQUEST-KEY> --status 201
stoobly-agent request response update <REQUEST-KEY> --header Content-Type:application/json
stoobly-agent request response update <REQUEST-KEY> --header X-Deprecated:
stoobly-agent request response update <REQUEST-KEY> --latency 500
```

**Request** — body, URL parts, method, or headers:

```bash
stoobly-agent request update <REQUEST-KEY> --body '{"name":"Alice"}'
stoobly-agent request update <REQUEST-KEY> --method POST --path /api/users
stoobly-agent request update <REQUEST-KEY> --header Authorization:Bearer token
```

Use `--header NAME:` with an empty value to delete a header.

### 5. Verify the update

```bash
stoobly-agent request response show <REQUEST-KEY>
```

Confirm the stored data matches expectations. If this change is for a failing E2E test, re-run that test to confirm the mock behaves correctly before persisting anything to version control.

### 6. Persist to version-controlled snapshots

```bash
stoobly-agent request snapshot create <REQUEST-KEY> --decode
```

Commit the updated snapshot file(s) under `.stoobly/snapshots/` so the change is available locally and in CI.

## Reference documentation

| Topic | Doc |
| ----- | --- |
| Request commands | [Request](../../docs/faq/request.md) |
| Snapshots | [Snapshot](../../docs/faq/snapshot.md) |
| Updating requests | [How to Update Requests](../../docs/guides/how-to-update-requests/README.md) |
| Troubleshoot E2E test | [Troubleshoot E2E Test](troubleshoot-e2e-test.md) |
