---
name: work-stream-start
description: Creates a new work stream for multi-session development efforts. Use when starting work on a new feature/topic or beginning multi-session effort. Creates work stream doc, sets up session folder, updates dev log.
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

## Purpose

Work streams provide semantic topic-based organization for development work:
- **Semantic naming**: Topic visible in path (e.g., `user-authentication`)
- **Parallel work**: Multiple work streams can be active simultaneously
- **Natural grouping**: All sessions for topic grouped together
- **Progress tracking**: Clear visibility into work status

## Instructions

### Step 1: Gather Work Stream Information

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

### Step 2: Check for Dependencies

Ask user using AskUserQuestion:

**Question**: "Does this work stream depend on other work?"
- Prompt for:
  - Related work streams (blocking or blocked by)
  - Prerequisite work that must be complete first
- Example: "Blocked by: database-migration (must complete first)"
- Example: "Related: api-v2-design"

### Step 3: Generate Filename and Path

Create work stream filename from topic:
- Convert to kebab-case
- Remove special characters
- Max 50 characters
- File: `docs/work-streams/topic-name.md`

Example transformations:
- "User Authentication" -> `user-authentication.md`
- "API Refactoring v2" -> `api-refactoring-v2.md`
- "Database Migration MVP" -> `database-migration-mvp.md`

Also create session folder:
- `docs/sessions/topic-name/` (matches work stream name)

### Step 4: Check for Existing Work Stream

Use Glob to check if work stream already exists:
```
Glob: docs/work-streams/*.md
```

If similar name found:
- Warn user: "Found similar work stream: [filename]"
- Ask: "Create new work stream or update existing one?"
- If update: Read existing, merge information, update status

### Step 5: Identify Success Criteria

Ask user using AskUserQuestion:

**Question**: "How do you know this work stream is complete? (success criteria)"
- Prompt for measurable outcomes
- Should be verifiable
- Example:
  - All API endpoints return valid JWT tokens
  - Auth middleware blocks unauthorized requests
  - All tests pass with 80%+ coverage

### Step 6: Create Work Stream Document

Write to `docs/work-streams/topic-name.md`:

```markdown
# [Topic Name]

**Status**: Active
**Created**: YYYY-MM-DD
**Priority**: [High|Medium|Low]

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

[Additional context, learnings, or references]
[This section grows over time as work progresses]
```

### Step 7: Create Session Folder

Create directory: `docs/sessions/topic-name/`

```bash
mkdir -p docs/sessions/[topic-name]
```

### Step 8: Update Development Log (if exists)

Check if `docs/development-log.md` exists. If yes, add work stream to "Active Work" section:

```markdown
### [Topic Name]
**Work Stream**: [work-streams/topic-name.md](work-streams/topic-name.md)
**Status**: Active (YYYY-MM-DD)
**Priority**: [High|Medium|Low]

[Brief 1-sentence description]

**Latest session**: (none yet - just started)
```

### Step 9: Confirm Creation

Display summary:
```
Work stream created!

Location: docs/work-streams/topic-name.md
Sessions folder: docs/sessions/topic-name/

Topic: [Topic Name]
Priority: [High|Medium|Low]
Goals: [N] goals defined
Success criteria: [N] criteria defined

Ready to start first session! Use /session-start
```

### Step 10: Suggest Next Steps

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
1. Asks topic: "User Authentication"
2. Asks purpose: "Implement secure user authentication with JWT tokens"
3. Asks goals:
   - Implement JWT token generation
   - Add refresh token support
   - Create auth middleware
   - Write tests
4. Asks priority: "High"
5. Asks dependencies: "None - can start immediately"
6. Asks success criteria:
   - All endpoints return valid tokens
   - Middleware blocks unauthorized requests
   - 80%+ test coverage
7. Creates: `docs/work-streams/user-authentication.md`
8. Creates: `docs/sessions/user-authentication/`
9. Updates dev log (if exists)
10. Suggests: "First goal is clear. Start session now?"

### Example 2: Refactoring Work Stream

**User**: "let's start work on API refactoring"

**Skill Actions**:
1. Asks topic: "API Refactoring"
2. Asks purpose: "Improve API performance and maintainability"
3. Asks goals:
   - Remove deprecated endpoints
   - Standardize response formats
   - Add pagination to list endpoints
   - Update documentation
4. Asks priority: "Medium"
5. Asks dependencies: "Depends on: database-migration"
6. Asks success criteria:
   - All endpoints follow REST conventions
   - Response times under 200ms
   - Documentation updated
7. Creates work stream and session folder
8. Notes dependency in docs
9. Suggests: "This depends on database-migration. Start when ready?"

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

**If docs folder doesn't exist:**
- Create it: `mkdir -p docs/work-streams docs/sessions`
- Continue with creation

**If user cancels mid-creation:**
- Ask: "Discard work stream or save as draft?"
- If save: Mark status as "Draft" and note incomplete sections

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
