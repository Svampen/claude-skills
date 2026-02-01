---
name: session-end
description: Automates completing a development session. Verifies task completion, runs project-configured quality checks, updates session log. Use when user says "finish session", "complete session", or "session is done".
allowed-tools: Read, Glob, Edit, Write, Bash, TodoWrite, AskUserQuestion
user-invocable: true
---

# Session End Skill

Automates the completion of a development session. Verifies tasks, runs quality checks, and updates documentation.

## When to Use

Activate this skill when:
- User says "finish session", "complete session", or "end session"
- User says "session is done" or "ready to wrap up"
- User asks "what's left to finish the session?"
- User wants to wrap up session work

## Project Configuration

This skill reads project-specific settings from `.claude/project-config.md`. If the config doesn't exist, the skill will prompt to create one.

**Expected config sections:**

```markdown
## Quality Checks

Commands to run before completing a session:

### Format
```bash
cargo fmt --all
```

### Build
```bash
cargo build --workspace
```

### Lint
```bash
cargo clippy --workspace --all-targets
```

### Test
```bash
cargo test --workspace
```

### Custom Validation (Optional)
```bash
cargo run --bin validate_ron
```
```

The skill reads these commands and executes them in order.

## Instructions

### Step 0: Read Project Config

1. Check if `.claude/project-config.md` exists
2. If **YES**:
   - Parse `## Documentation Paths` section to get:
     - Development log path (default: `docs/development-log.md`)
     - Work streams path (default: `docs/work-streams/`)
     - Sessions path (default: `docs/sessions/`)
   - Parse `## Quality Checks` section for build commands
3. If **NO**: Prompt user to create one:
   ```
   No project config found. Would you like to create one?

   I need to know your project's quality check commands:
   - Format command (e.g., "cargo fmt", "npm run format", "go fmt ./...")
   - Build command (e.g., "cargo build", "npm run build", "go build ./...")
   - Lint command (e.g., "cargo clippy", "npm run lint", "golangci-lint run")
   - Test command (e.g., "cargo test", "npm test", "go test ./...")
   - Any custom validation commands?
   ```

   Create `.claude/project-config.md` with user's answers, then continue.

### Step 1: Find Current Session

**Identify the active session:**

1. Check TodoWrite list for session context (if available)
2. Glob all sessions: `[sessions-path]/*/*.md` (from config)
3. Filter and sort to find most recent with "Status: In Progress"
4. Read session file

**If no active session found:**
- Error: "No active session found. Use /session-start first?"
- List available in-progress sessions if any

### Step 2: Verify Task Completion

1. Read session log "Tasks" section
2. Read session log "What Was Accomplished" section
3. Compare tasks vs accomplishments:
   - Are all tasks addressed?
   - Are there incomplete tasks?
4. Check TodoWrite list (if available):
   - Any todos still "pending" or "in_progress"?

5. If tasks incomplete:
   ```
   Incomplete tasks detected:
   - [ ] Task that's not done
   - [ ] Another incomplete task

   Options:
   1. Continue working (don't finish session yet)
   2. Mark these as deferred (update session log with reason)
   3. Finish anyway (not recommended)

   What would you like to do?
   ```
   Wait for user decision.

### Step 3: Run Quality Checks (From Config)

**Read quality check commands from `.claude/project-config.md`** and execute in order:

1. **Format** (if configured):
   - Run format command
   - Report: "Format: Passed" or show errors

2. **Build** (if configured):
   - Run build command
   - If build fails, STOP: "Build failed. Please fix errors before finishing session."
   - Report: "Build: Passed (Xs)"

3. **Lint** (if configured):
   - Run lint command
   - Count warnings (if any)
   - Report: "Lint: Passed (N warnings)"

4. **Test** (if configured):
   - Run test command
   - If tests fail, STOP: "Tests failed. Please fix before finishing session."
   - Report: "Tests: Passed"

5. **Custom Validation** (if configured):
   - Run any custom validation commands
   - Report results

**If any critical check fails (build, test), STOP and inform user.**

### Step 4: Update Session Log

1. Update "Build Status" section with results:
   ```markdown
   ## Build Status

   - [x] Format: Passed
   - [x] Build: Passed (12.5s)
   - [x] Lint: Passed (3 warnings)
   - [x] Tests: Passed
   - [x] Custom: Passed
   - [ ] Manual testing: [User to confirm]
   ```

2. Update "Status" at top from "In Progress" to "Complete"

3. Add completion timestamp:
   ```markdown
   **Date**: YYYY-MM-DD
   **Status**: Complete
   **Completed**: YYYY-MM-DD HH:MM
   ```

4. If "What Was Accomplished" is empty, prompt user:
   ```
   Session log needs accomplishments summary. Please provide:
   - What features were implemented?
   - What bugs were fixed?
   - What was learned?
   ```

### Step 5: Ask About Manual Testing

Ask user:
```
Have you tested the changes manually?
- If YES: Continue to next step
- If NO: "Please test before finishing."
```

Do NOT proceed until user confirms testing (or explicitly skips).

### Step 6: Update Work Stream Document (if exists)

1. Check if work stream document exists: `[work-streams-path]/[topic-name].md` (from config)
2. If yes:
   - Add completed session to "Completed" section
   - Update status if applicable
   - Link to session log

### Step 7: Update Development Log (if exists)

1. Check if development log exists at `[dev-log-path]` (from config)
2. If yes:
   - Add entry for this session
   - Update "Active Work" section if applicable

Suggested entry format:
```markdown
## YYYY-MM-DD

### [Topic Name] Session: [Brief Title]

**Session**: [link to session log]

**Accomplished**:
- [Key accomplishment 1]
- [Key accomplishment 2]
- [Key accomplishment 3]
```

### Step 8: Prompt to Commit

Inform the user that the session is complete and ready to commit:

```
Session complete!

Quality checks passed. Documentation updated.
Use /git-commit when you're ready to commit your changes.
```

Do NOT stage or commit files — the `/git-commit` skill handles that.

### Step 9: Clean Up TodoWrite

1. Mark all todos as "completed"
2. Or clear the todo list entirely
3. Inform user: "Todo list cleared. Use /session-start for next session."

---

## Project Config Template

When creating `.claude/project-config.md`, use this template:

```markdown
# Project Configuration for Claude Skills

Project-specific settings used by Claude skills.

## Quality Checks

Commands to run before completing a session:

### Format
```bash
# Format command (e.g., cargo fmt, npm run format, go fmt)
[YOUR_FORMAT_COMMAND]
```

### Build
```bash
# Build command (e.g., cargo build, npm run build)
[YOUR_BUILD_COMMAND]
```

### Lint
```bash
# Lint command (e.g., cargo clippy, npm run lint)
[YOUR_LINT_COMMAND]
```

### Test
```bash
# Test command (e.g., cargo test, npm test)
[YOUR_TEST_COMMAND]
```

### Custom Validation (Optional)
```bash
# Any project-specific validation (e.g., RON validation, schema checks)
# Remove this section if not needed
[YOUR_CUSTOM_COMMAND]
```

## Insight Tags

Common tags for insight-capture skill:

- architecture
- performance
- [YOUR_PROJECT_TAGS]

## Documentation Paths

<!-- Customize these paths for your project structure -->
<!-- All paths are relative to project root -->

- Development log: docs/development-log.md
- Work streams: docs/work-streams/
- Sessions: docs/sessions/
- Decisions: docs/decisions/
- Insights: docs/insights/
- Discovery: docs/discovery/
```

---

## Examples

### Example 1: Clean Session Completion (Rust Project)

**User**: "finish session"

**Skill Actions**:
1. Reads `.claude/project-config.md` - finds Rust commands
2. Finds session: `[sessions-path]/user-auth/2026-01-11-auth-middleware.md`
3. Checks tasks: All 5 tasks completed
4. Runs quality checks from config:
   - `cargo fmt --all` - passed
   - `cargo build --workspace` - passed (12.5s)
   - `cargo clippy --workspace --all-targets` - passed (2 warnings)
   - `cargo test --workspace` - passed
5. Updates session log build status
6. Changes status to "Complete"
7. Asks: "Have you tested manually?" -> User: "Yes"
8. Updates work stream doc, dev log
9. Prompts: "Session complete! Use `/git-commit` to commit."

### Example 2: Build Fails

**User**: "let's wrap up"

**Skill Actions**:
1. Reads config
2. Runs format - passed
3. Runs build -> **ERROR**
4. Stops immediately
5. Reports:
   ```
   Build failed! Cannot finish session with errors.

   Error: [error message]

   Please fix the errors before finishing the session.
   ```
6. Does NOT proceed with session completion

---

## Error Handling

**If no config and user cancels creation:**
- Warn: "Cannot run quality checks without config."
- Ask: "Skip quality checks and proceed anyway? (Not recommended)"

**If no session found:**
- Error: "No active session found. Use /session-start first?"
- List available sessions if any

**If session already complete:**
- Warn: "Session already marked complete. Finish anyway?"

**If quality checks fail:**
- STOP immediately
- Report errors to user
- Do NOT proceed until issues fixed

**If user hasn't tested:**
- STOP unless user explicitly skips
- Remind: "Manual testing is important for quality"

---

## Notes

- Quality check commands come from `.claude/project-config.md`
- If no config exists, skill prompts to create one
- Config is per-project, skills are generic
- All quality checks are optional - config defines what's needed
- Manual testing confirmation is always asked (project-agnostic)
