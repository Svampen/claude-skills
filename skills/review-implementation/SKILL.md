---
name: review-implementation
description: Reviews implementation against session/work stream goals before committing. Identifies shortcuts, hacks, incomplete work, and validates accomplishments match actual changes. Use before /session-end to ensure implementation quality.
allowed-tools: Read, Glob, Grep, Bash, AskUserQuestion, Task
user-invocable: true
---

# Review Implementation Skill

Reviews current implementation against stated goals. Identifies shortcuts, hacks, incomplete work, and validates that documented accomplishments match actual code changes.

## When to Use

Activate this skill when:
- User says "review implementation", "review my work", or "check implementation"
- User wants to verify work quality before committing
- Before running `/session-end` to catch issues early
- User asks "did I miss anything?" or "is this complete?"

## Instructions

### Step 0: Read Project Config

1. Read `.claude/project-config.md`
2. Parse `## Documentation Paths` section to get:
   - Sessions path (default: `docs/sessions/`)
   - Work streams path (default: `docs/work-streams/`)
3. Parse `## Implementation Review` section:
   - Get `Strictness:` setting (strict/medium/loose)
   - Default to `strict` if not configured

**If no config exists:**
- Use default paths: `docs/sessions/`, `docs/work-streams/`
- Use default strictness: `strict`

### Step 1: Find Current Session

1. Glob all sessions: `[sessions-path]/*/*.md`
2. Find most recent with "Status: In Progress"
3. Read session file to get:
   - Tasks section (what should be done)
   - "What Was Accomplished" section (what's claimed done)
   - Work stream reference (if any)

**If no active session found:**
- Report: "No active session found. Run `/session-start` first, or provide context about what you're reviewing."
- Ask user if they want to continue without session context

### Step 2: Read Work Stream Goals (if exists)

1. Check if work stream document exists: `[work-streams-path]/[topic-name].md`
2. If yes, extract:
   - Goals
   - Success criteria
   - Current session tasks

### Step 3: Get Git Diff

```bash
git diff HEAD
```

If no changes:
- Report: "No uncommitted changes detected."
- Ask: "Did you already commit? Should I review the last commit instead?"

### Step 4: Perform Review

Analyze the implementation against goals:

**A. Goal Alignment**

For each task/goal from session and work stream:
- Is it fully implemented in the diff?
- Is it partially done? What's missing?
- Is it not addressed?

**B. Code Quality Scan**

Search diff for red flags:
- `TODO`, `FIXME`, `HACK`, `XXX` comments
- "workaround", "quick fix", "temporary", "for now", "later"
- Hardcoded values without explanation
- Incomplete error handling

**C. Accomplishment Verification**

Compare "What Was Accomplished" to actual diff:
- Does diff support each claimed accomplishment?
- Are there undocumented significant changes?

**D. Deferred Work Detection**

Identify new deferrals:
- New TODO comments (vs existing)
- "Future work" mentions

### Step 5: Generate Report

Present a structured report:

```markdown
## Implementation Review

**Session**: [session file path]
**Strictness**: [strict/medium/loose]

### Goal Alignment

| Goal | Status | Evidence |
|------|--------|----------|
| [Task 1] | ✅ Complete | [file:line or description] |
| [Task 2] | ⚠️ Partial | [what's done / what's missing] |
| [Task 3] | ❌ Missing | [explanation] |

### Code Quality Concerns

**Critical** (must address before commit):
| Location | Type | Issue |
|----------|------|-------|
| file:line | TODO | "comment text" |

**Warnings** (should explain or fix):
| Location | Type | Issue |
|----------|------|-------|

### Accomplishment Verification

| Claimed | Verified | Notes |
|---------|----------|-------|
| "Added X" | ✅ | Confirmed in src/x.rs |
| "Fixed Y" | ⚠️ | Partial - edge case not covered |

### Deferred Work Detected

- [List any new TODOs/deferrals introduced]

### Summary

- Goals addressed: X/Y
- Critical issues: N
- Warnings: N
- New TODOs: N

**Status**: [PASS / CONCERNS / NEEDS WORK]
```

### Step 6: Request User Decision

**CRITICAL: You MUST ask the user and wait for their response. Never auto-accept or skip this step.**

Present options based on review status:

```
Implementation Review Complete.

[Show summary: goals X/Y, critical N, warnings N]

Options:
1. ACCEPT - Proceed (implementation meets goals)
2. ACCEPT WITH NOTES - Acknowledge concerns and proceed
3. NEEDS WORK - Address issues before proceeding

What would you like to do?
```

**Strictness behavior** (affects recommendation, NOT whether to ask):

- **Strict**: Critical issues → recommend NEEDS WORK
- **Medium**: Critical issues → CONCERNS, can acknowledge
- **Loose**: All informational, can always proceed

**Important**: Regardless of strictness level or review status, you MUST present the options and wait for user input. Even if the review passes with no issues, ask the user to confirm before proceeding.

### Step 7: Handle Decision

**CRITICAL: Only proceed after receiving explicit user input from Step 6. Never assume or auto-select an option.**

**If user selects ACCEPT:**
- Report: "Review complete. Ready for `/session-end`."

**If user selects ACCEPT WITH NOTES:**
- Ask user to provide notes/explanation for concerns
- Report: "Review complete with notes. Ready for `/session-end`."

**If user selects NEEDS WORK:**
- List specific items to address
- Report: "Please address issues and run `/review-implementation` again when ready."

---

## Examples

### Example 1: Clean Review

**User**: "/review-implementation"

**Output**:
```
## Implementation Review

**Session**: docs/sessions/auth-system/2026-01-18-login-flow.md
**Strictness**: strict

### Goal Alignment

| Goal | Status | Evidence |
|------|--------|----------|
| Add login endpoint | ✅ Complete | src/api/auth.rs:45-89 |
| Add session management | ✅ Complete | src/session.rs:1-120 |
| Add logout endpoint | ✅ Complete | src/api/auth.rs:91-105 |

### Code Quality Concerns

No critical issues found.

### Summary

- Goals addressed: 3/3
- Critical issues: 0
- Warnings: 0

**Status**: PASS

Options:
1. ACCEPT - Proceed (implementation meets goals)
2. NEEDS WORK - Address issues before proceeding

What would you like to do?
```

### Example 2: Issues Found

**User**: "/review-implementation"

**Output**:
```
## Implementation Review

**Session**: docs/sessions/auth-system/2026-01-18-login-flow.md
**Strictness**: strict

### Goal Alignment

| Goal | Status | Evidence |
|------|--------|----------|
| Add login endpoint | ✅ Complete | src/api/auth.rs:45-89 |
| Add session management | ⚠️ Partial | Missing session expiry handling |
| Add logout endpoint | ❌ Missing | Not implemented |

### Code Quality Concerns

**Critical**:
| Location | Type | Issue |
|----------|------|-------|
| src/api/auth.rs:67 | TODO | "// TODO: validate password strength" |
| src/session.rs:34 | HACK | "// hack: skip validation for now" |

### Summary

- Goals addressed: 1/3 complete, 1 partial, 1 missing
- Critical issues: 2
- Warnings: 0

**Status**: NEEDS WORK

Critical issues found. Please address before proceeding:
1. Missing: logout endpoint
2. Partial: session management (no expiry)
3. TODO: password strength validation (src/api/auth.rs:67)
4. Hack comment needs proper fix (src/session.rs:34)

Options:
1. ACCEPT - I understand and accept these as-is
2. ACCEPT WITH NOTES - Add explanation for why these are acceptable
3. NEEDS WORK - Stop and fix these issues

What would you like to do?
```

---

## Notes

- This skill is standalone - use it anytime to review implementation quality
- Run before `/session-end` to catch issues early
- Strictness setting comes from `.claude/project-config.md`
- If no session context, skill can still review git diff against user-provided goals
- The user makes the final decision - this skill surfaces concerns, doesn't block
