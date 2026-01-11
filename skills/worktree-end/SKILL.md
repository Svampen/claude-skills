---
name: worktree-end
description: Completes a worktree by checking for uncommitted changes, pushing to remote, merging to main, and removing the worktree. Use when finishing work on a feature.
allowed-tools: Read, Glob, Bash, AskUserQuestion
user-invocable: true
---

# Worktree End Skill

Completes a worktree workflow by safely finishing all work, pushing to remote for backup, merging into main, and cleaning up the worktree.

## When to Use

Activate this skill when:
- User wants to finish/complete a worktree and merge the work
- User says "end worktree", "finish worktree", "complete worktree", or "merge worktree"
- User has completed work on a feature and wants to integrate it
- User wants to merge a worktree branch and clean up

**For abandoned work or simple removal without merging**, use `/worktree-remove` instead.

## Prerequisites

- Must be in the main repository (not in the worktree being ended)
- Work in the worktree should be complete (all changes committed)
- Remote should be configured (for push step)

## Instructions

### Step 1: Get Available Worktrees

Run:
```bash
git worktree list
```

Parse output to get list of worktrees (excluding main).

### Step 2: Ask Which Worktree to End

If user didn't specify which worktree, use AskUserQuestion:

**Question**: "Which worktree do you want to end and merge?"
- List available worktrees as options
- Show branch name for each

### Step 3: Check for Uncommitted Changes

For the selected worktree, run:
```bash
git -C "[worktree-path]" status --porcelain
```

**If there are uncommitted changes:**
- Show the list of changed files
- Use AskUserQuestion:
  ```
  This worktree has uncommitted changes:
  - [list of files]

  What would you like to do?
  ```
  Options:
  1. **Commit changes** - I'll commit them with a message you provide
  2. **Discard changes** - Proceed anyway (changes will be lost)
  3. **Cancel** - Stop and let me handle it manually

**If "Commit changes" selected:**
- Ask for commit message
- Run: `git -C "[worktree-path]" add -A && git -C "[worktree-path]" commit -m "[message]"`

**If "Discard changes" selected:**
- Run: `git -C "[worktree-path]" checkout -- .`
- Continue to next step

**If clean:**
- Proceed to next step

### Step 4: Push Branch to Remote (Safety Backup)

Get the branch name from the worktree.

Check if remote tracking exists:
```bash
git -C "[worktree-path]" rev-parse --abbrev-ref --symbolic-full-name @{u} 2>/dev/null
```

**If no remote tracking:**
```bash
git -C "[worktree-path]" push -u origin [branch-name]
```

**If remote tracking exists:**
```bash
git -C "[worktree-path]" push
```

**If push fails:**
- Show error message
- Use AskUserQuestion:
  - **Retry** - Try pushing again
  - **Skip push** - Continue without pushing (not recommended)
  - **Cancel** - Stop and investigate

Display: "Branch pushed to remote for safety backup."

### Step 5: Rebase Branch onto Main

First, fetch latest from remote:
```bash
git fetch origin main
```

Check if branch has been merged to main:
```bash
git branch --merged main | grep "[branch-name]"
```

**If already merged:**
- Display: "Branch is already merged to main. Skipping rebase/merge step."
- Proceed to worktree removal

**If NOT merged:**
- Display: "Rebasing [branch-name] onto main for clean history..."
- Run rebase from the worktree:
  ```bash
  git -C "[worktree-path]" rebase main
  ```

**If rebase conflicts:**
- Abort the rebase immediately:
  ```bash
  git -C "[worktree-path]" rebase --abort
  ```
- Display:
  ```
  Rebase conflict detected. Aborting rebase.

  The worktree has NOT been modified.
  Please resolve this manually in the worktree:
    1. cd [worktree-path]
    2. git rebase main
    3. Resolve conflicts
    4. git rebase --continue
    5. Run /worktree-end again
  ```
- Stop skill execution (do NOT proceed to merge or removal)

**If rebase succeeds:**
- Display: "Successfully rebased [branch-name] onto main."

### Step 6: Fast-Forward Merge

After successful rebase, perform fast-forward merge:
```bash
git merge --ff-only [branch-name]
```

**If ff-only fails** (shouldn't happen after rebase, but safety check):
- Display error message
- Suggest manual resolution
- Stop skill execution

**If merge succeeds:**
- Display: "Fast-forward merged [branch-name] into main (clean linear history)."

### Step 7: Push Main to Remote

After successful merge:
```bash
git push origin main
```

Display: "Pushed updated main to remote."

### Step 8: Remove Worktree

Run:
```bash
git worktree remove "[worktree-path]"
```

Verify removal succeeded.

Display: "Worktree removed: [worktree-path]"

### Step 9: Ask About Branch Deletion

Use AskUserQuestion:

**Question**: "Do you want to delete the branch '[branch-name]'?"

Options:
1. **Delete local and remote** (Recommended) - Clean up completely
2. **Delete local only** - Keep remote branch
3. **Keep branch** - Don't delete anything

**If "Delete local and remote":**
```bash
git branch -d [branch-name]
git push origin --delete [branch-name]
```

**If "Delete local only":**
```bash
git branch -d [branch-name]
```

**If "Keep branch":**
- Do nothing, branch remains

### Step 10: Confirm Completion

Display summary:
```
Worktree ended successfully!

Summary:
- Worktree removed: ../[worktree-name]
- Branch rebased and fast-forward merged: [branch-name] -> main
- History: Clean linear (no merge commits)
- Branch status: [deleted / kept]
- Remote: [pushed / skipped]

Remaining worktrees: [N]
```

---

## Error Handling

**If not in main repository:**
- Detect if current directory is a worktree (not main)
- Error: "You appear to be in a worktree. Run this skill from the main repository."
- Suggest: "Navigate to [main-repo-path] and try again."

**If worktree doesn't exist:**
- Error: "Worktree not found. Use /worktree-list to see available worktrees."

**If push fails (network/auth):**
- Show error message
- Offer to skip push and continue (with warning)
- Offer to cancel

**If rebase fails (conflicts):**
- Immediately abort rebase: `git -C "[worktree-path]" rebase --abort`
- Do NOT attempt to resolve conflicts
- Instruct user to resolve manually in worktree, then retry /worktree-end
- This keeps the worktree intact for manual conflict resolution

**If worktree removal fails:**
- Check if VS Code or other process has files open
- Suggest closing editors/terminals in that worktree
- Offer force removal as last resort

**If branch deletion fails:**
- If `-d` fails (not fully merged): explain and offer `-D`
- If remote deletion fails: show error, continue with local deletion

---

## Safety Features

1. **Push before rebase**: Branch is pushed to remote before any destructive operations
2. **Uncommitted changes check**: Always check and prompt before proceeding
3. **Rebase abort on conflict**: Immediately aborts rebase on conflict, preserves worktree
4. **Fast-forward only**: Uses --ff-only to ensure clean linear history
5. **Explicit confirmation**: Each destructive step has user confirmation
6. **Graceful failure**: If any step fails, provide recovery options
7. **Main protection**: Only merges INTO main via ff-only, never force-pushes main

---

## Examples

### Example 1: Clean Completion

**User**: "/worktree-end"

**Skill Actions**:
1. Lists worktrees: `../feature-auth`
2. User selects it
3. Checks status - clean (no uncommitted changes)
4. Pushes branch to remote - success
5. Checks merge status - not merged
6. Rebases branch onto main - success
7. Fast-forward merges into main - success
8. Pushes main to remote - success
9. Removes worktree - success
10. Asks about branch deletion - user selects "Delete local and remote"
11. Deletes branch locally and on remote
12. Shows summary (clean linear history)

### Example 2: Has Uncommitted Changes

**User**: "/worktree-end feature-auth"

**Skill Actions**:
1. Finds worktree: `../feature-auth`
2. Checks status - HAS uncommitted changes:
   - `M src/auth/mod.rs`
3. Asks user what to do
4. User selects "Commit changes"
5. Asks for commit message
6. Commits changes
7. Continues with push, merge, removal...

### Example 3: Already Merged

**User**: "/worktree-end"

**Skill Actions**:
1. Lists worktrees, user selects one
2. Checks status - clean
3. Pushes branch to remote
4. Checks merge status - ALREADY merged
5. Displays: "Branch is already merged to main. Skipping merge step."
6. Removes worktree
7. Asks about branch deletion
8. Shows summary

### Example 4: Rebase Conflict

**User**: "/worktree-end"

**Skill Actions**:
1. Selects worktree
2. Checks status - clean
3. Pushes to remote - success
4. Attempts rebase - CONFLICT
5. Immediately aborts rebase: `git -C "../worktree" rebase --abort`
6. Shows:
   ```
   Rebase conflict detected. Aborting rebase.

   The worktree has NOT been modified.
   Please resolve this manually in the worktree:
     1. cd ../feature-auth
     2. git rebase main
     3. Resolve conflicts
     4. git rebase --continue
     5. Run /worktree-end again
   ```
7. Skill stops (worktree preserved for manual resolution)

---

## Comparison: worktree-end vs worktree-remove

| Feature | worktree-end | worktree-remove |
|---------|--------------|-----------------|
| Rebases onto main | Yes (clean history) | No |
| Merges to main | Yes (fast-forward) | No |
| Pushes to remote | Yes (safety) | No |
| Handles uncommitted changes | Prompts to commit | Warns only |
| Use case | Completing work | Abandoning/cleanup |
| Recommended for | Finished features | Experiments, already-merged |

---

## Notes

- Always run from main repository, not from within a worktree
- Push happens BEFORE rebase for safety (branch preserved on remote)
- Rebase + fast-forward merge ensures clean linear history (no merge commits)
- If rebase conflicts, skill aborts and preserves worktree for manual resolution
- If anything fails, branch is safely on remote for recovery
- Branch deletion is optional and happens last
