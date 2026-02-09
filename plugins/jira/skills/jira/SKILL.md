---
name: jira
description: Use this skill when the user asks to search, create, update, or manage Jira tickets/issues. Provides ACLI command patterns for Jira operations.
version: 2.3.0
---

# Jira CLI Skill

Use the Atlassian CLI (acli) for Jira operations. Reference: https://developer.atlassian.com/cloud/acli/reference/commands/jira/

## First Step: Bootstrap

**Before any Jira operation**, run `acli jira auth status`.
- If `command not found`: tell the user to install it (`brew tap atlassian/homebrew-acli && brew install acli`)
- If authenticated: proceed. Note the email shown - it's needed for `--assignee`.
- If not authenticated: tell the user to run `acli jira auth login --web` (requires interactive browser).

## Work Items

### View

**Default: use plain text** (compact, includes summary/description/status/type). Only add `--json` with specific `--fields` when you need structured data like parent, components, or links.

```bash
# Default - compact plain text (~1.4KB vs ~22KB for --fields '*all' --json)
acli jira workitem view PROJ-123

# When parent/components/links needed - use specific fields, NEVER '*all'
acli jira workitem view PROJ-123 --fields "summary,status,priority,assignee,parent,components,labels,issuelinks" --json

# Open in browser
acli jira workitem view PROJ-123 --web
```

**IMPORTANT:** Never use `--fields '*all' --json` - it returns ~22KB of mostly null custom fields. Use specific field names.

### Search
```bash
acli jira workitem search --jql "project = PROJ AND status != Done" --limit 20
acli jira workitem search --jql "assignee = currentUser()" --limit 10 --json
acli jira workitem search --jql "project = PROJ" --fields "key,summary,status,assignee" --csv
acli jira workitem search --jql "project = PROJ" --paginate  # Fetch all pages
```

### Create
```bash
# Simple
acli jira workitem create --project PROJ --type Task \
  --summary "Summary" --description "Description" \
  --parent PROJ-100 --assignee "user@email.com" --label "backend,urgent"

# Interactive (opens editor for summary + description)
acli jira workitem create --project PROJ --type Task --editor
```

### Create with JSON (for advanced fields)
Use `--from-json` when you need to set fields like components, custom fields, or ADF descriptions:
```bash
cat > /tmp/ticket.json << 'EOF'
{
  "projectKey": "PROJ",
  "type": "Task",
  "summary": "Summary here",
  "description": {
    "type": "doc",
    "version": 1,
    "content": [
      {"type": "paragraph", "content": [{"type": "text", "text": "Description text."}]}
    ]
  },
  "assignee": "user@email.com",
  "additionalAttributes": {
    "parent": {"key": "PROJ-100"},
    "components": [{"name": "ComponentName"}]
  }
}
EOF
acli jira workitem create --from-json /tmp/ticket.json --json
```

**Gotchas:**
- `parent` must go inside `additionalAttributes` with format `{"key": "PROJ-XXX"}`
- Do NOT use `parentIssueId` (only works for subtasks)
- Do NOT combine `--from-json` with CLI flags like `--parent` (flags are ignored when using JSON)
- Description in JSON must be Atlassian Document Format (ADF), not plain text
- Use `--generate-json` to get an example JSON template

### Edit
```bash
acli jira workitem edit --key PROJ-123 --summary "New summary" --yes
acli jira workitem edit --key PROJ-123 --description "New description" --yes
acli jira workitem edit --key PROJ-123 --type Story --yes
acli jira workitem edit --key PROJ-123 --remove-assignee --yes
acli jira workitem edit --key "PROJ-123,PROJ-124" --assignee "user@email.com" --yes  # Bulk
acli jira workitem edit --jql "project = PROJ AND label = stale" --assignee "@me" --yes  # Via JQL
```

### Delete
```bash
acli jira workitem delete --key PROJ-123 --yes
acli jira workitem delete --key "PROJ-123,PROJ-124" --yes  # Multiple
```

### Transition
```bash
acli jira workitem transition --key PROJ-123 --status "In Progress"
acli jira workitem transition --jql "project = PROJ AND assignee = currentUser() AND status = 'To Do'" --status "In Progress" --yes
```

### Assign
```bash
acli jira workitem assign --key PROJ-123 --assignee "user@email.com"
acli jira workitem assign --key PROJ-123 --assignee "@me"           # Self-assign
acli jira workitem assign --key PROJ-123 --remove-assignee          # Unassign
```

### Clone
```bash
acli jira workitem clone --key PROJ-123 --yes
acli jira workitem clone --key PROJ-123 --to-project OTHER --yes    # Clone to different project
```

## Comments

```bash
acli jira workitem comment-create --key PROJ-123 --body "Comment text"
acli jira workitem comment-list --key PROJ-123 --json
acli jira workitem comment-update --key PROJ-123 --edit-last --body "Updated text"
acli jira workitem comment-delete --key PROJ-123                    # Prompts for selection
```

## Links

```bash
# List link types available
acli jira workitem link type

# Create link (--out blocks --in)
acli jira workitem link create --out PROJ-123 --in PROJ-456 --type "blocks" --yes

# List links on an issue
acli jira workitem link list --key PROJ-123
```

## Boards & Sprints

```bash
# Find boards
acli jira board search --project PROJ --json
acli jira board search --name "My Board" --type scrum

# List sprints for a board
acli jira board list-sprints --board 42 --json

# List issues in a sprint
acli jira sprint list-workitems --board 42 --sprint 100 --json
acli jira sprint list-workitems --board 42 --sprint 100 --jql "assignee = currentUser()"
```

## Projects

```bash
acli jira project list --json
acli jira project list --recent    # Up to 20 recently viewed
acli jira project view PROJ --json
```

## JQL Examples

```
assignee = currentUser() AND status != Done
project = PROJ ORDER BY created DESC
project = PROJ AND sprint in openSprints()
project = PROJ AND status = "In Progress"
project = PROJ AND type = Epic AND status != Done
parent = PROJ-100
project = PROJ AND created >= -7d
project = PROJ AND labels in ("backend", "urgent")
project = PROJ AND component = "ComponentName"
```

Always scope JQL with a project or assignee filter - unbounded queries are not allowed.

## Common Flags

| Flag | Purpose |
|------|---------|
| `--json` | JSON output (NOT `--output json`) |
| `--csv` | CSV output |
| `--from-json FILE` | Read definition from JSON (NOT `--file`) |
| `--generate-json` | Generate example JSON template |
| `--yes` / `-y` | Skip confirmation prompt |
| `--fields "f1,f2"` | Select fields to return |
| `--limit N` | Max results |
| `--paginate` | Fetch all pages |
| `--web` / `-w` | Open in browser |
| `--editor` / `-e` | Open text editor for input |
| `--ignore-errors` | Continue on errors (bulk ops) |
| `--description-file` | Read description from file |
