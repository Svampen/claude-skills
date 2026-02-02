---
name: work-stream-start
description: Creates a new work stream for multi-session development efforts. Optionally seeds from a Jira ticket. Use when starting work on a new feature/topic or beginning multi-session effort. Creates work stream doc, sets up session folder, updates dev log.
allowed-tools: Read, Glob, Write, AskUserQuestion
user-invocable: true
---

# Work Stream Start Skill

Automates the creation of new work streams for organized, multi-session development efforts.

## When to Use

Activate this skill when:
- User says "start work on...", "let's work on...", "begin work stream for..."
- Starting work on a new feature/topic
- Beginning a multi-session effort
- User asks "create work stream", "set up new work"
- Planning significant new development effort
- User says "start work stream for ABC-123" or "create work stream from ticket"
- User provides a Jira ticket key (e.g., PROJ-456) when starting a work stream

## Purpose

Work streams provide semantic topic-based organization for development work:
- **Semantic naming**: Topic visible in path (e.g., `user-authentication`)
- **Parallel work**: Multiple work streams can be active simultaneously
- **Natural grouping**: All sessions for topic grouped together
- **Progress tracking**: Clear visibility into work status

## Instructions

### Step 0: Read Project Config

1. Read `.claude/project-config.md` if it exists
2. Parse `## Documentation Paths` section to get:
   - Development log path (default: `docs/development-log.md`)
   - Work streams path (default: `docs/work-streams/`)
   - Sessions path (default: `docs/sessions/`)

**If no config exists:**
- Use default paths listed above
- Continue with skill execution

### Step 1: Check for Jira Ticket (Optional)

Determine if the user provided a Jira ticket key:

1. Check if the user's request includes a Jira ticket key (pattern: 1-10 uppercase letters followed by a hyphen and digits, e.g., `PROJ-123`, `ABC-4567`)
2. If **no ticket key detected**:
   - Ask user: "Do you have a Jira ticket to seed this work stream from? (Enter ticket key, or skip for manual entry)"
   - If user skips or provides no key: continue to **Step 2** (normal manual flow)
   - If user provides a key: proceed to fetch below

3. If **ticket key detected or provided**:
   - Attempt to fetch ticket details via Jira MCP:
     - Key
     - Summary
     - Description
     - Type (Story, Bug, Task, Epic, Sub-task, etc.)
     - Priority
     - Acceptance criteria (if field exists)
     - Linked tickets (blocks, is blocked by, relates to)
     - Sub-tasks (if any)
     - Epic children (if ticket is an Epic)

4. **If Jira MCP unavailable or fetch fails**:
   - Inform user: "Could not connect to Jira MCP. Continuing with manual entry."
   - Continue to **Step 2** (normal manual flow)

5. **If fetch succeeds**, map Jira fields to work stream fields:

   | Jira Field | Work Stream Field | Notes |
   |---|---|---|
   | Summary | Topic name | Use as-is; user can refine |
   | Description | Purpose | Extract first 1-2 sentences or summary paragraph |
   | Priority (Highest/High/Medium/Low/Lowest) | Priority (High/Medium/Low) | Map: Highest/High → High, Medium → Medium, Low/Lowest → Low |
   | Acceptance criteria | Success criteria | Copy verbatim if present |
   | Linked tickets (blocks/is-blocked-by) | Dependencies (Depends on / Blocks) | Map relationship type to dependency direction |
   | Linked tickets (relates-to) | Dependencies (Related) | List as related work |
   | Sub-tasks | Goals | Each sub-task summary becomes a goal |
   | Epic children (stories/tasks) | Planned sessions | Each child summary becomes a planned session item |

   For tickets that are NOT Epics: if no sub-tasks exist, extract goals from acceptance criteria bullet points or description sections.

6. **Present pre-populated values for review**:

   ```
   Fetched details from Jira ticket [KEY]: [Summary]

   Pre-populated work stream details:
   - Topic: [Summary]
   - Purpose: [Extracted from description]
   - Priority: [Mapped priority]
   - Goals:
     - [Sub-task 1 or extracted goal 1]
     - [Sub-task 2 or extracted goal 2]
     - [...]
   - Dependencies:
     - Depends on: [blocked-by tickets]
     - Blocks: [blocks tickets]
     - Related: [relates-to tickets]
   - Success criteria:
     - [Acceptance criterion 1]
     - [Acceptance criterion 2]
     - [...]
   [If Epic] - Planned sessions:
     - [Child story/task 1]
     - [Child story/task 2]
     - [...]

   Accept these details? Or would you like to modify any fields?
   ```

7. **If user accepts**: skip to **Step 4** (Generate Filename and Path) with all fields populated
8. **If user wants modifications**: allow user to edit specific fields via AskUserQuestion, then skip to **Step 4**

### Step 2: Gather Work Stream Information

**If Jira ticket data was accepted in Step 1, skip to Step 4.**

Ask user to define the work stream using AskUserQuestion:

**Question 1**: "What is the topic/goal of this work stream?"
- Get brief descriptive name (2-5 words)
- Should be clear and searchable
- Examples: "User Authentication", "API Refactoring", "Database Migration"
- This becomes the work stream name

**Question 2**: "What is the purpose of this work stream? (1-2 sentences)"
- Get brief description of what problem it solves
- Focus on the "why" not the "how"
- Example: "Implement secure user authentication with JWT tokens"
- Example: "Refactor legacy API endpoints for better performance"

**Question 3**: "What are the main goals? (3-5 bullet points)"
- Prompt for specific, measurable goals
- Should be achievable outcomes
- Example:
  - Implement JWT token generation
  - Add refresh token support
  - Create auth middleware
  - Write comprehensive tests

**Question 4**: "What is the priority?"
- Options: High, Medium, Low
- High: Blocking other work, critical bug, important feature
- Medium: Valuable but not urgent
- Low: Nice to have, can be deferred

### Step 3: Check for Dependencies

**If Jira ticket data was accepted in Step 1 (dependencies already populated), skip to Step 4.**

Ask user using AskUserQuestion:

**Question**: "Does this work stream depend on other work?"
- Prompt for:
  - Related work streams (blocking or blocked by)
  - Prerequisite work that must be complete first
- Example: "Blocked by: database-migration (must complete first)"
- Example: "Related: api-v2-design"

### Step 4: Generate Filename and Path

Create work stream filename from topic:
- Convert to kebab-case
- Remove special characters
- Max 50 characters
- File: `[work-streams-path]/topic-name.md` (from config)

Example transformations:
- "User Authentication" -> `user-authentication.md`
- "API Refactoring v2" -> `api-refactoring-v2.md`
- "Database Migration MVP" -> `database-migration-mvp.md`

Also create session folder:
- `[sessions-path]/topic-name/` (from config, matches work stream name)

### Step 5: Check for Existing Work Stream

Use Glob to check if work stream already exists:
```
Glob: [work-streams-path]/*.md
```

If similar name found:
- Warn user: "Found similar work stream: [filename]"
- Ask: "Create new work stream or update existing one?"
- If update: Read existing, merge information, update status

### Step 6: Identify Success Criteria

**If Jira ticket data was accepted in Step 1 (success criteria already populated), skip to Step 7.**

Ask user using AskUserQuestion:

**Question**: "How do you know this work stream is complete? (success criteria)"
- Prompt for measurable outcomes
- Should be verifiable
- Example:
  - All API endpoints return valid JWT tokens
  - Auth middleware blocks unauthorized requests
  - All tests pass with 80%+ coverage

### Step 7: Create Work Stream Document

Write to `[work-streams-path]/topic-name.md` (from config):

```markdown
# [Topic Name]

**Status**: Active
**Created**: YYYY-MM-DD
**Priority**: [High|Medium|Low]
**Jira**: [TICKET-KEY](jira-url) *(only include if seeded from Jira)*

## Purpose

[What problem does this work stream address? 1-2 sentences]

## Goals

- Goal 1
- Goal 2
- Goal 3
- [Additional goals...]

## Dependencies

**Depends on**:
- [Related work stream or prerequisite work]

**Blocks**:
- [Work streams waiting on this to complete]

**Related**:
- [Related but not blocking work]

## Sessions

### Completed
[Empty initially - populated as sessions complete]

### Planned
- [ ] [First planned session goal]
- [ ] [Second planned session goal]
- [ ] [Additional planned sessions...]

## Success Criteria

[How do we know this work stream is complete?]

- Criterion 1
- Criterion 2
- Criterion 3

## Notes

[If seeded from Jira:]
- Seeded from Jira ticket [TICKET-KEY] on YYYY-MM-DD
- Jira description and acceptance criteria used as starting point
- Refer to Jira for latest ticket state

[Additional context, learnings, or references]
[This section grows over time as work progresses]
```

### Step 8: Create Session Folder

Create directory: `[sessions-path]/topic-name/` (from config)

```bash
mkdir -p [sessions-path]/[topic-name]
```

### Step 9: Update Development Log (if exists)

Check if development log exists at `[dev-log-path]` (from config). If yes, add work stream to "Active Work" section:

```markdown
### [Topic Name]
**Work Stream**: [work-streams/topic-name.md](work-streams/topic-name.md)
**Status**: Active (YYYY-MM-DD)
**Priority**: [High|Medium|Low]

[Brief 1-sentence description]

**Latest session**: (none yet - just started)
```

### Step 10: Confirm Creation

Display summary:
```
Work stream created!

Location: [work-streams-path]/topic-name.md
Sessions folder: [sessions-path]/topic-name/

Topic: [Topic Name]
Priority: [High|Medium|Low]
Goals: [N] goals defined
Success criteria: [N] criteria defined
[If Jira] Jira: [TICKET-KEY]

Ready to start first session! Use /session-start
```

### Step 11: Suggest Next Steps

Based on work stream, suggest appropriate next actions:

**If work stream has clear first session**:
```
First session goal seems clear: [goal 1]

Would you like to start the first session now?
```

**If work stream needs more planning**:
```
Consider planning out session breakdown before starting.

Questions to consider:
- What's the first deliverable?
- How many sessions needed?
- Any dependencies to resolve first?
```

---

## Examples

### Example 1: Feature Work Stream

**User**: "start work stream for user authentication"

**Skill Actions**:
1. Asks if user has a Jira ticket — user skips
2. Asks topic: "User Authentication"
3. Asks purpose: "Implement secure user authentication with JWT tokens"
4. Asks goals:
   - Implement JWT token generation
   - Add refresh token support
   - Create auth middleware
   - Write tests
5. Asks priority: "High"
6. Asks dependencies: "None - can start immediately"
7. Asks success criteria:
   - All endpoints return valid tokens
   - Middleware blocks unauthorized requests
   - 80%+ test coverage
8. Creates work stream file at `[work-streams-path]/user-authentication.md`
9. Creates session folder at `[sessions-path]/user-authentication/`
10. Updates dev log (if exists)
11. Suggests: "First goal is clear. Start session now?"

### Example 2: Refactoring Work Stream

**User**: "let's start work on API refactoring"

**Skill Actions**:
1. Asks if user has a Jira ticket — user skips
2. Asks topic: "API Refactoring"
3. Asks purpose: "Improve API performance and maintainability"
4. Asks goals:
   - Remove deprecated endpoints
   - Standardize response formats
   - Add pagination to list endpoints
   - Update documentation
5. Asks priority: "Medium"
6. Asks dependencies: "Depends on: database-migration"
7. Asks success criteria:
   - All endpoints follow REST conventions
   - Response times under 200ms
   - Documentation updated
8. Creates work stream and session folder
9. Notes dependency in docs
10. Suggests: "This depends on database-migration. Start when ready?"

### Example 3: Work Stream from Jira Ticket

**User**: "start work stream for PROJ-456"

**Skill Actions**:
1. Detects Jira ticket key `PROJ-456`
2. Fetches via Jira MCP: Summary="User Authentication Overhaul", Description, Priority=High, 3 sub-tasks, 2 acceptance criteria, 1 linked ticket
3. Presents pre-populated details:
   - Topic: "User Authentication Overhaul"
   - Purpose: (from description)
   - Priority: High
   - Goals: (from 3 sub-tasks)
   - Dependencies: Related: PROJ-400
   - Success criteria: (from 2 acceptance criteria)
4. User accepts with one modification: changes priority to Medium
5. Creates `user-authentication-overhaul.md` with Jira reference
6. Creates session folder
7. Updates dev log
8. Suggests: "First goal is clear. Start session now?"

### Example 4: Jira Unavailable Fallback

**User**: "start work stream for PROJ-789"

**Skill Actions**:
1. Detects Jira ticket key `PROJ-789`
2. Attempts Jira MCP fetch — connection fails
3. Informs user: "Could not connect to Jira MCP. Continuing with manual entry."
4. Falls through to Step 2 (normal manual flow)
5. Proceeds with standard question-based workflow

---

## Error Handling

**If work stream name is too vague:**
- Warn: "Work stream name should be specific (e.g., 'User Authentication' not 'Auth Work')"
- Ask user to provide more specific name

**If work stream name already exists:**
- Warn: "Work stream already exists: [filename]"
- Ask: "Update existing or create new with different name?"

**If no goals provided:**
- Warn: "Work stream should have at least 2-3 goals"
- Prompt: "What specific outcomes do you want to achieve?"

**If docs folders don't exist:**
- Create them: `mkdir -p [work-streams-path] [sessions-path]`
- Continue with creation

**If user cancels mid-creation:**
- Ask: "Discard work stream or save as draft?"
- If save: Mark status as "Draft" and note incomplete sections

**If Jira MCP unavailable:**
- Inform user: "Could not connect to Jira MCP. Continuing with manual entry."
- Fall through to Step 2 (normal manual flow)
- Do not block work stream creation

**If Jira ticket not found:**
- Inform user: "Ticket [KEY] not found in Jira. Check the key and try again, or continue with manual entry."
- Ask: "Try a different ticket key, or continue without Jira?"

**If Jira fields are sparse (no description, no acceptance criteria, etc.):**
- Pre-populate what is available
- Note missing fields in the confirmation prompt
- Ask user to fill in missing fields manually

---

## Quality Checks

Before finalizing, verify:
- Clear topic name (semantic, searchable)
- Purpose stated (why does this exist?)
- At least 2-3 goals defined
- Priority assigned (High/Medium/Low)
- Success criteria defined (measurable)
- Session folder created
- Development log updated (if exists)

---

## Notes

- Work streams are semantic (describe what, not just numbers)
- Multiple work streams can be active simultaneously
- Work streams can be paused/resumed (mark status accordingly)
- Sessions within work stream use date-based naming: `YYYY-MM-DD-title.md`
- Work stream status: Active, Planned, Paused, Complete
- Update work stream document as work progresses (add learnings, notes)
- Jira integration is read-only — it does not assign, transition, or modify the ticket
- When seeded from Jira, all fields are presented for review before use
