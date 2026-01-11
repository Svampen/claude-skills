---
name: dev-log-rotate
description: Checks development-log.md size and rotates to numbered archives (.1.md, .2.md) if too large. Use when log exceeds ~1000 lines or user says "rotate dev log". Maintains system log-style rotation pattern.
allowed-tools: Read, Write, Bash, Edit
user-invocable: true
---

# Dev Log Rotate Skill

Automates rotation of the development log when it becomes too large for efficient context loading.

## When to Use

Activate this skill when:
- User says "rotate dev log", "check dev log size", or "archive dev log"
- User mentions dev log is too big or difficult to load
- Before starting new work and dev log feels unwieldy
- Proactively when you notice dev log exceeds ~1000 lines

## Rotation Strategy

**System log-style numbered rotation**:
```
docs/development-log.md     # Current (most recent entries)
docs/development-log.1.md   # Previous rotation
docs/development-log.2.md   # Older rotation
docs/development-log.3.md   # Oldest rotation
```

**Threshold**: Rotate when `development-log.md` exceeds **1000 lines** (keeps file manageable)

**Rotation process**:
1. Rename `.2.md` -> `.3.md` (if `.2.md` exists)
2. Rename `.1.md` -> `.2.md` (if `.1.md` exists)
3. Rename `.md` -> `.1.md`
4. Create new `.md` with header template

## Instructions

### Step 1: Check Current Size

1. Count lines in development log:
   ```bash
   wc -l docs/development-log.md
   ```

2. Parse line count from output

3. Report current size:
   ```
   Development log size: [N] lines

   Status: [NEEDS ROTATION / OK]
   Threshold: 1000 lines
   ```

4. If **< 1000 lines**:
   - Report: "Development log is healthy ([N] lines). No rotation needed."
   - STOP (do not rotate)

5. If **>= 1000 lines**:
   - Report: "Development log exceeds threshold ([N] lines). Rotation recommended."
   - Proceed to rotation steps

### Step 2: Check for Existing Archives

Check which archive files exist:

```bash
ls docs/development-log.*.md 2>/dev/null || echo "No archives"
```

Parse results to determine:
- Does `.1.md` exist?
- Does `.2.md` exist?
- Does `.3.md` exist?

### Step 3: Perform Rotation

**IMPORTANT**: Rotate in reverse order to avoid overwriting!

1. **If `.2.md` exists**, rename to `.3.md`:
   ```bash
   mv docs/development-log.2.md docs/development-log.3.md
   ```
   - Report: "Rotated: .2.md -> .3.md"

2. **If `.1.md` exists**, rename to `.2.md`:
   ```bash
   mv docs/development-log.1.md docs/development-log.2.md
   ```
   - Report: "Rotated: .1.md -> .2.md"

3. **Rename current log** to `.1.md`:
   ```bash
   mv docs/development-log.md docs/development-log.1.md
   ```
   - Report: "Rotated: .md -> .1.md"

### Step 4: Create New Development Log

1. Read first 50 lines of archived log (`.1.md`) to extract header/template

2. Extract:
   - Title line (usually `# Development Log`)
   - Description/purpose section
   - Structure template (Active Work, Chronological, etc.)

3. Create new `docs/development-log.md` with template:
   ```markdown
   # Development Log

   This is the current development log. Older entries are archived in numbered files:
   - [development-log.1.md](development-log.1.md) - Previous rotation
   - [development-log.2.md](development-log.2.md) - Older rotation (if exists)
   - [development-log.3.md](development-log.3.md) - Oldest rotation (if exists)

   ---

   ## Active Work

   [Copy from archived log's Active Work section - latest state]

   ---

   ## Development History

   [Empty - will populate with new entries]
   ```

4. **Preserve Active Work section**:
   - Read archived log's "Active Work" section
   - Copy current active items to new log
   - Ensures continuity of ongoing work tracking

### Step 5: Report Rotation Summary

Generate rotation summary:

```
Development log rotated successfully!

Rotation Summary:
- development-log.md: [OLD_LINES] lines -> archived to .1.md
- development-log.1.md -> development-log.2.md ([if existed])
- development-log.2.md -> development-log.3.md ([if existed])

New log created:
- development-log.md: Fresh template with Active Work preserved
- Archive chain: .1.md -> .2.md -> .3.md

Tip: Archived logs remain searchable and accessible for historical reference.
```

### Step 6: Verify Rotation

1. Count lines in new log:
   ```bash
   wc -l docs/development-log.md
   ```

2. Verify new log is small (should be ~50-100 lines - just template)

3. Verify archived log exists:
   ```bash
   ls -lh docs/development-log.1.md
   ```

4. Report verification:
   ```
   Verification:
   - New log: [N] lines (healthy)
   - Archive .1.md: [SIZE] ([OLD_LINES] lines)
   ```

---

## Examples

### Example 1: Rotation Needed

**User**: "Check if we need to rotate the dev log"

**Skill Actions**:
1. Runs `wc -l docs/development-log.md` -> "2318 lines"
2. Reports: "Log exceeds threshold (2318 lines). Rotation recommended."
3. Checks archives: `.1.md` does not exist
4. Rotates:
   - `.md` -> `.1.md`
5. Reads archived log header (first 50 lines)
6. Extracts "Active Work" section
7. Creates new `development-log.md` with template + Active Work
8. Reports:
   ```
   Rotation complete!
   - Archived: 2318 lines -> development-log.1.md
   - New log: 87 lines (healthy)
   ```

### Example 2: Rotation Not Needed

**User**: "rotate dev log"

**Skill Actions**:
1. Runs `wc -l docs/development-log.md` -> "432 lines"
2. Reports:
   ```
   Development log is healthy (432 lines).
   No rotation needed (threshold: 1000 lines).
   ```
3. STOPS (does not rotate)

### Example 3: Multiple Archives Exist

**User**: "rotate the log"

**Skill Actions**:
1. Checks size: 1543 lines -> needs rotation
2. Checks archives:
   - `.1.md` exists (2100 lines)
   - `.2.md` exists (1987 lines)
   - `.3.md` does not exist
3. Rotates in reverse order:
   - `.2.md` -> `.3.md`
   - `.1.md` -> `.2.md`
   - `.md` -> `.1.md`
4. Creates new log with template
5. Reports:
   ```
   Rotation complete!

   Archive chain:
   - .1.md: 1543 lines (just archived)
   - .2.md: 2100 lines
   - .3.md: 1987 lines

   New log: 92 lines
   ```

---

## Error Handling

**If development-log.md not found**:
- Report: "Development log not found at docs/development-log.md"
- Suggest: "Check working directory or file path"
- STOP

**If rotation fails (file operation error)**:
- Report exact error message
- STOP immediately (do not create new log)
- Suggest manual intervention

**If Active Work section cannot be extracted**:
- Warn: "Could not extract Active Work. Creating minimal template."
- Create log without Active Work section
- Continue with rotation

**If new log creation fails**:
- Report error
- STOP
- Suggest: "Original log still exists at .1.md. Manually create new log."

---

## Quality Checks

Before completing, verify:
- Old log archived with correct number (.1.md)
- Archive chain rotated correctly (.1 -> .2 -> .3)
- New log exists and is small (~50-100 lines)
- Active Work section preserved (if present)
- Archive files readable and complete
- No data loss occurred

---

## Notes

**When to rotate**:
- Proactively suggest rotation when log exceeds 1000 lines
- User can manually trigger anytime
- Rotation is safe and reversible (archives preserved)

**Archive retention**:
- Keep up to 3 archive levels (.1, .2, .3)
- Oldest archive (.3) is overwritten when new rotation pushes .2 -> .3
- Adjust retention policy if needed

**Performance**:
- Rotation takes seconds (file renames + template creation)
- No data loss (everything archived)
- New log loads quickly

**Reverting rotation**:
- To undo: `mv docs/development-log.1.md docs/development-log.md`
- Archives remain untouched (easy rollback)
