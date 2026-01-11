---
name: session-start
description: Automates starting a new development session. Reads work stream or topic plan, creates session log, pre-populates tasks, sets up TodoWrite checklist. Use when user says "start session", "begin session", or "set up session".
allowed-tools: Read, Glob, Write, Bash, TodoWrite, AskUserQuestion
user-invocable: true
---

# Session Start Skill

Automates the initialization of a new development session. Creates session logs, pre-populates tasks, and sets up tracking.

## When to Use

Activate this skill when:
- User says "start new session", "begin next session", or "set up session"
- User asks "what's next?" or "what should we work on?"
- Beginning work after reviewing a work stream or plan
- Starting the day's development work

## Instructions

### Step 1: Check for Worktree Context File

**Check if this is a worktree with pre-configured context:**

1. Check if `.claude-worktree-context.md` exists in project root
2. If **YES** (worktree context found):
   - Read the context file
   - Extract:
     - Topic/Work stream name (from "**Topic**:" or "**Work Stream**:" field)
     - Session goals (from "## Session Goals" section)
     - Suggested tasks (from "## Suggested Tasks" section)
   - Inform user: "Detected worktree context for [topic]. Using pre-configured details..."
   - **Delete the context file** after reading (it's only needed for the first session; subsequent sessions should use normal flow)
   - Skip to **Step 4** with pre-populated values
3. If **NO** (no context file):
   - Continue to Step 2 (normal flow)

### Step 2: Find Active Work

**Check for active work streams or topics:**

1. Check if `docs/development-log.md` exists
   - If yes, look for "Active Work" section
   - Extract active topics/work streams
2. Check for work stream files: `docs/work-streams/*.md`
3. Build list of available options

**If work streams found:**
- List active work streams
- Ask user which one to work on

**If no work streams found:**
- Ask user: "What topic/feature will you work on this session?"
- User can provide a custom topic name

### Step 3: Read Work Stream Document (if exists)

1. If work stream selected, check if `docs/work-streams/[topic-name].md` exists
2. If yes:
   - Read the document
   - Extract: Current status, Goals, Planned sessions, Latest completed session
3. If no:
   - Continue with user-provided topic
   - Create minimal session structure

### Step 4: Prompt for Session Details

**If coming from worktree context:**

Present pre-populated values and ask for confirmation:

```
Detected worktree context for [topic].

Suggested session details:
- Goal: [session goals from context file]
- Tasks:
  - [task 1 from context]
  - [task 2 from context]
  - [task 3 from context]
- Title suggestion: [derived from topic name]

Accept these details? Or would you like to modify them?
```

**Normal flow (no worktree context):**

Ask user for session information:

**Question 1**: "What is the goal of this session? (1-2 sentences)"
- Brief description of what this session will accomplish
- Should align with work stream goals (if applicable)
- Example: "Implement user authentication middleware"

**Question 2**: "What are the main tasks for this session? (3-5 tasks)"
- Specific, actionable tasks
- Example:
  - Create auth middleware
  - Add JWT validation
  - Write tests
  - Update documentation

**Question 3**: "Brief title for this session?"
- Short title (2-5 words) for filename
- Example: "auth-middleware"
- Will become: `YYYY-MM-DD-auth-middleware.md`

### Step 5: Create Session Log File

1. Ensure session folder exists: `docs/sessions/[topic-name]/`
2. Generate filename: `docs/sessions/[topic-name]/YYYY-MM-DD-[brief-title].md`
3. Use current date for YYYY-MM-DD
4. Create session log with this template:

```markdown
# [Topic Name] Session: [Brief Title]

**Date**: YYYY-MM-DD
**Status**: In Progress

## Session Goals

[Goal provided by user]

## Tasks

- [ ] [Task 1 from user]
- [ ] [Task 2 from user]
- [ ] [Task 3 from user]
- [ ] Run tests/build (if applicable)
- [ ] Update documentation

## What Was Accomplished

[To be filled during session]

## Technical Details

[To be filled during session]

## Challenges & Solutions

[To be filled during session]

## Files Changed

**New**:
- [To be filled]

**Modified**:
- [To be filled]

## Build Status

- [ ] Build: [status]
- [ ] Tests: [status]
- [ ] Manual testing: [status]

## Next Steps

[To be filled at end of session]
```

### Step 6: Create TodoWrite Checklist

Use TodoWrite to create checklist with:
- User-provided tasks (pending)
- Standard tasks: build/test, update docs

### Step 7: Confirm with User

Display summary:
```
Session started for: [Topic Name]

Date: YYYY-MM-DD
Goal: [Session goal]
Tasks: [N] tasks in todo list

Session log: docs/sessions/[topic-name]/YYYY-MM-DD-[title].md

Ready to begin! First task: [first task]
```

---

## Examples

### Example 1: Start Session from Worktree

**User**: "/session-start"

**Skill Actions**:
1. Finds `.claude-worktree-context.md` in project root
2. Extracts: Topic = "user-authentication", Goals, Tasks
3. Deletes the context file (consumed - won't confuse future sessions)
4. Shows pre-populated details
5. User accepts
6. Creates: `docs/sessions/user-authentication/2026-01-11-auth-middleware.md`
7. Creates TodoWrite with tasks
8. Confirms: "Session started for User Authentication"

### Example 2: Start Session with Work Stream

**User**: "start session"

**Skill Actions**:
1. No worktree context found
2. Checks dev log - finds active work stream: "api-refactor"
3. Asks: "Work on api-refactor?" - User confirms
4. Reads `docs/work-streams/api-refactor.md`
5. Asks for session goal, tasks, title
6. Creates session log and TodoWrite
7. Confirms: "Session started!"

### Example 3: Start Session without Work Stream

**User**: "start new session"

**Skill Actions**:
1. No worktree context, no work streams found
2. Asks: "What topic/feature will you work on?"
3. User: "bugfix-login-redirect"
4. Creates: `docs/sessions/bugfix-login-redirect/`
5. Asks for goal, tasks, title
6. Creates session log and TodoWrite
7. Confirms: "Session started for bugfix-login-redirect"

---

## Error Handling

**If no topic provided:**
- Ask user to specify what they're working on
- Can't create session without at least a topic name

**If session folder doesn't exist:**
- Create it automatically: `mkdir -p docs/sessions/[topic-name]/`

**If previous session incomplete:**
- Check latest session in folder for "Status: In Progress"
- Warn user:
  ```
  Previous session appears incomplete:
  - Session: [filename]
  - Status: In Progress

  Options:
  1. Continue previous session
  2. Mark previous as complete, start new
  3. Start new anyway
  ```

**If session file already exists for today:**
- Suggest adding suffix: `YYYY-MM-DD-[title]-2.md`
- Or ask user to choose different title

---

## Quality Checks

Before finalizing, verify:
- Topic/work stream identified
- Session goal stated
- At least 2-3 tasks defined
- Session log created with correct template
- TodoWrite tasks match session log tasks
- Filename follows convention

---

## Notes

- Sessions use date-based filenames: `YYYY-MM-DD-title.md`
- Multiple sessions per day possible (add suffix if needed)
- Worktree context file (`.claude-worktree-context.md`) is consumed and deleted after first session - this prevents stale context from confusing subsequent sessions
- TodoWrite tasks should match session log tasks
- Keep goals focused - one session, one main objective
