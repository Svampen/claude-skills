---
name: team-stream-start
description: Reads a work stream and spawns an agent team to run its sessions in parallel across git worktrees. Use when the user wants to parallelize a multi-session work stream with agent teammates.
allowed-tools: Read, Glob, Write, Bash, AskUserQuestion, TeamCreate, TaskCreate, TaskUpdate, TaskList, Task, SendMessage
user-invocable: true
---

# Team Stream Start Skill

Reads an existing work stream document, analyzes session dependencies, and helps the team lead plan how to divide work across teammates and worktrees. The lead explicitly creates worktrees, assigns sessions to teammates, and manages the lifecycle across waves of work.

## When to Use

Activate this skill when:
- User says "start team stream", "team work on [topic]", "parallel sessions for [topic]"
- User wants to run multiple work stream sessions concurrently with agent teammates
- User says "run [topic] in parallel", "team up on [topic]"

**Prerequisites:**
- A work stream document must already exist (created by `/work-stream-start`)
- The environment variable `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` should be set

## Instructions

### Step 0: Read Project Config

1. Read `.claude/project-config.md` if it exists
2. Parse `## Documentation Paths` section to get:
   - Work streams path (default: `docs/work-streams/`)
   - Sessions path (default: `docs/sessions/`)
3. Parse `## Co-Author` section for commit trailer preference (default: `auto`)

**If no config exists:** use defaults and continue.

### Step 1: Find and Parse Work Stream

1. If the user specified a topic, look for `[work-streams-path]/<topic>.md`
2. If not specified, glob `[work-streams-path]/*.md` and list active work streams
   - Use AskUserQuestion: "Which work stream should the team work on?"
   - List active work streams as options
3. Read the selected work stream document
4. Extract:
   - **Topic name** (from heading)
   - **Purpose** (from `## Purpose`)
   - **Goals** (from `## Goals`)
   - **Planned sessions** (from `### Planned` under `## Sessions`)
   - **Dependencies** (from `## Dependencies`)
   - **Success criteria** (from `## Success Criteria`)

**If no planned sessions found:**
- Error: "Work stream has no planned sessions. Add sessions to the `### Planned` section first."
- Stop skill execution

### Step 2: Analyze Dependencies and Plan Assignments

For each planned session entry, derive:
- **Session name**: human-readable name from the checklist item text
- **Session slug**: kebab-case version (e.g., "Add JWT support" -> `add-jwt-support`)
- **Session goal**: the full checklist item text
- **Dependencies**: infer from ordering and any explicit dependency notes
  - By default, sessions are independent (can run in parallel)
  - If a session references output from another (e.g., "after auth middleware is done"), mark it as dependent

Then group sessions into **waves** based on dependencies:

- **Wave 1**: all sessions with no dependencies (can start immediately)
- **Wave 2**: sessions that depend on wave 1 results (need wave 1 merged first)
- **Wave N**: sessions that depend on wave N-1, and so on

Within each wave, sessions run in parallel — each gets its own worktree and teammate.

**Key principle**: `isolation: worktree` does NOT exist for agent team teammates — only for subagents. The lead MUST create worktrees manually and assign each teammate to work in a specific worktree directory. Dependent sessions cannot simply wait on a task — they need the prior wave's code merged back to the base branch so their worktree starts from the right state.

### Step 3: Present Plan and Get Approval

Display the full plan to the user:

```
Team Stream Plan for: [Topic Name]

Work stream: [work-streams-path]/[topic].md
Base branch: [current branch]

Wave 1 (starts immediately) — [N] teammates:
  1. [session-slug] — "[session goal]"
     Worktree: .worktrees/<topic>-<session-slug>/
     Branch: <topic>/<session-slug>

  2. [session-slug] — "[session goal]"
     Worktree: .worktrees/<topic>-<session-slug>/
     Branch: <topic>/<session-slug>

Wave 2 (after wave 1 merges) — [N] teammates:
  3. [session-slug] — "[session goal]"
     Worktree: .worktrees/<topic>-<session-slug>/
     Branch: <topic>/<session-slug>
     Depends on: #1, #2

  ...

Workflow:
  1. Create worktrees and spawn teammates for wave 1
  2. When wave 1 completes → review, merge worktrees back to [base branch]
  3. Create worktrees for wave 2 (branching from updated [base branch])
  4. Spawn teammates for wave 2
  5. Repeat until all waves complete

Proceed with wave 1?
```

Use AskUserQuestion with options:
1. **Yes, start wave 1** (Recommended)
2. **Modify plan** — adjust grouping, dependencies, or skip sessions
3. **Cancel**

**If "Modify plan":** ask what to change (move sessions between waves, combine sessions into one assignment, remove sessions, adjust dependencies). Rebuild and re-present.

**If "Cancel":** stop skill execution.

### Step 4: Create Worktrees for Current Wave

Create worktrees **only for the current wave** (wave 1 on first run). Dependent sessions get their worktrees later, after prior waves are merged.

For each session in the current wave:

```bash
mkdir -p .worktrees
git worktree add ".worktrees/<topic>-<session-slug>" -b "<topic>/<session-slug>"
```

After creating each worktree, copy Claude Code settings:

```bash
if [ -d ".claude" ]; then
  mkdir -p ".worktrees/<topic>-<session-slug>/.claude"
  [ -f ".claude/settings.local.json" ] && cp ".claude/settings.local.json" ".worktrees/<topic>-<session-slug>/.claude/"
  [ -f ".claude/settings.json" ] && cp ".claude/settings.json" ".worktrees/<topic>-<session-slug>/.claude/"
fi
```

**If worktree creation fails:**
- If branch already exists: warn and ask to reuse or pick a different name
- If path already exists: warn and ask to reuse or clean up first

Track the absolute path of each worktree for teammate prompts.

### Step 5: Create Agent Team

Use TeamCreate to create the team:

```
team_name: "<topic>-team"
description: "Parallel implementation of [Topic Name] work stream"
```

### Step 6: Create Tasks for Current Wave

Use TaskCreate for each session **in the current wave**. For each task:

- **subject**: `[session-slug]: [session goal]` (short imperative form)
- **description**: Include the full context the teammate needs (see reference doc for template)
- **activeForm**: `Implementing [session-slug]`

Do NOT create tasks for future waves yet — those worktrees don't exist and the base code may change after merging.

### Step 7: Spawn Teammates for Current Wave

For each session in the current wave, spawn a teammate using the Task tool:

```
Task:
  name: "<session-slug>"
  team_name: "<topic>-team"
  subagent_type: "general-purpose"
  prompt: <see references/team-patterns.md for the full prompt template>
```

**Key rules for teammate prompts:**
- Include the full work stream context (overall goal, their specific session)
- Include their worktree absolute path — instruct them to ONLY work within it
- Instruct them to commit their work before finishing (using conventional commits)
- Instruct them to message other teammates if they discover overlapping concerns
- Include the co-author setting from project config
- Include relevant CLAUDE.md / project conventions

See `references/team-patterns.md` for the complete prompt template.

### Step 8: Report Launch Status

Display:

```
Wave 1 launched!

Team: <topic>-team
Teammates: [N] agents spawned
Worktrees:
  - .worktrees/<topic>-<session-1>/  (branch: <topic>/<session-1>)
  - .worktrees/<topic>-<session-2>/  (branch: <topic>/<session-2>)

Monitor progress:
- Check TaskList for task status
- Teammates will message you when done or if they need help

When wave 1 completes:
1. Review each worktree's changes
2. Merge worktrees back to [base branch] using /worktree-end
3. I will then create worktrees for wave 2 and spawn the next teammates

Remaining waves:
  Wave 2: [session-slug-3], [session-slug-4]
  Wave 3: [session-slug-5]
```

### Step 9: Manage Subsequent Waves (Lead Responsibility)

When all teammates in a wave complete their tasks:

1. **Review results**: check each worktree for quality
2. **Merge back**: use `/worktree-end` for each worktree to merge branches into the base branch
3. **Create next wave's worktrees**: repeat Step 4 for the next wave — worktrees branch from the updated base so they include all prior wave's code
4. **Create tasks and spawn teammates**: repeat Steps 6-7 for the next wave
5. **Report**: repeat Step 8

Continue until all waves are complete.

**This is the lead's active responsibility** — the lead must drive the wave transitions because:
- Worktrees must be created manually (no `isolation: worktree` for teammates)
- Each new wave needs code from prior waves merged into the base branch first
- The lead reviews and approves before merging

---

## Examples

### Example 1: All Sessions Independent (Single Wave)

**User**: "team work on user-authentication"

**Skill Actions**:
1. Reads `.claude/project-config.md` — gets paths
2. Reads `docs/work-streams/user-authentication.md`
3. Finds 3 planned sessions — all independent:
   - "Implement JWT token generation"
   - "Create auth middleware"
   - "Write integration tests"
4. Presents plan: 1 wave, 3 parallel teammates
5. User approves
6. Creates 3 worktrees:
   - `.worktrees/user-authentication-implement-jwt-token-generation/`
   - `.worktrees/user-authentication-create-auth-middleware/`
   - `.worktrees/user-authentication-write-integration-tests/`
7. Creates team `user-authentication-team`
8. Creates 3 tasks, spawns 3 teammates
9. All 3 work in parallel — single wave, no follow-up needed

### Example 2: Multi-Wave with Dependencies

**User**: "/team-stream-start"

**Skill Actions**:
1. No topic specified — lists active work streams
2. User selects "api-refactor"
3. Reads work stream — finds 4 planned sessions:
   - "Define new API schema"
   - "Implement new endpoints" (depends on schema)
   - "Migrate existing clients" (depends on endpoints)
   - "Update documentation" (depends on endpoints)
4. Groups into waves:
   - **Wave 1**: "Define new API schema" (independent)
   - **Wave 2**: "Implement new endpoints" (needs schema merged first)
   - **Wave 3**: "Migrate existing clients" + "Update documentation" (need endpoints merged, can run in parallel)
5. Presents plan — user approves wave 1
6. Creates 1 worktree, spawns 1 teammate for wave 1
7. When wave 1 teammate finishes:
   - Lead reviews and merges worktree back to main via `/worktree-end`
   - Lead creates 1 worktree for wave 2 (branching from updated main)
   - Lead spawns 1 teammate for wave 2
8. When wave 2 finishes:
   - Lead merges, creates 2 worktrees for wave 3
   - Lead spawns 2 teammates — they run in parallel
9. When wave 3 finishes — lead merges both, work stream complete

### Example 3: User Regroups Sessions

**User**: "parallel sessions for database-migration"

**Skill Actions**:
1. Reads work stream — finds 5 planned sessions
2. Presents plan with 3 waves
3. User selects "Modify plan":
   - Moves session 3 from wave 2 into wave 1 (it's actually independent)
   - Combines sessions 4 and 5 into one assignment (small enough for one teammate)
4. Rebuilds plan: wave 1 has 4 teammates, wave 2 has 1 teammate
5. Re-presents — user approves
6. Creates 4 worktrees, spawns 4 teammates for wave 1

---

## Error Handling

**If work stream document not found:**
- Error: "No work stream found for '[topic]'. Run /work-stream-start first to create one."
- List available work streams if any exist

**If no planned sessions in work stream:**
- Error: "Work stream has no planned sessions under `### Planned`."
- Suggest: "Add session items to the work stream document and try again."

**If worktree creation fails (branch exists):**
- Ask: "Branch `<topic>/<session-slug>` already exists. Reuse it, or choose a different name?"
- If reuse: `git worktree add ".worktrees/..." "<topic>/<session-slug>"` (without `-b`)

**If worktree creation fails (path exists):**
- Ask: "Worktree path already exists. Remove it and recreate, or reuse?"
- If remove: `git worktree remove ".worktrees/..." && git worktree add ...`

**If TeamCreate fails (teams not enabled):**
- Error: "Agent teams require `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`."
- Suggest: "Set the environment variable and try again."

**If not in a git repository:**
- Error: "Must be in a git repository to create worktrees."

**If already in a worktree:**
- Error: "Run this from the main repository, not from within a worktree."

---

## Cleanup

After each wave completes, the lead handles merge and cleanup:

```
Wave [N] complete! Next steps:

For each worktree in this wave:
  - Review changes: git -C ".worktrees/<name>" log --oneline main..HEAD
  - Merge and clean up: /worktree-end
  - Or discard: /worktree-remove

[If more waves remain:]
  After merging, I'll create worktrees for wave [N+1].

[If all waves done:]
  All waves complete!
  - Update the work stream doc to mark sessions as completed
  - Clean up .worktrees/ directory if empty
  - Shut down the team
```

---

## Notes

- **No worktree isolation for teammates**: `isolation: worktree` is a subagent-only feature — it does NOT work for agent team teammates. This skill creates worktrees manually and instructs each teammate to work exclusively within their assigned directory. This is why the lead must actively manage worktree creation and assignment.
- **Lead drives the lifecycle**: the lead is responsible for creating worktrees, spawning teammates, reviewing results, merging completed waves back to the base branch, and then creating worktrees for the next wave. This cannot be fully automated because each new wave needs the prior wave's code in the base branch.
- **Waves, not task dependencies**: dependent sessions are grouped into later waves rather than using `blockedBy` task relationships. A wave 2 teammate needs the actual merged code from wave 1 in their worktree — a task dependency alone would unblock them but they'd be working from stale code.
- Worktrees are created inside `.worktrees/` in the project root (not as siblings) to keep them organized and easily discoverable.
- Each teammate gets the full work stream context so they understand the bigger picture, not just their slice.
- Teammates should use conventional commits (the prompt template includes this instruction).
- Co-author setting from project config is passed through to teammate commit instructions.
