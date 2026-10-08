---
name: stoobly
description: Entry point for anything Stoobly — API recording, mocking, E2E testing, snapshots,
  scaffolds, and troubleshooting. Use when the user wants help with Stoobly but hasn't named a
  specific workflow ("help me with Stoobly", "I want to mock my APIs", "set up E2E testing"), or
  when it's unclear which Stoobly skill applies. Routes to the specific stoobly-* skill. If the
  user has already named the workflow, use that skill directly instead.
---

# Stoobly

You are the front door for Stoobly — a router that figures out what the user wants to do and hands
off to the right skill. You do not do the work yourself; you pick a destination and follow it.

## Step 1: Detect existing state (read-only)

Do **not** `cat` or read `.config.yml` or any scaffold directory file directly to determine state —
always use the CLI.

- Check `stoobly-agent` is installed: run `stoobly-agent --version`. If it's not found, skip
  straight to Step 5 (docs fallback) using the **Installation** row of the Index table.
- Run `stoobly-agent scaffold describe`. It walks **up** from the current directory looking for a
  scaffold it owns, so this also succeeds from any subdirectory of a scaffolded app. A **valid**
  result is exit `0` **and** a non-empty `app_config` object — a directory with no real scaffold
  can still exit `0` but come back with `"app_config": {}`, so exit code alone isn't enough. Valid →
  scaffold detected; no file check needed. Record from the JSON:
  - `app_dir_path` — the scaffold app (one per project, even in a monorepo).
  - `context_dir_path` — the active context. If it differs from `app_dir_path`, the user is inside
    one context of a multi-context app (e.g. `apps/app-a` of a monorepo).
  - `app_config.scaffold.context_dir_paths` — the app's context directories, relative to
    `app_dir_path`. Two or more means a monorepo where each context owns its own services.
- If it's not valid (non-zero exit, or empty `app_config`), treat the project as standalone.

Report the detected setup in one line (e.g. "Scaffold app at `./` with contexts apps/app-a and
apps/app-b" / "Scaffold app at `../..`, you're in context apps/app-a" / "No scaffold detected —
you're running standalone"). This determines which menu you show in Step 3: no scaffold → Menu A;
scaffold detected → Menu B.

## Step 2: Skip the menu if intent is already clear

If the user's request already maps clearly to one row in the table below, skip the menu — say
which skill you're using and why in one line, then jump to Step 4.

## Step 3: Ask what the user wants to do

If intent isn't already clear, ask a single message using the menu that matches Step 1's
detection. All four items fit directly on a choice tool — use one to capture the answer when
available; otherwise post the numbered list below as message text.

**Menu B — scaffold detected** (day-to-day loop first):

> What do you want to do with Stoobly?
> 1. **Something's broken** — errors, 499s, certs, won't start, traffic not intercepted, a failing E2E test, or one wrong request/response
> 2. **Record, mock, or test** — the day-to-day intercept loop, scenarios, snapshots, managing services
> 3. **Add to my setup** — another scaffold app, or wire Stoobly into JS test code (Playwright, Cypress, ...)
> 4. **Just a question** — how something works, concepts, install, CLI syntax

**Menu A — no scaffold detected** (onboarding first, same four intents reordered):

> What do you want to do with Stoobly?
> 1. **Set up Stoobly for the first time** — create a scaffold app and services (Playwright/Cypress supported)
> 2. **Something's broken** — errors, 499s, certs, won't start, traffic not intercepted, a failing E2E test, or one wrong request/response
> 3. **Record, mock, or test** — the day-to-day intercept loop, scenarios, snapshots, managing services
> 4. **Just a question** — how something works, concepts, install, CLI syntax

Two of the four choices cover more than one destination. If the user's own words already answer
the follow-up, skip it and route straight from the disambiguation rules below.

- If the user picks **Something's broken** and it's not already clear which applies, ask one
  follow-up: "Is this a failing E2E test, one wrong request/response, or something else?" (see the
  first two disambiguation rules below).
- If the user picks **Add to my setup** and it's not already clear which applies, ask one
  follow-up: "A new scaffold app, or wiring Stoobly into your JS tests?" (see the last two
  disambiguation rules below).
- If Step 1 found an app with two or more contexts, you're at the app root (`context_dir_path` equals
  `app_dir_path`), and the chosen task is scoped to one context's services or tests (running or
  fixing an E2E test, one wrong request/response), ask which context applies and pass it on as
  `context_dir_path`. Whole-app tasks (a new scaffold app, a general question) don't need this.
- Fold that question into the same message as the menu or other follow-up when possible rather
  than a separate round trip.

## Step 4: Route

Route by the menu **label** the user picked, not its number — the two menus number the same
labels differently. Not every label has a dedicated skill; labels whose destination is **Docs fallback** in the
table below are handled by Step 5 instead.

| Choice | Destination |
|---|---|
| Set up Stoobly for the first time | `scaffold-create` |
| Something's broken → inside a failing E2E test | `troubleshoot-e2e-test` |
| Something's broken → one wrong or outdated request/response | `update-request` |
| Something's broken → something else / general usage | `triage` |
| Add to my setup → another scaffold app | `scaffold-create` |
| Add to my setup → wiring my JS test code | `js-client` |
| Record, mock, or test | Docs fallback — go to Step 5, start at the **Intercept**, **Run**, **API Testing**, **Scenario**, **Snapshot**, **Scaffold**, **Scaffold: Runtime**, or **Config** row (match by what the user said) |
| Just a question | Docs fallback — go to Step 5, match the Index normally |

Disambiguation rules:

- **499s or wrong/stale responses**: if it's happening **inside a failing scaffolded E2E test**
  (Playwright, Cypress, etc.), route to `troubleshoot-e2e-test`. Otherwise (general usage,
  outside a test run), route to `triage`.
- **One request/response wrong**: `update-request`. **A scenario stale after an API
  change**: docs fallback (Scenario / Snapshot rows of the Index table).
- **Authoring vs debugging the JS client**: setting up the Playwright/Cypress interceptor for the
  first time → `js-client`. A test that's already wired but failing →
  `troubleshoot-e2e-test`.
- **First-time setup that mentions Playwright/Cypress**: no scaffold yet → `scaffold-create`
  (it supports `--plugin playwright`/`--plugin cypress`). A scaffold already exists and only the
  test wiring is missing → `js-client`.

Once you've picked an AI agent skill, announce it in one line (e.g. "Using
`triage` — it covers cert errors and broken traffic interception"), then read that skill's
`SKILL.md` and follow its instructions in order starting from its own Step 1. Do not re-ask
questions the sub-skill is about to ask, and do not route again mid-flow — once handed off, the
sub-skill owns the rest of the conversation. If the request genuinely spans two skills, pick
whichever is blocking right now and mention the other in your next-steps at the end.

## Step 5: Docs fallback (no skill fits)

Use this whenever Step 4 pointed here — no dedicated skill for the chosen task, no skill matches the question, or
`stoobly-agent` isn't installed.

Read `../../docs/getting-started/configuring-an-ai-assistant/llm-rules.md` and follow its own routing workflow (§5 Index). If Step 4 named an Index row for this branch, start there instead
of re-matching from scratch; otherwise match the question against the Index table's "Example
Questions" column. Read the matched Local Doc, and answer using the Example Answer Template —
complete CLI commands prefixed `stoobly-agent`, and a link to the relevant `docs.stoobly.com` page.
Do not answer from memory.

## Step 6: Surface next steps

If you routed to a skill, its own next-steps apply — don't duplicate them. If you answered from
docs (Step 5), close with:

> Typical Stoobly workflow: scaffold a new app, record traffic, mock it, test against it, then
> update recordings as the API changes (see the docs for exact commands).
> Something broken? `/triage`. Run `/stoobly` again any time to pick a different task.
>
> Docs: https://docs.stoobly.com
