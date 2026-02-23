# Team Patterns Reference

Prompt templates and patterns for agent team teammates spawned by `team-stream-start`.

## Teammate Spawn Prompt Template

Use this template when spawning each teammate via the Task tool. Replace all `<placeholders>` with actual values.

```
You are a developer teammate working on the "<topic-name>" work stream.

## Your Assignment

**Session**: <session-slug>
**Goal**: <session-goal>
**Worktree**: <absolute-worktree-path>

## Work Stream Context

**Overall purpose**: <work-stream-purpose>
**Overall goals**:
<work-stream-goals as bullet list>

**Your session** is one of <N> sessions running in parallel (wave <W>). The other sessions in this wave are:
<list of other session slugs and their goals in this wave>

## Critical Rules

1. **Work ONLY in your worktree**: <absolute-worktree-path>
   - All file reads, writes, and edits MUST use paths under this directory
   - Do NOT modify files outside your worktree
   - Your working directory is: <absolute-worktree-path>

2. **Commit your work** before finishing:
   - Use conventional commits: `<type>(scope): description`
   - Stage and commit all changes
   - <co-author-instruction>

3. **Coordinate with teammates** if you discover:
   - Your work overlaps with another session's scope
   - You need something from another session that isn't available yet
   - You've made a decision that affects other sessions
   Use SendMessage to notify the relevant teammate by name.

4. **Follow project conventions**:
   - Read CLAUDE.md in your worktree for project-specific instructions
   - Follow existing code patterns and style
   - Write tests if the project has them

## Task Completion

When you finish your session:
1. Ensure all changes are committed
2. Run any quality checks defined in .claude/project-config.md (if it exists)
3. Mark your task as completed via TaskUpdate
4. Send a summary message to the team lead describing what you accomplished

## Your Goal

Implement: <session-goal>

Start by exploring the codebase in your worktree to understand the current state, then implement the changes needed to accomplish your session goal.
```

## Co-Author Instruction Variants

Based on the project config `## Co-Author` setting, insert one of these into the prompt:

**auto (default):**
```
Add a Co-Authored-By trailer to each commit:
Co-Authored-By: Claude <noreply@anthropic.com>
```

**none:**
```
Do NOT add a Co-Authored-By trailer to commits.
```

**Custom value (e.g., `Name <email>`):**
```
Add a Co-Authored-By trailer to each commit:
Co-Authored-By: <custom-value>
```

## Task Description Template

Use this when creating tasks via TaskCreate for each session:

```
Implement session "<session-slug>" for the <topic-name> work stream.

Goal: <session-goal>

Worktree: <absolute-worktree-path>
Branch: <topic>/<session-slug>

Context:
<relevant details from the work stream doc for this specific session>

Definition of done:
- Changes implemented and committed in the worktree
- Conventional commit messages used
- Quality checks pass (if configured)
```

## Wave Patterns

Dependencies between sessions are handled by grouping them into sequential **waves**, not by task `blockedBy` relationships. Each wave's code must be merged into the base branch before the next wave's worktrees are created — this is how dependent teammates get the code they depend on.

### Single Wave (All Independent)

All sessions can run in parallel. The lead creates all worktrees and spawns all teammates at once.

```
Wave 1: A, B, C (parallel)
→ All complete → merge all → done
```

### Linear Chain

Each session depends on the previous one. One teammate per wave.

```
Wave 1: A
→ Merge A → create worktree for B
Wave 2: B
→ Merge B → create worktree for C
Wave 3: C
→ Merge C → done
```

### Fan-out

Session A must complete before B, C, D can start in parallel.

```
Wave 1: A
→ Merge A → create worktrees for B, C, D
Wave 2: B, C, D (parallel)
→ Merge all → done
```

### Fan-in

Sessions A, B, C are independent, but D needs all of them merged first.

```
Wave 1: A, B, C (parallel)
→ Merge all → create worktree for D
Wave 2: D
→ Merge D → done
```

### Mixed

Combine the above patterns as needed. The lead decides the grouping.

```
Wave 1: A, B (parallel, independent)
→ Merge both
Wave 2: C (depends on A), D (depends on B) — parallel
→ Merge both
Wave 3: E (depends on C and D)
→ Merge → done
```

### Why Waves Instead of Task Dependencies

Task `blockedBy` relationships unblock a teammate to start working, but the teammate's worktree was created before the blocking work existed. The teammate would be working from stale code that doesn't include the dependency's changes.

By using waves:
1. Wave 1 completes and merges into the base branch
2. Wave 2 worktrees branch from the updated base
3. Wave 2 teammates start with all of wave 1's code already present

This is necessary because `isolation: worktree` does not exist for agent team teammates — only for subagents. The lead must create worktrees manually at the right time.
