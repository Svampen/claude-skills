# Claude Skills

A collection of reusable custom skills for Claude Code.

## About

This repository contains custom skills that extend Claude Code's capabilities. Skills are specialized tools that provide domain-specific functionality and can be invoked during conversations with Claude.

## Installation

### Option 1: Copy Skills

Copy the skill folders you want to your project's `.claude/skills/` directory:

```bash
cp -r skills/git-commit /path/to/your/project/.claude/skills/
```

### Option 2: Symlink (Recommended for Multiple Projects)

Symlink skills from this repo to keep them in sync across projects:

```bash
# Clone this repo somewhere central
git clone https://github.com/Svampen/claude-skills.git ~/repos/claude-skills

# Symlink skills to your project
ln -s ~/repos/claude-skills/skills/git-commit /path/to/your/project/.claude/skills/git-commit
```

### Option 3: Git Submodule

Add as a submodule for version control:

```bash
git submodule add https://github.com/Svampen/claude-skills.git .claude/shared-skills
```

## Available Skills

### Git & Version Control

| Skill | Description | Trigger |
|-------|-------------|---------|
| **git-commit** | Creates commits using Conventional Commits format | "commit", "/commit" |
| **worktree-start** | Creates git worktree for focused implementation work | "create worktree", "/worktree-start" |
| **worktree-end** | Completes worktree with merge and cleanup | "end worktree", "/worktree-end" |
| **worktree-list** | Shows all active worktrees and their status | "list worktrees", "/worktree-list" |
| **worktree-remove** | Removes worktree (for abandoned work) | "remove worktree", "/worktree-remove" |

### Session Management

| Skill | Description | Trigger |
|-------|-------------|---------|
| **session-start** | Starts a new development session with tracking | "start session", "/session-start" |
| **session-end** | Completes session with quality checks and commit | "finish session", "/session-end" |
| **session-list** | Lists all sessions for a topic | "list sessions", "/session-list" |
| **work-stream-start** | Creates work stream for multi-session effort | "start work stream", "/work-stream-start" |

### Architecture & Discovery

| Skill | Description | Trigger |
|-------|-------------|---------|
| **adr-create** | Creates Architecture Decision Records | "create ADR", "/adr-create" |
| **discovery-start** | Starts structured discovery session | "discovery", "let's explore", "/discovery-start" |
| **insight-capture** | Quick lightweight insight logging | "capture insight", "/insight-capture" |
| **gather-insight** | Iterative research to generate insights | "I want to explore...", "/gather-insight" |

### Documentation

| Skill | Description | Trigger |
|-------|-------------|---------|
| **dev-log-rotate** | Rotates development log when too large | "rotate dev log", "/dev-log-rotate" |

## Skill Details

### git-commit

Creates commits following [Conventional Commits](https://www.conventionalcommits.org/) specification.

**Features:**
- Auto-detects commit type (feat, fix, docs, etc.)
- Suggests scope based on changed files
- Generates concise descriptions
- Always includes Co-Authored-By trailer

**Usage:** Just say "commit" or "/commit" when you have changes ready.

---

### worktree-start / worktree-end / worktree-list / worktree-remove

Manages git worktrees for parallel development work.

**Workflow:**
1. `/worktree-start` - Create new worktree with context file
2. Work in the worktree with VS Code
3. `/worktree-end` - Merge branch and clean up

**Features:**
- Creates `.claude-worktree-context.md` for seamless session integration
- Copies Claude Code settings to new worktrees
- Handles uncommitted changes gracefully
- Clean linear history with rebase + fast-forward merge

---

### session-start / session-end / session-list

Tracks development sessions with documentation and commits.

**Features:**
- Creates dated session logs: `docs/sessions/topic-name/YYYY-MM-DD-title.md`
- Integrates with TodoWrite for task tracking
- Auto-detects worktree context
- Quality checks before commit (configurable per project)

---

### work-stream-start

Creates structured work streams for multi-session development.

**Creates:**
- Work stream document: `docs/work-streams/topic-name.md`
- Session folder: `docs/sessions/topic-name/`
- Updates development log (if exists)

---

### adr-create

Creates Architecture Decision Records with proper structure.

**Template includes:**
- Context & Decision Drivers
- Options Considered (with pros/cons)
- Decision & Rationale
- Consequences (positive/negative/neutral)
- Implementation Notes

---

### discovery-start

Guides through incremental discovery sessions.

**Structure:** Insights -> Opportunity -> Solutions -> Assumptions -> Validation

**Key principle:** Works incrementally, one section at a time, with explicit stops for user input.

---

### insight-capture / gather-insight

Quick insight logging vs. iterative research exploration.

- **insight-capture**: Quick 1-3 paragraph observations
- **gather-insight**: Conversational exploration leading to insight-capture

---

### dev-log-rotate

Rotates development log when it exceeds ~1000 lines.

**Pattern:** System log-style rotation (.md -> .1.md -> .2.md -> .3.md)

## Document Structure

These skills assume a documentation structure in your project:

```
docs/
├── development-log.md      # Main development log
├── work-streams/           # Work stream definitions
│   └── topic-name.md
├── sessions/               # Session logs
│   └── topic-name/
│       └── YYYY-MM-DD-title.md
├── decisions/              # ADRs
│   └── NNN-description.md
├── discovery/              # Discovery documents
│   └── topic-name.md
└── insights/               # Quick insights
    └── YYYY-MM-DD-title.md
```

Skills will create these directories as needed.

## Customization

Each skill can be customized for your project:

1. **Copy** the skill to your project's `.claude/skills/`
2. **Edit** the `SKILL.md` file to adjust:
   - Trigger phrases
   - Templates and formats
   - Tool permissions
   - Project-specific commands

## Contributing

When adding new skills:
1. Create skill folder with `SKILL.md`
2. Include clear documentation with examples
3. Test thoroughly
4. Update this README

## License

MIT License - see LICENSE file for details.
