# Request

## Stoobly Request CLI - Questions & Answers

The request CLI enables you to manage recorded HTTP requests, replay them, test responses, and work with request/response data. These commands help you work with individual requests outside of scenarios.

***

### Listing Requests

#### Q: How do I view all recorded requests?

**A:** Use `request list` to display all recorded requests with pagination.

**Example:**

```bash
stoobly-agent request list
```

#### Q: How do I paginate through requests?

**A:** Use the `--page` and `--size` options to control pagination.

**Example:**

```bash
# Show first 10 requests (default)
stoobly-agent request list --page 0 --size 10

# Show next 20 requests
stoobly-agent request list --page 1 --size 20

# Show 50 requests per page
stoobly-agent request list --page 0 --size 50
```

#### Q: How do I filter requests by scenario?

**A:** Use the `--scenario-key` option to show only requests from a specific scenario.

**Example:**

```bash
stoobly-agent request list --scenario-key "<SCENARIO-KEY>"
```

#### Q: How do I search for specific requests?

**A:** Use the `--search` option to filter requests by URL path or other criteria.

**Example:**

```bash
# Search by path
stoobly-agent request list --search "/api/users"

# Search by domain
stoobly-agent request list --search "example.com"
```

#### Q: How do I sort requests?

**A:** Use `--sort-by` and `--sort-order` options to control sorting.

**Example:**

```bash
# Sort by creation date (newest first, default)
stoobly-agent request list --sort-by created_at --sort-order desc

# Sort by creation date (oldest first)
stoobly-agent request list --sort-by created_at --sort-order asc

# Sort by path
stoobly-agent request list --sort-by path --sort-order asc
```

#### Q: How do I format the request list output?

**A:** Use the `--format` option to change output format (table, json, csv).

**Example:**

```bash
# Table format (default)
stoobly-agent request list --format table

# JSON format
stoobly-agent request list --format json

# CSV format
stoobly-agent request list --format csv
```

#### Q: How do I select specific columns to display?

**A:** Use the `--select` option to choose which columns to show.

**Example:**

```bash
# Show only specific columns
stoobly-agent request list --select id --select path --select method

# Show multiple columns
stoobly-agent request list --select id --select path --select method --select created_at
```

#### Q: How do I hide column headers?

**A:** Use the `--without-headers` flag to disable column headers.

**Example:**

```bash
stoobly-agent request list --without-headers
```

#### Q: How do I export requests to a file?

**A:** Use format options and shell redirection to export requests.

**Example:**

```bash
# Export as JSON
stoobly-agent request list --format json > requests.json

# Export as CSV
stoobly-agent request list --format csv > requests.csv

# Export specific scenario
stoobly-agent request list --scenario-key "<SCENARIO-KEY>" --format json > scenario-requests.json
```

***

### Replaying Requests

#### Q: How do I replay a recorded request?

**A:** Use `request replay` with the request key to re-execute a recorded request.

**Example:**

```bash
stoobly-agent request replay "<REQUEST-KEY>"
```

#### Q: How do I replay a request to a different host?

**A:** Use the `--host` option to override the request host.

**Example:**

```bash
# Replay to localhost
stoobly-agent request replay "<REQUEST-KEY>" --host localhost:8080

# Replay to staging
stoobly-agent request replay "<REQUEST-KEY>" --host staging.example.com
```

#### Q: How do I replay a request with a different scheme (HTTP/HTTPS)?

**A:** Use the `--scheme` option to change the protocol.

**Example:**

```bash
# Force HTTP
stoobly-agent request replay "<REQUEST-KEY>" --scheme http

# Force HTTPS
stoobly-agent request replay "<REQUEST-KEY>" --scheme https
```

#### Q: How do I replay a request and record the new response?

**A:** Use the `--record` flag to capture the replayed response.

**Example:**

```bash
stoobly-agent request replay "<REQUEST-KEY>" --record
```

#### Q: How do I replay a request and overwrite the existing response?

**A:** Use the `--overwrite` flag to replace the stored response (local mode only).

**Example:**

```bash
stoobly-agent request replay "<REQUEST-KEY>" --overwrite
```

#### Q: How do I replay a request to a specific scenario?

**A:** Use the `--scenario-key` option to record the replay into a scenario.

**Example:**

```bash
stoobly-agent request replay "<REQUEST-KEY>" --record --scenario-key "<SCENARIO-KEY>"
```

#### Q: How do I format the replay response output?

**A:** Use the `--format` option to control response display.

**Example:**

```bash
# Body only
stoobly-agent request replay "<REQUEST-KEY>" --format body

# Full JSON with metadata
stoobly-agent request replay "<REQUEST-KEY>" --format json
```

#### Q: How do I replay a request with custom lifecycle hooks?

**A:** Use the `--lifecycle-hooks-path` option to apply custom processing.

**Example:**

```bash
stoobly-agent request replay "<REQUEST-KEY>" --lifecycle-hooks-path ./hooks.py
```

#### Q: How do I increase logging verbosity when replaying?

**A:** Use the `--log-level` option to see more details.

**Example:**

```bash
# Debug level (most verbose)
stoobly-agent request replay "<REQUEST-KEY>" --log-level debug

# Info level
stoobly-agent request replay "<REQUEST-KEY>" --log-level info
```

***

### Working with Aliases (Remote Features)

#### Q: How do I replay a request with assigned alias values?

**A:** Use the `--assign` option to set alias values before replay.

**Example:**

```bash
# Assign single alias
stoobly-agent request replay "<REQUEST-KEY>" --assign userId=12345

# Assign multiple aliases
stoobly-agent request replay "<REQUEST-KEY>" --assign userId=12345 --assign token=abcde12345
```

#### Q: How do I validate alias values during replay?

**A:** Use the `--validate` option to specify validation rules for aliases.

**Example:**

```bash
# Validate userId is an integer
stoobly-agent request replay "<REQUEST-KEY>" --validate "userId=?int"

# Validate multiple aliases
stoobly-agent request replay "<REQUEST-KEY>" --validate "userId=?int" --validate "email=?string"
```

#### Q: How do I control alias resolution strategy?

**A:** Use the `--alias-resolve-strategy` option to specify how aliases are resolved.

**Example:**

```bash
# No alias resolution (default)
stoobly-agent request replay "<REQUEST-KEY>" --alias-resolve-strategy none

# First-in-first-out resolution
stoobly-agent request replay "<REQUEST-KEY>" --alias-resolve-strategy fifo

# Last-in-first-out resolution
stoobly-agent request replay "<REQUEST-KEY>" --alias-resolve-strategy lifo
```

#### Q: How do I repeat a request replay for each alias value?

**A:** Use the `--group-by` option to iterate over alias values.

**Example:**

```bash
# Replay once for each userId value
stoobly-agent request replay "<REQUEST-KEY>" --group-by userId

# Results in multiple replays with different userId values
```

#### Q: How do I use an existing trace for replay?

**A:** Use the `--trace-id` option to leverage a previous trace.

**Example:**

```bash
stoobly-agent request replay "<REQUEST-KEY>" --trace-id "<TRACE-ID>"
```

***

### Snapshots and Version Control

#### Q: How do I create a snapshot of a request?

**A:** Use `request snapshot` to create a committable file for the request (local mode only).

**Example:**

```bash
stoobly-agent request snapshot "<REQUEST-KEY>"
```

#### Q: How do I delete a request snapshot?

**A:** Use the `--action delete` option with snapshot command.

**Example:**

```bash
stoobly-agent request snapshot "<REQUEST-KEY>" --action delete
```

#### Q: How do I snapshot a request with decoded response body?

**A:** Use the `--decode` flag to decode the response body in the snapshot.

**Example:**

```bash
stoobly-agent request snapshot "<REQUEST-KEY>" --decode
```

#### Q: How do I reset a request to its snapshot state?

**A:** Use `request reset` to restore a request from its snapshot.

**Example:**

```bash
stoobly-agent request reset "<REQUEST-KEY>"
```

#### Q: How do I force delete when resetting a request?

**A:** Use the `--force` flag to hard delete the request data.

**Example:**

```bash
stoobly-agent request reset "<REQUEST-KEY>" --force
```

#### Q: How do I share requests with my team via git?

**A:** Create snapshots and commit them to version control.

**Example:**

```bash
# Create snapshot
stoobly-agent request snapshot "<REQUEST-KEY>"

# Commit to git
git add .stoobly/snapshots/
git commit -m "Add request snapshot"
git push

# Team member pulls and uses
git pull
stoobly-agent request reset "<REQUEST-KEY>"  # Restore from snapshot
stoobly-agent request replay "<REQUEST-KEY>"  # Use the request
```

***

### Updating Requests

The following snapshot-based workflows are the **recommended approach** when updating single requests or requests that belong to an API endpoint.

#### Q: How do I update a single request?

**A:** Use the snapshot workflow to edit a request's data directly by modifying its snapshot file.

**Steps:**

1. Find the request key for the request you want to update:

```bash
stoobly-agent request list --search "/api/users"
```

2. Snapshot it using `stoobly-agent request snapshot "<REQUEST-KEY>"`. The output of the command is the snapshot file path:

```bash
stoobly-agent request snapshot "<REQUEST-KEY>"
```

3. Update the snapshot file. The format of the file is a raw HTTP request and response separated by a delimiter.

4. Run `stoobly-agent snapshot apply` to apply the updated snapshot to the database:

```bash
stoobly-agent snapshot apply
```

More details: [Editing with Snapshots](https://docs.stoobly.com/guides/how-to-update-requests/editing-with-snapshots), [Snapshot FAQ](https://docs.stoobly.com/faq/snapshot)

#### Q: How do I update all requests matching an API endpoint whose contract has changed?

**A:** When an API endpoint's request or response contract changes, use the snapshot workflow to update all matching requests in bulk.

**Steps:**

1. Find all request keys matching the endpoint pattern:

```bash
stoobly-agent request list --search "<PATTERN>" --select key --without-headers
```

Where `<PATTERN>` uses SQLite search syntax. For endpoints where a path segment is an ID (e.g., `/users/1`), the search pattern should be `/users/%`. The output will be request keys separated by newlines.

2. For each key in the search results, snapshot it using `stoobly-agent request snapshot "<REQUEST-KEY>"`. The output of the command is a snapshot file path for each key:

```bash
stoobly-agent request list --search "<PATTERN>" --select key --without-headers | while read key; do
  stoobly-agent request snapshot "$key"
done
```

3. For each path in the search results, update the snapshot file. The format of the files is raw HTTP requests and responses separated by a delimiter.

4. Run `stoobly-agent snapshot apply` to apply all updated snapshots to the database:

```bash
stoobly-agent snapshot apply
```

More details: [Editing with Snapshots](https://docs.stoobly.com/guides/how-to-update-requests/editing-with-snapshots), [Snapshot FAQ](https://docs.stoobly.com/faq/snapshot)

If you need to **update all requests within a scenario**, use the scenario-specific workflows described in the [Scenario FAQ](https://docs.stoobly.com/faq/scenario) and the [How to Update Scenarios](https://docs.stoobly.com/guides/how-to-update-requests/how-to-update-scenarios/) guide.

***

### Managing Request Responses

#### Q: How do I view the response for a recorded request?

**A:** Use `request response show` to retrieve the stored response.

**Example:**

```bash
stoobly-agent request response show "<REQUEST-KEY>"
```

#### Q: How do I query specific properties in a response?

**A:** Use `request response query` with a query expression.

**Example:**

```bash
# Query JSON path
stoobly-agent request response query "<REQUEST-KEY>" --query "$.data.users[0].name"

# Query nested property
stoobly-agent request response query "<REQUEST-KEY>" --query "$.response.body"
```

#### Q: How do I extract data from a response for scripting?

**A:** Combine response query with shell processing.

**Example:**

```bash
# Extract user ID
userId=$(stoobly-agent request response query "<REQUEST-KEY>" --query "$.data.id")

# Use in another command
echo "User ID: $userId"

# Extract array of values
stoobly-agent request response query "<REQUEST-KEY>" --query "$.data.users[*].email"
```

***

### Deleting Requests

#### Q: How do I delete a recorded request?

**A:** Use `request delete` with the request key.

**Example:**

```bash
stoobly-agent request delete "<REQUEST-KEY>"
```

#### Q: How do I delete multiple requests?

**A:** Use a loop or script to delete requests in bulk.

**Example:**

```bash
# Delete multiple specific requests
stoobly-agent request delete "<REQUEST-KEY-1>"
stoobly-agent request delete "<REQUEST-KEY-2>"
stoobly-agent request delete "<REQUEST-KEY-3>"

# Delete all requests from a scenario (scripted)
for key in $(stoobly-agent request list --scenario-key "<SCENARIO-KEY>" --format json | jq -r '.[].id'); do
  stoobly-agent request delete $key
done
```

***

### Managing Intercepted Request Logs

#### Q: How do I view the intercepted requests log?

**A:** Use `request logs list` to display all logged intercepted requests. Use `-f` to stream logs in real time, or filter by method, URL, status code, log level, and more.

**Examples:**

```bash
# List all intercepted request log entries
stoobly-agent request logs list

# Follow logs in real time (-f / --follow, Ctrl-C to stop)
stoobly-agent request logs list --follow

# Filter by HTTP method
stoobly-agent request logs list --method get

# Filter by URL substring
stoobly-agent request logs list --url /api/users

# Filter by HTTP status code
stoobly-agent request logs list --status-code 500

# Filter by log level
stoobly-agent request logs list --level error

# Filter by log message
stoobly-agent request logs list --message "Mock failure"

# Filter by scenario name
stoobly-agent request logs list --scenario-name my-scenario

# Output as JSON
stoobly-agent request logs list --format json

# Select specific columns
stoobly-agent request logs list --select method,url,status-code

# Combine filters with follow
stoobly-agent request logs list --follow --level error --method post
```

#### Q: How do I clear the intercepted requests log?

**A:** Use `request logs delete` to truncate the log file.

**Example:**

```bash
stoobly-agent request logs delete
```

#### Q: How do I enable request logging?

**A:** Use the `--request-log-enable` flag when starting the agent.

**Example:**

```bash
# Enable request logging
stoobly-agent run --request-log-enable

# View logs in another terminal
stoobly-agent request logs list
```

#### Q: How do I set the log level for intercepted requests?

**A:** Use the `--request-log-level` option when starting the agent.

**Example:**

```bash
stoobly-agent run --request-log-enable --request-log-level debug
```

#### Q: How do I prevent log truncation on startup?

**A:** Set `--request-log-truncate` to false when starting the agent.

**Example:**

```bash
# Don't truncate log on startup
stoobly-agent run --request-log-enable --request-log-truncate=false
```

***

### Advanced Request Operations

#### Q: How do I replay multiple requests in sequence?

**A:** Use a loop or script to replay requests sequentially.

**Example:**

```bash
# Replay multiple requests
for key in "<REQUEST-KEY-1>" "<REQUEST-KEY-2>" "<REQUEST-KEY-3>"; do
  stoobly-agent request replay $key
done

# Replay all requests in a scenario
for key in $(stoobly-agent request list --scenario-key "<SCENARIO-KEY>" --format json | jq -r '.[].id'); do
  stoobly-agent request replay $key --record
done
```

#### Q: How do I compare requests before and after changes?

**A:** Replay and record, then use snapshots to track changes.

**Example:**

```bash
# Create initial snapshot
stoobly-agent request snapshot "<REQUEST-KEY>"

# Make changes and replay
stoobly-agent request replay "<REQUEST-KEY>" --overwrite

# Compare with snapshot
git diff .stoobly/snapshots/requests/<REQUEST-KEY>.json
```

#### Q: How do I chain requests with dependencies?

**A:** Use aliases and assign values from one request to another.

**Example:**

```bash
# First request creates a user, extract ID
userId=$(stoobly-agent request response query "<REQUEST-KEY-1>" --query "$.data.id")

# Second request uses that ID
stoobly-agent request replay "<REQUEST-KEY-2>" --assign "userId=$userId"
```

#### Q: How do I replay a request with custom headers?

**A:** Use lifecycle hooks to modify request headers before replay.

**Example:**

```bash
# Create hooks.py with custom header logic
cat > hooks.py << 'EOF'
def before_request(context):
    context.request.headers['X-Custom-Header'] = 'custom-value'
    return context
EOF

# Replay with hooks
stoobly-agent request replay "<REQUEST-KEY>" --lifecycle-hooks-path ./hooks.py
```

***

### Filtering and Formatting

#### Q: How do I create a custom report of requests?

**A:** Use format options and select specific fields for custom reporting.

**Example:**

```bash
# Custom CSV report
stoobly-agent request list \
  --select id --select method --select path --select created_at \
  --format csv \
  --without-headers > requests-report.csv

# JSON report with filtering
stoobly-agent request list \
  --scenario-key "<SCENARIO-KEY>" \
  --sort-by created_at \
  --sort-order asc \
  --format json > scenario-report.json
```

#### Q: How do I find requests by method?

**A:** Use the search option or filter the output.

**Example:**

```bash
# Search for POST requests
stoobly-agent request list --format json | jq '.[] | select(.method=="POST")'

# Search for GET requests to specific path
stoobly-agent request list --search "/api/users" --format json | jq '.[] | select(.method=="GET")'
```

***

### Working with Remote Projects

#### Q: How do I list requests from a remote project?

**A:** Use the `--project-key` option to specify the remote project (remote features).

**Example:**

```bash
stoobly-agent request list --project-key "<PROJECT-KEY>"
```

#### Q: How do I test against a remote project's endpoint definitions?

**A:** Use the `--remote-project-key` option when testing.

**Example:**

```bash
stoobly-agent request test "<REQUEST-KEY>" --remote-project-key "<PROJECT-KEY>"
```

***

### Troubleshooting

#### Q: How do I debug a failing request replay?

**A:** Increase log level and use verbose output.

**Example:**

```bash
stoobly-agent request replay "<REQUEST-KEY>" --log-level debug
```

#### Q: How do I see the full request and response details?

**A:** Use JSON format and query specific fields.

**Example:**

```bash
# Get full response
stoobly-agent request response show "<REQUEST-KEY>" --format json

# Query specific parts
stoobly-agent request response query "<REQUEST-KEY>" --query "$.headers"
stoobly-agent request response query "<REQUEST-KEY>" --query "$.body"
```

#### Q: What do I do if a request key is not found?

**A:** List all requests to find the correct key.

**Example:**

```bash
# List all requests to find the key
stoobly-agent request list

# Search for specific request
stoobly-agent request list --search "/api/endpoint"
```

***

### Quick Reference

#### Q: What are the most common request commands?

**A:** Here's a quick reference of frequently used commands:

**Example:**

```bash
# List requests
stoobly-agent request list
stoobly-agent request list --scenario-key "<SCENARIO-KEY>"
stoobly-agent request list --search "/api/users"

# Replay requests
stoobly-agent request replay "<REQUEST-KEY>"
stoobly-agent request replay "<REQUEST-KEY>" --host localhost:8080
stoobly-agent request replay "<REQUEST-KEY>" --record

# Test requests
stoobly-agent request test "<REQUEST-KEY>"
stoobly-agent request test "<REQUEST-KEY>" --strategy diff
stoobly-agent request test "<REQUEST-KEY>" --strategy contract

# Snapshots
stoobly-agent request snapshot "<REQUEST-KEY>"
stoobly-agent request reset "<REQUEST-KEY>"

# View responses
stoobly-agent request response show "<REQUEST-KEY>"
stoobly-agent request response query "<REQUEST-KEY>" --query "$.data"

# Delete requests
stoobly-agent request delete "<REQUEST-KEY>"

# Manage logs
stoobly-agent request logs list
stoobly-agent request logs delete
```

***

