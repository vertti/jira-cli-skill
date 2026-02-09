# Jira CLI Skill for Claude Code

A [Claude Code](https://docs.anthropic.com/en/docs/claude-code) skill for managing Jira issues using the [Atlassian CLI (acli)](https://developer.atlassian.com/cloud/acli/).

Supports searching, creating, updating, transitioning, and commenting on Jira issues — all from within Claude Code.

## Why this instead of Atlassian MCP?

Context efficiency. MCP tools load their schema into every session whether you use them or not.

| Scenario | Atlassian MCP | This skill |
|---|---|---|
| Idle (no Jira actions) | ~6,500 tokens | ~37 tokens |
| Fetch a ticket + parent epic | ~14,700 tokens | ~10,500 tokens |

The skill only loads its full instructions when you actually use it.

## Install

```
/plugin marketplace add vertti/jira-cli-skill
```

```
/plugin install jira@jira-cli-skill
```

Restart Claude Code after installing for `/jira` autocomplete to work.

## Usage

The skill activates automatically when you ask Claude Code about Jira tasks. You can also invoke it directly with `/jira`.

Examples:
- "Show me my open Jira tickets"
- "Create a new bug in PROJECT for the login issue"
- "Transition PROJECT-123 to In Progress"
- `/jira search for unassigned tasks in PROJECT`
