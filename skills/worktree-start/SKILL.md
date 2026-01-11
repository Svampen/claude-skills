---
name: worktree-start
description: Creates a git worktree for focused implementation work. Sets up context file and opens VS Code. Use when starting work on a feature/topic in a separate folder/branch.
allowed-tools: Read, Glob, Write, Bash, AskUserQuestion
user-invocable: true
---

# Worktree Start Skill

Creates a git worktree with context file for focused implementation work. The worktree is a separate working directory with its own branch, allowing parallel work on different features.

## When to Use

Activate this skill when:
- User wants to start implementation work in a separate folder
- User says "create worktree", "start worktree", or "set up worktree"
- User wants to work on a specific feature/topic in isolation
- User wants to separate planning (main repo) from implementation (worktree)

## Prerequisites

- Must be in the main repository (not already in a worktree)
- Work stream/feature should be defined (recommended)

## Instructions

### Step 1: Verify We're in Main Repo

1. Run `git worktree list` to see current worktrees
2. Verify we're in the main worktree (not a secondary one)
3. If already in a worktree, warn user:
   ```
   You appear to be in a worktree already.
   Run this skill from the main repository to create new worktrees.
   ```

### Step 2: Find Available Work Streams (Optional)

1. Check if `docs/development-log.md` exists
2. If yes, look for "Active Work" section and extract active topics
3. Also glob `docs/work-streams/*.md` for all available work streams
4. Build list of options (if any exist)

### Step 3: Ask User for Topic

Use AskUserQuestion to ask:

**Question 1**: "What topic/feature is this worktree for?"
- List active work streams as options (if any found)
- Include "Other (custom topic)" option
- If user selects "Other", ask for custom topic name

### Step 4: Ask for Worktree Configuration

**Question 2**: "Worktree folder name?"
- Suggest: topic name in kebab-case
- Example: if topic is "User Authentication", suggest "user-authentication"
- User can customize if desired

**Question 3**: "Branch strategy?"
Options:
1. **Create new branch** (Recommended) - Creates branch with same name as folder
2. **Use existing branch** - Ask which branch to check out
3. **Detached HEAD** - Use current commit without creating branch

### Step 5: Read Work Stream Document (if exists)

1. Check if `docs/work-streams/[topic-name].md` exists
2. If yes:
   - Read the document
   - Extract: Purpose, Goals, Next session tasks, Key files mentioned
3. If no:
   - Continue with minimal context (this is fine for ad-hoc work)

### Step 6: Create Git Worktree

Run the appropriate git command based on branch strategy:

**If creating new branch:**
```bash
git worktree add "../[worktree-name]" -b "[branch-name]"
```

**If using existing branch:**
```bash
git worktree add "../[worktree-name]" "[branch-name]"
```

**If detached HEAD:**
```bash
git worktree add --detach "../[worktree-name]"
```

Handle errors:
- If worktree already exists: "Worktree already exists at that location. Choose a different name?"
- If branch already exists (for new branch): "Branch already exists. Use existing branch instead?"

### Step 7: Create Context File

Write `.claude-worktree-context.md` in the new worktree root:

```markdown
# Worktree Context

**Topic**: [topic-name]
**Branch**: [branch-name]
**Created**: [YYYY-MM-DD]
**Source Repo**: [main repo folder name]

## Purpose

[Copied from work stream document, or user-provided description, or "Implementation work for [topic]"]

## Session Goals

[Next session goals from work stream, or "To be defined with /session-start"]

## Suggested Tasks

- [ ] [Task from work stream plan]
- [ ] [Task from work stream plan]
- [ ] [Additional tasks...]

[If no tasks defined: "Run /session-start to define session tasks"]

## Key Files

[Files/directories mentioned in work stream, or relevant directories for this work]

## Quick Start

1. This file is auto-read by `/session-start` skill
2. Run `/session-start` to create session log and todo list
3. Implement tasks
4. Run `/session-end` when done
5. Merge branch and remove worktree when work complete

## Links

- Work Stream: docs/work-streams/[topic-name].md (if exists)
- Dev Log: docs/development-log.md (if exists)
```

### Step 8: Copy Claude Code Settings

Copy personal Claude Code settings to the new worktree so permissions persist:

1. Check if `.claude/settings.local.json` exists in main repo
2. If yes:
   - Create `.claude/` directory in worktree (if not exists): `mkdir -p "../[worktree-name]/.claude"`
   - Copy settings: `cp ".claude/settings.local.json" "../[worktree-name]/.claude/settings.local.json"`
3. If no, skip (no action needed)

This ensures the new Claude Code session inherits your personal permission settings.

### Step 9: Open VS Code

Run:
```bash
code "../[worktree-name]"
```

This opens VS Code in the new worktree folder.

### Step 10: Display Confirmation

Show user:
```
Worktree created successfully!

Location: ../[worktree-name]
Branch: [branch-name]
Topic: [topic-name]
Context File: .claude-worktree-context.md

VS Code is opening in the worktree folder.

Next steps:
1. In the new VS Code window, open a terminal
2. Run `claude` to start Claude Code
3. Run `/session-start` - it will auto-detect the worktree context
4. Start implementing!

When done with the work:
- Commit your changes
- Use `/worktree-end` to merge and clean up (or `/worktree-remove` to just remove)

[If settings were copied: "Claude Code settings copied - permissions will be preserved."]
```

---

## Error Handling

**If git worktree add fails:**
- Parse error message
- Common issues:
  - Path already exists: suggest different name
  - Branch already exists: suggest using existing branch
  - Not a git repository: verify we're in main repo
- Show helpful error message with recovery options

**If work stream document doesn't exist:**
- Don't fail completely
- Continue with minimal context
- Context file will have placeholder values

**If VS Code fails to open:**
- Don't fail the skill
- Show manual instructions: "VS Code didn't open. Navigate to: ../[worktree-name]"
- Worktree is still created successfully

**If already in a worktree:**
- Detect via `git worktree list` output
- Refuse to create nested worktree
- Direct user to main repo

---

## Examples

### Example 1: Basic Worktree Creation

**User**: "/worktree-start"

**Skill Actions**:
1. Runs `git worktree list` - confirms we're in main repo
2. Asks: "What topic/feature?" - User enters "user-authentication"
3. Asks: "Worktree folder name?" - User accepts default
4. Asks: "Branch strategy?" - User selects "Create new branch"
5. Runs: `git worktree add "../user-authentication" -b "user-authentication"`
6. Writes `.claude-worktree-context.md` with details
7. Copies `.claude/settings.local.json` to worktree
8. Runs: `code "../user-authentication"`
9. Shows confirmation with next steps

### Example 2: With Existing Work Stream

**User**: "/worktree-start"

**Skill Actions**:
1. Finds active work streams in dev log
2. Lists options: "api-refactor", "database-migration", "Other"
3. User selects "api-refactor"
4. Reads `docs/work-streams/api-refactor.md` - extracts purpose/goals
5. Creates worktree with rich context from work stream
6. Opens VS Code

### Example 3: Using Existing Branch

**User**: "/worktree-start"

**Skill Actions**:
1. Asks: "What topic?" - User enters "bugfix-login"
2. Asks: "Worktree folder name?" - User accepts "bugfix-login"
3. Asks: "Branch strategy?" - User selects "Use existing branch"
4. Runs `git branch --list` to show available branches
5. Asks: "Which branch?" - User selects "feature/login-fix"
6. Runs: `git worktree add "../bugfix-login" "feature/login-fix"`
7. Writes context file
8. Opens VS Code

---

## Notes

- Worktrees are created in the parent folder (siblings of main repo)
- Each worktree has its own working directory but shares git history
- The context file `.claude-worktree-context.md` enables seamless session-start integration
- Branch name defaults to folder name for consistency
- VS Code opening is best-effort; worktree is still created even if VS Code fails
- Claude Code settings (`.claude/settings.local.json`) are copied to preserve personal permissions
