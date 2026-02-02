---
name: init-project
description: Generates a tailored .claude/project-config.md for the current project. Asks about project type, quality check commands, documentation paths, insight tags, and review strictness.
allowed-tools: Read, Write, Bash, Glob, AskUserQuestion
user-invocable: true
---

# Init Project Skill

Interactively generates a `.claude/project-config.md` tailored to the current project. The config file is read by most other skills in their "Step 0" to discover paths and settings.

## When to Use

Activate this skill when:
- User says "init project", "initialize project", "set up project config"
- User says "/init-project"
- User asks about configuring Claude skills for their project
- User asks "how do I set up project config?"
- Starting a new project that will use Claude skills

## Instructions

### Step 0: Check for Existing Config

1. Check if `.claude/project-config.md` already exists
2. **If it exists:**
   - Read the file and display a summary of current settings
   - Ask user:
     ```
     .claude/project-config.md already exists.

     Options:
     1. Overwrite — Start fresh with new settings
     2. Update — Keep existing values as defaults, modify specific sections
     3. Cancel — Keep current config unchanged

     What would you like to do?
     ```
   - If Cancel, stop here
   - If Update, pre-fill answers with existing values
   - If Overwrite, proceed as if no config exists

### Step 1: Detect Project Type

1. Look for project indicators:
   - `Cargo.toml` → Rust
   - `package.json` → Node/TypeScript
   - `go.mod` → Go
   - `pyproject.toml` or `requirements.txt` or `setup.py` → Python
   - `pom.xml` or `build.gradle` → Java
2. Ask user to confirm or override:

```
Detected project type: [type] (based on [indicator file])

Is this correct, or would you like to specify a different type?
Options:
1. [Detected type]
2. Rust
3. Node / TypeScript
4. Go
5. Python
6. Java
7. Other
```

### Step 2: Configure Quality Checks

Based on project type, pre-fill commands and let user customize.

**Preset defaults by project type:**

| Type | Format | Build | Lint | Test |
|------|--------|-------|------|------|
| Rust | `cargo fmt` | `cargo build` | `cargo clippy -- -D warnings` | `cargo test` |
| Node/TS | `npm run format` | `npm run build` | `npm run lint` | `npm test` |
| Go | `go fmt ./...` | `go build ./...` | `golangci-lint run` | `go test ./...` |
| Python | `black .` | *(none)* | `ruff check .` | `pytest` |
| Java | *(none)* | `./gradlew build` | `./gradlew check` | `./gradlew test` |

Present pre-filled commands and ask user to confirm or customize each:

```
Quality check commands (leave blank to skip):

  Format:  [pre-filled command]
  Build:   [pre-filled command]
  Lint:    [pre-filled command]
  Test:    [pre-filled command]
  Custom:  (none)

Would you like to customize any of these commands?
```

If user wants to customize, ask for each command they want to change.

### Step 3: Configure Documentation Paths

Present defaults and ask if user wants changes:

```
Documentation paths (relative to project root):

  Development log:  docs/development-log.md
  Work streams:     docs/work-streams/
  Sessions:         docs/sessions/
  Decisions:        docs/decisions/
  Insights:         docs/insights/
  Discovery:        docs/discovery/

Would you like to customize any paths?
```

If user wants to customize, ask for each path they want to change.

### Step 4: Configure Insight Tags

Ask about project-specific insight tags:

```
Insight tags help categorize observations captured with /insight-capture.

Default tags: architecture, performance, developer-experience, technical-debt,
              patterns, security, testing, api, database

Would you like to:
1. Use defaults
2. Add project-specific tags (e.g., ecs, combat, auth-system)
3. Replace with fully custom tag list
```

If adding or replacing, ask user for their tags.

### Step 5: Configure Review Strictness

```
Implementation review strictness (/review-implementation):

  strict — Critical issues trigger NEEDS WORK recommendation
  medium — Critical issues trigger CONCERNS, can acknowledge and proceed
  loose  — All findings are informational

Which strictness level?
```

### Step 6: Write Config File

1. Ensure `.claude/` directory exists (`mkdir -p .claude`)
2. Write `.claude/project-config.md` with all configured values
3. Use the following template structure:

```markdown
# Project Configuration for Claude Skills

Project-specific settings used by Claude skills.

## Documentation Paths

- Development log: [path]
- Work streams: [path]
- Sessions: [path]
- Decisions: [path]
- Insights: [path]
- Discovery: [path]

## Quality Checks

Commands to run before completing a session:

### Format
```bash
[command]
```

### Build
```bash
[command]
```

### Lint
```bash
[command]
```

### Test
```bash
[command]
```

[Include Custom Validation section only if user provided a command]

## Insight Tags

- [tag1]
- [tag2]
- ...

## Implementation Review

Strictness: [strict/medium/loose]
```

**Important:**
- Omit subsections where the user left the command blank (e.g., if no format command, omit the Format subsection entirely)
- Do not include placeholder text like `[YOUR_COMMAND]` — either include a real command or omit the subsection

### Step 7: Offer to Create Docs Directory Structure

```
Would you like to create the documentation directory structure now?

This will create:
  [paths listed from Step 3]

Options:
1. Yes — Create directories now
2. No — I'll create them as needed (skills create them automatically)
```

If yes, run `mkdir -p` for each configured path's parent directory.

### Step 8: Summary

Display a summary of what was created:

```
Project configuration complete!

Created: .claude/project-config.md

  Quality Checks:
    Format: [command or "not configured"]
    Build:  [command or "not configured"]
    Lint:   [command or "not configured"]
    Test:   [command or "not configured"]

  Documentation Paths:
    [list of paths]

  Insight Tags: [tag1, tag2, ...]
  Review Strictness: [level]

  [Directories created: list (if Step 7 was yes)]

Skills that use this config: session-start, session-end, session-list,
work-stream-start, worktree-start, dev-log-rotate, adr-create,
discovery-start, insight-capture, review-implementation

See examples/project-config.md in the claude-skills repo for a fully
annotated reference.
```

---

## Examples

### Example 1: New Rust Project

**User**: "/init-project"

1. No existing config found
2. Detected: Rust (Cargo.toml)
3. Quality checks pre-filled: `cargo fmt`, `cargo build`, `cargo clippy -- -D warnings`, `cargo test`
4. User accepts default paths
5. User adds tags: `ecs`, `bevy`, `game-systems`
6. User picks `strict` strictness
7. Config written to `.claude/project-config.md`

### Example 2: Existing Config Update

**User**: "/init-project"

1. Found existing `.claude/project-config.md`
2. User selects "Update"
3. Existing values shown as defaults at each step
4. User only changes lint command
5. Updated config written

### Example 3: Node/TypeScript Project

**User**: "set up project config"

1. No existing config found
2. Detected: Node/TypeScript (package.json)
3. Quality checks pre-filled: `npm run format`, `npm run build`, `npm run lint`, `npm test`
4. User customizes sessions path to `documentation/sessions/`
5. User uses default tags
6. User picks `medium` strictness
7. Config written

---

## Error Handling

**If `.claude/` directory doesn't exist:**
- Create it with `mkdir -p .claude`

**If project type can't be detected:**
- Skip detection, ask user directly for project type
- If user picks "Other", ask for quality check commands manually with no pre-fill

**If write fails (permissions, etc.):**
- Report the error clearly
- Suggest: "Check directory permissions or try creating `.claude/` manually"

---

## Notes

- This skill generates config; it does not enforce it. Each skill reads the config independently.
- All config sections are optional. Skills fall back to defaults when sections are missing.
- The generated config includes no HTML comments or placeholder text — only real values.
- For a fully annotated reference with explanations of each section, see `examples/project-config.md` in the claude-skills repository.
