---
name: session-list
description: Lists all sessions for a work stream or topic. Use when user asks "what have we done on X?", reviewing session history, or looking for specific session.
allowed-tools: Read, Glob
user-invocable: true
---

# Session List Skill

Provides quick overview of all sessions for a given topic, showing chronological history of work.

## When to Use

Activate this skill when:
- User asks "what have we done on X?"
- User says "list sessions for..."
- Reviewing session history for a topic
- Looking for specific session
- Understanding work cadence/velocity
- Preparing summary or report

## Purpose

Provides visibility into session history:
- See all work done on a topic at a glance
- Understand chronological progression
- Identify patterns or gaps
- Find specific sessions quickly
- Estimate complexity of remaining work

## Instructions

### Step 1: Identify Topic

Parse user request to extract topic:

- Extract topic name from user query
- Example: "list sessions for user-authentication" -> topic: `user-authentication`
- Example: "what did we do on the API?" -> topic: `api` (fuzzy search)

If ambiguous or not specified, ask user:
```
Which topic would you like to see sessions for?
```

### Step 2: Find Session Files

Use Glob to find all session files:

```
Glob: docs/sessions/[topic-name]/*.md
```

**For fuzzy search** (user provided partial topic):
```
Glob: docs/sessions/*[partial-topic]*/*.md
```

If no sessions found:
- Inform user: "No sessions found for [topic]"
- Suggest: "Check if topic exists? Typo in name?"
- Offer to list all available topics

### Step 3: Read Session Metadata

For each session file found, extract key information:

Read first ~30 lines of each session file to get:
- **Title**: From first heading (# Title)
- **Date**: From `**Date**: YYYY-MM-DD` line
- **Status**: From `**Status**: Complete|In Progress` line
- **Goal**: From `## Session Goals` section (first line)

Store this information for formatting.

### Step 4: Sort Sessions

Sort sessions chronologically:
- Sort by date (filename prefix or Date field)
- Default: **Oldest first** (shows progression)

### Step 5: Format and Display

Create formatted list:

```
Sessions for '[Topic Name]':

2026-01-08: Initial setup
  Goal: Set up project structure and dependencies
  Status: Complete
  File: docs/sessions/topic-name/2026-01-08-initial-setup.md

2026-01-09: Core implementation
  Goal: Implement core functionality
  Status: Complete
  File: docs/sessions/topic-name/2026-01-09-core-implementation.md

2026-01-11: Testing and fixes
  Goal: Add tests and fix issues
  Status: In Progress
  File: docs/sessions/topic-name/2026-01-11-testing.md

Total: 3 sessions
Completed: 2 sessions
In Progress: 1 session
```

**Status indicators**:
- Complete = finished session
- In Progress = active session
- Unknown = couldn't parse status

### Step 6: Provide Summary Statistics

Calculate and display:
- **Total sessions**: Count of all sessions
- **Completed sessions**: Sessions marked complete
- **In-progress sessions**: Sessions marked in-progress
- **Date range**: First session date -> Last session date

Example:
```
Summary:
- Total sessions: 3
- Completed: 2 (67%)
- In Progress: 1
- Date range: 2026-01-08 to 2026-01-11
```

### Step 7: Suggest Follow-up Actions

Based on the session list, suggest relevant actions:

**If all sessions complete**:
```
All sessions for this topic are complete!

Consider:
- Creating a summary document
- Marking work stream as complete
- Archiving this work
```

**If sessions in-progress**:
```
Found 1 in-progress session:
- 2026-01-11: Testing and fixes

Consider:
- Resuming work on in-progress session
- Completing session before starting new one
```

**If many sessions (8+)**:
```
This topic has 12 sessions (substantial work!).

Consider:
- Creating a summary of accomplishments
- Reviewing learnings for documentation
- Extracting patterns for future work
```

---

## Examples

### Example 1: List Topic Sessions

**User**: "list sessions for user-authentication"

**Skill Actions**:
1. Identifies topic: `user-authentication`
2. Globs: `docs/sessions/user-authentication/*.md`
3. Finds 3 sessions
4. Reads metadata from each
5. Formats output:
   ```
   Sessions for 'User Authentication':

   2026-01-08: Initial setup
     Goal: Set up auth middleware structure
     Status: Complete

   2026-01-09: JWT implementation
     Goal: Add JWT token validation
     Status: Complete

   2026-01-11: Testing
     Goal: Add comprehensive tests
     Status: In Progress

   Summary:
   - Total: 3 sessions
   - Completed: 2 (67%)
   - In Progress: 1
   ```

### Example 2: Find In-Progress Sessions

**User**: "what sessions are in progress?"

**Skill Actions**:
1. User didn't specify topic - search all sessions
2. Globs: `docs/sessions/*/*.md`
3. Reads all session files, filters for "Status: In Progress"
4. Formats output:
   ```
   In-Progress Sessions:

   user-authentication: Testing
     Date: 2026-01-11
     Goal: Add comprehensive tests
     File: docs/sessions/user-authentication/2026-01-11-testing.md

   api-refactor: Endpoint cleanup
     Date: 2026-01-10
     Goal: Clean up deprecated endpoints
     File: docs/sessions/api-refactor/2026-01-10-endpoint-cleanup.md

   Total: 2 in-progress sessions
   ```

### Example 3: Fuzzy Topic Search

**User**: "list sessions for auth"

**Skill Actions**:
1. Identifies partial topic: `auth`
2. Globs: `docs/sessions/*auth*/*.md`
3. Finds multiple matches:
   - `user-authentication/` (3 sessions)
   - `oauth-integration/` (2 sessions)
4. Asks user:
   ```
   Found multiple topics matching 'auth':
   - user-authentication (3 sessions)
   - oauth-integration (2 sessions)

   Which one would you like to see?
   ```

### Example 4: Empty Results

**User**: "list sessions for database-migration"

**Skill Actions**:
1. Identifies topic: `database-migration`
2. Globs: `docs/sessions/database-migration/*.md`
3. No files found
4. Informs user:
   ```
   No sessions found for 'database-migration'

   Possible reasons:
   - Topic hasn't started yet
   - Typo in name
   - Work hasn't been documented

   Available topics:
   - user-authentication
   - api-refactor
   - oauth-integration
   ```

---

## Error Handling

**If topic not found:**
- List available topics
- Suggest similar names (fuzzy matching)
- Ask if user wants to create new topic

**If no sessions exist for topic:**
- Check if work stream document exists (but no sessions yet)
- Inform: "Topic exists but no sessions started yet"
- Suggest: "Use /session-start to begin first session?"

**If session files are malformed:**
- Skip sessions with parsing errors
- Note: "Skipped N sessions with formatting issues"
- Continue processing valid sessions

**If user provides no topic:**
- Ask: "Which topic would you like to see sessions for?"
- Offer to list all available topics

---

## Quality Checks

Before displaying results:
- At least try to find sessions
- Metadata extracted successfully (where possible)
- Sessions sorted chronologically
- Status indicators correct
- Summary statistics calculated

---

## Notes

- Works with date-based filenames: `YYYY-MM-DD-title.md`
- Supports fuzzy search (partial topic names)
- Fast operation (just file listing + light parsing)
- Useful for quick status checks
- Can be called repeatedly without side effects
- Output can feed into summary for deeper analysis
