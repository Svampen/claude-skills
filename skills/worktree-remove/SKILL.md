---
name: worktree-remove
description: Removes a git worktree and optionally deletes its branch. Use for abandoned experiments or when branch is already merged. For completing work with merge, use worktree-end instead.
allowed-tools: Read, Glob, Bash, AskUserQuestion
user-invocable: true
---

# Worktree Remove Skill

Safely removes a git worktree and optionally deletes its associated branch.

## When to Use

Activate this skill when:
- User wants to remove/delete a worktree
- User says "remove worktree", "delete worktree", or "clean up worktree"
- User has finished work and manually merged the branch
- User wants to abandon work in a worktree

**For completing work with automatic merge**, use `/worktree-end` instead.

## Prerequisites

- Must be in the main repository (not in the worktree being removed)
- Worktree should ideally have no uncommitted changes
- Branch should be merged if keeping the work

## Instructions

### Step 1: Get Available Worktrees

Run:
```bash
git worktree list
```

Parse output to get list of worktrees (excluding main).

### Step 2: Ask Which Worktree to Remove

If user didn't specify which worktree:

Use AskUserQuestion to ask:

**Question**: "Which worktree do you want to remove?"
- List available worktrees as options
- Show branch name for each
- Example options:
  - "../feature-auth (branch: feature-auth)"
  - "../bugfix-login (branch: bugfix-login)"

### Step 3: Check for Uncommitted Changes

For the selected worktree, run:
```bash
git -C "[worktree-path]" status --porcelain
```

**If there are uncommitted changes:**
- Warn user:
  ```
  Warning: This worktree has uncommitted changes!

  Changes:
  - [list of changed files]

  These changes will be LOST if you proceed.
  Are you sure you want to remove this worktree?
  ```
- Require explicit confirmation before proceeding
- Options:
  1. **Cancel** - Don't remove, go back
  2. **Force remove** - Remove anyway, losing changes

**If clean (no uncommitted changes):**
- Proceed to next step

### Step 4: Check Branch Merge Status

Get the branch name from the worktree.

Check if branch has been merged to main:
```bash
git branch --merged main | grep "[branch-name]"
```

**If NOT merged:**
- Warn user:
  ```
  Warning: Branch "[branch-name]" has NOT been merged to main!

  If you delete this branch, you may lose commits.

  Options:
  1. Remove worktree, keep branch (recommended)
  2. Remove worktree AND delete branch (commits will be lost)
  3. Cancel
  ```

**If merged (or user doesn't care):**
- Proceed to ask about branch deletion

### Step 5: Ask About Branch Deletion

Use AskUserQuestion:

**Question**: "Do you want to delete the branch '[branch-name]' as well?"

Options:
1. **Keep branch** - Only remove worktree, keep branch for potential reuse
2. **Delete branch** - Remove worktree AND delete the branch
3. **Cancel** - Don't remove anything

### Step 6: Remove Worktree

Run:
```bash
git worktree remove "[worktree-path]"
```

**If fails due to uncommitted changes and user confirmed force:**
```bash
git worktree remove --force "[worktree-path]"
```

Verify removal succeeded.

### Step 7: Delete Branch (if requested)

If user chose to delete branch:

**If branch was merged:**
```bash
git branch -d "[branch-name]"
```

**If branch was NOT merged (and user confirmed):**
```bash
git branch -D "[branch-name]"
```

### Step 8: Confirm Completion

Display summary:
```
Worktree removed successfully!

Removed: ../[worktree-name]
Branch: [kept / deleted]

Remaining worktrees: [N]
```

If there was a context file, mention:
```
Note: The worktree's context file (.claude-worktree-context.md) was removed with the worktree.
```

---

## Error Handling

**If not in main repository:**
- Detect if current directory is a worktree (not main)
- Error: "You appear to be in a worktree. Run this skill from the main repository."

**If worktree doesn't exist:**
- Check if path exists
- If not, suggest: "Worktree path not found. Run `git worktree prune` to clean up stale entries?"

**If git worktree remove fails:**
- Parse error message
- Common issues:
  - Uncommitted changes: offer force option
  - Path doesn't exist: suggest prune
  - Locked worktree: suggest checking if VS Code is open there
- Show helpful recovery options

**If branch delete fails:**
- `-d` fails if not merged: suggest `-D` with warning
- Branch doesn't exist: just note it and continue
- Branch is checked out elsewhere: explain the situation

---

## Safety Features

1. **Uncommitted changes warning**: Always check and warn before removing
2. **Merge status check**: Warn if branch isn't merged to prevent data loss
3. **Explicit confirmation**: Require user confirmation for destructive actions
4. **Force is opt-in**: Never force-remove without explicit user approval
5. **Branch deletion is separate**: Worktree removal and branch deletion are separate decisions

---

## Examples

### Example 1: Clean Removal (Merged Branch)

**User**: "/worktree-remove"

**Skill Actions**:
1. Lists worktrees:
   - ../feature-auth
   - ../bugfix-login
2. Asks: "Which worktree?" - User selects "feature-auth"
3. Checks status - clean (no uncommitted changes)
4. Checks merge status - branch IS merged to main
5. Asks: "Delete branch too?" - User selects "Delete branch"
6. Runs: `git worktree remove "../feature-auth"`
7. Runs: `git branch -d "feature-auth"`
8. Confirms: "Worktree removed, branch deleted"

### Example 2: Uncommitted Changes

**User**: "/worktree-remove feature-auth"

**Skill Actions**:
1. Parses: worktree = "feature-auth"
2. Checks status - HAS uncommitted changes:
   - `M src/auth/mod.rs`
   - `?? src/auth/new_file.rs`
3. Warns:
   ```
   Warning: This worktree has uncommitted changes!

   Changes:
   - M src/auth/mod.rs
   - ?? src/auth/new_file.rs

   These changes will be LOST if you proceed.
   ```
4. Asks: "Remove anyway?" - User selects "Cancel"
5. Aborts: "Cancelled. Worktree not removed."

### Example 3: Unmerged Branch

**User**: "/worktree-remove bugfix-login"

**Skill Actions**:
1. Finds worktree: ../bugfix-login
2. Checks status - clean
3. Checks merge status - branch NOT merged
4. Warns:
   ```
   Warning: Branch "bugfix-login" has NOT been merged to main!

   Options:
   1. Remove worktree, keep branch (recommended)
   2. Remove worktree AND delete branch
   3. Cancel
   ```
5. User selects "Keep branch"
6. Runs: `git worktree remove "../bugfix-login"`
7. Does NOT delete branch
8. Confirms: "Worktree removed, branch kept"

### Example 4: Force Remove

**User**: "/worktree-remove"

**Skill Actions**:
1. Lists worktrees, user selects one
2. Checks status - HAS uncommitted changes
3. Warns about losing changes
4. User selects "Force remove"
5. Runs: `git worktree remove --force "../[worktree]"`
6. Asks about branch
7. Completes removal

---

## Notes

- Always run from main repository, not from within a worktree
- Uncommitted changes are permanently lost when force-removing
- Keeping branches is safer; they can be deleted later
- After removing worktree, the branch still exists unless explicitly deleted
- `git worktree prune` can clean up stale worktree entries
