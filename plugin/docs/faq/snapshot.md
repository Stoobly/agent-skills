# Snapshot

## Stoobly Snapshot CLI - Questions & Answers

The snapshot CLI manages version-controlled snapshots of requests and scenarios. Snapshots create committable files that enable team collaboration, historical tracking, and reproducible testing through version control systems like Git.

***

### Understanding Snapshots

#### Q: What are snapshots in Stoobly?

**A:** Snapshots are version-controlled, file-based representations of requests and scenarios that can be committed to Git. They enable team collaboration and historical tracking of API tests.

**Example:**

```bash
# Create snapshot of a request
stoobly-agent request snapshot create "<REQUEST-KEY>"

# Create snapshot of a scenario
stoobly-agent scenario snapshot create "<SCENARIO-KEY>"

# Snapshots are stored in .stoobly/snapshots/
ls .stoobly/snapshots/requests/
ls .stoobly/snapshots/scenarios/
```

#### Q: Why should I use snapshots?

**A:** Snapshots enable version control of your tests, team collaboration through Git, reproducible testing across environments, and historical tracking of API changes.

**Example:**

```bash
# Create snapshots
stoobly-agent scenario snapshot create user-login

# Commit to Git
git add .stoobly/snapshots/
git commit -m "Add user login test scenario"
git push

# Team members get the tests
git pull
stoobly-agent snapshot apply  # Apply all snapshots
```

#### Q: What's the difference between requests in the database and snapshots?

**A:** Database requests are local and ephemeral, while snapshots are file-based, version-controlled, and shareable across the team.

**Example:**

```bash
# Database: Local only
stoobly-agent request list  # Shows local requests

# Snapshots: Shareable via Git
stoobly-agent snapshot list  # Shows version-controlled snapshots
git add .stoobly/snapshots/ && git commit -m "Share tests"
```

***

### Listing Snapshots

#### Q: How do I view all snapshots?

**A:** Use resource-specific list commands to display snapshots. List request snapshots with `request snapshot list` or scenario snapshots with `scenario snapshot list`.

**Example:**

```bash
# List all request snapshots
stoobly-agent request snapshot list

# List all scenario snapshots
stoobly-agent scenario snapshot list
```

#### Q: How do I list only request snapshots?

**A:** Use `request snapshot list` to display all request snapshots.

**Example:**

```bash
# List request snapshots
stoobly-agent request snapshot list

# Show more results
stoobly-agent request snapshot list --size 50
```

#### Q: How do I list only scenario snapshots?

**A:** Use `scenario snapshot list` to display all scenario snapshots.

**Example:**

```bash
stoobly-agent scenario snapshot list
```

#### Q: How do I search for specific snapshots?

**A:** Use the `--search` option with a regex pattern to filter snapshots. The search behavior depends on the resource type:

- **For request snapshots** (using `request snapshot list`): The regex matches from the start of the URL or the start of the path. For example, `docs.stoobly.com` will not match `https://docs.stoobly.com` because the search starts from the beginning of the URL. Use `https://docs.stoobly.com` or `.*?docs.stoobly.com` to match URLs containing the domain.
- **For scenario snapshots** (using `scenario snapshot list`): The regex matches from the start of the scenario name or the start of the scenario description.

**Example:**

```bash
# Search request snapshots by URL path (matches from start of path)
stoobly-agent request snapshot list --search "/api/users"

# Search request snapshots by domain (use .*? to match anywhere in URL)
stoobly-agent request snapshot list --search ".*?example.com"

# Search request snapshots with regex (matches paths starting with /api/ and containing login)
stoobly-agent request snapshot list --search "^/api/.*login"

# Search scenario names/descriptions (matches from start)
stoobly-agent scenario snapshot list --search "^User.*"
```

#### Q: How do I filter snapshots by scenario key?

**A:** Use the `--scenario-key` option with `request snapshot list` to filter request snapshots that belong to a specific scenario.

**Example:**

```bash
# Filter request snapshots by scenario key
stoobly-agent request snapshot list --scenario-key "<SCENARIO-KEY>"

# Filter with regex pattern
stoobly-agent request snapshot list --scenario-key "^Login.*"
```

#### Q: How do I list pending (unprocessed) snapshots?

**A:** Use the `--pending` flag with resource-specific list commands to show snapshots that haven't been applied yet.

**Example:**

```bash
# List pending request snapshots
stoobly-agent request snapshot list --pending

# List pending scenario snapshots
stoobly-agent scenario snapshot list --pending
```

#### Q: How do I limit the number of snapshots displayed?

**A:** Use the `--size` option with resource-specific list commands to control the number of results.

**Example:**

```bash
# Show 20 request snapshots
stoobly-agent request snapshot list --size 20

# Show 50 scenario snapshots
stoobly-agent scenario snapshot list --size 50
```

#### Q: How do I format snapshot list output?

**A:** Use the `--format` option with resource-specific list commands to change output format.

**Example:**

```bash
# Table format (default)
stoobly-agent request snapshot list --format table

# JSON format
stoobly-agent request snapshot list --format json

# CSV format
stoobly-agent scenario snapshot list --format csv
```

#### Q: How do I select specific columns to display?

**A:** Use the `--select` option with resource-specific list commands to choose which columns to show.

**Example:**

```bash
# Show specific columns
stoobly-agent request snapshot list --select uuid --select path --select method

# Minimal output
stoobly-agent request snapshot list --select uuid --select snapshot --without-headers
```

***

### Applying Snapshots

#### Q: How do I apply all snapshots?

**A:** Use `snapshot apply` without arguments to apply all available snapshots.

**Example:**

```bash
# Apply all snapshots from .stoobly/snapshots/
stoobly-agent snapshot apply
```

#### Q: How do I apply a specific snapshot?

**A:** Use `snapshot apply` with the snapshot UUID.

**Example:**

```bash
# Get UUID from list
stoobly-agent snapshot list

# Apply specific snapshot
stoobly-agent snapshot apply "<SNAPSHOT-UUID>"
```

#### Q: What happens when I apply a snapshot?

**A:** Applying a snapshot creates or updates the corresponding request or scenario in your local database from the snapshot file.

**Example:**

```bash
# Team member gets snapshots from git
git pull

# Apply snapshots to local database
stoobly-agent snapshot apply

# Now can replay/test the requests
stoobly-agent scenario replay user-login
```

#### Q: How do I force apply snapshots with hard delete?

**A:** Use the `--force` flag to hard delete existing resources when applying.

**Example:**

```bash
# Force apply all snapshots
stoobly-agent snapshot apply --force

# Force apply specific snapshot
stoobly-agent snapshot apply "<SNAPSHOT-UUID>" --force
```

#### Q: What's the difference between apply and reset?

**A:** These commands serve different purposes:

- **`snapshot apply`**: Creates or updates requests/scenarios in your database **from snapshot files**. Use this when you have snapshot files (e.g., after pulling from git) and want to build your database from them. This is a bulk operation that processes snapshot files.

- **`request reset` or `scenario reset`**: Reverts a specific request or scenario that **already exists in your database** back to its snapshot state. Use this when you've made changes to a request/scenario and want to undo those changes by restoring from its snapshot.

**Example:**

```bash
# Apply all snapshots (build database from snapshot files)
# Use this after pulling snapshots from git or when setting up a new environment
stoobly-agent snapshot apply

# Reset specific request (revert changes to an existing request)
# Use this when you want to undo changes to a request you've modified
stoobly-agent request reset "<REQUEST-KEY>"

# Reset specific scenario (revert changes to an existing scenario)
# Use this when you want to undo changes to a scenario you've modified
stoobly-agent scenario reset "<SCENARIO-KEY>"
```

***

### Updating Snapshots

#### Q: How do I create/update a request snapshot?

**A:** Use `request snapshot create` with the request key to create or update a request snapshot.

**Example:**

```bash
# Create or update a request snapshot
stoobly-agent request snapshot create "<REQUEST-KEY>"

# Create snapshot with decoded response bodies
stoobly-agent request snapshot create "<REQUEST-KEY>" --decode
```

#### Q: How do I create/update a scenario snapshot?

**A:** Use `scenario snapshot create` with the scenario key to create or update a scenario snapshot.

**Example:**

```bash
# Create or update a scenario snapshot
stoobly-agent scenario snapshot create "<SCENARIO-KEY>"

# Create snapshot with decoded response bodies
stoobly-agent scenario snapshot create "<SCENARIO-KEY>" --decode
```

#### Q: How do I update a snapshot without verification?

**A:** Use the `--no-verify` flag to skip request verification.

**Example:**

```bash
stoobly-agent request snapshot create "<REQUEST-KEY>" --no-verify
stoobly-agent scenario snapshot create "<SCENARIO-KEY>" --no-verify
```

#### Q: What does snapshot verification do?

**A:** Verification ensures the raw HTTP request format is valid and fixes any formatting issues before creating the snapshot.

**Example:**

```bash
# With verification (default) - fixes malformed requests
stoobly-agent request snapshot create "<REQUEST-KEY>"

# Without verification - takes request as-is
stoobly-agent request snapshot create "<REQUEST-KEY>" --no-verify
```

***

### Resetting Snapshots

#### Q: How do I reset a request to its snapshot state?

**A:** Use `request snapshot reset` with the request key to revert a request back to its last snapshot state. This will discard any changes made to the request since the snapshot was created.

**Example:**

```bash
stoobly-agent request snapshot reset "<REQUEST-KEY>"
```

#### Q: What happens when I reset a request?

**A:** Reset will:
1. Load the request data from the last snapshot file
2. Overwrite the current request in your local database with snapshot data
3. Restore the exact request state from when the snapshot was created
4. Discard all changes made since the snapshot was created

**Example:**

```bash
# Before reset: Request has modifications
# After reset: Request matches snapshot state exactly

stoobly-agent request snapshot reset "<REQUEST-KEY>"
```

#### Q: How do I know what will be lost when I reset?

**A:** Use `request snapshot diff` to preview changes before resetting.

**Example:**

```bash
# See what will be lost
stoobly-agent request snapshot diff "<REQUEST-KEY>"

# Review the diff output carefully

# If you want to proceed with reset:
stoobly-agent request snapshot reset "<REQUEST-KEY>"
```

#### Q: How do I reset a request with force delete?

**A:** Use the `--force` flag to hard delete the request before restoring it from snapshot. This is useful if the request is in a corrupted state.

**Example:**

```bash
# Hard delete and restore from snapshot
stoobly-agent request snapshot reset "<REQUEST-KEY>" --force
```

#### Q: What's the difference between reset with and without --force?

**A:**

- **Without `--force`**: Updates the existing request with snapshot data. Safer option that preserves record history.
- **With `--force`**: Hard deletes the request completely, then recreates it from snapshot. Use this if the request is corrupted or if you want a clean state.

**Example:**

```bash
# Normal reset: Updates existing request
stoobly-agent request snapshot reset "<REQUEST-KEY>"

# Force reset: Delete and recreate from snapshot
stoobly-agent request snapshot reset "<REQUEST-KEY>" --force
```

#### Q: Can I undo a reset?

**A:** No, reset is permanent. Once a reset is applied, the changes are lost. However, you can:
1. Use `request snapshot diff` before resetting to understand what will change
2. Check Git history if you have snapshots committed
3. Use your database backups if available

**Example:**

```bash
# Always check diff before resetting
stoobly-agent request snapshot diff "<REQUEST-KEY>"

# If you made a mistake, check if there's a Git history
git log .stoobly/snapshots/

# Or restore from backup if available
```

#### Q: How do I reset a scenario to its snapshot state?

**A:** Use `scenario snapshot reset` with the scenario key to revert a scenario back to its last snapshot state.

**Example:**

```bash
stoobly-agent scenario snapshot reset "<SCENARIO-KEY>"
```

#### Q: What happens when I reset a scenario?

**A:** Reset will:
1. Load the scenario metadata from the last snapshot file
2. Update the scenario name and description from the snapshot
3. Restore all requests in the scenario to their snapshot states
4. Discard all changes made to the scenario and its requests since the snapshot

**Example:**

```bash
# Before reset: Scenario has modifications
# After reset: Scenario and all its requests match snapshot state

stoobly-agent scenario snapshot reset "<SCENARIO-KEY>"
```

#### Q: How do I know what will be lost when I reset a scenario?

**A:** Use `scenario snapshot diff` to preview all changes before resetting.

**Example:**

```bash
# See what will change in the scenario
stoobly-agent scenario snapshot diff "<SCENARIO-KEY>"

# See full raw HTTP diffs for all requests
stoobly-agent scenario snapshot diff "<SCENARIO-KEY>" --full

# Review carefully, then reset if needed
stoobly-agent scenario snapshot reset "<SCENARIO-KEY>"
```

#### Q: How do I reset a scenario with force delete?

**A:** Use the `--force` flag to hard delete the scenario before restoring it from snapshot.

**Example:**

```bash
# Hard delete scenario and restore from snapshot
stoobly-agent scenario snapshot reset "<SCENARIO-KEY>" --force
```

#### Q: When should I use reset?

**A:** Reset is useful when:
- You made accidental changes to requests or scenarios
- You want to undo modifications and return to a known good state
- You want to sync with the committed snapshot in Git
- You need to verify that the snapshot is still valid
- The scenario got corrupted and needs a clean restore

**Example Workflow:**

```bash
# Make changes to scenario
stoobly-agent record https://api.example.com/new-endpoint --scenario-key "<SCENARIO-KEY>"

# Realize the changes aren't needed
stoobly-agent scenario snapshot diff "<SCENARIO-KEY>"

# Check what changed, decide it's not needed

# Reset to last snapshot
stoobly-agent scenario snapshot reset "<SCENARIO-KEY>"

# Scenario is back to snapshot state
```

#### Q: Can I reset multiple requests or scenarios at once?

**A:** Use a script to reset multiple resources.

**Example:**

```bash
#!/bin/bash
# Reset all requests in a scenario

# List all request snapshots and reset each
stoobly-agent request snapshot list --format json | jq -r '.[].key' | while read key; do
  echo "Resetting request: $key"
  stoobly-agent request snapshot reset "$key"
done

# Reset all scenarios
stoobly-agent scenario snapshot list --format json | jq -r '.[].key' | while read key; do
  echo "Resetting scenario: $key"
  stoobly-agent scenario snapshot reset "$key"
done
```

#### Q: What's the difference between reset and apply?

**A:**

- **`reset`**: Reverts a specific resource (request or scenario) back to its snapshot state. Works on individual resources you've already modified.
- **`apply`**: Applies snapshots from snapshot files to create or update resources in your database. Works on unprocessed snapshots (typically after pulling from Git).

**Example:**

```bash
# Use apply when you pull snapshots from Git
git pull
stoobly-agent snapshot apply  # Creates/updates resources from snapshot files

# Use reset when you've modified a resource and want to undo
stoobly-agent request snapshot reset "<REQUEST-KEY>"  # Discard changes

# Use diff to understand the difference
stoobly-agent request snapshot diff "<REQUEST-KEY>"
```

***

### Global Snapshot Reset

#### Q: How do I reset all requests and scenarios to their snapshot states at once?

**A:** Use `snapshot reset` (without a resource key) to reset all requests and scenarios that have snapshots back to their last snapshot state. This is useful when you've made many changes and want to reset everything.

**Example:**

```bash
# Reset all requests and scenarios to snapshot states
stoobly-agent snapshot reset

# Will prompt for confirmation showing count of items to reset
# This will reset 5 request(s) and 3 scenario(s) to their last snapshot state (8 total). Continue?
```

#### Q: What happens when I run snapshot reset?

**A:** The command will:
1. Count all requests and scenarios that have snapshots
2. Ask for confirmation (unless `--yes` is used)
3. Reset each resource to its snapshot state
4. Report success/failure count for the operation

**Example:**

```bash
stoobly-agent snapshot reset
# Output:
# This will reset 5 request(s) and 3 scenario(s) to their last snapshot state (8 total).
# WARNING: --hard is set and will delete ALL requests and scenarios before resetting.
# Continue? [y/N]: y
# Successfully reset the request!
# Successfully reset the request!
# ... (more items)
```

#### Q: How do I skip the confirmation prompt?

**A:** Use the `--yes` flag to proceed without confirmation. Useful for automated scripts.

**Example:**

```bash
# Skip confirmation and reset all immediately
stoobly-agent snapshot reset --yes
```

#### Q: What's the --hard option for snapshot reset?

**A:** The `--hard` flag deletes ALL requests and scenarios in your database before resetting them from snapshots. This is a destructive operation that should be used carefully.

**Important:** `--hard` will delete:
- All requests not in snapshots
- All scenarios not in snapshots
- All requests in snapshots (they'll be recreated)
- All scenarios in snapshots (they'll be recreated)

**Example:**

```bash
# Hard reset: delete everything, then restore from snapshots
stoobly-agent snapshot reset --hard

# This will show a warning before proceeding:
# This will reset 5 request(s) and 3 scenario(s) to their last snapshot state (8 total).
# WARNING: --hard is set and will delete ALL requests and scenarios before resetting.
# Continue? [y/N]:
```

#### Q: When should I use --hard?

**A:** Use `--hard` in these situations:

- **Clean slate needed**: You want to remove all local changes and start fresh from snapshots
- **Database corruption**: Your database has inconsistent or corrupted data
- **Fresh environment setup**: Setting up a new environment from snapshots
- **Testing snapshot integrity**: Verify that all snapshots can be applied cleanly
- **Cleanup**: Remove clutter and unwanted requests/scenarios not in snapshots

**Example Workflows:**

```bash
# Workflow 1: Clean environment from Git
git clone <repo>
cd <repo>
stoobly-agent snapshot reset --hard --yes
# Result: Database contains only snapshot-defined requests/scenarios

# Workflow 2: Recover from corruption
stoobly-agent snapshot reset --hard
# All data reset from snapshots

# Workflow 3: Test snapshot integrity
stoobly-agent snapshot reset --hard --yes
stoobly-agent snapshot list  # Verify all snapshots present
```

#### Q: What's the difference between --hard and normal reset?

**A:**

| Aspect | Normal Reset | With --hard |
|--------|-------------|-----------|
| Deletes unsnapshot items? | No | Yes |
| Deletes snapshot items? | No | Yes (recreates them) |
| Safe for normal use? | Yes | No (destructive) |
| Data loss risk? | Low | High |
| Best for? | Undo changes | Fresh start |

**Example:**

```bash
# Setup:
# Database has: request-a (snapshot), request-b (no snapshot)

# Normal reset:
stoobly-agent snapshot reset
# Result: request-a reset to snapshot, request-b unchanged

# With --hard:
stoobly-agent snapshot reset --hard
# Result: request-a deleted and recreated, request-b deleted
```

#### Q: What should I do before using --hard?

**A:** Before using `--hard`, take these safety measures:

1. Backup your database
2. Review what will be deleted with `snapshot list`
3. Commit snapshots to Git
4. Consider the impact on your team

**Example Safety Workflow:**

```bash
# Step 1: Check what will change
stoobly-agent request snapshot list
stoobly-agent scenario snapshot list

# Step 2: Backup database
cp -r .stoobly/db .stoobly/db.backup.$(date +%s)

# Step 3: Commit snapshots to Git
git add .stoobly/snapshots/
git commit -m "Backup before hard reset"
git push

# Step 4: Run with caution
stoobly-agent snapshot reset --hard --yes
```

#### Q: Can I undo a hard reset?

**A:** Hard reset is permanent. To recover:

1. **Restore from backup**: If you created a database backup
2. **Restore from Git**: If you committed snapshots before the reset
3. **Re-record data**: If you have the original traffic

**Example Recovery:**

```bash
# Option 1: Restore database backup
rm -rf .stoobly/db
cp -r .stoobly/db.backup.1234567 .stoobly/db

# Option 2: Restore from Git (if you committed before reset)
git log --oneline .stoobly/snapshots/
git checkout <commit-hash> -- .stoobly/snapshots/
stoobly-agent snapshot apply

# Option 3: Re-record from traffic
# Re-run your application with recording enabled
stoobly-agent run --intercept --intercept-mode record &
# Run your tests/app
# Stop and record new snapshots
```

#### Q: How does --lock-timeout work?

**A:** The `--lock-timeout` option prevents concurrent reset operations that could corrupt your database. Only one reset can run at a time within the specified timeout.

**Example:**

```bash
# Default timeout is 60 seconds
stoobly-agent snapshot reset

# Custom timeout (wait up to 120 seconds for another reset to finish)
stoobly-agent snapshot reset --lock-timeout 120

# If another reset is already running and timeout expires:
# Error: Another snapshot reset command is already running. Please wait for it to complete.
```

#### Q: Can I reset while other processes are running?

**A:** The lock mechanism ensures only one `snapshot reset` can run at a time. However, it's best practice to:

1. Stop your application
2. Stop the Stoobly agent if running
3. Run the reset
4. Restart applications

**Example Safe Reset:**

```bash
# Stop running processes
stoobly-agent stop  # if running

# Perform reset
stoobly-agent snapshot reset --hard --yes

# Verify everything is correct
stoobly-agent snapshot list
stoobly-agent request list

# Restart
stoobly-agent run --intercept
```

#### Q: How do I use snapshot reset in scripts or CI/CD?

**A:** Use the `--yes` flag to skip confirmation, and check exit codes to handle errors.

**Example Bash Script:**

```bash
#!/bin/bash
set -e  # Exit on error

echo "Resetting all snapshots..."
stoobly-agent snapshot reset --yes

if [ $? -eq 0 ]; then
  echo "Reset successful"
  stoobly-agent snapshot list
else
  echo "Reset failed" >&2
  exit 1
fi
```

**Example GitHub Actions:**

```yaml
- name: Reset snapshots
  run: stoobly-agent snapshot reset --hard --yes
  continue-on-error: true  # Optional: continue if reset fails

- name: Verify snapshots
  run: stoobly-agent request snapshot list
```

#### Q: What error messages might I see?

**A:** Common error messages and what they mean:

- **"Another snapshot reset command is already running"**: Wait for the other reset to complete
- **"Completed with X failures (Y succeeded)"**: Some resources failed to reset; check error logs
- **"Aborted."**: You rejected the confirmation prompt

**Example Troubleshooting:**

```bash
# If getting lock timeout error
# Check if another reset is running
ps aux | grep snapshot

# Kill stuck process if needed (use with caution)
pkill -f "snapshot reset"

# Try again
stoobly-agent snapshot reset --yes
```

#### Q: What counts as a "snapshot" for the reset operation?

**A:** Snapshots are counted from the snapshot log. Only requests and scenarios with PUT_ACTION events in the snapshot log are included.

**Example:**

```bash
# To see what will be reset:
stoobly-agent request snapshot list
stoobly-agent scenario snapshot list

# These define what "snapshot reset" will operate on
```

***

### Comparing Snapshots

#### Q: How do I see the diff between a request and its snapshot?

**A:** Use `request snapshot diff` to show differences between the current request stored in your database and its last snapshot state.

**Example:**

```bash
# Show diff for a specific request
stoobly-agent request snapshot diff "<REQUEST-KEY>"
```

#### Q: What information does the diff show for a request?

**A:** The diff displays:
- Current request data from your local database
- Snapshot request data from the last saved version
- Detailed comparison of request properties (URL, method, headers, body)
- Response body comparisons

**Example:**

```bash
# See what changed in a request
stoobly-agent request snapshot diff "<REQUEST-KEY>"
# Output shows: added/removed/modified fields in headers, body, or response
```

#### Q: How do I see the full raw diff for all requests?

**A:** Use the `--full` flag to display the complete raw HTTP request diff for a specific request.

**Example:**

```bash
# Show full raw HTTP request diff
stoobly-agent request snapshot diff "<REQUEST-KEY>" --full

# Show raw diff for all requests with diffs
stoobly-agent request snapshot diff --full
```

#### Q: What does the --full flag show?

**A:** The `--full` flag displays the complete raw HTTP request format, showing exactly how the request has changed at the HTTP level.

**Example:**

```bash
# Without --full (summary view)
stoobly-agent request snapshot diff "<REQUEST-KEY>"
# Output: Shows parsed fields like method, path, headers, body

# With --full (raw HTTP view)
stoobly-agent request snapshot diff "<REQUEST-KEY>" --full
# Output: Shows full raw HTTP request format with all changes highlighted
```

#### Q: How do I see diffs for all requests that have changed?

**A:** Run `request snapshot diff` without specifying a request key to see diffs for all requests with snapshots.

**Example:**

```bash
# Show diffs for all requests with changes
stoobly-agent request snapshot diff

# Show full raw diffs for all requests
stoobly-agent request snapshot diff --full
```

#### Q: How do I see the diff between a scenario and its snapshot?

**A:** Use `scenario snapshot diff` to show differences between the current scenario stored in your database and its last snapshot state.

**Example:**

```bash
# Show diff for a specific scenario
stoobly-agent scenario snapshot diff "<SCENARIO-KEY>"
```

#### Q: What information does the scenario diff show?

**A:** The scenario diff displays:
- Scenario metadata changes (name, description)
- Request changes within the scenario
- For each request: added/removed/modified properties
- Overall scenario composition differences

**Example:**

```bash
# See what changed in a scenario
stoobly-agent scenario snapshot diff "<SCENARIO-KEY>"
# Output shows: scenario name/description changes and request changes
```

#### Q: How do I see full raw diffs for all requests in a scenario?

**A:** Use the `--full` flag to display complete raw HTTP diffs for all requests in the scenario.

**Example:**

```bash
# Show full raw request diffs within a scenario
stoobly-agent scenario snapshot diff "<SCENARIO-KEY>" --full

# Show raw diffs for all scenarios with changes
stoobly-agent scenario snapshot diff --full
```

#### Q: How do I see diffs for all scenarios that have changed?

**A:** Run `scenario snapshot diff` without specifying a scenario key to see diffs for all scenarios with snapshots.

**Example:**

```bash
# Show diffs for all scenarios with changes
stoobly-agent scenario snapshot diff

# Show full raw diffs for all scenarios
stoobly-agent scenario snapshot diff --full
```

#### Q: When should I use diff before resetting?

**A:** Always check the diff before resetting to ensure you want to discard the changes. The diff shows you exactly what will be lost.

**Example:**

```bash
# Check what changed
stoobly-agent request snapshot diff "<REQUEST-KEY>"

# Review the changes carefully

# If you want to keep the changes, don't reset
# If you want to discard them and go back to snapshot:
stoobly-agent request snapshot reset "<REQUEST-KEY>"
```

#### Q: Can I use diff to find requests that match a pattern?

**A:** The `request snapshot diff` command will show diffs for all requests with snapshots. You can filter by specifying a request key or by piping output.

**Example:**

```bash
# Show diffs for all requests to find which ones changed
stoobly-agent request snapshot diff

# Save diffs to a file for review
stoobly-agent request snapshot diff --full > changes.txt
```

***

### Copying Snapshots

#### Q: How do I copy snapshots to a different directory?

**A:** Use `snapshot copy` with the destination path to copy snapshots between data directories.

**Example:**

```bash
# Copy request snapshot to different directory
stoobly-agent snapshot copy /path/to/other/project --request-key "<REQUEST-KEY>"

# Copy scenario snapshot
stoobly-agent snapshot copy /path/to/other/project --scenario-key "<SCENARIO-KEY>"
```

#### Q: How do I copy multiple requests at once?

**A:** Use multiple `--request-key` options to copy several requests.

**Example:**

```bash
stoobly-agent snapshot copy /path/to/destination \
  --request-key "<REQUEST-KEY-1>" \
  --request-key "<REQUEST-KEY-2>" \
  --request-key "<REQUEST-KEY-3>"
```

#### Q: How do I copy multiple scenarios at once?

**A:** Use multiple `--scenario-key` options to copy several scenarios.

**Example:**

```bash
stoobly-agent snapshot copy /path/to/destination \
  --scenario-key "<SCENARIO-KEY-1>" \
  --scenario-key "<SCENARIO-KEY-2>" \
  --scenario-key "<SCENARIO-KEY-3>"
```

#### Q: How do I copy both requests and scenarios together?

**A:** Combine both `--request-key` and `--scenario-key` options.

**Example:**

```bash
stoobly-agent snapshot copy /path/to/destination \
  --request-key "<REQUEST-KEY-1>" \
  --request-key "<REQUEST-KEY-2>" \
  --scenario-key "<SCENARIO-KEY-1>" \
  --scenario-key "<SCENARIO-KEY-2>"
```

#### Q: Why would I copy snapshots to a different directory?

**A:** Copying snapshots is useful for moving tests between projects, creating backups, or setting up separate test environments.

**Example:**

```bash
# Copy production tests to staging environment
stoobly-agent snapshot copy /path/to/staging \
  --scenario-key "<SCENARIO-KEY-1>" \
  --scenario-key "<SCENARIO-KEY-2>"

# In staging directory
cd /path/to/staging
stoobly-agent snapshot apply
```

***

### Pruning Snapshots

#### Q: How do I clean up deleted snapshots?

**A:** Use `snapshot prune` to remove snapshot files for deleted resources.

**Example:**

```bash
stoobly-agent snapshot prune
```

#### Q: How do I preview what will be pruned without deleting?

**A:** Use the `--dry-run` flag to see what would be deleted.

**Example:**

```bash
# Preview prune operation
stoobly-agent snapshot prune --dry-run

# Actually prune
stoobly-agent snapshot prune
```

#### Q: When should I prune snapshots?

**A:** Prune after deleting scenarios or requests to keep your snapshot directory clean and your Git repository size manageable.

**Example:**

```bash
# Delete a scenario
stoobly-agent scenario delete old-scenario

# Clean up its snapshots
stoobly-agent snapshot prune

# Commit the cleanup
git add .stoobly/snapshots/
git commit -m "Remove old scenario snapshots"
```

***

### Version Control Workflows

#### Q: How do I set up snapshots for Git?

**A:** Create snapshots and add the `.stoobly/snapshots/` directory to Git.

**Example:**

```bash
# Create scenario with requests
stoobly-agent scenario create "User Login"
# ... add requests ...

# Create snapshot
stoobly-agent scenario snapshot create user-login --decode

# Add to Git
git add .stoobly/snapshots/
git commit -m "Add user login test scenario"
git push
```

#### Q: How do team members use snapshots from Git?

**A:** Pull the repository and apply snapshots to get the tests.

**Example:**

```bash
# Clone or pull repository
git clone <repo-url>
cd <repo>

# Apply all snapshots
stoobly-agent snapshot apply

# Use the tests
stoobly-agent scenario list
stoobly-agent scenario test user-login
```

#### Q: How do I update existing snapshots in Git?

**A:** Re-create the snapshot and commit the changes.

**Example:**

```bash
# Make changes to scenario (add/remove requests)
stoobly-agent record <new-request> --scenario-key "<SCENARIO-KEY>"

# Update snapshot
stoobly-agent scenario snapshot create user-login --decode

# Commit changes
git add .stoobly/snapshots/
git commit -m "Update user login with new endpoint"
git push
```

#### Q: How do I handle merge conflicts in snapshots?

**A:** Resolve conflicts manually in snapshot files or re-create snapshots from one version.

**Example:**

```bash
# Conflict occurs during merge
git merge feature-branch

# Option 1: Manual resolution
# Edit conflicted snapshot files in .stoobly/snapshots/
git add .stoobly/snapshots/
git commit

# Option 2: Re-create from one version
git checkout --theirs .stoobly/snapshots/scenarios/<SNAPSHOT-UUID>
stoobly-agent snapshot apply "<SNAPSHOT-UUID>"
# Test and verify
git add .stoobly/snapshots/
git commit
```

#### Q: Should I commit snapshot files or the database?

**A:** Commit snapshot files in `.stoobly/snapshots/`, NOT the database files in `.stoobly/db/`. Add `.stoobly/db/` to `.gitignore`.

**Example:**

```bash
# .gitignore
.stoobly/db/
.stoobly/logs/
.stoobly/tmp/

# DO commit:
# .stoobly/snapshots/
```

***

### Advanced Snapshot Operations

#### Q: How do I export snapshots for backup?

**A:** Copy the `.stoobly/snapshots/` directory or use snapshot copy.

**Example:**

```bash
# Option 1: Direct copy
tar -czf snapshots-backup-$(date +%Y%m%d).tar.gz .stoobly/snapshots/

# Option 2: Use snapshot copy
stoobly-agent snapshot copy /backup/location \
  --scenario-key "<SCENARIO-KEY-1>" \
  --scenario-key "<SCENARIO-KEY-2>"
```

#### Q: How do I share snapshots across projects?

**A:** Use snapshot copy to move snapshots between project directories.

**Example:**

```bash
# In project A
stoobly-agent snapshot copy /path/to/project-b \
  --scenario-key "<SCENARIO-KEY>"

# In project B
cd /path/to/project-b
stoobly-agent snapshot apply
```

#### Q: How do I create snapshots in CI/CD?

**A:** Record requests during CI, create snapshots, and commit them back if needed.

**Example:**

```bash
#!/bin/bash
# CI/CD snapshot creation

# Run application and record tests
stoobly-agent run --intercept --intercept-mode record &
AGENT_PID=$!

# Run test suite (gets recorded)
npm test

# Stop agent
kill $AGENT_PID

# Create snapshots
for scenario in $(stoobly-agent scenario list --format json | jq -r '.[].key'); do
  stoobly-agent scenario snapshot create $scenario --decode
done

# Check if snapshots changed
if git diff --exit-code .stoobly/snapshots/; then
  echo "No snapshot changes"
else
  echo "Snapshots updated"
  # Optionally commit back to repo
fi
```

#### Q: How do I validate snapshots in CI/CD?

**A:** Apply snapshots and run tests to ensure they're valid.

**Example:**

```bash
#!/bin/bash
# CI/CD snapshot validation

# Apply all snapshots
if ! stoobly-agent snapshot apply; then
  echo "Failed to apply snapshots"
  exit 1
fi

# List applied snapshots
stoobly-agent request snapshot list

# Test scenarios from snapshots
for scenario in $(stoobly-agent scenario list --format json | jq -r '.[].key'); do
  echo "Testing scenario: $scenario"
  if ! stoobly-agent scenario test $scenario --strategy diff; then
    echo "Failed: $scenario"
    exit 1
  fi
done

echo "All snapshot scenarios validated"
```

***

### Troubleshooting

#### Q: What do I do if snapshot apply fails?

**A:** Check for errors, use force option, or manually inspect the snapshot files.

**Example:**

```bash
# Try with force
stoobly-agent snapshot apply --force

# List pending snapshots to see what's not applied
stoobly-agent request snapshot list --pending

# Check for specific errors
stoobly-agent snapshot apply "<SNAPSHOT-UUID>"
```

#### Q: How do I verify snapshot integrity?

**A:** Use snapshot create to verify and fix snapshot formatting.

**Example:**

```bash
# Verify and fix request snapshot
stoobly-agent request snapshot create "<REQUEST-KEY>"

# Verify and fix scenario snapshot
stoobly-agent scenario snapshot create "<SCENARIO-KEY>"

# List all snapshots and update each
for uuid in $(stoobly-agent request snapshot list --format json | jq -r '.[].uuid'); do
  stoobly-agent request snapshot create $uuid
done
```

#### Q: How do I find which snapshot corresponds to a request?

**A:** List snapshots with search or match request keys to UUIDs.

**Example:**

```bash
# Search request snapshots by path
stoobly-agent request snapshot list --search "/api/users"

# List with specific format
stoobly-agent request snapshot list --select uuid --select path --select snapshot
```

***

### Best Practices

#### Q: When should I create snapshots?

**A:** Create snapshots after recording important test flows, before releases, and when sharing tests with the team.

**Example:**

```bash
# After recording a critical flow
stoobly-agent record <requests> --scenario-key "<SCENARIO-KEY>"
stoobly-agent scenario snapshot create "<SCENARIO-KEY>" --decode
git add .stoobly/snapshots/ && git commit -m "Add checkout flow"

# Before release
./create-all-snapshots.sh
git tag -a v1.0.0 -m "Release 1.0.0 with test snapshots"
```

#### Q: How often should I prune snapshots?

**A:** Prune after deleting old tests or periodically to keep the repository clean.

**Example:**

```bash
# Monthly cleanup
stoobly-agent snapshot prune --dry-run  # Preview
stoobly-agent snapshot prune            # Execute
git add .stoobly/snapshots/
git commit -m "Prune deleted snapshots"
```

#### Q: When should I decode snapshots?

**A:** Use `--decode` when creating snapshots for better Git diffs and readability.

**Example:**

```bash
# Always use --decode for version control
stoobly-agent scenario snapshot create "<SCENARIO-KEY>" --decode
stoobly-agent request snapshot create "<REQUEST-KEY>" --decode
```

***

### Quick Reference

#### Q: What are the most common snapshot commands?

**A:** Here's a quick reference of frequently used commands:

**Example:**

```bash
# List request snapshots
stoobly-agent request snapshot list
stoobly-agent request snapshot list --search "/api/users"
stoobly-agent request snapshot list --pending
stoobly-agent request snapshot list --scenario-key "<SCENARIO-KEY>"

# List scenario snapshots
stoobly-agent scenario snapshot list
stoobly-agent scenario snapshot list --search "login"
stoobly-agent scenario snapshot list --pending

# Create snapshots
stoobly-agent request snapshot create "<REQUEST-KEY>"
stoobly-agent request snapshot create "<REQUEST-KEY>" --decode
stoobly-agent scenario snapshot create "<SCENARIO-KEY>"
stoobly-agent scenario snapshot create "<SCENARIO-KEY>" --decode

# Reset snapshots
stoobly-agent request snapshot reset "<REQUEST-KEY>"
stoobly-agent request snapshot reset "<REQUEST-KEY>" --force
stoobly-agent scenario snapshot reset "<SCENARIO-KEY>"
stoobly-agent scenario snapshot reset "<SCENARIO-KEY>" --force

# Diff snapshots
stoobly-agent request snapshot diff "<REQUEST-KEY>"
stoobly-agent request snapshot diff --full
stoobly-agent scenario snapshot diff "<SCENARIO-KEY>"

# Apply all snapshots
stoobly-agent snapshot apply
stoobly-agent snapshot apply "<SNAPSHOT-UUID>"
stoobly-agent snapshot apply --force

# Copy snapshots
stoobly-agent snapshot copy /path/to/dest --scenario-key "<SCENARIO-KEY>"
stoobly-agent snapshot copy /path/to/dest --request-key "<REQUEST-KEY>"

# Prune snapshots
stoobly-agent snapshot prune --dry-run
stoobly-agent snapshot prune
```

***

### Integration Examples

#### Q: How do I automate snapshot creation?

**A:** Use a script to create snapshots for all scenarios.

**Example:**

```bash
#!/bin/bash
# Create snapshots for all scenarios

for scenario in $(stoobly-agent scenario list --format json | jq -r '.[].key'); do
  echo "Creating snapshot for: $scenario"
  stoobly-agent scenario snapshot create $scenario --decode
done

echo "All snapshots created"
```

#### Q: How do I sync snapshots across environments?

**A:** Use Git to sync and snapshot copy for different data directories.

**Example:**

```bash
# Development: Create and push snapshots
stoobly-agent scenario snapshot create critical-tests --decode
git add .stoobly/snapshots/ && git commit -m "Update tests" && git push

# Staging: Pull and apply
git pull
stoobly-agent snapshot apply

# Production: Copy from staging
stoobly-agent snapshot copy /production/path --scenario-key "<SCENARIO-KEY>"
```
