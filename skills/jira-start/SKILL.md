---
name: jira-start
description: Start working on a Jira ticket. Fetches ticket details via Jira MCP, assigns to current user, transitions to "In Progress", creates a git branch or worktree based on ticket type, and generates a TICKET.md context file. Use when user says "start ticket", "work on ABC-123", "pick up ticket", "begin work on", or similar phrases indicating they want to start a Jira ticket.
allowed-tools: Read, Write, Bash, AskUserQuestion
user-invocable: true
---

# Jira Start Workflow

Begin work on a Jira ticket with proper git setup and context.

## Prerequisites

- Jira MCP server connected (authentication provides current user identity)
- Git repository as working directory

## Workflow

### Step 1: Connect to Jira and Fetch Ticket

Use Jira MCP to fetch ticket details. If MCP unavailable or connection fails:
→ Inform user, ask if they want to provide ticket details manually or troubleshoot MCP

Required fields to fetch:
- Key (e.g., `ABC-123`)
- Summary (title)
- Description
- Type (Story, Bug, Task, etc.)
- Status
- Assignee
- Priority
- Reporter
- Acceptance criteria (if field exists)

### Step 2: Handle Assignment

Check current assignee:

**Unassigned** → Assign to current user via MCP (use auth token identity)

**Assigned to current user** → Continue

**Assigned to someone else** → Prompt user:
> "This ticket is assigned to [name]. Would you like to:
> 1. Reassign to yourself
> 2. Continue without reassigning
> 3. Abort"

### Step 3: Transition to In Progress

Query available transitions for the ticket via MCP. Look for transitions matching (case-insensitive):
- "In Progress"
- "Start Progress"
- "Start"
- "Begin"

**Match found** → Execute transition

**Already in progress-like status** → Skip, inform user

**No match found** → Show available transitions, ask user which to use (or skip)

### Step 4: Check Git State

Before creating branch/worktree:

**Uncommitted changes detected** → Warn user:
> "You have uncommitted changes. Proceed anyway, stash them, or abort?"

### Step 5: Determine Branch Prefix

Map Jira ticket type to branch prefix:

| Ticket Type | Branch Prefix |
|-------------|---------------|
| Story       | `story/`      |
| Bug         | `bugfix/`     |
| Task        | `task/`       |
| Epic        | `epic/`       |
| Subtask     | `subtask/`    |

**Type not in table** → Ask user for preferred prefix

### Step 6: Ask Branch or Worktree

Prompt user:
> "How would you like to set up your workspace?
> 1. **Branch** — Create `<prefix>/<TICKET-KEY>` in current repo
> 2. **Worktree** — Create worktree at `../<repo-name>-<TICKET-KEY>` with that branch"

### Step 7: Create Branch/Worktree

Determine branch name: `<prefix>/<TICKET-KEY>` (e.g., `story/ABC-123`)

**If branch already exists:**
> "Branch `story/ABC-123` already exists. Check it out, or create with different name?"

**Branch option:**
```bash
git checkout -b <branch-name>
```

**Worktree option:**
```bash
# From repo at /path/to/repo-name
git worktree add -b <branch-name> ../repo-name-<TICKET-KEY> <branch-name>
```

### Step 8: Create TICKET.md

Create `<TICKET-KEY>.md` (e.g., `ABC-123.md`) in repository root (or worktree root):

```markdown
# <TICKET-KEY>: <Summary>

**Type:** <Type> | **Priority:** <Priority> | **Reporter:** <Reporter>

[View in Jira](<jira-url>)

## Description

<Full description from Jira>

## Acceptance Criteria

<If available, otherwise omit section>

## Technical Notes

<!-- Add implementation notes, decisions, blockers here -->

## References

<!-- Links to docs, related tickets, PRs -->
```

### Step 9: VS Code (Worktree Only)

If worktree was created, ask:
> "Open VS Code in the new worktree folder?"

If yes:
```bash
code ../repo-name-<TICKET-KEY>
```

## Error Handling

- **MCP connection failure**: Offer manual ticket details or MCP troubleshooting
- **Transition failure**: Show error, offer to continue without transition
- **Git errors**: Show error details, suggest resolution
- **Permission errors**: Explain and abort gracefully
