---
name: worktree-list
description: Shows all active git worktrees with their associated topics, branches, and status. Use when reviewing active worktrees or checking what work is in progress.
allowed-tools: Read, Glob, Bash
user-invocable: true
---

# Worktree List Skill

Displays all active git worktrees with their associated topics and status information.

## When to Use

Activate this skill when:
- User wants to see active worktrees
- User says "list worktrees", "show worktrees", or "what worktrees do I have"
- User wants to check what implementation work is in progress
- Before removing a worktree, to see options

## Instructions

### Step 1: Get Git Worktree List

Run:
```bash
git worktree list
```

This returns output like:
```
/path/to/main-repo                abc1234 [main]
/path/to/feature-auth             def5678 [feature-auth]
/path/to/bugfix-login             ghi9012 [bugfix-login]
```

### Step 2: Parse Worktree Information

For each line in the output:
1. Extract path (first column)
2. Extract commit hash (second column)
3. Extract branch name (in brackets)
4. Identify which is the main worktree (typically the one matching current directory pattern)

### Step 3: Read Context Files

For each non-main worktree:
1. Check if `.claude-worktree-context.md` exists at that path
2. If yes, read and extract:
   - Topic name
   - Created date
   - Purpose (first line of Purpose section)
3. If no context file, mark as "No context file"

### Step 4: Check Worktree Status

For each non-main worktree, optionally run:
```bash
git -C "[worktree-path]" status --porcelain
```

This shows if there are uncommitted changes:
- Empty output = clean
- Non-empty = has uncommitted changes

### Step 5: Display Formatted List

Show the information in a clear format:

```
Git Worktrees
=============

Main Repository:
  /path/to/main-repo
  Branch: main
  Status: Clean

Active Worktrees:
-----------------

1. feature-auth
   Path: ../feature-auth
   Branch: feature-auth
   Topic: User Authentication
   Created: 2026-01-08
   Status: Clean
   Purpose: Implement JWT authentication

2. bugfix-login
   Path: ../bugfix-login
   Branch: bugfix-login
   Topic: Login Bug Fix
   Created: 2026-01-05
   Status: Has uncommitted changes
   Purpose: Fix login redirect issue

-----------------
Total: 2 active worktrees
```

**If no worktrees (besides main):**
```
Git Worktrees
=============

Main Repository:
  /path/to/main-repo
  Branch: main

No active worktrees.

To create a worktree, run: /worktree-start
```

---

## Error Handling

**If git worktree list fails:**
- Check if we're in a git repository
- Show error message with troubleshooting steps

**If context file can't be read:**
- Don't fail the entire list
- Show "Context file unreadable" for that worktree
- Continue with other worktrees

**If worktree path doesn't exist:**
- This indicates a stale worktree entry
- Note: "Path not found (may be stale)"
- Suggest running `git worktree prune`

---

## Examples

### Example 1: Multiple Worktrees

**User**: "/worktree-list"

**Output**:
```
Git Worktrees
=============

Main Repository:
  /path/to/my-project
  Branch: main
  Status: Clean

Active Worktrees:
-----------------

1. feature-auth
   Path: ../feature-auth
   Branch: feature-auth
   Topic: User Authentication
   Created: 2026-01-08
   Status: Clean

2. api-refactor
   Path: ../api-refactor
   Branch: api-refactor
   Topic: API Refactoring
   Created: 2026-01-07
   Status: Has uncommitted changes

-----------------
Total: 2 active worktrees

Tip: Use /worktree-remove to clean up finished worktrees
     Use /worktree-end to merge and clean up
```

### Example 2: No Worktrees

**User**: "/worktree-list"

**Output**:
```
Git Worktrees
=============

Main Repository:
  /path/to/my-project
  Branch: main

No active worktrees.

To create a worktree, run: /worktree-start
```

### Example 3: Worktree Without Context File

**User**: "/worktree-list"

**Output**:
```
Git Worktrees
=============

Main Repository:
  /path/to/my-project
  Branch: main

Active Worktrees:
-----------------

1. experiment-branch
   Path: ../experiment-branch
   Branch: experiment
   Topic: (no context file)
   Status: Clean

-----------------
Total: 1 active worktree

Note: Worktree "experiment-branch" has no context file.
Consider adding .claude-worktree-context.md for better session-start integration.
```

---

## Notes

- This skill is read-only; it doesn't modify anything
- Main repository is always shown first
- Worktrees without context files are still listed
- Status check helps identify worktrees with unsaved work
- Stale worktrees (path doesn't exist) are flagged for cleanup
